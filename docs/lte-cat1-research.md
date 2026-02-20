# LTE Cat 1 芯片研究与系统架构分析

> 研究日期：2026-02-20

---

## 一、LTE Cat 1 技术概述

### 1.1 标准定义

LTE Category 1（Cat 1）是 3GPP Release 8 定义的最低吞吐量 LTE 类别，后续 Release 13 引入 **Cat 1 bis** 变体，将原来需要 2 根天线（MIMO）的要求简化为 **单天线**，进一步降低了终端硬件成本和复杂度。

| 参数 | LTE Cat 1 | LTE Cat 1 bis |
|------|-----------|---------------|
| 3GPP Release | Rel. 8 | Rel. 13 |
| 下行峰值速率 | 10 Mbps | 10 Mbps |
| 上行峰值速率 | 5 Mbps | 5 Mbps |
| 天线数量 | 2 (MIMO 2×2) | 1 (SISO) |
| VoLTE 支持 | 是 | 是 |
| PSM/eDRX | 支持 | 支持 |
| 调制方式 | DL: 64QAM / UL: 16QAM | DL: 64QAM / UL: 16QAM |

### 1.2 技术定位

Cat 1 填补了以下空白：

```
NB-IoT (250kbps)  →  Cat 1 (10Mbps)  →  Cat 4 (150Mbps)
低速率/低功耗         中速率/中功耗         高速率/高功耗
```

**适用场景**：需要比 NB-IoT 更高数据速率，但不需要 Cat 4 的应用——如 POS 机、共享单车、工业 DTU、车载追踪、可穿戴设备、移动支付、智能表计。

---

## 二、主要芯片厂商与产品

### 2.1 国内主要厂商

#### 2.1.1 翱捷科技（ASR Microelectronics）

**市场地位**：2024 年全球 Cat 1 芯片市场份额约 **50%**，出货量领先。

| 芯片型号 | 工艺节点 | 主处理器 | 关键特性 |
|---------|---------|---------|---------|
| ASR1601 | — | ARM Cortex-R5 @ 624 MHz | 多模（LTE Cat1 + GSM），集成相机/音频 |
| ASR1603 | — | ARM Cortex-R5 @ 624 MHz | ASR1601 增强版，更多频段 |
| ASR3601 | — | ARM Cortex-R5 | 集成 RF，单芯片方案 |
| **ASR1606** | **22nm** | **ARM Cortex-R5 @ 624 MHz** | 集成 PMIC + Codec + pSRAM + Flash，最新旗舰 |

**ASR1601 架构亮点**：
- 通信子系统：LTE Cat1/1bis + GSM 基带，RF 收发器覆盖 450 MHz～2.7 GHz
- 应用子系统：Cortex-R5，集成摄像头接口（30W 像素）、SPI LCD、Audio Codec（含 VAD）
- 为客户应用预留 **70% 以上算力和 RAM 空间**
- 接口：UART、USB 2.0、SPI、I2C、GPIO、ADC、PWM、USIM

**ASR1606（新一代）架构亮点**：
- **22nm 制程**，更小封装，更低功耗
- 单 SoC 集成：基带 Modem + Cortex-R5 + Audio Codec + pSRAM + Flash + **PMIC**
- 继承 ASR160x 亿级稳定出货软件基线

---

#### 2.1.2 移芯通信（Eigencomm）

**市场地位**：与 UNISOC 共分剩余市场份额，以极致集成度见长。

| 芯片型号 | 封装 | 主处理器 | 关键特性 |
|---------|------|---------|---------|
| **EC618** | LFBGA 134球, 6.1×6.1mm | ARM Cortex-M3 @ 204 MHz | 全球首款基带+RF+PMIC三合一 |

**EC618 系统架构**：

