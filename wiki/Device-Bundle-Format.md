# 设备备份文件格式说明（device_bundle v2.0）

本文档定义本工具的 **设备整树** 备份 / 迁移 JSON 格式（`device_bundle`）。

备份粒度为：

```text
设备 (Device)  →  从站 (Slave)  →  寄存器 (Register)
```

用途：

1. **数据备份**：把选中设备的配置整包导出到本地文件；
2. **跨环境迁移**：在测试 / 生产、或两台机器之间导入；
3. **配置交接**：把设备整树交给同事复现同样的从站与寄存器布局。

> 入口：设备列表页 →「管理」→ 勾选 →「导出」；「导入」按钮常驻工具栏。
>
> v1 旧格式（`format: "modbus-slave-monitor.devices"`，内部 `Device` 原样快照）**只读兼容**，
> 导出端不再产出。

如果其他工具（含 AI）需要生成可供本工具导入的 JSON 文件，请严格按本文档生成。

---

## 1. 顶层结构

备份文件是一个 **JSON 对象**（不是数组）：

| 字段             | 类型     | 必填 | 说明 |
| ---------------- | -------- | ---- | ---- |
| `format_version` | string   | 是   | 格式版本号。当前为 `"2.0"`（主版本必须是 `2`） |
| `export_type`    | string   | 是   | 固定字符串 `"device_bundle"`。导入时据此识别格式 |
| `exported_at`    | string   | 否   | 导出时间，ISO 8601，仅记录，不参与校验 |
| `source`         | object   | 否   | 导出来源统计（仅人类阅读，导入整体忽略） |
| `devices`        | object[] | 是   | 设备数组，**不能为空**；元素结构见第 3 节 |

### 1.1 `source` 对象（可选）

| 字段           | 类型    | 说明 |
| -------------- | ------- | ---- |
| `app`          | string  | 固定 `shulink-iot-monitor`（旧版本导出的文件可能为 `modbus-slave-monitor`，导入时不校验） |
| `app_version`  | string  | 导出版本号 |
| `device_count` | integer | 设备数量 |
| `slave_count`  | integer | 从站数量 |
| `register_count` | integer | 寄存器条目数量（不含空白占位） |

---

## 2. 与内部存储的差异（务必理解）

内部存储（`%APPDATA%\com.shulink.iot.monitor\devices\{id}.json`）与本交换格式**刻意不同**：

| 对比项 | 内部存储 | `device_bundle` |
| ------ | -------- | --------------- |
| 地址表达 | 数组下标（隐式） | `address` 字段（显式） |
| 寄存器组织 | 四个数组 `coils` / `discrete_inputs` / `input_registers` / `holding_registers` | 一个扁平数组，每条带 `type` |
| 空白占位 | 数组里保留（地址空间） | **不导出**，改由 `space` 记录长度 |
| 运行态 `status` | 落盘 | 不导出，导入后一律回到 `Created` |
| `created_at` | 落盘 | 不导出，导入时按「当前时间 + 序号」重排以保持列表顺序 |

因此导出的文件体积通常只有内部设备文件的十分之一。

---

## 3. `devices[]` 设备对象

| 字段               | 类型           | 必填 | 说明 |
| ------------------ | -------------- | ---- | ---- |
| `id`               | string         | 是   | 设备 ID（如 `device_1760625389`）。主键；已存在时按冲突策略处理 |
| `name`             | string         | 否   | 设备名称；为空时回落到 `id` |
| `master_connection`| object         | 否   | 主站连接配置，见 3.1 |
| `registration`     | object         | 否   | DTU 注册包配置，见 3.2 |
| `heartbeat`        | object         | 否   | 心跳包配置，见 3.3 |
| `sim_config`       | object[]       | 否   | 随机模拟配置，见 3.4 |
| `slaves`           | object[]       | 是   | 从站数组，可为空 `[]` |

**不导出的设备字段**：`status`（运行态）、`created_at`（派生）。

### 3.1 `master_connection`

| 字段                 | 类型    | 默认值     | 说明 |
| -------------------- | ------- | ---------- | ---- |
| `master_ip`          | string  | `127.0.0.1`| 主站 IP；为空时导入回落默认值 |
| `master_port`        | integer | `502`      | 主站端口；为 0 时回落默认值 |
| `auto_reconnect`     | boolean | `true`     | 是否自动重连 |
| `reconnect_interval` | integer | `5`        | 重连间隔（秒），导入时下限 1 |
| `modbus_mode`        | string  | `Tcp`      | 帧格式：`Tcp`（MBAP）/ `Rtu`（RTU over TCP） |

