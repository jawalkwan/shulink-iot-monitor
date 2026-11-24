# 更新日志 (Changelog)

本项目的版本真源为内部 `Cargo.toml`，所有版本以 GitHub Release 为准。
安装包请前往 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases) 页面下载。

## [1.0.5] - 当前版本

- MODBUS TCP 从站（服务端）模拟器，支持功能码 0x01–0x06、0x0F、0x10
- 四类内存区：线圈 (0x)、离散输入 (1x)、输入寄存器 (3x)、保持寄存器 (4x)，每区最多 10000 个地址
- 自定义注册包（支持 `{device_id}`、`{timestamp}` 变量替换）与可配置心跳（1–300 秒）
- 实时寄存器监控与编辑（DEC/HEX 切换）
- 通信日志查看器
- 配置导入/导出（TOML 格式）、设备包导入/导出
- 深色/浅色主题
- 提供两种发行形态：`setup.exe`（NSIS 安装包，含 WebView2 引导）与 `portable.exe`（免安装绿色版）

<!-- 新版本发布时在上方按相同格式追加条目 -->