```
┌─────────────────────────────────────────────┐
│                   EC618 SoC                  │
│  ┌──────────────┐   ┌──────────────────────┐ │
│  │  基带子系统   │   │    应用子系统          │ │
│  │  LTE Cat1bis │   │  Cortex-M3 @ 204MHz  │ │
│  │  调制解调器   │   │  256KB SRAM          │ │
│  └──────┬───────┘   └──────────────────────┘ │
│         │                                     │
│  ┌──────▼───────┐   ┌──────────────────────┐ │
│  │  RF 收发器   │   │    PMIC（内置电源管理） │ │
│  │  单天线SISO  │◄──┤  无需外部32K晶体       │ │
│  └──────────────┘   └──────────────────────┘ │
│                                               │
│  接口：SPI×2, I2C×2, UART×3, GPIO×32,       │
│         USB 2.0, PWM×6, ADC×4, USIM×2,      │
│         I2S, OneWire, Camera                  │
└─────────────────────────────────────────────┘
```

**EC618 核心优势**：
- **全球首款**基带 + RF + PMIC 三合一集成
- PSM 功耗低至 **1.3 μA**，连接态功耗下降 50%+
- 外围器件减少 **30%+**
- 封装仅 **6.1×6.1×1.14 mm**，pitch 0.5mm
- TDD 频段接收灵敏度达 **–101.x dBm**（优于竞品 2~3 dB）
- 支持 WiFi Scan（无需 WiFi 模块即可定位）
- 无需外部 32K 晶体

---

#### 2.1.3 紫光展锐（UNISOC）

| 芯片型号 | 特性 |
|---------|------|
| UIS8910DM (8910DM) | LTE Cat 1 bis，多模，丰富运营商生态 |
| V8850 | Cat 1 bis 优化版本 |

**8910DM 定位**：填补低功耗 NB-IoT 与传统宽带 IoT 之间的通信方案空白，打破国外垄断，拥有完善的运营商网络认证生态。

---

#### 2.1.4 上海海思（HiSilicon）

| 芯片型号 | 定位 | 特性 |
|---------|------|------|
| **Hi2131** | LTE Cat 1 | 2024年7月正式发布，优化功耗和信号接收，聚焦移动支付、共享经济 |
| Hi2115 | NB-IoT | 第二代"Boudica"系列，集成 MCU+存储+RF+PMIC+应用处理器，支持 nuSIM |

注：Hi2115 为 NB-IoT 而非 Cat 1。Hi2131 是海思首款 Cat 1 芯片，依托华为强大 R&D 资源，分析师预计 2026 年起快速抢占 ASR/移芯份额。

#### 2.1.5 信翼科技（XINYISEMI）

| 芯片型号 | 定位 | 特性 |
|---------|------|------|
| XY1100 | NB-IoT | 全球首款量产片内集成 CMOS PA 的 NB-IoT SoC，PSM 电流 700nA，累计出货超 1 亿片 |
| **XY4100LC/LD** | **LTE Cat 1 bis** | **MWC 2025 发布**，XY4100LD 在 ISSCC 2025 发表论文（中国大陆唯一受邀）|

ISSCC（IEEE 国际固态电路大会）是全球最顶级的芯片设计学术会议，与三星、英特尔同台意义重大。

---

### 2.2 国际厂商

#### 2.2.1 Sequans Communications

**代表产品：Calliope 系列**

| 芯片型号 | 代次 | 主要特性 |
|---------|------|---------|
| SQN3520 (Calliope 1) | 第一代（2015年）| 首款 Cat 1 工业 IoT 芯片 |
| **SQN3530 (Calliope 2)** | **第二代（2021年）**| 单芯片全集成，Rel-15，eUICC/ieUICC/UICC 灵活 SIM 管理 |
| Calliope 3（规划中）| 第三代 | 5G NR eRedCap + LTE Cat 1 bis 双模，与 Calliope 2 模组封装兼容 |

