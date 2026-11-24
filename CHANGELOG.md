# 更新日志 (Changelog)

本项目的版本真源为内部 `Cargo.toml`，所有版本以 GitHub Release 为准。
安装包请前往 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases) 页面下载。

## [1.1.0] - 2026-11-24

- 【增】应用内检查更新（数据源为 GitHub Releases）：启动时静默检查（24 小时节流）并在发现新版时提示；设置页可开关自动检查、跳过指定版本；关于页可手动检查
- 【增】更新下载按形态自动匹配附件（安装包 / 绿色版），流式下载并显示进度、校验文件字节数；下载完成后安装包直接启动 NSIS 安装器，绿色版自动替换自身（旧文件以 `.old` 备份清理）
- 【增】更新安全策略：客户端不持有任何 Token，下载地址与打开的外链均校验 GitHub 域名白名单
- 【增】发布产物双形态：`setup.exe`（NSIS 安装包，含 WebView2 引导）与 `portable.exe`（绿色单文件，免安装）
- 【增】GitHub Release 一键打包发布脚本（`release` / `release:publish`）
- 【改】应用更名为 **Shulink-Iot-Monitor（SIM）**，中文名「枢灵 Iot 模拟器」：窗口标题、关于页与安装包 / 绿色版产物名（`Shulink-Iot-Monitor-x.y.z-*`）同步更新
- 【改】运行期数据目录迁移至 `%APPDATA%\com.shulink.iot.monitor`；旧导出格式标识 `modbus-slave-monitor.devices` 保持兼容，历史设备备份文件可正常导入
- 【改】修复发布流程在 Windows 下的多个兼容性问题：新版 Node 禁止直接 spawn `.cmd` 导致打包失败、`PATH` 环境变量键名大小写导致找不到 cargo、Release 附件经命令行参数传输被 `\0` 截断（改经 stdin 传输并在上传后校验字节数）、删除附件 API 路径 404

## [1.0.5] - 2026-11-23

- 【增】DTU 接入模式：设备作为 TCP 客户端主动连接主站平台，断线自动重连（间隔 1–300 秒可配）
- 【增】支持 Modbus TCP（MBAP）与 Modbus RTU over TCP（带 CRC16）两种帧格式
- 【增】设备 → 从站 → 寄存器三级模型：每台设备可挂多从站（Unit ID 1–247，按 Unit ID 路由），每从站四类内存区各最多 10000 地址，支持预留地址空间扩容
- 【增】功能码支持：0x01 / 0x02 / 0x03 / 0x04 / 0x05 / 0x06 / 0x0F / 0x10
- 【增】注册包与心跳在同一条 TCP 长连接收发；报文支持 Hex / String / Int8 / Int16 / Int32 / Uint8 / Uint16 / Uint32 八种数据类型；心跳间隔 1–300 秒
- 【增】寄存器随机模拟：按寄存器配置取值范围与刷新间隔自动写入，运行中支持热重载
- 【增】运行监控：实时查看四类内存区当前值、在线修改运行值、每设备 500 条环形通信日志（HEX / ASCII 双视图）
- 【增】寄存器 JSON 批量导入（含模板下载与导入预览）
- 【增】设备整树备份 device_bundle v2.0：导出 / 只读预检 / 导入，冲突策略支持覆盖 / 跳过 / 副本
- 界面：中文 / English / 日本語 三语言，浅色 / 深色主题，宽松 / 紧凑排版密度，无边框自绘标题栏，从站与寄存器独立子窗口编辑
- 提供两种发行形态：`setup.exe`（NSIS 安装包，含 WebView2 引导）与 `portable.exe`（免安装绿色版）

<!-- 新版本发布时在上方按相同格式追加条目 -->
