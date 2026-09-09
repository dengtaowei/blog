---
home: false
---

# Linux 内核

Linux 内核子系统源码分析（Linux 6.8）。

## 中断 / softirq

专题页：[中断 / softirq](/analysis/kernel/irq/)

- [硬中断、softirq 与 arm64 路径](/analysis/kernel/irq/hardirq-softirq-arm64) — 栈与上下文、`irq_exit`、`local_bh_disable`、加锁
- [ARM64 异常路径上的栈切换](/analysis/kernel/irq/arm64-stack-switch) — 硬件换 `SP_EL1`、劫持 `SP_EL0`、IRQ 栈与 `cpu_switch_to`

## USB 子系统

专题页：[USB 子系统](/analysis/kernel/usb/)

1. [USB 2.0 枚举流程](/analysis/kernel/usb/usb-enumeration)
2. [hub_port_init 调用链](/analysis/kernel/usb/hub-port-init)
3. [usb_get_descriptor 调用链](/analysis/kernel/usb/get-descriptor-trace)
4. [枚举与两轮 Probe](/analysis/kernel/usb/enumeration-and-probe)
5. [UVC 驱动分析](/analysis/kernel/usb/uvc-driver)
6. [Gadget 子系统概览](/analysis/kernel/usb/gadget-subsystem) — UDC / composite / configfs 四层架构
7. [Configfs 组装分析](/analysis/kernel/usb/gadget-configfs-assembly) — `gadget_info` / `cdev` 脚本拼装与 bind
8. [UDC bind 分析](/analysis/kernel/usb/gadget-udc-core-bind) — `udc_bind_to_driver`、pending、pullup
9. [DWC2 接口总览](/analysis/kernel/usb/gadget-dwc2-interface) — `gadget_ops`、`ep_ops` 与框架交付边界
10. [DWC2 USBTRDTIM 选值](/analysis/kernel/usb/gadget-dwc2-turnaround-time) — 按 UTMI+ 位宽给 5 / 9，对照 ST HAL 按 AHB 频率查表
11. [Composite EP0 枚举](/analysis/kernel/usb/gadget-composite-ep0) — `composite_setup`、`SET_CONFIGURATION`
12. [ACM Function 路径](/analysis/kernel/usb/gadget-function-acm) — `f_acm.c`、`gserial_connect` 与 ttyGS
13. [Gadget CDC ACM 串口实践](/analysis/kernel/usb/gadget-cdc-acm) — configfs + `ttyGS0` / Host `cdc_acm`

## Media / V4L2 子系统

专题页：[Media / V4L2](/analysis/kernel/media/)

- [i.MX6ULL OV5640：设备树到 /dev/video](/analysis/kernel/media/imx6-ov5640-dts-video) — DVP DTS、引脚冲突与 mx6s 出现 video 节点（Linux 4.9.88 BSP）
- [OV5640 横纹与偏色](/analysis/kernel/media/imx6-ov5640-artifacts-color-stripes) — CSI 同步极性 vs ffplay 像素格式（Linux 4.9.88 BSP）
- [PXP PRIMARY 直写 fb0](/analysis/kernel/media/imx6-pxp-display-fb0) — YUYV→RGB 硬件显示与内核 patch（Linux 4.9.88 BSP）
- [PXP USERPTR 零拷贝](/analysis/kernel/media/imx6-pxp-userptr-zero-copy) — CSI 缓冲直交 PXP，降低 CPU 占用（Linux 4.9.88 BSP）
- [V4L2 设备注册与 video 节点](/analysis/kernel/media/v4l2-device-registration)
- [V4L2 ioctl 分发](/analysis/kernel/media/v4l2-ioctl-dispatch)
- [videobuffer2：Buffer 状态机与双链表](/analysis/kernel/media/v4l2-vb2-queue)

## Pinctrl / GPIO 子系统

专题页：[Pinctrl / GPIO 子系统](/analysis/kernel/pinctrl/)

- [STM32 Pinctrl 分析](/analysis/kernel/pinctrl/stm32-pinctrl)
- [STM32 GPIO 分析](/analysis/kernel/gpio/stm32-gpio)