```
Calliope 2 (SQN3530) 架构：
┌──────────────────────────────────────┐
│         SQN3530 单芯片（WLP）         │
│  ┌────────────┐  ┌─────────────────┐ │
│  │  基带处理器 │  │  RF 收发器       │ │
│  │  LTE协议栈  │  │  晶圆级封装(WLP)│ │
│  └────────────┘  └─────────────────┘ │
│  ┌──────────────────────────────────┐ │
│  │  集成 IoT 应用处理器 + IMS 客户端 │ │
│  │  + Sequans AIR™ 干扰抑制技术     │ │
│  │  + eUICC/ieUICC/UICC 支持        │ │
│  └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

- 超低功耗 + 紧凑形态（基带和 RF 均采用晶圆级封装 WLP）
- 已通过：Verizon、AT&T、Bell、FirstNet、T-Mobile、Telus、Rogers、KDDI、NTT Docomo 认证
- 适用于可穿戴、M2M、智能计量、家庭自动化、车载
- **注**：2023-2024 年高通收购了 Sequans 的 4G IoT 技术组合

#### 2.2.2 Qualcomm

| 芯片型号 | 技术 | 核心 | 备注 |
|---------|------|------|------|
| MDM9205 | LTE Cat-M1 + NB-IoT | Cortex-A7 @ 800MHz，32MB DRAM+64MB Flash 片内集成 | 功耗比 MDM9206 降低 70% |
| MDM9206 | LTE Cat-M1 + NB-IoT | Cortex-A7 | 多模低功耗，早期 IoT 旗舰 |
| **QCX216** | **LTE Cat 1 bis** | **Cortex-M3 @ 204MHz** | **2022年12月发布，专用 Cat1bis IoT modem，FreeRTOS，Wi-Fi 扫描定位** |

**QCX216 关键规格**：
- RAM 1.25MB + Flash 4MB（片内）
- UART×4、USB 2.0、ADC×2、GPIO×4
- 工业温度范围 -40°C～+85°C
- 典型模组：Quectel EG916Q-GL、SIMCom SIM7672、Cavli C16QS（目标 $5 以内）
- Qualcomm 2024 年发布白皮书《Understanding the Benefits of LTE Cat 1bis Technology》

#### 2.2.3 Sony Altair（索尼旗下，原 Altair Semiconductor）

| 芯片型号 | 特性 |
|---------|------|
| **ALT1160** | LTE Cat 1，片内集成 DDR 内存 + PMIC，MCU 子系统，VoLTE，eDRX 省电 80%，可软件升级为 Cat-M1 |

已通过：AT&T、T-Mobile、NTT DoCoMo、KDDI 认证。

#### 2.2.4 RDA（现为 UNISOC 旗下）

- RDA8910：早期 Cat 1 方案，被 Logicrom SDK 等生态广泛支持

---

## 三、LTE Cat 1 系统架构深度分析

### 3.1 整体架构层次

```
┌─────────────────────────────────────────────────────────┐
│                      应用层（APP）                        │
├─────────────────────────────────────────────────────────┤
│              AT 命令接口 / OpenCPU API                   │
├─────────────────────────────────────────────────────────┤
│                   协议栈层                               │
│  ┌──────────────────────────────────────────────────┐   │
│  │  NAS（非接入层）: EMM / ESM                       │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  RRC（无线资源控制）                               │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  PDCP（分组数据汇聚协议）                          │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  RLC（无线链路控制）                               │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  MAC（媒体访问控制）                               │   │
│  ├──────────────────────────────────────────────────┤   │
│  │  PHY（物理层）：OFDM/SC-FDMA 调制解调              │   │
│  └──────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────┤
│              RF 子系统（射频收发器）                      │
│  PA / LNA / 滤波器 / 天线开关 / 单天线(Cat1bis)          │
├─────────────────────────────────────────────────────────┤
│                    天线                                   │
└─────────────────────────────────────────────────────────┘
```

### 3.2 基带处理器架构

#### 3.2.1 处理器核心

| 厂商 | 处理器 | 主频 | 说明 |
|------|--------|------|------|
| ASR | ARM Cortex-R5 | 624 MHz | 实时处理器，适合通信低延迟需求 |
| Eigencomm EC618 | ARM Cortex-M3 | 204 MHz | 低功耗微控制器内核 |
| Sequans | 自研 IoT CPU | — | 集成于基带 |
| UNISOC 8910DM | ARM 架构 | — | 多核设计 |

**Cortex-R5 选用原因**：
- 专为实时性要求高的嵌入式系统设计（Real-time 系列）
- 支持 Tightly Coupled Memory（TCM），降低内存访问延迟
- 支持 ECC 内存保护，提高可靠性
- 适合同时运行通信协议栈和用户应用

#### 3.2.2 内存子系统

```
┌─────────────────────────────────────┐
│           内存架构                   │
│  ┌─────────┐  ┌──────────────────┐  │
│  │  SRAM   │  │  Flash（NOR/NAND)│  │
│  │（片内）  │  │  （固件存储）     │  │
│  └─────────┘  └──────────────────┘  │
│  ┌─────────────────────────────────┐ │
│  │  pSRAM（伪静态 RAM，外挂或集成） │ │
│  │  用于运行时数据缓冲              │ │
│  └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

