# 安装指南

## 系统要求

- Windows 10 / 11（x64）
- WebView2 Runtime（Win11 自带；Win10 若缺失，安装包会自动引导安装）

## 安装包形态

前往 [Releases](https://github.com/jawalkwan/shulink-iot-monitor/releases/latest) 页面下载：

| 文件 | 形态 | 说明 |
|---|---|---|
| `Shulink-Iot-Monitor-x.y.z-x64-setup.exe` | NSIS 安装包 | **推荐**。标准安装流程，含 WebView2 检测与引导安装 |
| `Shulink-Iot-Monitor-x.y.z-x64-portable.exe` | 绿色单文件 | 免安装，解压即用 / U 盘便携携带 |

## 便携版说明

- 便携版直接双击运行，无需安装
- 需目标机器已安装 [WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)，否则程序无法启动（无任何窗口）
- 便携版与安装版的数据目录相同，配置可互通

## 数据存放位置

所有配置数据保存在：

```
%APPDATA%\com.shulink.iot.monitor\
├── devices\{id}.json     # 每台设备的完整配置
├── sim_config.json       # 寄存器随机模拟配置
└── config.toml           # 应用与界面设置（语言、主题、排版密度）
```

备份整个目录即可完整迁移全部配置；也可使用应用内「导出 / 导入」功能做设备级迁移，见 [导入与导出](Import-and-Export)。

## 升级与卸载

- **升级**：直接下载新版本 setup.exe 覆盖安装，配置数据保留
- **卸载**：通过系统「设置 → 应用」卸载；配置数据目录需手动删除
