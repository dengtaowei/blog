---
date: 2026-09-09
homeTag: 内核 · 调试
homeTitle: 根据 Oops 的 PC 定位到出错指令
homeDesc: 从 fault_null_write+0x24 到 str [r2]，用 kallsyms 与 objdump 对齐 C 行
sidebarOrder: 5
sidebarTitle: Oops PC 定位出错指令
---

# 根据 Oops 的 PC 定位到出错指令 / C 行

> **环境**：Linux 5.4.31 · ARM（STM32MP157）· 故意空指针模块 `pc_fault`  
> **目标**：只拿到 `pc : [<bf1db09c>]` 时，如何确认是哪条汇编、对应哪行 C  

---

## 目录

- [1. 现象（Oops 关键字段）](#1-现象oops-关键字段)
- [2. 函数名 + 偏移](#2-函数名--偏移)
- [3. 只有裸 PC：用 kallsyms](#3-只有裸-pc用-kallsyms)
- [4. 定位到指令](#4-定位到指令)

---

## 1. 现象（Oops 关键字段）

人为构造空指针写后，dmesg 摘要如下：

```text
Unable to handle kernel NULL pointer dereference at virtual address 00000000
PC is at fault_null_write+0x24/0x2c [pc_fault]
LR is at fault_null_write+0x18/0x2c [pc_fault]
pc : [<bf1db09c>]    lr : [<bf1db090>]
r3 : 12345678  r2 : 00000000  r1 : 00000001  r0 : 0000003f
Code: eb3e88f6 e3a02000 e3053678 e3413234 (e5823000)
```

| 字段 | 含义 |
|------|------|
| `PC is at func+off/size [mod]` | 已给出函数名、模块内偏移；优先用它 |
| `pc : [<addr>]` | PC 绝对值；模块区常见 `0xbf......` |
| `Code: ... (xxxxxxxx)` | 括号内是**触发异常的那条指令机器码** |

对应 C（节选）：

```c
static noinline void fault_null_write(void)
{
	volatile unsigned int *p = NULL;
	pr_emerg("...");
	*p = 0x12345678;   /* 目标：定位到这一行 */
}
```

---

## 2. 函数名 + 偏移

Oops 若已打印：

```text
PC is at fault_null_write+0x24/0x2c [pc_fault]
```

则：

1. 在**与板上一致**的 `pc_fault.ko` 上反汇编；
2. 看 `fault_null_write` 里偏移 `+0x24` 的那条指令；
3. 对照源码 / `-S` 输出，得到 C 行。

---

## 3. 只有裸 PC：用 kallsyms

假设只知道 `pc = 0xbf1db09c`：

```bash
cat /proc/kallsyms | grep bf1db0
```

示例：

```text
bf1db000 t trigger_store        [pc_fault]
bf1db064 t trigger_show         [pc_fault]
bf1db078 t fault_null_write     [pc_fault]
bf1db0a4 t fault_bad_pc         [pc_fault]
...
```

判断 PC 落在哪个符号区间，再减函数起始地址：

```text
0xbf1db09c - 0xbf1db078 = 0x24
→ fault_null_write + 0x24
```

也可用：

```bash
cat /sys/module/pc_fault/sections/.text   # .text 加载基址
```

---

## 4. 定位到指令

```bash
arm-buildroot-linux-gnueabihf-objdump -d pc_fault.ko > pc_fault.dis
# 或带源码行：
arm-buildroot-linux-gnueabihf-objdump -dS pc_fault.ko | less
```

`fault_null_write` 片段：

```text
00000000 <fault_null_write>:
   0:   e92d4010    push    {r4, lr}
   ...
  14:   ebfffffe    bl      printk
  18:   e3a02000    mov     r2, #0
  1c:   e3053678    movw    r3, #0x5678
  20:   e3413234    movt    r3, #0x1234
  24:   e5823000    str     r3, [r2]    ← +0x24，出问题的指令
  28:   e8bd8010    pop     {r4, pc}
```

与 Oops 对齐：

- 偏移 `+0x24` 一致；
- 括号内机器码 `(e5823000)` 与 `str r3, [r2]` 一致；
- 语义：`r2=0`，向地址 0 写 `r3`，即 `*NULL = 0x12345678`。