### 3.3 RF 子系统架构

#### 3.3.1 频段支持

LTE Cat 1 芯片通常支持以下频段组合：

**中国频段**：
- Band 1（2100 MHz）、Band 3（1800 MHz）、Band 5（850 MHz）
- Band 8（900 MHz）、Band 34（2010 MHz）
- Band 38（2600 MHz）、Band 39（1900 MHz）、Band 40（2300 MHz）、Band 41（2500 MHz）

**全球频段（以 ASR1601 为例）**：450 MHz～2.7 GHz 全覆盖

#### 3.3.2 RF 链路架构

```
天线
  │
  ▼
┌─────────────────────────────────────────────┐
│               RF 前端模块（FEM）              │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ 天线开关  │  │  滤波器  │  │  PA/LNA   │  │
│  │ (ANT SW) │  │ (SAW/BAW)│  │(放大器)   │  │
│  └──────────┘  └──────────┘  └───────────┘  │
└──────────────────────┬──────────────────────┘
                       │
┌──────────────────────▼──────────────────────┐
│              RF 收发器（RFIC）               │
│  ┌──────────────┐  ┌──────────────────────┐  │
│  │   接收链路   │  │     发射链路          │  │
│  │  LNA→混频器 │  │  DAC→上变频→滤波      │  │
│  │  →ADC→解调  │  │  →PA 控制            │  │
│  └──────────────┘  └──────────────────────┘  │
└──────────────────────┬──────────────────────┘
                       │
                  基带处理器
```

**Cat 1 bis 单天线优势**：
- 省去第二路 RF 链路（LNA + 混频器 + ADC）
- PCB 面积减小约 20-30%
- BOM 成本降低

### 3.4 协议栈详解

#### 3.4.1 用户面协议栈

```
UE 侧                          eNB 侧
─────────────────────────────────────────
应用层数据
    │
   PDCP（压缩/加密/完整性保护）
    │
   RLC（ARQ、分段重组）
    │   ← 无线链路 →
   MAC（调度、HARQ）
    │
   PHY（OFDMA-DL / SC-FDMA-UL）
    │
  ~~~~~ 空中接口 ~~~~~
```

#### 3.4.2 控制面协议栈

```
NAS（非接入层）: 移动性管理(EMM) + 会话管理(ESM)
    │
   RRC（无线资源控制）: 连接管理、测量上报、切换
    │
   PDCP / RLC / MAC / PHY
```

#### 3.4.3 PHY 层关键参数（Cat 1）

| 参数 | 下行（DL） | 上行（UL） |
|------|-----------|-----------|
| 多址方式 | OFDMA | SC-FDMA |
| 子载波间隔 | 15 kHz | 15 kHz |
| FFT 大小 | 512（5MHz）～2048（20MHz） | — |
| 最大带宽 | 20 MHz | 20 MHz |
| 最高调制阶数 | 64QAM | 16QAM |
| MIMO 层数 | 1（Cat1bis） / 2（Cat1） | 1 |
| HARQ 进程数 | 8 | 8 |

