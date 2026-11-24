# Shulink-Iot-Monitor (SIM) · 枢灵 Iot 模拟器 Wiki

**Shulink-Iot-Monitor** 是一款 Windows 桌面 Modbus 从站模拟器：模拟 DTU 设备主动接入主站平台（物联网监控平台 / SCADA），为上位联调提供可靠数据源。

基于 **Rust + Tauri 2 + Vue 3 + Element Plus** 构建，原生性能，安装包体积小。

## 功能总览

- **设备 → 从站 → 寄存器** 三级模型，多设备、多从站（Unit ID 1–247）
- DTU 接入模式：TCP 客户端主动拨号接入主站，断线自动重连
- Modbus TCP（MBAP）与 Modbus RTU over TCP（CRC16）两种帧格式
- 功能码 0x01 / 0x02 / 0x03 / 0x04 / 0x05 / 0x06 / 0x0F / 0x10
- 四类内存区各最多 10,000 地址，支持预留地址空间
- 注册包与心跳（同一条 TCP 长连接，8 种报文数据类型）
- 寄存器随机模拟（支持热重载）
- 运行监控 + 通信日志（HEX / ASCII 双视图）
- 寄存器批量导入、设备整树备份（device_bundle v2.0）
- 中文 / English / 日本語 三语言，浅色 / 深色主题

## Wiki 目录

| 页面 | 内容 |
|---|---|
| [[Installation|安装指南]] | 下载安装包、便携版说明、WebView2 依赖 |
| [[Quick-Start|快速上手]] | 五分钟跑通：创建设备 → 配置主站 → 启动 → 监控 |
| [[Device-and-Slave-Management|设备与从站管理]] | 设备生命周期、从站配置、Unit ID 路由 |
| [[Register-Configuration|寄存器配置]] | 四类内存区、数据类型、批量导入、地址空间预留 |
| [[Registration-and-Heartbeat|注册包与心跳]] | 半双工长连接模型、报文数据类型、心跳间隔 |
| [[Random-Simulation|随机模拟]] | 取值范围、刷新间隔、运行中热重载 |
| [[Runtime-Monitor|运行监控]] | 实时值查看、运行值修改、通信日志 |
| [[Import-and-Export|导入与导出]] | 寄存器批量导入、设备整树备份与冲突策略 |
| [[Device-Bundle-Format|设备包格式]] | device_bundle v2.0 完整格式规范 |
| [[FAQ|常见问题]] | 连接排查、高地址读取、WebView2 等常见问题 |

## 快速链接

- 下载安装包：[Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest)
- 项目主页：[README](https://github.com/jawalkwan/shulink-iot-monitor#readme)
- 问题反馈：[Issues](https://github.com/jawalkwan/shulink-iot-monitor/issues)
