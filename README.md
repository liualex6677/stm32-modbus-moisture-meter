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
