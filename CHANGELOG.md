# 更新日志 (Changelog)

本项目的版本真源为内部 `Cargo.toml`，所有版本以 GitHub Release 为准。
安装包请前往 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases) 页面下载。

## [1.0.5] - 当前版本

- DTU 接入模式：设备作为 TCP 客户端主动连接主站平台，断线自动重连（间隔 1–300 秒可配）
- 支持 Modbus TCP（MBAP）与 Modbus RTU over TCP（带 CRC16）两种帧格式
- 设备 → 从站 → 寄存器三级模型：每台设备可挂多从站（Unit ID 1–247，按 Unit ID 路由），每从站四类内存区各最多 10000 地址，支持预留地址空间扩容
- 功能码支持：0x01 / 0x02 / 0x03 / 0x04 / 0x05 / 0x06 / 0x0F / 0x10
- 注册包与心跳在同一条 TCP 长连接收发；报文支持 Hex / String / Int8 / Int16 / Int32 / Uint8 / Uint16 / Uint32 八种数据类型；心跳间隔 1–300 秒
- 寄存器随机模拟：按寄存器配置取值范围与刷新间隔自动写入，运行中支持热重载
- 运行监控：实时查看四类内存区当前值、在线修改运行值、每设备 500 条环形通信日志（HEX / ASCII 双视图）
- 寄存器 JSON 批量导入（含模板下载与导入预览）
- 设备整树备份 device_bundle v2.0：导出 / 只读预检 / 导入，冲突策略支持覆盖 / 跳过 / 副本
- 界面：中文 / English / 日本語 三语言，浅色 / 深色主题，宽松 / 紧凑排版密度，无边框自绘标题栏，从站与寄存器独立子窗口编辑
- 提供两种发行形态：`setup.exe`（NSIS 安装包，含 WebView2 引导）与 `portable.exe`（免安装绿色版）

<!-- 新版本发布时在上方按相同格式追加条目 -->
