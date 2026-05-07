# STM32 Modbus TCP/RTU 水分仪模拟系统

基于 STM32F103C8T6 + NT1-B 的 Modbus TCP/RTU 工业通信模拟项目

---

# 1. 项目简介

本项目基于 STM32F103C8T6 实现了一个模拟工业水分仪系统。

STM32 作为 Modbus RTU 从机，通过 USART1 与 NT1-B 串口转网口模块进行通信。  
电脑端使用 MThings 作为 Modbus TCP 主机，通过路由器访问 NT1-B，实现对 STM32 寄存器数据的远程读取与写入。

本项目主要实现了：

- STM32 UART 串口通信
- Modbus RTU 协议实现
- Modbus TCP 与 RTU 协议转换
- TCP 客户端 / 服务端通信
- 串口转网口网关配置
- 工业通信调试流程

---

# 2. 系统架构

```text
电脑 / MThings（Modbus TCP 主机）
                │
                │ Modbus TCP
                ▼
            路由器 / 局域网
                │
                │ Ethernet
                ▼
 NT1-B 网关（TCP ⇄ RTU 协议转换）
                │
                │ UART / Modbus RTU
                ▼
 STM32F103C8T6（Modbus RTU 从机）
```

---

# 3. 硬件平台

| 硬件 | 说明 |
|---|---|
| STM32F103C8T6 | 主控制器 |
| NT1-B | 串口转网口模块 |
| 路由器 | 局域网通信 |
| ST-Link | 程序下载与调试 |
| USB-to-TTL | 串口调试 |
| 杜邦线 | 硬件连接 |

---

# 4. 软件工具

| 软件 | 功能 |
|---|---|
| STM32CubeIDE | STM32 工程开发 |
| STM32CubeMX | 外设配置 |
| MThings | Modbus TCP 主机调试 |
| Ebyte ConfigTool | NT1-B 参数配置 |
| ATK-XCOM | 串口调试 |

---

# 5. 硬件连接

## STM32 ↔ NT1-B

| STM32 | NT1-B |
|---|---|
| PA9（USART1_TX） | RX |
| PA10（USART1_RX） | TX |
| GND | GND |
| 5V | VCC |

---

# 6. 通信原理

电脑作为 Modbus TCP 主机，通过 TCP/IP 网络与 NT1-B 建立连接。

NT1-B 作为 TCP 服务端和协议转换网关，将 Modbus TCP 数据帧转换为 Modbus RTU 数据帧，通过串口发送给 STM32。

STM32 作为 Modbus RTU 从机，对收到的数据帧进行解析，根据功能码执行对应操作，并返回响应数据。

随后 NT1-B 再将 RTU 响应帧重新封装为 Modbus TCP 数据，发送回电脑端。

---

# 7. TCP 客户端 / 服务端结构

| 设备 | 角色 |
|---|---|
| 电脑 / MThings | TCP 客户端 |
| NT1-B | TCP 服务端 |
| STM32 | Modbus RTU 从机 |

电脑主动发起 TCP 连接，因此属于客户端。  
NT1-B 监听 502 端口等待连接，因此属于服务端。

---

# 8. Modbus 寄存器设计

本项目使用保持寄存器（Holding Registers）进行数据存储。

| 寄存器地址 | 名称 | 权限 | 说明 |
|---|---|---|---|
| 0x0010（16） | 砂石种类 | R/W | 可读写 |
| 0x0011（17） | 水分值 | R | 固定返回 128 |

---

# 9. 支持的 Modbus 功能码

| 功能码 | 功能 |
|---|---|
| 03H | 读取保持寄存器 |
| 06H | 写单个寄存器 |

---

# 10. Modbus 通信示例

## 读取寄存器

读取地址 16 和 17。

### 请求帧

```text
01 03 00 10 00 02 C5 CE
```

### 响应帧

```text
01 03 04 00 03 00 80 0B 93
```

### 响应帧解析

| 数据 | 含义 |
|---|---|
| 01 | 从机地址 |
| 03 | 功能码（读取寄存器） |
| 04 | 返回数据字节数 |
| 00 03 | 地址16数据：砂石种类 = 3 |
| 00 80 | 地址17数据：水分值 = 128 |

---

## 写入寄存器

将地址16写为5。

### 请求帧

```text
01 06 00 10 00 05 49 CF
```

### 响应帧

```text
01 06 00 10 00 05 49 CF
```

功能码06写入成功后，从机会返回原请求帧作为响应。

---

# 11. STM32 固件逻辑

STM32 固件主要完成以下功能：

1. 初始化 USART1
2. 接收 Modbus RTU 数据帧
3. 解析功能码
4. 执行寄存器读写操作
5. 构造响应帧
6. 计算 CRC16 校验
7. 通过 UART 返回数据

---

# 12. USART1 串口通信

USART1 用于 STM32 与 NT1-B 之间的串口通信。

| STM32 引脚 | 功能 |
|---|---|
| PA9 | USART1_TX |
| PA10 | USART1_RX |

串口参数：

```text
波特率：9600
数据位：8
校验位：None
停止位：1
```

---

# 13. Modbus TCP ⇄ RTU 协议转换

Modbus TCP 与 Modbus RTU 的主要区别在于数据帧格式。

## Modbus TCP

```text
包含 MBAP Header
不使用 CRC
基于以太网通信
```

## Modbus RTU

```text
使用 CRC16 校验
基于 UART 串口通信
数据帧更紧凑
```

NT1-B 在 TCP → RTU 转换过程中：

- 去除 TCP Header
- 添加 CRC16 校验

在 RTU → TCP 转换过程中：

- 去除 CRC16
- 添加 TCP Header

从而实现 Modbus TCP 与 RTU 的双向协议转换。

---

# 14. 实验结果

本项目成功实现：

- TCP 网络通信
- Modbus TCP 与 RTU 协议转换
- STM32 Modbus RTU 从机模拟
- 远程寄存器读取
- 远程寄存器写入

电脑端成功通过以太网读取和修改 STM32 中的寄存器数据。

---

# 15. 项目演示

## 成功读取寄存器

- 地址16 → 砂石种类
- 地址17 → 水分值

## 成功写入寄存器

- 写入地址16 = 5
- 再次读取验证成功

---

# 16. 项目收获

通过本项目，我学习并实践了：

- STM32 UART 串口通信
- Modbus RTU 数据帧结构
- Modbus TCP 通信
- TCP 客户端 / 服务端原理
- 工业通信调试流程
- 协议网关配置
- 嵌入式固件开发流程

---

# 17. 后续可扩展方向

后续可进一步扩展：

- 动态水分值模拟
- 更多 Modbus 功能码支持
- 接入真实传感器
- Web 监控界面
- 数据记录与可视化

---

# 18. 项目作者

Alex Liu

STM32 / Modbus / 工业通信学习项目
