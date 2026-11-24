# Shulink-Iot-Monitor (SIM) · 枢灵 Iot 模拟器

**Shulink-Iot-Monitor** 是一款 Windows 桌面应用，用于模拟 MODBUS Slave（服务端）设备，为上位机 / 监控平台的联调与测试提供可靠的数据源。基于 Tauri 构建，原生性能，安装包体积小。

> A Windows desktop application for simulating MODBUS TCP Slave (server) devices — built for testing SCADA / IoT platforms without real hardware.

[![Latest Release](https://img.shields.io/github/v/release/jawalkwan/shulink-iot-monitor?label=%E6%9C%80%E6%96%B0%E7%89%88%E6%9C%AC)](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest)
[![Platform](https://img.shields.io/badge/platform-Windows%20x64-blue)](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest)
[![License](https://img.shields.io/badge/license-MIT-green)](#license)

## 截图

<!-- 截图就绪后替换下面的占位；图片放 screenshots/ 目录，用相对路径引用 -->
| 主界面 | 寄存器监控 |
|:---:|:---:|
| ![主界面](screenshots/main.png) | ![寄存器监控](screenshots/registers.png) |

## 功能特性

### MODBUS TCP 从站模拟

- 完整支持标准功能码：`0x01` `0x02` `0x03` `0x04` `0x05` `0x06` `0x0F` `0x10`
- 可配置监听 IP / 端口（默认 `0.0.0.0:502`）与 Unit ID（1–247）
- 支持多客户端并发连接

### 四类内存区

| 内存区 | 读写 | 说明 |
|---|---|---|
| 线圈 Coils (0x) | 读/写 | 位寄存器 |
| 离散输入 Discrete Inputs (1x) | 只读 | 位寄存器 |
| 输入寄存器 Input Registers (3x) | 只读 | 16 位寄存器 |
| 保持寄存器 Holding Registers (4x) | 读/写 | 16 位寄存器 |

每个内存区最多支持 10,000 个地址，界面实时监控并可直接编辑寄存器值（DEC / HEX 切换）。

### 注册包与心跳

- 自定义注册包内容（HEX 字符串），支持 `{device_id}`、`{timestamp}` 等变量替换
- 可配置心跳间隔（1–300 秒），注册与心跳走独立 TCP 连接
- 支持手动"立即发送注册包"
- 实时通信日志，收发报文一目了然

### 其他

- 配置导入 / 导出（TOML 格式），设备配置打包导入 / 导出
- 深色 / 浅色主题切换
- 中文界面，自适应系统字体

## 下载安装

前往 [**Releases**](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest) 页面下载对应形态的安装包：

| 文件 | 形态 | 适合人群 |
|---|---|---|
| `Shulink-Iot-Monitor-x.y.z-x64-setup.exe` | NSIS 安装包（含 WebView2 引导） | 普通用户，**推荐** |
| `Shulink-Iot-Monitor-x.y.z-x64-portable.exe` | 绿色单文件，免安装 | 便携使用 / U 盘携带 |

> 便携版需目标机器已安装 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)（Win11 自带）；安装包会自动检测并引导安装。

## 快速开始

1. **启动应用**：运行安装好的程序
2. **配置服务端**：进入「配置」页，设置监听 IP（默认 `0.0.0.0`）、端口（默认 `502`）与 Unit ID
3. **启动服务**：点击工具栏绿色「启动」按钮，状态指示灯变绿即表示服务已运行
4. **监控寄存器**：进入「寄存器」页，选择内存区查看实时值，可直接编辑
5. **注册 / 心跳**（可选）：进入「Reg/HB」页，配置目标服务器 IP / 端口与报文内容，开启自动心跳

### 用上位机工具验证

任意 MODBUS Master 工具均可连接测试，例如 Python + pymodbus：

```python
from pymodbus.client import ModbusTcpClient

client = ModbusTcpClient('127.0.0.1', port=502)
client.connect()

# 读保持寄存器 0-9（Unit ID = 1）
result = client.read_holding_registers(0, 10, slave=1)
print(result.registers)

client.close()
```

GUI 工具推荐 [QModMaster](https://sourceforge.net/projects/qmodmaster/)：连接 `localhost:502`，Unit ID 与配置保持一致即可读写。

## 配置文件

应用配置保存为 TOML 格式，可导出备份或跨机器迁移：

```toml
[modbus_server]
listen_ip = "0.0.0.0"
listen_port = 502
unit_id = 1
max_connections = 50
coil_count = 1000
discrete_input_count = 1000
input_register_count = 1000
holding_register_count = 1000

[registration]
target_ip = "127.0.0.1"
target_port = 9000
packet_hex = "01000000000601030000000A"
send_on_startup = false
use_variables = false
device_id = "DEVICE001"

[heartbeat]
target_ip = "127.0.0.1"
target_port = 9000
packet_hex = "010000000006010300000001"
interval_seconds = 30
enabled = false
```

## 文档

详细使用说明请查看 **[Wiki](https://github.com/jawalkwan/shulink-iot-monitor/wiki)**：

- [安装指南](https://github.com/jawalkwan/shulink-iot-monitor/wiki)
- [使用手册](https://github.com/jawalkwan/shulink-iot-monitor/wiki)
- [设备包格式](https://github.com/jawalkwan/shulink-iot-monitor/wiki)
- [FAQ](https://github.com/jawalkwan/shulink-iot-monitor/wiki)

## 常见问题 (FAQ)

**端口 502 被拒绝访问？**
Windows 上低于 1024 的端口可能需要管理员权限：以管理员身份运行，或改用其他端口（如 5020）。

**Windows 防火墙弹窗？**
服务启动时防火墙可能弹窗提示，请点击「允许访问」，否则上位机无法连入。

**便携版启动无反应？**
大概率缺少 WebView2 Runtime，安装 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/) 后重试，或改用 setup.exe 安装包。

更多问题见 [Wiki FAQ](https://github.com/jawalkwan/shulink-iot-monitor/wiki)。

## License

[MIT](LICENSE)