### 3.2 `registration`（注册包）

| 字段             | 类型    | 默认值 | 说明 |
| ---------------- | ------- | ------ | ---- |
| `packet_data`    | string  | `""`   | 注册包内容（HEX 字符串） |
| `data_type`      | string  | `Hex`  | `Hex` / `String` / `Int8` / `Int16` / `Int32` / `Uint8` / `Uint16` / `Uint32` |
| `send_on_startup`| boolean | `true` | 连接主站后是否自动发送 |

### 3.3 `heartbeat`（心跳包）

| 字段              | 类型    | 默认值 | 说明 |
| ----------------- | ------- | ------ | ---- |
| `packet_data`     | string  | `""`   | 心跳包内容 |
| `data_type`       | string  | `Hex`  | 同 3.2 |
| `interval_seconds`| integer | `30`   | 心跳间隔（秒），导入时限制在 1–300 |
| `enabled`         | boolean | `true` | 是否启用 |

### 3.4 `sim_config[]`（随机模拟配置）

| 字段          | 类型    | 说明 |
| ------------- | ------- | ---- |
| `slave_id`    | string  | 所属从站 ID |
| `type`        | string  | 寄存器区，取值同 5.1 |
| `address`     | integer | 地址 |
| `min`         | integer | 随机下界 |
| `max`         | integer | 随机上界 |
| `interval_ms` | integer | 刷新间隔（毫秒） |
| `enabled`     | boolean | 是否启用 |

> 内部用拼接字符串键 `device|slave|area|address` 存储，交换格式改为结构化字段，便于人工编辑与跨工具生成。

---

## 4. `devices[].slaves[]` 从站对象

| 字段       | 类型     | 必填 | 说明 |
| ---------- | -------- | ---- | ---- |
| `id`       | string   | 是   | 从站 ID，在同一设备内唯一；重复或缺失会被跳过并记原因码 |
| `name`     | string   | 否   | 从站名称；为空时回落到 `id` |
| `unit_id`  | integer  | 否   | Modbus 从机地址，导入时限制在 1–247，默认 `1` |
| `space`    | object   | 否   | 每个区预留的地址数，见 4.1 |
| `registers`| object[] | 是   | 寄存器数组，可为空 `[]` |

**不导出的从站字段**：`status`（运行态）；从站级遗留的 `registration` / `heartbeat`（已统一收归设备级）。

### 4.1 `space`（地址空间声明，可选）

```json
"space": { "coils": 0, "discrete_inputs": 0, "input_registers": 0, "holding_registers": 100 }
```

每个值是该区**可寻址的地址数量**，导入后数组长度即为该值。

**何时需要写**：主站固定轮询 0–99，而实际只用了个别地址时——不写 `space`，
导入后的数组会收缩到「实际用到的最大地址 + 1」，主站轮询高地址将收到 `IllegalDataAddress` 异常。

导出端只在「该区存在尾部空白占位」时才写 `space`；从站不需要固定区间时可以整段省略。

---

## 5. `slaves[].registers[]` 寄存器对象

扁平数组，每条自带区域类型与地址：

| 字段         | 类型    | 必填 | 说明 |
| ------------ | ------- | ---- | ---- |
| `address`    | integer | 是   | 地址（0 起）。无法解析为整数 → 跳过，记 `invalid_address` |
| `type`       | string  | 否   | 寄存器区，默认 `holding`，见 5.1 |
| `name`       | string  | 否   | 寄存器名称 |
| `value`      | integer | 否   | **配置初值**，默认 `0`，范围 0–65535；位区按 0/1 处理 |
| `data_type`  | string  | 否   | `bool` / `uint16` / `int16` / `uint32` / `int32` / `float32`；位区强制 `bool` |
| `unit`       | string  | 否   | 单位，如 `℃` |
| `description`| string  | 否   | 描述 |

> `value` 是**配置初值**而非实时值：本工具的 Modbus 处理程序只读写运行内存，
> 从不回写配置文件，因此该值不会因为设备运行而变化。

### 5.1 `type` 取值

| `type` | 对应内部区域 | 说明 |
| ------ | ------------ | ---- |
| `coil` | `coils` | 线圈（可读写） |
| `discrete` | `discrete_inputs` | 离散输入（只读） |
| `input` | `input_registers` | 输入寄存器（只读） |
| `holding` | `holding_registers` | 保持寄存器（可读写，默认） |

也接受复数写法 `coils` / `discrete_inputs` / `input_registers` / `holding_registers`。

---

