---
title: FPGA基本知识
date: 2026-05-31 15:00:00
tags: FPGA
category: FPGA
---

### 一、FPGA 简介

#### 1. FPGA 是什么

FPGA（Field Programmable Gate Array，现场可编程门阵列）是一种可以通过硬件描述语言（HDL）或高层次综合（HLS）进行**反复配置**的半导体芯片。

与 ASIC（专用集成电路）不同，FPGA 不需要流片即可修改逻辑；与 CPU 不同，FPGA 通过并行硬件电路实现功能，适合对时序、延迟、IO 速率要求高的场景。

#### 2. FPGA 与 CPU、MCU 的区别

| 对比项 | CPU / MCU | FPGA |
| --- | --- | --- |
| 执行方式 | 顺序执行指令 | 并行逻辑电路 |
| 灵活性 | 软件易改 | 硬件逻辑可重构 |
| 性能特点 | 通用计算强 | 低延迟、高并行 IO |
| 开发方式 | C/C++ 等 | Verilog / VHDL / HLS |
| 典型场景 | 业务逻辑、操作系统 | 接口转换、信号处理、原型验证 |

简单理解：CPU 像「一位全能工人按步骤干活」，FPGA 像「一群专用工人同时干活」。

#### 3. 常见应用领域

- 通信：5G 基带、光口、以太网 MAC/PHY 适配
- 工业：高速采集、运动控制、PLC 扩展
- 视频：图像采集、编解码预处理、显示驱动
- 芯片验证：ASIC 原型验证
- 人工智能：低延迟推理加速（配合 DSP/ARM 硬核）

### 二、FPGA 内部结构

#### 1. 基本组成

一块 FPGA 芯片通常包含：

- **CLB / LUT**（可配置逻辑块 / 查找表）：实现组合逻辑与寄存器
- **布线资源**（Routing）：连接各逻辑单元
- **Block RAM**：片上存储器
- **DSP Slice**：乘法累加，适合数字信号处理
- **IO Block**：连接外部引脚，支持多种电平标准
- **时钟网络**（Clock Tree）：全局/ regional 时钟

#### 2. LUT 与寄存器

FPGA 的核心是可编程逻辑。以 LUT（Look-Up Table）为例，本质是用存储表实现任意布尔函数；再配合 D 触发器（FF）构成时序电路。

```
输入信号 → LUT（组合逻辑）→ FF（寄存器）→ 输出
                ↑
              时钟
```

同一个 FPGA，通过不同配置可以变成 UART、SPI 主从、FIFO、计数器等不同电路。

#### 3. 硬核与软核

| 类型 | 说明 | 示例 |
| --- | --- | --- |
| 硬核（Hard IP） | 芯片内固化电路 | ARM 处理器、PCIe、GTX 收发器 |
| 软核（Soft IP） | 用逻辑资源实现 | MicroBlaze、Nios、RISC-V 软核 |
| 软逻辑 | 用户自定义模块 | 自定义 SPI、PWM、状态机 |

带 ARM 硬核的 FPGA 常称为 **SoC FPGA**（如 Xilinx Zynq、Intel Cyclone V SoC）。

### 三、开发流程

#### 1. 典型流程

```
需求分析 → 架构设计 → RTL 编码 → 功能仿真
    → 综合 → 实现（布局布线）→ 时序分析 → 下载验证
```

#### 2. 各阶段说明

| 阶段 | 工具/产物 | 目标 |
| --- | --- | --- |
| RTL 设计 | Verilog / VHDL | 描述电路行为 |
| 仿真 | ModelSim、Vivado Simulator | 验证功能正确 |
| 综合 | Vivado Synthesis | RTL → 门级网表 |
| 实现 | Place & Route | 映射到 FPGA 资源 |
| 约束 | XDC / SDC | 时钟、引脚、时序 |
|  bitstream | .bit / .bin | 下载到 FPGA |

#### 3. 开发工具

| 厂商 | 代表芯片 | 主要工具 |
| --- | --- | --- |
| AMD（原 Xilinx） | Artix、Kintex、Zynq | Vivado |
| Intel（原 Altera） | Cyclone、Arria | Quartus |
| 国产 | 紫光同创、复旦微、安路 | 对应厂商 IDE |

### 四、硬件描述语言基础

#### 1. Verilog 与 VHDL

FPGA 开发最常用 **Verilog** 和 **VHDL**。Verilog 语法简洁，国内工程使用广泛；VHDL 语法严谨，欧洲及军工领域较多。

#### 2. Verilog 示例：D 触发器

```verilog
module d_ff (
    input  wire clk,
    input  wire rst_n,
    input  wire d,
    output reg  q
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            q <= 1'b0;
        else
            q <= d;
    end
endmodule
```

#### 3. Verilog 示例：4 位计数器

```verilog
module counter_4bit (
    input  wire       clk,
    input  wire       rst_n,
    input  wire       en,
    output reg [3:0]  cnt
);
    always @(posedge clk or negedge rst_n) begin
        if (!rst_n)
            cnt <= 4'd0;
        else if (en)
            cnt <= cnt + 4'd1;
    end
endmodule
```