## SPI 子系统

专题页：[SPI 子系统](/analysis/kernel/spi/)

- [STM32MP157 SPI 子系统](/analysis/kernel/spi/spi-sync-trace)

## Sound / ALSA

专题页：[Sound / ALSA](/analysis/kernel/sound/)

内核以百问 **Linux 4.9.88 BSP** 为主（文首已注明；上面多数是 6.8）。

1. [ASoC 四层架构](/analysis/kernel/sound/imx6ull-asoc-layers) — Machine / Platform / CPU DAI / Codec
2. [`/dev/snd` 设备节点](/analysis/kernel/sound/imx6ull-snd-devices) — `ls /dev/snd` 看到的那些文件是干什么的
3. [声卡播放路径](/analysis/kernel/sound/imx6ull-audio-playback-flow) — `aplay` → SDMA / SAI / WM8960
4. [声卡录音路径](/analysis/kernel/sound/imx6ull-audio-capture-flow) — `arecord`、`read`、SAI RX
5. [ALSA PCM 状态机](/analysis/kernel/sound/alsa-pcm-state-xrun) — 状态、`start`/`stop`、underrun/overrun
6. [ALSA hw_params 参数协商](/analysis/kernel/sound/alsa-hw-params-negotiate) — 为什么 48 kHz 能播，96 kHz 就不行
7. [WM8960 kcontrol 构造与使用](/analysis/kernel/sound/wm8960-kcontrol) — `tinymix` 里那些音量名字是怎么来的
8. [amixer 改音量、拨开关：内核的不同调用](/analysis/kernel/sound/alsa-ctl-write-flow) — 一条只改数字，一条才会接通通路
9. [从 tinymix 到 WM8960 DAPM 路由](/analysis/kernel/sound/wm8960-dapm-routes) — 那些开关对应芯片里哪条线
10. [DAPM widget 上电：谁判、何时判](/analysis/kernel/sound/dapm-widget-power) — 电源管理

## BPF / kprobe

专题页：[BPF / kprobe](/analysis/kernel/bpf/)

内核以 Ubuntu HWE **5.15** 为主（文首已注明；上面多数是 6.8）。

1. [eBPF kprobe：load / attach / 命中](/analysis/kernel/bpf/ebpf-kprobe-load-attach) — PROG_LOAD / 插桩 / LINK / 多 BPF 分发
2. [kprobe-on-ftrace 插桩实测](/analysis/kernel/bpf/kprobe-on-ftrace-lab) — 入口 NOP↔CALL 与 handler 上下文

## 调试与实践

具体问题的排查记录，和上面的流程分析互补。概览：[调试与实践](/analysis/kernel/debug/)。

- [DJI Osmo UVC/UAC2 枚举排查](/analysis/kernel/debug/usb/dji-osmo-uvc-uac2-id-table) — `2ca3:8004` id_table 与 UAC2 probe
- [USB Device 公头悬空误报 Suspend](/analysis/kernel/debug/usb/floating-male-false-suspend) — 悬空 D± 触发 `USBSUSP`
- [UVC 拔出后 DQBUF 不返回](/analysis/kernel/debug/usb/uvc-disconnect-dqbuf-hang) — 断连未唤醒 `done_wq`，`vb2_queue_error()`
- [重启后 USB WiFi 概率枚举不到](/analysis/kernel/debug/usb/usb-wifi-reboot-power-residue) — 上电序列轮询超时，按序下电
- [libusb Windows 枚举失败](/analysis/kernel/debug/usb/libusb-windows-hcd-enum-fail) — 空 root hub 让 HCD 扫描轮整体失败
- [IMX6ULL SPI 片选 GPIO 时好时坏](/analysis/kernel/debug/gpio/imx6ull-spi-cs-gpio-runtime-pm) — runtime PM 覆盖 CS
- [根据 Oops 的 PC 定位到出错指令](/analysis/kernel/debug/oops-pc-to-source) — `func+off` / kallsyms / `objdump`
- [写作模板](/analysis/kernel/debug/template)