## 6. 导入行为说明

### 6.1 流程

```text
选择 JSON 文件
    → preview_import_devices（只读预检，不落盘）
    → 界面展示：可导入 N 台 / 跳过 M 台，逐条列出动作与原因
    → 用户选择冲突策略并确认
    → import_devices（按计划逐台落盘）
    → 返回逐设备结果摘要
```

### 6.2 冲突策略

| 策略       | 设备 ID 已存在时 |
| ---------- | ---------------- |
| `overwrite` | 覆盖原设备（导入前若在运行会先停止） |
| `skip`     | 整台跳过，原因 `device_id_exists` |
| `copy`     | 分配新 ID 作为副本导入，原设备不动 |

### 6.3 三级判定

| 层级     | 条件                                | 结果                              | `reason`               |
| -------- | ----------------------------------- | --------------------------------- | ---------------------- |
| 设备     | 包内 `devices[].id` 重复            | **整包预检失败**                  | —                      |
| 设备     | 缺少 `id`                           | **整包预检失败**                  | —                      |
| 设备     | `id` 已存在且策略为 `skip`          | 整台跳过                          | `device_id_exists`      |
| 从站     | 所属设备被跳过                      | 随设备跳过                        | 随设备                 |
| 从站     | 同一设备下 `id` 重复                | 跳过该从站及其寄存器              | `slave_id_duplicated`   |
| 从站     | 缺少 `id`                           | 跳过该从站                        | `slave_id_missing`      |
| 寄存器   | 所属从站被跳过                      | 随从站跳过                        | 随从站                 |
| 寄存器   | 同从站同区 `address` 重复（包内）   | 跳过该条（保留首条）              | `address_exists`        |
| 寄存器   | `address` 无法解析为整数            | 跳过该条                          | `invalid_address`       |
| 设备     | 写入时落盘异常                      | 该设备回滚，不影响其他设备        | `import_error: ...`     |

### 6.4 落盘细则

1. **数组重建**：先按 `space` 预分配，再按 `address` 写入；最终长度 = `max(space, 最大地址 + 1)`
2. **重复地址**：同从站同区出现两次时保留首条，其余计入 `register_skipped`
3. **模拟配置**：`sim_config` 随设备写入 `sim_config.json`
4. **运行态**：导入后设备与从站一律 `Created`，**不自动启动**
5. **顺序保持**：`created_at` 重排为「导入时刻 + 序号」，使设备列表顺序与文件一致
6. **失败隔离**：单台设备落盘异常只回滚该台并标记 `import_error`，其余设备照常导入

### 6.5 预检响应示例

```json
{
  "summary": {
    "deviceTotal": 2, "deviceImportable": 1, "deviceSkipped": 1,
    "slaveTotal": 2, "slaveImportable": 1, "slaveSkipped": 1,
    "registerTotal": 6, "registerImportable": 3, "registerSkipped": 3
  },
  "items": [
    {
      "deviceId": "device_a1b2c3d4",
      "deviceName": "1号DTU",
      "action": "skip",
      "reason": "device_id_exists",
      "slaveTotal": 1, "slaveImported": 0, "slaveSkipped": 1,
      "registerImported": 0, "registerSkipped": 3,
      "slaves": []
    },
    {
      "deviceId": "device_new0001",
      "deviceName": "新设备",
      "action": "create",
      "reason": null,
      "slaveTotal": 1, "slaveImported": 1, "slaveSkipped": 0,
      "registerImported": 3, "registerSkipped": 0,
      "slaves": [
        {
          "slaveId": "sensor_new0001",
          "slaveName": "温湿度传感器",
          "action": "import",
          "reason": null,
          "registerImported": 3,
          "registerSkipped": 0
        }
      ]
    }
  ]
}
```

> 命令返回结构为 camelCase；**文件内容**为 snake_case，两者不要混淆。

---

## 7. 完整模板示例

可直接复制修改后用于导入测试：

