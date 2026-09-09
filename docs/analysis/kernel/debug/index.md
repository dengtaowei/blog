---
home: false
---

# 调试与实践

针对**具体问题**的排查记录、实验与调试笔记。与 [源码流程分析](/analysis/kernel/) 互补：后者讲「内核通常怎么走」，这里记「某次实际出了什么问题、怎么查的」。

---

## 与流程分析的区别

| | 流程分析 | 调试与实践 |
|---|---------|-----------|
| 目标 | 理解子系统设计与调用链 | 解决或记录一次具体问题 |
| 结构 | 分阶段、成体系 | 现象 → 排查 → 根因 |
| 时效 | 长期有效 | 常含环境、版本、设备信息 |
| 目录 | `analysis/kernel/<子系统>/` | `analysis/kernel/debug/<子系统>/` |

写流程分析时不必塞进某次踩坑；调试文里用链接指回对应流程文章即可。

---

## 分类

- [USB](/analysis/kernel/debug/usb/) — 枚举、probe、UVC、抓包与 trace 等
- [Pinctrl / GPIO](/analysis/kernel/debug/gpio/) — runtime PM、pin 复用、片选时序等
- [Sound](/analysis/kernel/debug/sound/) — 板载麦/喇叭、DTS 与原理图对照等
- 通用工具与方法
  - [根据 Oops 的 PC 定位到出错指令](/analysis/kernel/debug/oops-pc-to-source)
  - [从 Oops 栈转储手工回溯调用栈](/analysis/kernel/debug/oops-stack-unwind)
- [写作模板](/analysis/kernel/debug/template)

---

## 附件

大体积日志、抓包、trace 文本放在 `docs/public/files/`，构建后通过 `/files/文件名` 访问（例如 `/files/usb.pcapng`）。

---

## 新增一篇

1. 复制 [写作模板](/analysis/kernel/debug/template) 到对应子目录
2. 文件名建议：`问题简述-环境.md`（英文 kebab-case 亦可）
3. 在子目录 `index.md` 和本页分类里加链接
4. 若与某篇流程分析相关，文末互链
