<div align="center">

# Shulink-Iot-Monitor (SIM) · 枢灵 Iot 模拟器

**一款 Windows 桌面 Modbus 从站模拟器 —— 模拟 DTU 设备主动接入主站平台，为物联网监控平台、SCADA 系统的上位联调提供可靠数据源。**

*A Windows desktop Modbus slave simulator that emulates DTU devices dialing into a master platform — built for testing IoT / SCADA systems without real hardware.*

[![Latest Release](https://img.shields.io/github/v/release/jawalkwan/shulink-iot-monitor?label=%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC)](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest)
[![Platform](https://img.shields.io/badge/platform-Windows%20x64-blue)](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest)
[![Built with Tauri](https://img.shields.io/badge/built%20with-Tauri%202-24C8DB)](https://tauri.app)

</div>

---

## 目录

- [项目简介](#项目简介)
- [功能特性](#功能特性)
- [下载安装](#下载安装)
- [快速开始](#快速开始)
- [工作原理](#工作原理)
- [数据文件](#数据文件)
- [文档（Wiki）](#文档wiki)
- [常见问题](#常见问题-faq)
- [更新日志](#更新日志)
- [问题反馈](#问题反馈)
- [版权说明](#版权说明)

## 项目简介

在开发物联网监控平台或 SCADA 系统时，往往还没有真实的传感器 / DTU 硬件可用于联调。
**Shulink-Iot-Monitor** 在一台 Windows 电脑上就能批量模拟出整批从站设备：注册、心跳、寄存器读写、随机数据变化全部真实呈现，让上位平台的开发与测试不再依赖硬件。

技术栈：**Rust + Tauri 2 + Vue 3 + Element Plus**，原生性能，安装包体积小。

## 功能特性

### 设备 · 从站 · 寄存器三级模型

- 管理多台**设备（Device）**，每台设备下可挂载多个**从站（Slave）**，每个从站独立配置 Modbus 从机地址（Unit ID，1–247），请求按 Unit ID 精确路由
- 每个从站独立维护四类内存区：**线圈 (0x) / 离散输入 (1x) / 输入寄存器 (3x) / 保持寄存器 (4x)**，每区最多 10,000 个地址
- 寄存器条目可配置**名称、初值、数据类型**（`bool` / `uint16` / `int16` / `uint32` / `int32` / `float32`）、**单位与描述**，支持 JSON 批量导入与模板下载
- 支持**预留地址空间**扩容：主站固定轮询 0–99 这类场景下，避免读到未用地址时返回 `IllegalDataAddress` 异常

### DTU 接入模式

- 设备作为 TCP **客户端主动拨号**连接主站平台（可配置主站 IP / 端口），复现真实 DTU 的接入行为
- 支持 **Modbus TCP（MBAP 封装）** 与 **Modbus RTU over TCP（带 CRC16 校验）** 两种帧格式
- 断线**自动重连**，重连间隔可配置（1–300 秒）

### 注册包与心跳

- 注册包与心跳报文在**同一条 TCP 长连接**上收发（半双工长连接模型），与真实 DTU 行为一致
- 连接成功后可自动发送注册包（`send_on_startup`），心跳间隔可配置（1–300 秒）
- 报文内容支持 8 种数据类型：`Hex` / `String` / `Int8` / `Int16` / `Int32` / `Uint8` / `Uint16` / `Uint32`

### 随机数据模拟

- 可对任意寄存器配置**随机模拟**：设定取值范围（min / max）与刷新间隔，自动周期写入新值
- 设备运行中修改模拟配置支持**热重载**，立即生效，无需重启

### 运行监控与通信日志

- 实时监控各从站四类内存区的当前值，支持在线修改运行值（模拟主站写入）
- 每台设备独立的通信日志缓冲（500 条环形缓冲）：Modbus 请求 / 响应、注册包、心跳收发报文，同时提供 **HEX 与 ASCII** 双视图

### 导入 / 导出

- **寄存器批量导入**：上传 JSON 文件批量导入寄存器，支持下载导入模板、导入前预览
- **设备整树备份**（`device_bundle v2.0`）：设备 → 从站 → 寄存器整树导出为 JSON，导入前**只读预检**，支持三种冲突策略（覆盖 / 跳过 / 副本），可跨机器迁移、跨用户交接
- 格式细节见 [Wiki · 设备包格式](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Device-Bundle-Format)

### 界面

- 中文 / English / 日本語 三语言界面，一键切换
- 浅色 / 深色主题，宽松 / 紧凑两档排版密度
- 无边框自绘标题栏，从站编辑、寄存器编辑使用独立子窗口

## 下载安装

前往 [**Releases**](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest) 页面下载对应形态的安装包：

| 文件 | 形态 | 适合人群 |
|---|---|---|
| `Shulink-Iot-Monitor-x.y.z-x64-setup.exe` | NSIS 安装包（含 WebView2 引导） | 普通用户，**推荐** |
| `Shulink-Iot-Monitor-x.y.z-x64-portable.exe` | 绿色单文件，免安装 | 便携使用 / U 盘携带 |

> 便携版需目标机器已安装 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)（Win11 自带）；安装包会自动检测并引导安装。

## 快速开始

1. **创建设备**：启动应用，在「设备列表」新建一台设备
2. **配置主站连接**：进入设备「配置」页，填写主站 IP / 端口、帧格式（Modbus TCP / RTU over TCP）、自动重连；按需配置注册包与心跳
3. **添加从站与寄存器**：在「从站管理」页添加从站（Unit ID 1–247），为其配置寄存器名称、初值、数据类型
4. **启动设备**：设备主动连接主站，状态变为「已连接」
5. **监控寄存器**：进入「运行监控」查看实时值、修改运行值、开启随机模拟、查看通信日志

### 用 Python 脚本验证（模拟主站平台）

设备会主动拨号接入主站，因此在主站侧监听即可收发 Modbus 报文。下面用 20 行 Python 模拟一个最简主站平台：

```python
import socket

# 1. 在 9000 端口监听，等待设备（DTU）拨号接入
srv = socket.socket()
srv.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
srv.bind(("0.0.0.0", 9000))
srv.listen(1)
print("等待设备接入 ...")
conn, addr = srv.accept()
print("设备已接入:", addr)

# 2. 向从站 1 读取保持寄存器 0-9（FC 0x03，Modbus TCP 帧）
conn.send(bytes.fromhex("00010000000601030000000A"))
resp = conn.recv(1024)
print("响应 HEX:", resp.hex().upper())
```

在应用中把主站地址配置为 `127.0.0.1:9000`，帧格式选 `Modbus TCP`，启动设备即可看到完整收发报文。

## 工作原理

```
┌─────────────────────────────┐          TCP 长连接（注册 / 心跳 / Modbus 共用）
│  Shulink-Iot-Monitor (DTU)  │ ────────────────────────────────────►  主站平台 / SCADA
│                             │                                              │
│  设备 Device                │   ① 连接后自动发送注册包（可选）               │
│   ├─ 从站 Slave (Unit 1)    │   ② 周期发送心跳包（可选）                     │
│   │    └─ 四类内存区         │   ③ 响应主站的 Modbus 请求（FC 01/02/03/04/05/06/0F/10）
│   └─ 从站 Slave (Unit 2)    │                                              │
└─────────────────────────────┘                                              │
                                                                 按请求中的 Unit ID 路由到对应从站内存区
```

## 数据文件

应用数据保存在 `%APPDATA%\com.shulink.iot.monitor\`：

| 文件 / 目录 | 内容 |
|---|---|
| `devices\{id}.json` | 每台设备的完整配置（从站、寄存器、连接、注册包、心跳） |
| `sim_config.json` | 各设备寄存器随机模拟配置 |
| `config.toml` | 应用与界面设置（语言、主题、排版密度） |

设备配置可通过「设备列表 → 管理 → 导出 / 导入」整树备份为 JSON，实现跨机器迁移。

## 文档（Wiki）

详细使用说明请查看 **[Wiki](https://github.com/jawalkwan/shulink-iot-monitor/wiki)**：

- [安装指南](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Installation)
- [快速上手](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Quick-Start)
- [设备与从站管理](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Device-and-Slave-Management)
- [寄存器配置](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Register-Configuration)
- [注册包与心跳](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Registration-and-Heartbeat)
- [随机模拟](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Random-Simulation)
- [运行监控](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Runtime-Monitor)
- [导入与导出](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Import-and-Export)
- [设备包格式](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Device-Bundle-Format)
- [FAQ](https://github.com/jawalkwan/shulink-iot-monitor/wiki/FAQ)

## 常见问题 (FAQ)

**设备连接不上主站？**
确认主站 IP / 端口可达（可先用 `telnet <ip> <port>` 验证）；主站侧监听端口需放行防火墙；检查帧格式是否与主站一致（Modbus TCP / RTU over TCP）。

**主站读不到某些高地址？**
寄存器数组只覆盖实际用到的地址。请在对应内存区使用「预留地址空间」扩容，或在导入的设备包中补上 `space` 字段。详见 [Wiki · 设备包格式](https://github.com/jawalkwan/shulink-iot-monitor/wiki/Device-Bundle-Format)。

**注册包 / 心跳对端收不到？**
注册包与心跳走同一条连接，需主站按原始字节流读取；确认报文 HEX 内容与数据类型（Hex / String / 整型）选择正确。

**便携版启动无反应？**
大概率缺少 WebView2 Runtime，安装 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) 后重试，或改用 setup.exe 安装包。

更多问题见 [Wiki · FAQ](https://github.com/jawalkwan/shulink-iot-monitor/wiki/FAQ)。

## 更新日志

见 [CHANGELOG.md](CHANGELOG.md)；完整版本与安装包请前往 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases) 页面。

## 问题反馈

使用中遇到问题或有功能建议，欢迎提交 [Issue](https://github.com/jawalkwan/shulink-iot-monitor/issues)，请附上：

- 操作系统版本与应用版本号（「关于」页可见）
- 问题现象与复现步骤
- 相关通信日志（注意隐去敏感信息）

## 版权说明

**本项目未开源**。本仓库仅用于发布版本说明、使用文档与问题反馈，不包含源码；安装包请通过 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases) 页面获取。

未经授权，不得对本软件进行反编译、逆向工程或再分发。