### 3.5 电源管理架构

#### 3.5.1 工作状态

```
┌─────────────────────────────────────────────┐
│              功耗状态机                      │
│                                             │
│  ACTIVE ──→ IDLE ──→ PSM(省电模式)          │
│   (mA级)    (μA级)   (< 5μA)               │
│                ↑                            │
│           eDRX（扩展非连续接收）             │
│           周期最长 43.69 分钟               │
└─────────────────────────────────────────────┘
```

| 功耗模式 | EC618 | ASR1606 | 说明 |
|---------|-------|---------|------|
| PSM | **1.3 μA** | — | 超低功耗待机 |
| 连接态（RRC Connected） | 降低 50%+ | — | 相比上代 |
| 发射峰值 | ~500 mA | ~500 mA | 取决于频段和功率等级 |

#### 3.5.2 PMIC 集成趋势

- **EC618**：首款集成 PMIC 的 Cat 1 芯片，支持多路 LDO 和 DC-DC 变换器
- **ASR1606**：新一代集成 PMIC + pSRAM + Flash
- 效果：外围器件减少 30%+，PCB 面积缩小，BOM 成本降低

### 3.6 主机接口

```
┌───────────────────────────────────────────────────┐
│                 主机接口层                         │
│                                                   │
│  ┌─────────┐  ┌─────────┐  ┌─────────────────┐   │
│  │UART(AT) │  │USB 2.0  │  │SPI / SDIO       │   │
│  │主要控制  │  │数据传输  │  │嵌入式高速场景   │   │
│  └─────────┘  └─────────┘  └─────────────────┘   │
│                                                   │
│  ┌──────────────────────────────────────────┐     │
│  │    OpenCPU / 内置应用处理器模式           │     │
│  │  用户代码直接运行在芯片内（无需外部 MCU） │     │
│  └──────────────────────────────────────────┘     │
└───────────────────────────────────────────────────┘
```

**两种集成模式**：

| 模式 | 说明 | 优缺点 |
|------|------|--------|
| AT 命令模式 | 外部 MCU 通过 UART/USB 发送 AT 指令 | 灵活，开发简单；需额外 MCU |
| OpenCPU 模式 | 用户代码运行在芯片内置处理器上 | 省去外部 MCU；需了解 SDK |

---

## 四、主流芯片横向对比

| 对比项 | ASR1606 | Eigencomm EC618 | UNISOC 8910DM | Qualcomm QCX216 | Sequans SQN3530 |
|--------|---------|----------------|--------------|----------------|----------------|
| 标准 | Cat.1 bis Rel-14 | Cat.1 bis Rel-14 | Cat.1 bis + GSM | Cat.1 bis | Cat.1 bis Rel-15 |
| 制程 | **22nm** | — | 28nm | — | — |
| 处理器 | Cortex-R5 @ 624MHz | Cortex-M3 @ 204MHz | Cortex-A5 @ 500MHz | Cortex-M3 @ 204MHz | 自研 IoT CPU |
| 片内内存 | PSRAM+Flash集成 | 1MB SRAM+4MB Flash | 需外挂 | 1.25MB RAM+4MB Flash | — |
| 封装 | 小 | 6.1×6.1mm（极小） | 8.9×8.9mm | — | WLP（极小） |
| PSM功耗 | — | **1.3μA** | 低 | ~10mA 空闲 | 超低 |
| PMIC 集成 | 是 | 是 | 外置 | 外置 | 是 |
| 蓝牙 | 否 | 否 | **BT 4.2（片内）** | 否 | 否 |
| Wi-Fi Scan | 否 | 是 | 是 | 是 | 否 |
| VoLTE | 是 | 是 | 是 | — | 是 |
| OpenCPU | 是 | 是 | 是（V8850） | 是（FreeRTOS）| 是 |
| 多模（含2G） | LTE+GSM | 纯4G | LTE+GSM | 纯4G | 纯4G |
| 主要市场 | 中国/全球 | 中国 | 中国/全球 | 全球（欧美） | 欧美日韩 |
| 市场份额 | **~50%** | ~18% | ~25% | 入局 | 国际市场 |
| 运营商认证 | 国内三大 | 国内三大 | 45+国家 | AT&T/Verizon等 | 9大运营商 |