```json
{
  "format_version": "2.0",
  "export_type": "device_bundle",
  "exported_at": "2026-10-19T09:00:00+00:00",
  "source": {
    "app": "shulink-iot-monitor",
    "app_version": "1.0.1",
    "device_count": 1,
    "slave_count": 1,
    "register_count": 4
  },
  "devices": [
    {
      "id": "device_1760625389",
      "name": "1号DTU",
      "master_connection": {
        "master_ip": "192.168.1.10",
        "master_port": 502,
        "auto_reconnect": true,
        "reconnect_interval": 5,
        "modbus_mode": "Tcp"
      },
      "registration": {
        "packet_data": "01000000000601030000000A",
        "data_type": "Hex",
        "send_on_startup": true
      },
      "heartbeat": {
        "packet_data": "010000000006010300000001",
        "data_type": "Hex",
        "interval_seconds": 30,
        "enabled": true
      },
      "sim_config": [
        {
          "slave_id": "device_1760625389_slave_1760625400",
          "type": "holding_registers",
          "address": 2,
          "min": 0,
          "max": 100,
          "interval_ms": 1000,
          "enabled": true
        }
      ],
      "slaves": [
        {
          "id": "device_1760625389_slave_1760625400",
          "name": "温湿度传感器",
          "unit_id": 1,
          "space": {
            "coils": 1,
            "discrete_inputs": 0,
            "input_registers": 0,
            "holding_registers": 100
          },
          "registers": [
            { "address": 0, "type": "coil",     "name": "风机",   "value": 0, "data_type": "bool",   "unit": "",     "description": "启动信号" },
            { "address": 2, "type": "holding", "name": "温度",   "value": 265, "data_type": "int16", "unit": "℃",    "description": "环境温度" },
            { "address": 3, "type": "holding", "name": "湿度",   "value": 55, "data_type": "uint16", "unit": "%RH",  "description": "环境湿度" },
            { "address": 10, "type": "holding", "name": "阈值",  "value": 300, "data_type": "uint16", "unit": "℃",   "description": "温度上限" }
          ]
        }
      ]
    }
  ]
}
```

### 7.1 最小可导入示例

```json
{
  "format_version": "2.0",
  "export_type": "device_bundle",
  "devices": [
    {
      "id": "device_min0001",
      "name": "最小示例设备",
      "slaves": [
        {
          "id": "slave_min0001",
          "name": "最小示例从站",
          "registers": [
            { "address": 0, "name": "寄存器1" }
          ]
        }
      ]
    }
  ]
}
```

### 7.2 无从站设备

允许 `"slaves": []`。导入后得到一台空设备，可再在界面上添加从站。

---

## 8. `reason` 取值一览

| 值                        | 层级   | 含义 |
| ------------------------- | ------ | ---- |
| `device_id_exists`         | 设备   | 设备 ID 已存在（`skip` 策略） |
| `slave_id_duplicated`      | 从站   | 同一设备下从站 ID 重复 |
| `slave_id_missing`         | 从站   | 从站缺少 `id` |
| `address_exists`           | 寄存器 | 同从站同区地址重复 |
| `invalid_address`          | 寄存器 | `address` 无法解析为整数 |
| `not_planned`              | 设备   | 未纳入导入计划（异常兜底） |
| `import_error: ...`        | 设备   | 该设备写入时异常，整台回滚 |

---

## 9. 常见错误对照

| 现象 | 原因 | 处理 |
| ---- | ---- | ---- |
| `不支持的导出类型 'xxx'` | `export_type` 不是 `device_bundle`，也不是旧版格式 | 检查是否误用了别的格式文件 |
| `文件版本 3.0 高于当前支持的版本 2.0` | 主版本不兼容 | 用当前工具导出的文件 |
| `文件内存在重复的设备 ID: xxx` | 同一 JSON 里 `devices[].id` 重复 | 去重后重试 |
| `存在缺少 ID 的设备，无法导入` | 某台设备没有 `id` | 补上 `id` |
| `文件中没有可导入的设备` | `devices` 为空数组 | 检查导出时是否真的选中了设备 |
| 预检全是 skip | 目标机器已有相同 ID 且策略为跳过 | 换成「覆盖」或「副本」 |
| 导入后主站读不到高地址 | 文件未带 `space`，或该区未预留 | 用「预留地址空间」扩容，或在文件里补 `space` |
| 导入后设备列表顺序变了 | `created_at` 不参与交换，按导入时刻重排 | 同一次导入内相对顺序与文件一致 |

---

## 10. 不在本格式范围内

以下数据**不会**随 `device_bundle` 导出或导入：

- 连接运行态（连接中 / 已连接 / 断开 / 错误）
- 设备与从站的 `status`
- 主站轮询到的实时值（`RegisterEntry.value` 是配置初值，不是实时值）
- 通信日志
- 界面偏好（主题、语言、排版密度）

需要这些数据时，请在对应页面单独处理。

---

## 11. 生成文件命名建议

导出接口不强制文件名；前端默认形如：

```text
devices_bundle_YYYY-MM-DDTHH-mm-ss.json
```

跨环境迁移时建议在文件名或沟通中注明来源与导出时间，避免多版本混淆。
