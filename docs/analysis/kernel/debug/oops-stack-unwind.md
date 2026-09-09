---
date: 2026-09-09
homeTag: 内核 · 调试
homeTitle: 从 Oops 栈转储手工回溯调用栈
homeDesc: 从 SP 往高地址认 push 保存的 lr，串起 fault_null_write → sysfs → write
sidebarOrder: 6
sidebarTitle: Oops 栈转储回溯调用栈
---

# 从 Oops 栈转储手工回溯调用栈

> **环境**：Linux 5.4.31 · ARM（STM32MP157）· 模块 `pc_fault`（写 `/sys/kernel/pc_fault/trigger` 触发）  
> **关联**：[根据 Oops 的 PC 定位到出错指令](/analysis/kernel/debug/oops-pc-to-source)  

---

## 目录

- [1. 现象](#1-现象)
- [2. 方法概要](#2-方法概要)
- [3. 第一帧：fault_null_write](#3-第一帧fault_null_write)
- [4. 第二帧：trigger_store](#4-第二帧trigger_store)
- [5. 再往上：kernfs / VFS / syscall](#5-再往上kernfs--vfs--syscall)
- [6. 完整调用链](#6-完整调用链)

---

## 1. 现象

寄存器与栈：

```text
sp : df289e90
Stack: (0xdf289e90 to 0xdf28a000)
9e80:                                     c1204e88 bf1db05c 00000000 d3408596
9ea0: 00000002 00000002 d1d0a380 c034bed4 00000000 00000000 c1204e88 d1ed5240
9ec0: c034bddc df289f78 00000000 000f1ba8 00000002 c02c4390 ...
9f40: ... c02c72b4 ...
9f60: ... c02c7540 ...
9fa0: ... c0101000 ...
```

行首 `9e80` / `9ea0` 是地址低 16 位；一行 8 个字。`SP=…9e90` 时，`9e80` 行前半空白，真正从 **`9e90`** 开始才是当前栈内容。

---

## 2. 方法概要

1. 记下 Oops 里的 **`sp`**，从该地址往**高地址**读。
2. 对照当前函数的序言（`objdump -d`）：`push {r4, lr}` → 低地址是 r4，高地址是 **lr（返回地址）**。
3. 返回地址用 `/proc/kallsyms`（或同版本 `System.map`）看落在哪个符号。
4. 再对照**上一层**函数的 `push` / `sub sp`，找到它保存的 lr，继续往上。
5. 筛选「像代码指针」的值：模块区常见 `0xbf……`，内核镜像常见 `0xc0……`；用户态多在 `0xb6……` / `0xbe……`。

---

## 3. 第一帧：fault_null_write

反汇编入口：

```text
fault_null_write:
   0:  push {r4, lr}
  ...
  24:  str  r3, [r2]     ← PC
  28:  pop  {r4, pc}
```

从 `SP`：

| 地址 | 值 | 含义 |
|------|-----|------|
| `…9e90` | `c1204e88` | 保存的 **r4**（与寄存器 dump 里 `r4` 一致） |
| `…9e94` | `bf1db05c` | 保存的 **lr** |

```bash
cat /proc/kallsyms | grep bf1db0
# bf1db000 trigger_store
# bf1db05c 落在 trigger_store 内（+0x5c）
# bf1db064 trigger_show
```

`trigger_store+0x5c` 正是 `bl do_fault` **之后**那条指令 → 返回到 **`trigger_store`**。

---

## 4. 第二帧：trigger_store

`trigger_store` 入口大致是：

```text
push {r4, r5, lr}
sub  sp, #12          // 局部 / canary
...
bl   do_fault
mov  r0, r5           // +0x5c，上一帧要返回的位置
```

紧接 `fault_null_write` 的 8 字节帧之后：

- 先是 `sub sp,#12` 的 3 个字（`9e98`～`9ea0`）
- 再是保存的 r4、r5、**lr**

| 地址 | 值 | 含义 |
|------|-----|------|
| `…9ea4` | `00000002` | `trigger_store` 的 r4 |
| `…9ea8` | `d1d0a380` | `trigger_store` 的 r5 |
| `…9eac` | `c034bed4` | `trigger_store` 的 **lr** |

板上：

```text
c034bddc  kernfs_fop_write
c034bed4  ← +0xf8
c034bfec  kernfs_drain_open_files
```

→ 返回 **`kernfs_fop_write`**（写 sysfs 节点的路径）。

---

## 5. 再往上：kernfs / VFS / syscall

继续在栈里挑 `0xc0……` 且落在「下一函数体内」的值（`kallsyms`）：

| 栈上值 | 落入符号（示例） | 角色 |
|--------|------------------|------|
| `c034bed4` | `kernfs_fop_write+…` | sysfs write |
| `c02c4390` | `__vfs_write+…` | VFS |
| `c02c72b4` | `vfs_write+…` | VFS |
| `c02c7540` | `ksys_write+…` | `write` 系统调用 |
| `c0101000` | `ret_fast_syscall` | 从内核返回用户态 |

---

## 6. 完整调用链

手工结果：

```text
bash  write(2)
  → ksys_write
  → vfs_write
  → __vfs_write
  → kernfs_fop_write
  → trigger_store
  → fault_null_write  ※ PC 停在 str [r2]，*NULL
```