#### 4. 组合逻辑与 always 块

- **组合逻辑**：使用 `always @(*)` 或 `assign`，不能产生 latch（需覆盖所有分支）
- **时序逻辑**：使用 `always @(posedge clk)`，在时钟边沿更新

### 五、时序与时钟

#### 1. 为什么时序很重要

FPGA 是同步数字电路，时钟决定数据何时采样。若 setup/hold 不满足，会出现**亚稳态**，导致系统不稳定。

#### 2. 关键概念

| 概念 | 说明 |
| --- | --- |
| 时钟频率 | 如 50MHz、100MHz、200MHz |
| 建立时间（Setup Time） | 时钟沿到来前，数据需稳定 |
| 保持时间（Hold Time） | 时钟沿到来后，数据需保持 |
| 时钟域 | 不同时钟驱动的逻辑区域 |
| CDC | Clock Domain Crossing，跨时钟域处理 |

#### 3. 跨时钟域处理

两个不同频率时钟之间传递信号时，不能简单直连，常见方法：

- 单 bit 信号：**双触发器同步器**
- 多 bit 数据：**异步 FIFO**
- 控制信号：**握手协议**

### 六、常见 IP 与模块

#### 1. 基础 IP

- **PLL / MMCM**：时钟产生与倍频分频
- **FIFO**：跨时钟、缓冲数据
- **BRAM**：数据缓存、查找表
- **UART / SPI / I2C**：常见通信接口
- **AXI 总线**：ARM + FPGA 互联常用

#### 2. 状态机（FSM）

大量 FPGA 逻辑通过有限状态机实现，例如 UART 发送、协议解析、按键消抖。

典型三段式写法：

1. 时序逻辑：状态寄存器更新
2. 组合逻辑：下一状态判断
3. 时序逻辑：输出寄存

#### 3. Testbench 仿真

功能验证阶段需编写 Testbench 激励模块：

```verilog
module tb_counter;
    reg clk = 0;
    reg rst_n = 0;
    reg en = 0;
    wire [3:0] cnt;

    always #5 clk = ~clk; // 100MHz

    counter_4bit uut (
        .clk(clk),
        .rst_n(rst_n),
        .en(en),
        .cnt(cnt)
    );

    initial begin
        #20 rst_n = 1;
        #10 en = 1;
        #200 $finish;
    end
endmodule
```

### 七、FPGA 与嵌入式/Linux 协作

#### 1. PS + PL 架构

SoC FPGA（如 Zynq-7000）包含：

- **PS**（Processing System）：双核 ARM，可跑 Linux
- **PL**（Programmable Logic）：FPGA 逻辑

分工常见模式：

- ARM 负责网络、文件系统、业务逻辑
- FPGA 负责高速采集、协议 offload、实时控制

#### 2. 与 Csharp / 上位机的关系

PC 端 Csharp 工具可通过以太网、串口与 FPGA 或 SoC 通信，完成配置下发、数据采集、固件升级。FPGA 负责底层时序与 IO，上位机负责交互与存储。

### 八、学习路径建议

#### 1. 入门阶段

1. 了解数字电路基础：与门、或门、触发器、计数器
2. 学习 Verilog 基本语法
3. 在开发板上实现 LED 流水灯、按键控制、UART 回环
4. 学习 ModelSim / Vivado 仿真

#### 2. 进阶阶段

1. 状态机、FIFO、跨时钟域
2. AXI 总线与 IP 集成
3. 时序约束与 Timing Report 阅读
4. 简单 DSP：FIR 滤波、FFT（配合 IP）

#### 3. 实战方向

| 方向 | 内容 |
| --- | --- |
| 通信 | 以太网、SRIO、光纤 |
| 采集 | ADC 驱动、DMA |
| 显示 | HDMI、LVDS、VGA |
| SoC | Zynq + Linux + 驱动开发 |

### 九、常见问题

| 问题 | 原因 | 建议 |
| --- | --- | --- |
| 综合通过但无输出 | 引脚约束错误 | 检查 XDC 引脚与电平 |
| 时序不满足 | 路径过长、频率过高 | 降频、加流水线、优化逻辑 |
| 仿真正常、上板异常 | 未考虑异步信号 | 加同步、约束、复位设计 |
| 资源不够用 | LUT/BRAM 耗尽 | 换更大型号或优化算法 |

### 十、小结

FPGA 是一种**可编程硬件平台**，核心价值在于并行、低延迟和灵活 IO。学习时应同时掌握：

1. 数字电路与 Verilog 基础
2. 时序与跨时钟域
3. 工具链：仿真、综合、约束、下载
4. 与 CPU / 上位机协同的系统思维

从 LED、UART 等简单实验入手，再逐步接触 FIFO、AXI、SoC，是较为稳妥的学习路线。结合本博客中的《网络通讯协议与抓包工具学习》《网线烧录芯片与Csharp传输实现》，可以把 FPGA 作为底层 IO 与通讯加速，PC 端作为控制与展示，形成完整嵌入式系统能力。
