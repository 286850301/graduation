# 电信系统：协议与技术

![License](https://img.shields.io/badge/Academic-Coursework-blue)
![Status](https://img.shields.io/badge/Status-进行中-yellow)

本仓库包含为《电信系统协议与技术》课程开发的一系列仿真模型、分析报告和程序。工作重点在于深入研究和实践实现网络栈各层的各种通信协议。

---

## 📚 实验列表

该课程包含8个核心实验和1个个人任务，涵盖从物理层分析到网络管理和流量优先级的各个方面。

| 序号 | 实验名称 | 研究领域 | 关键技术 |
|---|-------|------------|------------------|
| [实验 1](Simulations/lab1/) | **物理层与数据链路层分析** | 底层协议性能分析 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 2](Simulations/lab2/) | **网络分析与管理系统研究** | 网络监控与管理工具 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 3](Reports/LR3/) | **应用自动化监控的设备管理** | 设备管理的自动化监控 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 4](Reports/LR4/) | **广域网络架构研究** | 广域网架构 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 5](Reports/LR5/) | **网络层交互原理研究** | 网络层协议与路由 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 6](Reports/LR6/) | **流量优先级划分原理分析** | 现代网络中的QoS与流量管理 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 7](Reports/LR7/) | **自动化网络监控系统与协议分析** | 网络监控协议与系统 | IEEE 802.11系列, LTE, 5G, LoRaWAN, ZigBee, Ethernet, PLC |
| [实验 8](Reports/LR8/) | **IEEE 802.11系列介质访问控制协议** | Wi-Fi中MAC协议的对比分析 | IEEE 802.11系列 (b, a, g, n, ac, ax, be等) |

## 🎯 个人任务

**状态图设计、程序开发与协议分析**

个人任务需要从以下列表中选择一个特定协议进行深入研究：
- **应用层协议:** `CoAP`, `BOOTP`, `SMTP`, `MQTT`, `FTP`, `HTTP`, `AMQP`, `XMTP`, `DHCP`
- **传输层/网络层协议:** `X.25`, `ICMP`
- **数据链路层协议:** `MAC`
- **串行通信协议:** `I2C`, `SPI`, `UART`, `RS232`, `USB`
- **概念:** `IoT`, `LAN`, `WLAN`, `冗余协议`

## 🛠 涉及的技术与协议

### 无线技术
- **Wi-Fi (IEEE 802.11):** a, b, g, n, ac, ax, be, ad, ay, ah, af, ai, aq, i
- **蜂窝网络:** GSM (CSD), GPRS, 3G, LTE, 5G
- **无线个域网与物联网:** IEEE 802.15 (如 ZigBee), LoRaWAN, NB-IoT
- **无线城域网:** IEEE 802.16 (WiMAX)

### 有线及其他技术
- **有线局域网:** IEEE 802.3 (以太网)
- **电力线通信:** PLC

### 按层划分的核心协议
- **应用层:** HTTP, SMTP, FTP, MQTT, CoAP, AMQP, DHCP
- **传输层:** TCP/UDP (隐含), X.25
- **网络层:** ICMP
- **数据链路层:** MAC, 各种LLC协议
- **物理层:** SPI, I2C, UART, RS232, USB

## 📁 仓库结构

```bash
Telecom-Systems-Protocols-and-Technologies/
├── 📂 Reports/               # 所有实验报告的目录
│   ├── LR1/                 # 实验1：物理层与数据链路层
│   ├── LR2/                 # 实验2：网络分析与管理系统
│   ├── ...                  # 其他实验 (LR3 到 LR8)
│   └── Individual/          # 个人任务
├── 📂 Simulations/          # 仿真模型源代码 (如 NS-3, Python)
│   ├── LR1_802.11ac_Model/
│   ├── LR8_MAC_Comparison/
│   └── ...
├── 📂 Diagrams/             # 状态图、网络架构图 (如 .drawio, .png)
│   └── Individual_Protocol_State_Diagram.png
├── 📜 README.md             # 本文件
└── 📜 .gitignore
```

## 🚀 开始使用

### 环境准备
- **仿真工具:** [NS-3](https://www.nsnam.org/), OMNeT++ 或类似工具。
- **编程语言:** Python (使用 Matplotlib, NumPy 等库), C++。
- **建模工具:** 绘图工具 (如 Draw.io, Dia)。
- **文档撰写:** LaTeX 或 Word，用于按照 ГОСТ 7.32 标准撰写报告。

### 如何使用本仓库
1.  **导航**到特定的实验文件夹 (例如 `Reports/LR1/`) 查找报告和相关材料。
2.  **查看** `Simulations/` 目录下的仿真代码。每个子文件夹对应一个特定的实验或分析。
3.  **查看** `Diagrams/` 文件夹，获取协议状态机和网络架构的可视化图表。

## 📄 报告格式

所有实验报告均按照 **俄罗斯国家标准 ГОСТ 7.32** 进行格式化。每份报告包括：
- 工作名称
- 引言 (目标、任务)
- 理论背景
- 仿真模型描述
- 带分析的仿真结果 (图表、表格)
- 结论
