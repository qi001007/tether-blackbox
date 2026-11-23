# Tether Blackbox · 小黑盒

> 随身携带的树莓派 AI 控制台：小屏对话 + 终端 + 通过 USB 网线/SSH 控制目标电脑。
> **状态：设计阶段，暂无实现代码。**

## 1. 这是什么

一台基于 Raspberry Pi 4B 的随身设备：

- 自己有一块小屏，主界面是**对话式 AI**，终端只是其中一个页签；
- 跑 Tether Node，使用与 Tether 相同的账号、Hub、设备注册；
- 通过 USB 网线与目标电脑建立点对点连接，用目标机自带的 SSH 控制它；
- 目标电脑可以完全离线；LLM/API 走黑盒自己的 WiFi/4G；
- 另一台电脑若本来就能装 Tether Node，仍然走 Node（最高档）；没有 Node 的机器走 SSH（无侵入档）。

定位：FDE/实验室现场的授权部署与操作工具，不是通用入侵工具。

## 2. 不做什么

- 不做“桌面 APP 的缩小版”；Pi 上是一套单独适配的轻量 UI。
- 不把纯视觉 Computer Use 当主循环；视觉只做兜底。
- 不承诺对未授权设备提权；没有 SSH/ADB/串口/预置执行器就没有高权限入口。
- 不把 Hub 搬进黑盒；Hub 仍是现有服务器。

## 3. 三档接入

| 档 | 目标上有什么 | 能力 | 前提 |
|---|---|---|---|
| 1. Node | Tether Node | 文件、终端、DSH、Agents、完整图形界面 | 能安装 Node |
| 2. SSH | 目标机自带 SSH | 终端、命令、文件；GUI 用 RDP/VNC 补 | 开一次 SSH + 放公钥 |
| 3. 兜底 | 无（KVM/HID/串口/ADB） | 看屏幕/键鼠/串口/Android | 目标支持相应通道 |

黑盒自己始终是 Tether 的一个 Node；目标是它下面的资源。桌面 APP 和黑盒 UI 都只是控制面。

## 4. 硬件原型

- Raspberry Pi 4B 4GB（第一版）
- 3.5"/5" 触摸屏（小屏对话 UI；也可用手机浏览器）
- 5V/3A 电源 + 电池/充电宝（后续做 UPS）
- USB-C OTG（对 PC 当 USB 设备：USB 网卡/存储/串口）
- USB-A host（对 Android/嵌入式当主机：ADB/串口）
- WiFi；4G 模块第二版
- 3D 打印外壳、风扇
- 后续：CM4/Zero 2 W + 自定义载板缩小体积

## 5. 软件原型

- 系统：Raspberry Pi OS Trixie Lite 64-bit
- 黑盒 UI：轻量 Web UI（本地 kiosk + 手机浏览器），不是 Electron APP
  - 对话主页（主入口）
  - 终端页签（xterm.js）
  - 目标列表 / 连接状态 / 延迟
  - 文件浏览（可选）
  - 设置：账号/Hub、WiFi/4G、API Key、电源/温度/风扇
- 黑盒服务：Tether Node + target adapters
  - `ssh` adapter：USB 网卡 + SSH（第一版）
  - `adb` / `serial` adapter：后续
  - `kvm/hid` adapter：兜底
- 目标侧：第一版不写自定义程序；用目标自带的 SSH。需要执行器的场景另开议题。

## 6. 与 Tether 的关系

- 同一个账号、同一个 Hub、同一套设备注册/身份；
- 黑盒是新的设备类型/Node 形态，不是桌面 APP 的分支；
- Node 协议尽量复用；UI 和目标适配器单独做；
- 具体复用边界见 `docs/`（待补）。

## 7. 当前待定

- Node 复用边界：哪些协议/身份/心跳直接复用，哪些要为新 UI 重写；
- UI 技术栈与小屏信息架构；
- USB 复合设备（网卡+存储+串口）与供电方案；
- 目标侧 SSH 开一次 + 公钥预置的流程；
- Windows 管理员 SSH 的 UAC 提权配置；
- 4G 模块、电池、外壳、散热；
- 许可证（软件可能 MIT，硬件可能 CERN-OHL，待定）。

## 8. 安全与授权

- 只连接你拥有或明确获授权管理的设备；
- 不实现隐蔽植入、自动提权、绕过目标安全策略；
- 黑盒默认显式连接、可审计、有物理授权开关；
- 目标机的权限由目标机自己给（SSH 账号/密钥、UAC、sudo），USB 线不制造权限。

## 9. 下一步

先把设计补齐：

1. `docs/architecture.md`：黑盒服务、UI、target adapters、Node 复用边界；
2. `docs/ui.md`：小屏信息架构与对话/终端交互；
3. `docs/hardware.md`：BOM、供电、USB 复合设备、外壳；
4. `docs/threat-model.md`：授权边界与审计。

代码在设计和验收判据对齐后开始。
