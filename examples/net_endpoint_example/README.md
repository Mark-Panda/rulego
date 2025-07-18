# NET端点示例

这个示例演示了如何使用RuleGo的NET端点来处理二进制数据，并使用JavaScript转换节点进行数据处理。

## 新增功能 🆕

NET端点现在支持**多种数据包分割模式**，提供更加灵活和通用的数据包处理能力：

### 支持的分割模式

1. **按行分割 (line)** - 默认模式，向后兼容
   - 以 `\n` 或 `\r\n` 分隔数据包
   - 适用于文本协议

2. **固定长度分割 (fixed)** 
   - 每个数据包固定字节数
   - 适用于结构化二进制协议

3. **自定义分隔符分割 (delimiter)**
   - 支持自定义分隔符（字符串或十六进制）
   - 适用于特殊分隔符协议

4. **长度前缀分割 (length_prefix)**
   - 数据包前缀包含长度信息
   - 适用于变长数据包协议

## 功能特性

- **NET端点服务器**：基于TCP/UDP协议的网络端点
- **灵活数据包分割**：支持4种不同的数据包分割模式
- **二进制数据处理**：自动识别和处理二进制设备命令
- **JavaScript处理**：使用js_transform_node进行灵活的数据转换
- **协议解析**：解析自定义二进制协议格式
- **响应处理**：支持各种格式的响应数据
- **安全防护**：包大小限制、超时保护等安全机制

## 项目结构

```
net_endpoint_example/
├── server/                    # 服务端
│   ├── server.go              # NET端点服务器
│   └── chain_dsl.json         # 规则链DSL配置（使用delimiter模式）
├── client/                    # 客户端  
│   └── client.go              # NET客户端示例
├── alternative_solutions/     # 各种分割模式示例 🆕
│   ├── README.md              # 详细的模式说明文档
│   ├── fixed_length_example.json      # 固定长度协议示例
│   ├── length_prefix_example.json     # 长度前缀协议示例
│   └── custom_delimiter_example.json  # 自定义分隔符示例
└── README.md                  # 本文档
```

## 配置示例

### 基础配置（原有）
```json
{
  "protocol": "tcp",
  "server": ":8088",
  "readTimeout": 300
}
```

### 新增配置选项
```json
{
  "protocol": "tcp",
  "server": ":8088",
  "readTimeout": 300,
  "packetMode": "delimiter",        // 数据包分割模式
  "delimiter": "0x0D0A",           // 自定义分隔符（CRLF）
  "maxPacketSize": 1024            // 最大包大小限制
}
```

### 固定长度模式
```json
{
  "packetMode": "fixed",
  "packetSize": 16                 // 固定16字节数据包
}
```

### 长度前缀模式
```json
{
  "packetMode": "length_prefix",
  "lengthPrefixSize": 2,           // 2字节长度前缀
  "lengthPrefixBigEndian": true,   // 大端序
  "lengthIncludesPrefix": false    // 长度不包含前缀本身
}
```

## 数据处理功能

### 二进制数据处理（原有）
- **协议解析**：deviceId(2字节) + command(1字节) + value(4字节)
- **命令支持**：
  - `0x01` SET_PARAMETER - 设置参数
  - `0x02` GET_STATUS - 获取状态  
  - `0x03` RESET - 设备重置
  - `0x04` SET_THRESHOLD - 设置阈值
- **响应格式**：状态(1字节) + 设备ID回显(2字节) + 命令回显(1字节) + 换行符(1字节)

### 新增协议支持 🆕
- **AT命令协议**：支持类似 `AT+INFO\r\n` 的命令格式
- **CSV格式协议**：支持 `SENSOR,001,TEMP,25.6\r\n` 格式
- **Modbus ASCII**：支持简化的Modbus ASCII协议
- **固定长度协议**：支持工业设备的固定格式协议
- **变长协议**：支持带长度前缀的消息队列协议

## 快速开始