---

## 五、典型应用场景架构

### 5.1 资产追踪器

```
GPS 模块
    │
    ▼
MCU（可选，OpenCPU模式下省略）
    │
    ▼
LTE Cat 1 模组（基于 EC618/ASR1601）
    │  UART/USB
    ▼
4G LTE 网络 → 云平台
    │
    ▼
Web/App 管理界面
```

### 5.2 POS 机 / 移动支付

```
扫码模块（Camera/扫描头）
    │
    ▼
应用处理器（本地业务逻辑）
    │
    ▼
LTE Cat 1 芯片（ASR1606/EC618）
    │  10Mbps DL + VoLTE
    ▼
银行/支付清算网络
```

### 5.3 工业 DTU

```
传感器 / 工业设备（RS485/RS232）
    │
    ▼
LTE Cat 1 DTU 模块
    ├── AT 命令接口（UART）
    ├── 透明数据传输
    └── TCP/UDP/MQTT/HTTP 协议栈
         │
         ▼
    云端工业平台
```

---

## 六、2024-2025 市场趋势

### 6.1 市场规模

- 2024 年全球 Cat 1 芯片出货量 **超过 2.5 亿片**
- Cat 1 bis 芯片市场 2024 年规模约 **10.45 亿美元**，预计 2031 年达 **16.32 亿美元**
- 蜂窝物联网模组（工业+车载）2024 年增长 **约 16%**；中国模组出货量同比增长 **21%**
- 2025 年全球蜂窝物联网模组预计出货 **5.44 亿片**，营收 **39.3 亿美元**（+23% YoY）
- 中国市场驱动：POS、共享经济、工业物联网、2G/3G 退网替换
- 2025 年中国市场预计增长 **10-15%**；西方市场库存去化后反弹至高个位数增长

### 6.2 技术趋势

1. **单芯片 SoC 化**：基带 + RF + PMIC 三合一（EC618 引领，ASR1606 跟进）
2. **制程升级**：向 22nm 及以下演进（ASR1606、UNISOC V8821 已达 22nm）
3. **功耗优化**：PSM 电流向 1μA 以下挑战（XY1100 NB-IoT 已达 700nA）
4. **生态扩展**：OpenCPU 使芯片直接承载客户应用，省去外部 MCU
5. **全球化**：Cat 1 bis 从中国向全球扩张，替代 2G/3G 退网市场
6. **向 5G 演进路径**：Sequans Calliope 3、ASR1903（5G RedCap）提供升级路径，eRedCap 填补 Cat 1 bis 与 5G 间的空白
7. **非地面网络（NTN）**：UNISOC V8821 已面向卫星 IoT，代表蜂窝物联网的下一前沿

### 6.3 竞争格局

```
市场份额（2024年估算）：
ASR 翱捷        ████████████████████ ~50%
UNISOC 展锐     ████████████ ~25%
Eigencomm 移芯  ████████ ~18%
其他（海思、信翼、高通等）██ ~7%
```

**格局展望**：
- 海思 Hi2131 于 2024 年 7 月正式入局，依托华为资源预计 2026 年起快速扩张
- 信翼科技（ISSCC 2025）技术能力获国际认可，高端市场潜力大
- 高通 QCX216 主攻欧美市场，不与中国厂商在国内正面竞争

### 6.4 NB-IoT vs Cat 1 bis 趋势

- NB-IoT 在中国部分场景（燃气表）出现**向 Cat 1 bis 反向迁移**
- 原因：Cat 1 bis 速率更高、支持 VoLTE、覆盖相近、成本差距缩小
- NB-IoT 仍在中国燃气表、印度及部分东南亚场景保持份额

---

