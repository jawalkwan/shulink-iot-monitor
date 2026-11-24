# 快速上手

本文带你五分钟跑通一次完整流程：模拟一台 DTU 设备接入主站并响应 Modbus 请求。

## 前置条件

- 已安装 Shulink-Iot-Monitor（见 [安装指南](Installation)）
- 主站平台 / SCADA 已就绪并监听端口（没有现成平台？往下看有 Python 脚本方案）

## 第 1 步：创建设备

1. 启动应用，进入「设备列表」页
2. 点击「新建设备」，输入设备名称
3. 新设备默认处于 **Created（未启动）** 状态

## 第 2 步：配置主站连接

进入设备「配置」页：

| 配置项 | 说明 |
|---|---|
| 主站 IP / 端口 | 设备主动拨号的目标地址 |
| 帧格式 | `Modbus TCP`（MBAP 封装）或 `Modbus RTU over TCP`（带 CRC16），需与主站一致 |
| 自动重连 | 断线后自动重拨，重连间隔 1–300 秒 |
| 注册包 / 心跳 | 按需配置，详见 [注册包与心跳](Registration-and-Heartbeat) |

## 第 3 步：添加从站与寄存器

1. 在「从站管理」页为设备添加从站（Unit ID 1–247）
2. 为从站配置寄存器：名称、初值、数据类型（`bool` / `uint16` / `int16` / `uint32` / `int32` / `float32`）、单位、描述
3. 寄存器较多时可使用「批量导入」，详见 [寄存器配置](Register-Configuration)

## 第 4 步：启动设备

点击「启动」，设备会主动连接主站：

- 状态从 `Created` → `Connecting` → `Connected`
- 若开启了注册包，连接成功后自动发送
- 若开启了心跳，按配置周期发送

## 第 5 步：监控运行

进入「运行监控」页：

- 查看各从站四类内存区的实时值（支持自动刷新）
- 在线修改运行值（模拟主站写入效果）
- 开启寄存器随机模拟，让数据"活"起来（见 [随机模拟](Random-Simulation)）
- 查看 Modbus 请求 / 响应、注册包、心跳的收发日志（HEX / ASCII 双视图）

## 没有 主站平台？用 Python 模拟一个

设备是主动拨号接入的，主站侧只需监听端口并按帧收发。最简示例：

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

应用中把主站地址配置为 `127.0.0.1:9000`、帧格式选 `Modbus TCP`，启动设备即可看到完整报文交互。
