# STM32 Modbus Moisture Meter Simulation

## 项目简介

本项目基于 STM32F103C8T6 实现了一个模拟水分仪系统。STM32 作为 Modbus RTU 从机，通过 USART1 与 NT1-B 串口转网口模块通信。电脑端使用 MThings 作为 Modbus TCP 主机，通过路由器访问 NT1-B，实现对 STM32 寄存器数据的读取与写入。

## 系统结构

```text
PC / MThings
    ↓ Modbus TCP
Router
    ↓ Ethernet
NT1-B Gateway
    ↓ Modbus RTU / UART
STM32F103C8T6