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

## 硬件
STM32F103C8T6
NT1-B 串口转网口模块
路由器
ST-Link
USB-to-TTL 模块
杜邦线
## 软件
STM32CubeIDE
STM32CubeMX
MThings
Ebyte Network ConfigTool
ATK-XCOM
## 功能
STM32 模拟 Modbus RTU 从机
支持功能码 03：读取保持寄存器
支持功能码 06：写单个寄存器
NT1-B 实现 Modbus TCP 与 Modbus RTU 的双向转换
上位机可读取和修改寄存器数据
## 寄存器设计
地址	名称	权限	说明
0x0010 / 16	砂石种类	R/W	可由上位机读取和修改
0x0011 / 17	水分值	R	固定返回 128，表示 12.8%
## Modbus 功能码
功能码	作用
03	读取保持寄存器
06	写单个寄存器

## 示例报文

读取地址 16 和 17：

请求：01 03 00 10 00 02 C5 CE
响应：01 03 04 00 03 00 80 0B 93

写地址 16 为 5：

请求：01 06 00 10 00 05 49 CF
响应：01 06 00 10 00 05 49 CF

## 演示结果
成功读取地址 16：砂石种类
成功读取地址 17：水分值
成功写入地址 16 并再次读取验证
项目收获

## 通过本项目，我学习并实践了：

STM32 UART 串口通信
Modbus RTU 数据帧解析
Modbus TCP 与 RTU 协议转换
NT1-B 网关配置
MThings 上位机调试
嵌入式通信系统的调试流程