## 七、开发生态

| 生态组件 | 描述 |
|---------|------|
| **LuatOS** | 基于 EC618 等平台的开源 Lua 物联网开发框架 |
| **Logicrom SDK** | 支持 ASR1601/1603、RDA8910 的 C 语言 SDK |
| **ASR 官方 SDK** | ASR 提供的完整开发套件，支持 OpenCPU 和 AT 模式 |
| **UNISOC SDK** | 展锐官方开发环境 |
| **Sequans SDK** | 包含 LTE 协议栈 + 应用开发接口 |
| **AT 命令标准** | 3GPP TS 27.007 + 各厂商扩展 AT 指令集 |

---

## 参考资料

### 官方规格与数据手册
- [ASR1601 官方规格](https://www.asrmicro.com/en/goods/proinfo/7.html)
- [ASR1606 官方规格](http://www.asrmicro.com/en/goods/proinfo/9.html)
- [ASR3601 官方规格](http://www.asrmicro.com/en/goods/proinfo/39.html)
- [UNISOC 8910DM 产品页](https://www.unisoc.com/cn_zh/home/TGYWLW-8910DM-7)
- [Sequans Calliope 2 产品页](https://sequans.com/products/calliope-2/)
- [Qualcomm QCX216 产品简介（PDF）](https://docs.qualcomm.com/bundle/publicresource/87-PW324-1_REV_C_Qualcomm_QCX216_LTE_IOT_Modem_Product_Brief.pdf)
- [Qualcomm MDM9205 产品简介（PDF）](https://www.qualcomm.com/content/dam/qcomm-martech/dm-assets/documents/9205-lte-modem-product-brief_87-pw321-1.pdf)
- [Sony Altair ALT1160](https://altair.sony-semicon.com/products/alt1160/)

### 开源与开发者资源
- [Eigencomm EC618 - SoCXin](https://github.com/SoCXin/EC618)
- [ASR1601 - SoCXin](https://github.com/SoCXin/ASR1601)
- [LuatOS EC618 文档](https://wiki.luatos.org/chips/air780e/mcu.html)
- [Logicrom SDK（ASR1601/RDA8910）](https://github.com/waybyte/logicromsdk)

### 技术分析与市场报告
- [Qualcomm LTE Cat 1bis 白皮书（2024）](https://www.qualcomm.com/content/dam/qcomm-martech/dm-assets/documents/whitepaper_understanding_the_benefits_of_lte_cat_1bis_technology.pdf)
- [2024 Cellular IoT Module Market Update - IoT Business News](https://iotbusinessnews.com/2025/02/19/02010-2024-cellular-iot-module-market-update/)
- [2025-2026 蜂窝物联网模组市场展望 - IoT Business News](https://iotbusinessnews.com/2026/02/10/cellular-iot-modules-market-outlook-2025-2026-strong-growth-in-2025-structural-pressures-ahead/)
- [Cat.1 bis 芯片市场 2025-2032 预测](https://www.intelmarketresearch.com/cat-bis-chip-2025-2032-359-5192)
- [高新兴 GM196H（基于 ASR1606）发布](https://www.gosuncn.com/article/565.html)
- [Cat.1 芯片模组产业分析 - CSDN](https://blog.csdn.net/a1809032425/article/details/134057126)
- [网红 Cat.1 诞生背景 - 知乎](https://zhuanlan.zhihu.com/p/123924614)
- [非"魔改"专属 Cat 1 芯片解析 - 半导体行业观察](http://www.semiinsights.com/s/package_test/35/39162.shtml)
- [Assessing the LTE Cat-1 bis market - RCR Wireless（2023）](https://www.rcrwireless.com/20230109/carriers/assessing-the-lte-cat-1-bis-market-and-qualcomms-late-entry-into-it-reader-forum)
- [LTE Cat 1 bis 综合指南 - Cavli Wireless](https://www.cavliwireless.com/blog/nerdiest-of-things/ultimate-guide-to-lte-cat-1bis-technology)