### 1. 启动服务器（原有示例）

```bash
cd examples/net_endpoint_example/server
go run server.go
```

服务器将在 `:8088` 端口启动，使用delimiter模式处理以 `\r\n` 分隔的数据。

### 2. 启动不同模式的服务器 🆕

#### 固定长度服务器（端口8090）
```bash
go run server.go -config=../alternative_solutions/fixed_length_example.json
```

#### 长度前缀服务器（端口8091）
```bash  
go run server.go -config=../alternative_solutions/length_prefix_example.json
```

#### 自定义分隔符服务器（端口8092）
```bash
go run server.go -config=../alternative_solutions/custom_delimiter_example.json
```

### 3. 运行客户端

```bash
cd examples/net_endpoint_example/client  
go run client.go
```

客户端将发送相应格式的数据，并显示服务器响应。

## 性能与安全

### 性能特性 🆕
- **固定长度** - 性能最佳，CPU开销最小
- **长度前缀** - 性能良好，支持变长数据
- **自定义分隔符** - 性能中等，需要逐字节扫描
- **按行分割** - 性能中等，针对文本优化

### 安全防护 🆕
- **包大小限制** - 默认64KB，防止内存攻击
- **读取超时** - 防止慢速攻击
- **格式验证** - 严格的协议格式检查
- **错误处理** - 完善的异常处理机制

## 示例数据

### 二进制命令格式（原有）
```
设备ID(2字节) + 命令(1字节) + 值(4字节)
例如：03 E9 01 00 00 00 64 (设备1001, 命令0x01, 值100)
```

### 新增格式示例 🆕

#### AT命令格式
```
AT+INFO\r\n          -> 设备信息查询
AT+CONFIG=mode=auto\r\n -> 配置设置
```

#### CSV格式
```
SENSOR,001,TEMP,25.6\r\n -> 传感器数据上传
COMMAND,002,RESET\r\n    -> 设备命令
```

#### 长度前缀格式
```
00 05 10 12 34 56 78  -> 长度5字节 + 消息类型0x10 + 数据
```

## 预期输出

### 客户端输出示例（更新）
```
NET Client Example
Connecting to server at localhost:8088
Connected to server successfully

=== Sending Binary Data ===
Protocol Mode: delimiter (0x0A)
Sending binary data 1: Device=1001, Command=0x01, Value=100 (hex: 03 E9 01 00 00 00 64)
Response Time: 15.2ms
=== Binary Response ===
Length: 5 bytes
Hex: 01 03 E9 01 0A
Decoded:
  Status: 0x01 (Success)  
  Device ID: 1001 (0x03E9)
  Command Echo: 0x01
  Protocol: delimiter_0x0A
========================
```

## 配置说明

### 端点配置（chain_dsl.json）
- **协议**：TCP/UDP
- **端口**：8088（delimiter）、8090（fixed）、8091（length_prefix）、8092（custom_delimiter）
- **超时**：300秒
- **分割模式**：delimiter（\r\n分隔符）
- **包大小限制**：1024字节

### 处理器配置
- **setBinaryDataType**：设置二进制数据类型
- **responseToBody**：格式化响应内容  
- **协议特定处理**：根据不同分割模式的专门处理逻辑

## 技术要点

1. **向后兼容**：默认使用line模式，保持与现有代码兼容
2. **灵活配置**：支持多种数据包分割模式的配置
3. **安全可靠**：包大小限制、超时保护、错误处理
4. **高性能**：根据协议特点选择最优的分割算法
5. **易于扩展**：基于接口的设计，可以轻松添加新的分割模式

## 注意事项

- 确保对应端口未被占用
- 客户端和服务器需要使用相同的协议格式
- 服务器使用Ctrl+C优雅关闭
- 支持多个客户端同时连接
- 新增的分割模式需要相应的客户端支持

## 更多示例

详细的配置示例和使用方法请参考 `alternative_solutions/` 目录中的文档和配置文件。