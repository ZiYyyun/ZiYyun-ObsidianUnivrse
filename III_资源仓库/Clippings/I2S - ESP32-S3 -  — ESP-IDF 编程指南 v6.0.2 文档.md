---
title: I2S - ESP32-S3 -  — ESP-IDF 编程指南 v6.0.2 文档
source: https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/peripherals/i2s.html#i2s
author:
published:
created: 2026-07-06
description: esp32 HTTP客户端
tags:
  - clippings
  - "#esp32"
  - "#I2S"
  - "#小智"
---
## I2S

[\[English\]](https://docs.espressif.com/projects/esp-idf/en/v6.0.2/esp32s3/api-reference/peripherals/i2s.html)

## 简介

I2S（Inter-IC Sound，集成电路内置音频总线）是一种同步串行通信协议，通常用于在两个数字音频设备之间传输音频数据。

ESP32-S3 包含 2 个 I2S 外设。通过配置这些外设，可以借助 I2S 驱动来输入和输出采样数据。

标准和 TDM 模式下的 I2S 总线包含以下几条线路：

- **MCLK** ：主时钟线。该信号线可选，具体取决于从机，主要用于向 I2S 从机提供参考时钟。
- **BCLK** ：位时钟线。用于数据线的位时钟。
- **WS** ：字（声道）选择线。通常用于识别声道（除 PDM 模式外）。
- **DIN/DOUT** ：串行数据输入/输出线。如果 DIN 和 DOUT 被配置到相同的 GPIO，数据将在内部回环。

PDM 通信模式下的 I2S 总线包含以下几条线路：

- **CLK** ：PDM 时钟线。
- **DIN/DOUT** ：串行数据输入/输出线。

每个 I2S 控制器都具备以下功能，可由 I2S 驱动进行配置：

- 可用作系统主机或从机
- 可用作发射器或接收器
- DMA 控制器支持流数据采样，CPU 无需单独复制每个采样数据

每个控制器都有独立的 RX 和 TX 通道，连接到不同 GPIO 管脚，能够在不同的时钟和声道配置下工作。注意，尽管在一个控制器上 TX 通道和 RX 通道的内部 MCLK 相互独立，但输出的 MCLK 信号只能连接到一个通道。如果需要两个互相独立的 MCLK 输出，必须将其分配到不同的 I2S 控制器上。

## I2S 时钟

### 时钟源

- `i2s_clock_src_t::I2S_CLK_SRC_DEFAULT` ：默认 PLL 时钟。

### 时钟术语

- **采样率** ：单声道每秒采样数据数量。
- **SCLK** ：源时钟频率，即时钟源的频率。
- **MCLK** ：主时钟频率，BCLK 由其产生。MCLK 信号通常作为参考时钟，用于同步 I2S 主机和从机之间的 BCLK 和 WS。
- **BCLK** ：位时钟频率，一个 BCLK 时钟周期代表数据管脚上的一个数据位。通过 配置的通道位宽即为一个声道中的 BCLK 时钟周期数量，因此一个声道中可以有 8/16/24/32 个 BCLK 时钟周期。
- **LRCK** / **WS** ：左/右时钟或字选择时钟。在非 PDM 模式下，其频率等于采样率。

> [!note] 备注
> 通常，MCLK 应该同时是 `采样率` 和 BCLK 的倍数。字段 表示 MCLK 相对于 `采样率` 的倍数。在大多数情况下，将其设置为 `I2S_MCLK_MULTIPLE_256` 即可。但如果 `slot_bit_width` 被设置为 `I2S_SLOT_BIT_WIDTH_24BIT` ，为了保证 MCLK 是 BCLK 的整数倍，应该将 设置为能被 3 整除的倍数，如 `I2S_MCLK_MULTIPLE_384` ，否则 WS 会不精准。

## I2S 通信模式

### 模式概览

| 芯片 | I2S 标准 | PCM-to-PDM | PDM-to-PCM | PDM | TDM | ADC/DAC | LCD/摄像头 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ESP32 | I2S 0/1 | I2S 0 | I2S 0 | I2S 0/1 | 无 | I2S 0 | I2S 0 |
| ESP32-S2 | I2S 0 | 无 | 无 | 无 | 无 | 无 | I2S 0 |
| ESP32-C3 | I2S 0 | I2S 0 | 无 | I2S 0 | I2S 0 | 无 | 无 |
| ESP32-C6 | I2S 0 | I2S 0 | 无 | I2S 0 | I2S 0 | 无 | 无 |
| ESP32-S3 | I2S 0/1 | I2S 0 | I2S 0 | I2S 0/1 | I2S 0/1 | 无 | 无 |
| ESP32-H2 | I2S 0 | I2S 0 | 无 | I2S 0 | I2S 0 | 无 | 无 |
| ESP32-P4 | I2S 0~2 | I2S 0 | I2S 0 | I2S 0~2 | I2S 0~2 | 无 | 无 |
| ESP32-C5 | I2S 0 | I2S 0 | 无 | I2S 0 | I2S 0 | 无 | 无 |
| ESP32-C61 | I2S 0 | I2S 0 | 无 | I2S 0 | I2S 0 | 无 | 无 |

> [!note] 备注
> 如需使用 PDM 模式，请注意不是所有 I2S 端口都支持原始 PDM 格式与 PCM 格式之间的转换，因为有些端口在 TX 方向上没有 PCM-to-PDM 数据格式转换器，或在 RX 方向上没有 PDM-to-PCM 数据格式转换器。因此，这些没有硬件格式转换器的端口只能读写原始 PDM 格式的数据。如果需要在这些端口上处理 PCM 格式的数据，则需额外采用一个软件滤波器来实现 PDM 格式和 PCM 格式之间的转换。

### 标准模式

标准模式中有且仅有左右两个声道，驱动中将声道称为 slot。这些声道可以支持 8/16/24/32 位宽的采样数据，声道的通信格式主要包括以下几种：

- **Philips 格式** ：数据信号与 WS 信号相比有一个位的位移。WS 信号的占空比为 50%。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_0" height="226" width="720" viewBox="0 0 720 226" overflow="hidden"><defs><g id="socket"><rect y="15" x="6" height="20" width="20"></rect></g><g id="pclk"><path d="M0,20 0,0 20,0"></path></g><g id="nclk"><path d="m0,0 0,20 20,0"></path></g><g id="000"><path d="m0,20 20,0"></path></g><g id="0m0"><path d="m0,20 3,0 3,-10 3,10 11,0"></path></g><g id="0m1"><path d="M0,20 3,20 9,0 20,0"></path></g><g id="0mx"><path d="M3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 5,20"></path><path d="M20,0 4,16"></path><path d="M15,0 6,9"></path><path d="M10,0 9,1"></path><path d="m0,20 20,0"></path></g><g id="0md"><path d="m8,20 10,0"></path><path d="m0,20 5,0"></path></g><g id="0mu"><path d="m0,20 3,0 C 7,10 10.107603,0 20,0"></path></g><g id="0mz"><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="111"><path d="M0,0 20,0"></path></g><g id="1m0"><path d="m0,0 3,0 6,20 11,0"></path></g><g id="1m1"><path d="M0,0 3,0 6,10 9,0 20,0"></path></g><g id="1mx"><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 6,9"></path><path d="M10,0 5,5"></path><path d="M3.5,1.5 5,0"></path></g><g id="1md"><path d="m0,0 3,0 c 4,10 7,20 17,20"></path></g><g id="1mu"><path d="M0,0 5,0"></path><path d="M8,0 18,0"></path></g><g id="1mz"><path d="m0,0 3,0 c 7,10 12,10 17,10"></path></g><g id="xxx"><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path><path d="M0,5 5,0"></path><path d="M0,10 10,0"></path><path d="M0,15 15,0"></path><path d="M0,20 20,0"></path><path d="M5,20 20,5"></path><path d="M10,20 20,10"></path><path d="m15,20 5,-5"></path></g><g id="xm0"><path d="M0,0 4,0 9,20"></path><path d="m0,20 20,0"></path><path d="M0,5 4,1"></path><path d="M0,10 5,5"></path><path d="M0,15 6,9"></path><path d="M0,20 7,13"></path><path d="M5,20 8,17"></path></g><g id="xm1"><path d="M0,0 20,0"></path><path d="M0,20 4,20 9,0"></path><path d="M0,5 5,0"></path><path d="M0,10 9,1"></path><path d="M0,15 7,8"></path><path d="M0,20 5,15"></path></g><g id="xmx"><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path><path d="M0,5 5,0"></path><path d="M0,10 10,0"></path><path d="M0,15 15,0"></path><path d="M0,20 20,0"></path><path d="M5,20 20,5"></path><path d="M10,20 20,10"></path><path d="m15,20 5,-5"></path></g><g id="xmd"><path d="m0,0 4,0 c 3,10 6,20 16,20"></path><path d="m0,20 20,0"></path><path d="M0,5 4,1"></path><path d="M0,10 5.5,4.5"></path><path d="M0,15 6.5,8.5"></path><path d="M0,20 8,12"></path><path d="m5,20 5,-5"></path><path d="m10,20 2.5,-2.5"></path></g><g id="xmu"><path d="M0,0 20,0"></path><path d="m0,20 4,0 C 7,10 10,0 20,0"></path><path d="M0,5 5,0"></path><path d="M0,10 10,0"></path><path d="M0,15 10,5"></path><path d="M0,20 6,14"></path></g><g id="xmz"><path d="m0,0 4,0 c 6,10 11,10 16,10"></path><path d="m0,20 4,0 C 10,10 15,10 20,10"></path><path d="M0,5 4.5,0.5"></path><path d="M0,10 6.5,3.5"></path><path d="M0,15 8.5,6.5"></path><path d="M0,20 11.5,8.5"></path></g><g id="ddd"><path d="m0,20 20,0"></path></g><g id="dm0"><path d="m0,20 10,0"></path><path d="m12,20 8,0"></path></g><g id="dm1"><path d="M0,20 3,20 9,0 20,0"></path></g><g id="dmx"><path d="M3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 5,20"></path><path d="M20,0 4,16"></path><path d="M15,0 6,9"></path><path d="M10,0 9,1"></path><path d="m0,20 20,0"></path></g><g id="dmd"><path d="m0,20 20,0"></path></g><g id="dmu"><path d="m0,20 3,0 C 7,10 10.107603,0 20,0"></path></g><g id="dmz"><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="uuu"><path d="M0,0 20,0"></path></g><g id="um0"><path d="m0,0 3,0 6,20 11,0"></path></g><g id="um1"><path d="M0,0 10,0"></path><path d="m12,0 8,0"></path></g><g id="umx"><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 6,9"></path><path d="M10,0 5,5"></path><path d="M3.5,1.5 5,0"></path></g><g id="umd"><path d="m0,0 3,0 c 4,10 7,20 17,20"></path></g><g id="umu"><path d="M0,0 20,0"></path></g><g id="umz"><path d="m0,0 3,0 c 7,10 12,10 17,10"></path></g><g id="zzz"><path d="m0,10 20,0"></path></g><g id="zm0"><path d="m0,10 6,0 3,10 11,0"></path></g><g id="zm1"><path d="M0,10 6,10 9,0 20,0"></path></g><g id="zmx"><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 6.5,8.5"></path><path d="M10,0 9,1"></path></g><g id="zmd"><path d="m0,10 7,0 c 3,5 8,10 13,10"></path></g><g id="zmu"><path d="m0,10 7,0 C 10,5 15,0 20,0"></path></g><g id="zmz"><path d="m0,10 20,0"></path></g><g id="gap"><path d="m7,-2 -4,0 c -5,0 -5,24 -10,24 l 4,0 C 2,22 2,-2 7,-2 z"></path><path d="M-7,22 C -2,22 -2,-2 3,-2"></path><path d="M-3,22 C 2,22 2,-2 7,-2"></path></g><g id="Pclk"><path d="M-3,12 0,3 3,12 C 1,11 -1,11 -3,12 z"></path><path d="M0,20 0,0 20,0"></path></g><g id="Nclk"><path d="M-3,8 0,17 3,8 C 1,9 -1,9 -3,8 z"></path><path d="m0,0 0,20 20,0"></path></g><g id="0mv-2"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-2"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-2"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-2"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-2"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-2"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-2"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-2"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-2"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-2"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-2"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-3"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-3"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-3"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-3"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-3"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-3"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-3"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-3"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-3"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-3"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-3"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-4"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-4"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-4"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-4"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-4"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-4"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-4"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-4"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-4"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-4"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-4"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-5"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-5"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-5"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-5"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-5"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-5"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-5"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-5"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-5"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-5"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-5"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-6"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-6"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-6"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-6"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-6"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-6"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-6"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-6"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-6"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-6"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-6"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-7"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-7"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-7"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-7"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-7"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-7"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-7"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-7"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-7"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-7"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-7"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-8"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-8"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-8"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-8"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-8"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-8"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-8"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-8"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-8"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-8"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-8"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="0mv-9"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="1mv-9"><path d="M2.875,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="xmv-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,5 3.5,1.5"></path><path d="M0,10 4.5,5.5"></path><path d="M0,15 6,9"></path><path d="M0,20 4,16"></path></g><g id="dmv-9"><path d="M9,0 20,0 20,20 3,20 z"></path><path d="M3,20 9,0 20,0"></path><path d="m0,20 20,0"></path></g><g id="umv-9"><path d="M3,0 20,0 20,20 9,20 z"></path><path d="m3,0 6,20 11,0"></path><path d="M0,0 20,0"></path></g><g id="zmv-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="m6,10 3,10 11,0"></path><path d="M0,10 6,10 9,0 20,0"></path></g><g id="vvv-9"><path d="M20,20 0,20 0,0 20,0"></path><path d="m0,20 20,0"></path><path d="M0,0 20,0"></path></g><g id="vm0-9"><path d="M0,20 0,0 3,0 9,20"></path><path d="M0,0 3,0 9,20"></path><path d="m0,20 20,0"></path></g><g id="vm1-9"><path d="M0,0 0,20 3,20 9,0"></path><path d="M0,0 20,0"></path><path d="M0,20 3,20 9,0"></path></g><g id="vmx-9"><path d="M0,0 0,20 3,20 6,10 3,0"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path><path d="m20,15 -5,5"></path><path d="M20,10 10,20"></path><path d="M20,5 8,17"></path><path d="M20,0 7,13"></path><path d="M15,0 7,8"></path><path d="M10,0 9,1"></path></g><g id="vmd-9"><path d="m0,0 0,20 20,0 C 10,20 7,10 3,0"></path><path d="m0,0 3,0 c 4,10 7,20 17,20"></path><path d="m0,20 20,0"></path></g><g id="vmu-9"><path d="m0,0 0,20 3,0 C 7,10 10,0 20,0"></path><path d="m0,20 3,0 C 7,10 10,0 20,0"></path><path d="M0,0 20,0"></path></g><g id="vmz-9"><path d="M0,0 3,0 C 10,10 15,10 20,10 15,10 10,10 3,20 L 0,20"></path><path d="m0,0 3,0 c 7,10 12,10 17,10"></path><path d="m0,20 3,0 C 10,10 15,10 20,10"></path></g><g id="vmv-2-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-2"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-3"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-4"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-5"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-6"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-7"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-8"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-2-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-3-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-4-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-5-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-6-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-7-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-8-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="vmv-9-9"><path d="M9,0 20,0 20,20 9,20 6,10 z"></path><path d="M3,0 0,0 0,20 3,20 6,10 z"></path><path d="m0,0 3,0 6,20 11,0"></path><path d="M0,20 3,20 9,0 20,0"></path></g><g id="arrow0"><path d="m-12,-3 9,3 -9,3 c 1,-2 1,-4 0,-6 z"></path><path d="M0,0 -15,0"></path></g><marker id="arrowhead" style="fill:#0041c4" markerHeight="7" markerWidth="10" markerUnits="strokeWidth" viewBox="0 -4 11 8" refX="15" refY="0" orient="auto"><path d="M0 -4 11 0 0 4z"></path></marker><marker id="arrowtail" style="fill:#0041c4" markerHeight="7" markerWidth="10" markerUnits="strokeWidth" viewBox="-11 -4 11 8" refX="-15" refY="0" orient="auto"><path d="M0 -4 -11 0 0 4z"></path></marker><marker id="tee" style="fill:#0041c4" markerHeight="6" markerWidth="1" markerUnits="strokeWidth" viewBox="0 0 1 6" refX="0" refY="3" orient="auto"><path d="M 0 0 L 0 6" style="stroke:#0041c4;stroke-width:2"></path></marker></defs><g id="waves_0"><rect width="720" height="226" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_0"><g id="gmarks_0"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_0" x1="0" y1="0" x2="0" y2="180"></line><line id="gmark_1_0" x1="40" y1="0" x2="40" y2="180"></line><line id="gmark_2_0" x1="80" y1="0" x2="80" y2="180"></line><line id="gmark_3_0" x1="120" y1="0" x2="120" y2="180"></line><line id="gmark_4_0" x1="160" y1="0" x2="160" y2="180"></line><line id="gmark_5_0" x1="200" y1="0" x2="200" y2="180"></line><line id="gmark_6_0" x1="240" y1="0" x2="240" y2="180"></line><line id="gmark_7_0" x1="280" y1="0" x2="280" y2="180"></line><line id="gmark_8_0" x1="320" y1="0" x2="320" y2="180"></line><line id="gmark_9_0" x1="360" y1="0" x2="360" y2="180"></line><line id="gmark_10_0" x1="400" y1="0" x2="400" y2="180"></line><line id="gmark_11_0" x1="440" y1="0" x2="440" y2="180"></line><line id="gmark_12_0" x1="480" y1="0" x2="480" y2="180"></line><line id="gmark_13_0" x1="520" y1="0" x2="520" y2="180"></line><line id="gmark_14_0" x1="560" y1="0" x2="560" y2="180"></line><line id="gmark_15_0" x1="600" y1="0" x2="600" y2="180"></line></g><text x="300" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>Standard Philips Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_0"><g transform="translate(14)" id="wavelane_draw_0_0"></g></g><g transform="translate(0,35)" id="wavelane_1_0"><g transform="translate(14)" id="wavelane_draw_1_0"></g></g><g transform="translate(0,65)" id="wavelane_2_0"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_2_0"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#pclk"></use><use transform="translate(180)" xlink:href="#nclk"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#pclk"></use><use transform="translate(300)" xlink:href="#nclk"></use><use transform="translate(320)" xlink:href="#pclk"></use><use transform="translate(340)" xlink:href="#nclk"></use><use transform="translate(360)" xlink:href="#0md"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#pclk"></use><use transform="translate(420)" xlink:href="#nclk"></use><use transform="translate(440)" xlink:href="#pclk"></use><use transform="translate(460)" xlink:href="#nclk"></use><use transform="translate(480)" xlink:href="#0md"></use><use transform="translate(500)" xlink:href="#ddd"></use><use transform="translate(520)" xlink:href="#pclk"></use><use transform="translate(540)" xlink:href="#nclk"></use><use transform="translate(560)" xlink:href="#pclk"></use><use transform="translate(580)" xlink:href="#nclk"></use></g></g><g transform="translate(0,95)" id="wavelane_3_0"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_0"><use xlink:href="#xmx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xm0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#000"></use><use transform="translate(100)" xlink:href="#000"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#dm0"></use><use transform="translate(180)" xlink:href="#000"></use><use transform="translate(200)" xlink:href="#000"></use><use transform="translate(220)" xlink:href="#000"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#dm1"></use><use transform="translate(300)" xlink:href="#111"></use><use transform="translate(320)" xlink:href="#111"></use><use transform="translate(340)" xlink:href="#111"></use><use transform="translate(360)" xlink:href="#1mu"></use><use transform="translate(380)" xlink:href="#uuu"></use><use transform="translate(400)" xlink:href="#um1"></use><use transform="translate(420)" xlink:href="#111"></use><use transform="translate(440)" xlink:href="#1mu"></use><use transform="translate(460)" xlink:href="#uuu"></use><use transform="translate(480)" xlink:href="#um1"></use><use transform="translate(500)" xlink:href="#111"></use><use transform="translate(520)" xlink:href="#1m0"></use><use transform="translate(540)" xlink:href="#000"></use><use transform="translate(560)" xlink:href="#000"></use><use transform="translate(580)" xlink:href="#000"></use></g></g><g transform="translate(0,125)" id="wavelane_4_0"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_4_0"><use xlink:href="#xmx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xxx"></use><use transform="translate(60)" xlink:href="#xxx"></use><use transform="translate(80)" xlink:href="#xmv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmx-2"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xxx"></use><use transform="translate(180)" xlink:href="#xxx"></use><use transform="translate(200)" xlink:href="#xmv-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmx-2"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vmx-2"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xxx"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xmv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmx-2"></use><use transform="translate(500)" xlink:href="#xxx"></use><use transform="translate(520)" xlink:href="#xxx"></use><use transform="translate(540)" xlink:href="#xxx"></use><use transform="translate(560)" xlink:href="#xmv-2"></use><use transform="translate(580)" xlink:href="#vvv-2"></use><text x="106" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="346" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="466" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="586" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,155)" id="wavelane_5_0"><g transform="translate(14)" id="wavelane_draw_5_0"></g></g><g id="wavearcs_0"><path id="gmark_C_D" d="M 100,15 340,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(220,15)"><rect x="-45" y="-5" width="90.77" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>slot_bit_width</tspan></text></g> <path id="gmark_A_B" d="M 100,45 260,45" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,45)"><rect x="-48" y="-5" width="96.38" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>data_bit_width</tspan></text></g> <path id="gmark_M_E" d="M 60,165 100,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,165)"><rect x="-22" y="-5" width="44.46" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>bit shift</tspan></text></g> <path id="gmark_E_F" d="M 100,165 340,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(220,165)"><rect x="-25" y="-5" width="51.72" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Left Slot</tspan></text></g> <path id="gmark_F_G" d="M 340,165 580,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(460,165)"><rect x="-30" y="-5" width="61.51" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Right Slot</tspan></text></g><path id="gmark_P_M" d="M 60,105 60,165" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_E_C" d="M 100,165 100,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 260,135 260,45" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_F_D" d="M 340,165 340,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_L_G" d="M 580,135 580,165" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_0"><g transform="translate(0,65)" id="wavegap_2_0"></g><g transform="translate(0,95)" id="wavegap_3_0"></g><g transform="translate(0,125)" id="wavegap_4_0"><use transform="translate(194)" xlink:href="#gap"></use><use transform="translate(314)" xlink:href="#gap"></use><use transform="translate(434)" xlink:href="#gap"></use><use transform="translate(554)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_0"><g></g></g></g></svg>
- **MSB 格式** ：与 Philips 格式基本相同，但其数据没有位移。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_1" height="226" width="680" viewBox="0 0 680 226" overflow="hidden"><g id="waves_1"><rect width="680" height="226" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_1"><g id="gmarks_1"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_1" x1="0" y1="0" x2="0" y2="180"></line><line id="gmark_1_1" x1="40" y1="0" x2="40" y2="180"></line><line id="gmark_2_1" x1="80" y1="0" x2="80" y2="180"></line><line id="gmark_3_1" x1="120" y1="0" x2="120" y2="180"></line><line id="gmark_4_1" x1="160" y1="0" x2="160" y2="180"></line><line id="gmark_5_1" x1="200" y1="0" x2="200" y2="180"></line><line id="gmark_6_1" x1="240" y1="0" x2="240" y2="180"></line><line id="gmark_7_1" x1="280" y1="0" x2="280" y2="180"></line><line id="gmark_8_1" x1="320" y1="0" x2="320" y2="180"></line><line id="gmark_9_1" x1="360" y1="0" x2="360" y2="180"></line><line id="gmark_10_1" x1="400" y1="0" x2="400" y2="180"></line><line id="gmark_11_1" x1="440" y1="0" x2="440" y2="180"></line><line id="gmark_12_1" x1="480" y1="0" x2="480" y2="180"></line><line id="gmark_13_1" x1="520" y1="0" x2="520" y2="180"></line><line id="gmark_14_1" x1="560" y1="0" x2="560" y2="180"></line></g><text x="280" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>Standard MSB Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_1"><g transform="translate(14)" id="wavelane_draw_0_1"></g></g><g transform="translate(0,35)" id="wavelane_1_1"><g transform="translate(14)" id="wavelane_draw_1_1"></g></g><g transform="translate(0,65)" id="wavelane_2_1"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_2_1"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#pclk"></use><use transform="translate(140)" xlink:href="#nclk"></use><use transform="translate(160)" xlink:href="#0md"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#ddd"></use><use transform="translate(220)" xlink:href="#ddd"></use><use transform="translate(240)" xlink:href="#pclk"></use><use transform="translate(260)" xlink:href="#nclk"></use><use transform="translate(280)" xlink:href="#0md"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#pclk"></use><use transform="translate(380)" xlink:href="#nclk"></use><use transform="translate(400)" xlink:href="#0md"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#ddd"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#pclk"></use><use transform="translate(500)" xlink:href="#nclk"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use></g></g><g transform="translate(0,95)" id="wavelane_3_1"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_1"><use xlink:href="#xmx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xm0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#000"></use><use transform="translate(100)" xlink:href="#000"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#dm0"></use><use transform="translate(180)" xlink:href="#000"></use><use transform="translate(200)" xlink:href="#000"></use><use transform="translate(220)" xlink:href="#000"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#dm1"></use><use transform="translate(300)" xlink:href="#111"></use><use transform="translate(320)" xlink:href="#111"></use><use transform="translate(340)" xlink:href="#111"></use><use transform="translate(360)" xlink:href="#1mu"></use><use transform="translate(380)" xlink:href="#uuu"></use><use transform="translate(400)" xlink:href="#um1"></use><use transform="translate(420)" xlink:href="#111"></use><use transform="translate(440)" xlink:href="#1mu"></use><use transform="translate(460)" xlink:href="#uuu"></use><use transform="translate(480)" xlink:href="#um1"></use><use transform="translate(500)" xlink:href="#111"></use><use transform="translate(520)" xlink:href="#1m0"></use><use transform="translate(540)" xlink:href="#000"></use></g></g><g transform="translate(0,125)" id="wavelane_4_1"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_4_1"><use xlink:href="#xmx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmv-2"></use><use transform="translate(60)" xlink:href="#vvv-2"></use><use transform="translate(80)" xlink:href="#vmx-2"></use><use transform="translate(100)" xlink:href="#xxx"></use><use transform="translate(120)" xlink:href="#xxx"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xmv-2"></use><use transform="translate(180)" xlink:href="#vvv-2"></use><use transform="translate(200)" xlink:href="#vmx-2"></use><use transform="translate(220)" xlink:href="#xxx"></use><use transform="translate(240)" xlink:href="#xxx"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xmv-2"></use><use transform="translate(300)" xlink:href="#vvv-2"></use><use transform="translate(320)" xlink:href="#vmx-2"></use><use transform="translate(340)" xlink:href="#xxx"></use><use transform="translate(360)" xlink:href="#xxx"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xmv-2"></use><use transform="translate(420)" xlink:href="#vvv-2"></use><use transform="translate(440)" xlink:href="#vmx-2"></use><use transform="translate(460)" xlink:href="#xxx"></use><use transform="translate(480)" xlink:href="#xxx"></use><use transform="translate(500)" xlink:href="#xxx"></use><use transform="translate(520)" xlink:href="#xmv-2"></use><use transform="translate(540)" xlink:href="#vvv-2"></use><text x="66" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="186" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="306" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="426" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="546" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,155)" id="wavelane_5_1"><g transform="translate(14)" id="wavelane_draw_5_1"></g></g><g id="wavearcs_1"><path id="gmark_C_D" d="M 60,15 300,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,15)"><rect x="-45" y="-5" width="90.77" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>slot_bit_width</tspan></text></g> <path id="gmark_A_B" d="M 60,45 220,45" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(140,45)"><rect x="-48" y="-5" width="96.38" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>data_bit_width</tspan></text></g> <path id="gmark_E_F" d="M 60,165 300,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,165)"><rect x="-25" y="-5" width="51.72" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Left Slot</tspan></text></g> <path id="gmark_F_G" d="M 300,165 540,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(420,165)"><rect x="-30" y="-5" width="61.51" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Right Slot</tspan></text></g><path id="gmark_E_C" d="M 60,165 60,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 220,135 220,45" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_F_D" d="M 300,165 300,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_J_G" d="M 540,105 540,165" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_1"><g transform="translate(0,65)" id="wavegap_2_1"></g><g transform="translate(0,95)" id="wavegap_3_1"></g><g transform="translate(0,125)" id="wavegap_4_1"><use transform="translate(154)" xlink:href="#gap"></use><use transform="translate(274)" xlink:href="#gap"></use><use transform="translate(394)" xlink:href="#gap"></use><use transform="translate(514)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_1"><g></g></g></g></svg>
- **PCM 帧同步** ：数据有一个位的位移，同时 WS 信号变成脉冲，持续一个 BCLK 周期。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_2" height="226" width="720" viewBox="0 0 720 226" overflow="hidden"><g id="waves_2"><rect width="720" height="226" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_2"><g id="gmarks_2"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_2" x1="0" y1="0" x2="0" y2="180"></line><line id="gmark_1_2" x1="40" y1="0" x2="40" y2="180"></line><line id="gmark_2_2" x1="80" y1="0" x2="80" y2="180"></line><line id="gmark_3_2" x1="120" y1="0" x2="120" y2="180"></line><line id="gmark_4_2" x1="160" y1="0" x2="160" y2="180"></line><line id="gmark_5_2" x1="200" y1="0" x2="200" y2="180"></line><line id="gmark_6_2" x1="240" y1="0" x2="240" y2="180"></line><line id="gmark_7_2" x1="280" y1="0" x2="280" y2="180"></line><line id="gmark_8_2" x1="320" y1="0" x2="320" y2="180"></line><line id="gmark_9_2" x1="360" y1="0" x2="360" y2="180"></line><line id="gmark_10_2" x1="400" y1="0" x2="400" y2="180"></line><line id="gmark_11_2" x1="440" y1="0" x2="440" y2="180"></line><line id="gmark_12_2" x1="480" y1="0" x2="480" y2="180"></line><line id="gmark_13_2" x1="520" y1="0" x2="520" y2="180"></line><line id="gmark_14_2" x1="560" y1="0" x2="560" y2="180"></line><line id="gmark_15_2" x1="600" y1="0" x2="600" y2="180"></line></g><text x="300" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>Standard PCM Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_2"><g transform="translate(14)" id="wavelane_draw_0_2"></g></g><g transform="translate(0,35)" id="wavelane_1_2"><g transform="translate(14)" id="wavelane_draw_1_2"></g></g><g transform="translate(0,65)" id="wavelane_2_2"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_2_2"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#pclk"></use><use transform="translate(180)" xlink:href="#nclk"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#pclk"></use><use transform="translate(300)" xlink:href="#nclk"></use><use transform="translate(320)" xlink:href="#pclk"></use><use transform="translate(340)" xlink:href="#nclk"></use><use transform="translate(360)" xlink:href="#0md"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#pclk"></use><use transform="translate(420)" xlink:href="#nclk"></use><use transform="translate(440)" xlink:href="#pclk"></use><use transform="translate(460)" xlink:href="#nclk"></use><use transform="translate(480)" xlink:href="#0md"></use><use transform="translate(500)" xlink:href="#ddd"></use><use transform="translate(520)" xlink:href="#pclk"></use><use transform="translate(540)" xlink:href="#nclk"></use><use transform="translate(560)" xlink:href="#pclk"></use><use transform="translate(580)" xlink:href="#nclk"></use></g></g><g transform="translate(0,95)" id="wavelane_3_2"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_2"><use xlink:href="#xm1"></use><use transform="translate(20)" xlink:href="#111"></use><use transform="translate(40)" xlink:href="#1m0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#0md"></use><use transform="translate(100)" xlink:href="#ddd"></use><use transform="translate(120)" xlink:href="#dm0"></use><use transform="translate(140)" xlink:href="#000"></use><use transform="translate(160)" xlink:href="#000"></use><use transform="translate(180)" xlink:href="#000"></use><use transform="translate(200)" xlink:href="#0md"></use><use transform="translate(220)" xlink:href="#ddd"></use><use transform="translate(240)" xlink:href="#dm1"></use><use transform="translate(260)" xlink:href="#111"></use><use transform="translate(280)" xlink:href="#1m0"></use><use transform="translate(300)" xlink:href="#000"></use><use transform="translate(320)" xlink:href="#0md"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#dm0"></use><use transform="translate(380)" xlink:href="#000"></use><use transform="translate(400)" xlink:href="#000"></use><use transform="translate(420)" xlink:href="#000"></use><use transform="translate(440)" xlink:href="#0md"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#dm1"></use><use transform="translate(500)" xlink:href="#111"></use><use transform="translate(520)" xlink:href="#1m0"></use><use transform="translate(540)" xlink:href="#000"></use></g></g><g transform="translate(0,125)" id="wavelane_4_2"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_4_2"><use xlink:href="#xmx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xxx"></use><use transform="translate(60)" xlink:href="#xxx"></use><use transform="translate(80)" xlink:href="#xmv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmx-2"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xxx"></use><use transform="translate(180)" xlink:href="#xxx"></use><use transform="translate(200)" xlink:href="#xmv-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmx-2"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vmx-2"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xxx"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xmv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmx-2"></use><use transform="translate(500)" xlink:href="#xxx"></use><use transform="translate(520)" xlink:href="#xxx"></use><use transform="translate(540)" xlink:href="#xxx"></use><use transform="translate(560)" xlink:href="#xmv-2"></use><use transform="translate(580)" xlink:href="#vvv-2"></use><text x="106" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="346" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="466" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="586" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,155)" id="wavelane_5_2"><g transform="translate(14)" id="wavelane_draw_5_2"></g></g><g id="wavearcs_2"><path id="gmark_C_D" d="M 100,15 340,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(220,15)"><rect x="-45" y="-5" width="90.77" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>slot_bit_width</tspan></text></g> <path id="gmark_A_B" d="M 100,45 260,45" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,45)"><rect x="-48" y="-5" width="96.38" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>data_bit_width</tspan></text></g> <path id="gmark_M_E" d="M 60,165 100,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,165)"><rect x="-22" y="-5" width="44.46" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>bit shift</tspan></text></g> <path id="gmark_E_F" d="M 100,165 340,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(220,165)"><rect x="-25" y="-5" width="51.72" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Left Slot</tspan></text></g> <path id="gmark_F_G" d="M 340,165 580,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(460,165)"><rect x="-30" y="-5" width="61.51" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Right Slot</tspan></text></g><path id="gmark_P_M" d="M 60,105 60,165" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_E_C" d="M 100,165 100,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 260,135 260,45" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_F_D" d="M 340,165 340,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_G_L" d="M 580,165 580,135" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_2"><g transform="translate(0,65)" id="wavegap_2_2"></g><g transform="translate(0,95)" id="wavegap_3_2"></g><g transform="translate(0,125)" id="wavegap_4_2"><use transform="translate(194)" xlink:href="#gap"></use><use transform="translate(314)" xlink:href="#gap"></use><use transform="translate(434)" xlink:href="#gap"></use><use transform="translate(554)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_2"><g></g></g></g></svg>

### PDM 模式

PDM（Pulse-density Modulation，脉冲密度调制）通过采样的方式将模拟信号数字化为 1 位精度的数字信号。它以脉冲密度的方式呈现模拟信号的大小，即密度越高，对应的模拟信号值越大。PDM 时序图如下所示：

<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_3" height="166" width="800" viewBox="0 0 800 166" overflow="hidden"><g id="waves_3"><rect width="800" height="166" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_3"><g id="gmarks_3"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_3" x1="0" y1="0" x2="0" y2="120"></line><line id="gmark_1_3" x1="40" y1="0" x2="40" y2="120"></line><line id="gmark_2_3" x1="80" y1="0" x2="80" y2="120"></line><line id="gmark_3_3" x1="120" y1="0" x2="120" y2="120"></line><line id="gmark_4_3" x1="160" y1="0" x2="160" y2="120"></line><line id="gmark_5_3" x1="200" y1="0" x2="200" y2="120"></line><line id="gmark_6_3" x1="240" y1="0" x2="240" y2="120"></line><line id="gmark_7_3" x1="280" y1="0" x2="280" y2="120"></line><line id="gmark_8_3" x1="320" y1="0" x2="320" y2="120"></line><line id="gmark_9_3" x1="360" y1="0" x2="360" y2="120"></line><line id="gmark_10_3" x1="400" y1="0" x2="400" y2="120"></line><line id="gmark_11_3" x1="440" y1="0" x2="440" y2="120"></line><line id="gmark_12_3" x1="480" y1="0" x2="480" y2="120"></line><line id="gmark_13_3" x1="520" y1="0" x2="520" y2="120"></line><line id="gmark_14_3" x1="560" y1="0" x2="560" y2="120"></line><line id="gmark_15_3" x1="600" y1="0" x2="600" y2="120"></line><line id="gmark_16_3" x1="640" y1="0" x2="640" y2="120"></line><line id="gmark_17_3" x1="680" y1="0" x2="680" y2="120"></line></g><text x="340" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>PDM Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_3"><g id="wavelane_draw_0_3"></g></g><g transform="translate(0,35)" id="wavelane_1_3"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>CLK</tspan></text> <g id="wavelane_draw_1_3"><use xlink:href="#111"></use><use transform="translate(20)" xlink:href="#111"></use><use transform="translate(40)" xlink:href="#1m0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#000"></use><use transform="translate(100)" xlink:href="#000"></use><use transform="translate(120)" xlink:href="#0m1"></use><use transform="translate(140)" xlink:href="#111"></use><use transform="translate(160)" xlink:href="#111"></use><use transform="translate(180)" xlink:href="#111"></use><use transform="translate(200)" xlink:href="#1mx"></use><use transform="translate(220)" xlink:href="#xxx"></use><use transform="translate(240)" xlink:href="#xxx"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xm0"></use><use transform="translate(340)" xlink:href="#000"></use><use transform="translate(360)" xlink:href="#000"></use><use transform="translate(380)" xlink:href="#000"></use><use transform="translate(400)" xlink:href="#0m1"></use><use transform="translate(420)" xlink:href="#111"></use><use transform="translate(440)" xlink:href="#111"></use><use transform="translate(460)" xlink:href="#111"></use><use transform="translate(480)" xlink:href="#1m0"></use><use transform="translate(500)" xlink:href="#000"></use><use transform="translate(520)" xlink:href="#000"></use><use transform="translate(540)" xlink:href="#000"></use><use transform="translate(560)" xlink:href="#0m1"></use><use transform="translate(580)" xlink:href="#111"></use><use transform="translate(600)" xlink:href="#111"></use><use transform="translate(620)" xlink:href="#111"></use><use transform="translate(640)" xlink:href="#1mx"></use><use transform="translate(660)" xlink:href="#xxx"></use></g></g><g transform="translate(0,65)" id="wavelane_2_3"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g id="wavelane_draw_2_3"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmv-2"></use><use transform="translate(60)" xlink:href="#vvv-2"></use><use transform="translate(80)" xlink:href="#vvv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmv-2-2"></use><use transform="translate(140)" xlink:href="#vvv-2"></use><use transform="translate(160)" xlink:href="#vvv-2"></use><use transform="translate(180)" xlink:href="#vvv-2"></use><use transform="translate(200)" xlink:href="#vmx-2"></use><use transform="translate(220)" xlink:href="#xxx"></use><use transform="translate(240)" xlink:href="#xxx"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vvv-2"></use><use transform="translate(380)" xlink:href="#vvv-2"></use><use transform="translate(400)" xlink:href="#vmv-2-2"></use><use transform="translate(420)" xlink:href="#vvv-2"></use><use transform="translate(440)" xlink:href="#vvv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmv-2-2"></use><use transform="translate(500)" xlink:href="#vvv-2"></use><use transform="translate(520)" xlink:href="#vvv-2"></use><use transform="translate(540)" xlink:href="#vvv-2"></use><use transform="translate(560)" xlink:href="#vmv-2-2"></use><use transform="translate(580)" xlink:href="#vvv-2"></use><use transform="translate(600)" xlink:href="#vvv-2"></use><use transform="translate(620)" xlink:href="#vvv-2"></use><use transform="translate(640)" xlink:href="#vmx-2"></use><use transform="translate(660)" xlink:href="#xxx"></use><text x="86" y="15" text-anchor="middle" xml:space="preserve"><tspan>LMSB</tspan></text> <text x="166" y="15" text-anchor="middle" xml:space="preserve"><tspan>RMSB</tspan></text> <text x="366" y="15" text-anchor="middle" xml:space="preserve"><tspan>LLSB</tspan></text> <text x="446" y="15" text-anchor="middle" xml:space="preserve"><tspan>RLSB</tspan></text> <text x="526" y="15" text-anchor="middle" xml:space="preserve"><tspan>LMSB</tspan></text> <text x="606" y="15" text-anchor="middle" xml:space="preserve"><tspan>RMSB</tspan></text></g></g> <g transform="translate(0,95)" id="wavelane_3_3"><g id="wavelane_draw_3_3"></g></g><g id="wavearcs_3"><path id="gmark_A_B" d="M 46,15 126,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(86,15)"><rect x="-10" y="-5" width="21.58" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>left</tspan></text></g> <path id="gmark_B_C" d="M 126,15 206,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(166,15)"><rect x="-15" y="-5" width="30.49" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>right</tspan></text></g> <path id="gmark_D_E" d="M 326,15 406,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(366,15)"><rect x="-10" y="-5" width="21.58" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>left</tspan></text></g> <path id="gmark_E_F" d="M 406,15 486,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(446,15)"><rect x="-15" y="-5" width="30.49" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>right</tspan></text></g> <path id="gmark_F_G" d="M 486,15 566,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(526,15)"><rect x="-10" y="-5" width="21.58" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>left</tspan></text></g> <path id="gmark_G_H" d="M 566,15 646,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(606,15)"><rect x="-15" y="-5" width="30.49" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>right</tspan></text></g> <path id="gmark_I_J" d="M 46,105 486,105" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(266,105)"><rect x="-52" y="-5" width="105.62" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>left slot &amp; right slot</tspan></text></g><path id="gmark_A_I" d="M 46,15 46,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_B_L" d="M 126,15 126,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_C_M" d="M 206,15 206,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_O" d="M 326,15 326,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_E_P" d="M 406,15 406,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_F_J" d="M 486,15 486,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_G_R" d="M 566,15 566,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_H_S" d="M 646,15 646,75" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_3"><g transform="translate(0,35)" id="wavegap_1_3"><use transform="translate(260)" xlink:href="#gap"></use></g><g transform="translate(0,65)" id="wavegap_2_3"><use transform="translate(260)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_3"><g></g></g></g></svg>

PDM 格式的数据通常可以经过以下几个步骤转换为 PCM 格式：

1. 低通滤波：用于还原模拟信号波形。一般采用 FIR 滤波器；
2. 下采样：用于将 PDM 的过采样率降低到期望的 PCM 采样率。下采样可以用简单的抽值法实现；
3. 高通滤波：用于去除信号的直流部分；
4. 放大：用于调整转换后的 PCM 数据的増益。一般由转换后的 PCM 信号乘以一个系数得到最终 PCM 信号。

对于具有 `PCM-to-PDM` 格式转换器的 I2S 端口，可以在发送数据的时候，将 PCM 数据转换为 PDM 格式发送。 对于具有 `PDM-to-PCM` 格式转换器的 I2S 端口，可以再接收数据的时候，将收到的 PDM 格式的数据转换为 PCM 格式。 若硬件不具备上述的格式转换器，则 PDM 模式只能收发原始的 PDM 格式数据。需要在软件上实现 PDM-to-PCM 的转换逻辑以此得到常用的 PCM 格式数据。

> [!note] 备注
> 无论原始 PDM 格式还是 PCM 格式，PDM 模式下的一个数据单元总是 16 比特的位宽。例如，用原始 PDM 格式发送数据，那么您数组中的数据应该像这样排列：CH0 0x1234，CH1 0x5678，CH0 0x9abc，CH1 0xdef0。RX 方向同理。

#### PDM TX 模式原始 PDM 数据格式

要发送原始 PDM 格式的数据，您需要将 设为 。另外在设置 时请注意，PDM 的采样率通常在若干 MHz，典型值范围一般是 1.024MHz 到 6.144MHz 之间，您可以根据需求来设置。

而原始 PDM 数据格式下的声道配置，可以通过帮助宏 或: 来配置。

#### PDM TX 模式 PCM 数据格式（采用 PCM-to-PDM 格式转换器）

ESP32-S3 在 `I2S0` 上支持 PCM-to-PDM 格式转换器，您可以通过 设为 来启用 PCM-to-PDM 格式转换器。启用后会将发送的 PCM 格式的数据转换为 PDM 格式发送。另外在设置 时请注意，PCM 的采样率通常低于 100 KHz，典型值的范围一般是 16KHz 到 48KHz 之间，您可以根据需求来设置。

另外 PCM-to-PDM 转换器可配置上采样参数 和 。上采样率可以通过公式 `up_sample_rate = i2s_pdm_tx_clk_config_t::up_sample_fp / i2s_pdm_tx_clk_config_t::up_sample_fs` 来计算。在 PDM TX 中有以下两种上采样模式，输出的 PDM 采样频率和配置的 PCM 采样频率关系如下：

- **固定时钟频率模式** ：在这种模式下，上采样率将根据采样率的变化而变化。设置 `fp = 960` 、 `fs = (PCM)sample_rate / 100` ，则 CLK 管脚上的输出的 PDM 时钟频率将固定为 `128 * 48 KHz = 6.144 MHz` 。
- **固定上采样率模式** ：在这种模式下，上采样率固定为 2。即设置 `fp = 960` 、 `fs = 480` ，则 CLK 管脚上的 PDM 的时钟频率将为 `128 * sample_rate` 。

而 PCM 数据格式下的声道配置，您可以通过帮助宏 和 来配置。

#### PDM RX 模式原始 PDM 数据格式

要接收原始 PDM 格式的数据，您需要将 设为 。另外在设置 时请注意，PDM 的采样率通常在若干 MHz，典型值范围一般是 1.024MHz 到 6.144MHz 之间，您可以根据需求来设置。

而原始 PDM 数据格式下的声道配置，可以通过帮助宏 来配置。

#### PDM RX 模式 PCM 数据格式（采用 PDM-to-PCM 格式转换器）

ESP32-S3 在 `I2S0` 上支持 PDM-to-PCM 格式转换器，您可以通过 设为 来启用 PDM-to-PCM 格式转换器。启用后会将接收到的 PDM 格式的数据转换为 PCM 格式。另外在设置 时请注意，PCM 的采样率通常低于 100 KHz，典型值的范围一般是 16KHz 到 48KHz 之间，您可以根据需求来设置。

另外 PDM-to-PCM 转换器可配置下采样参数 。在 PDM RX 中有以下两种下采样模式，输出的 PDM 采样频率和配置的 PCM 采样频率关系如下：

- ：在这种模式下，CLK 管脚的 PDM 时钟频率将为 `(PCM) sample_rate * 64` 。
- ： 在这种模式下，CLK 管脚的 PDM 时钟频率将为 `(PCM) sample_rate * 128` 。

而 PCM 数据格式下的声道配置，可以通过帮助宏 来配置。

### TDM 模式

TDM（Time Division Multiplexing，时分多路复用）模式最多支持 16 个声道，可通过 启用通道。

但由于硬件限制，声道设置为 32 位宽时最多只能支持 4 个声道，16 位宽时最多只能支持 8 个声道，8 位宽时最多只能支持 16 个声道。TDM 的声道通信格式与标准模式基本相同，但有一些细微差别。

- **Philips 格式** ：数据信号与 WS 信号相比有一个位的位移。无论一帧中包含多少个声道，WS 信号的占空比将始终保持为 50%。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_4" height="196" width="1040" viewBox="0 0 1040 196" overflow="hidden"><g id="waves_4"><rect width="1040" height="196" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_4"><g id="gmarks_4"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_4" x1="0" y1="0" x2="0" y2="150"></line><line id="gmark_1_4" x1="40" y1="0" x2="40" y2="150"></line><line id="gmark_2_4" x1="80" y1="0" x2="80" y2="150"></line><line id="gmark_3_4" x1="120" y1="0" x2="120" y2="150"></line><line id="gmark_4_4" x1="160" y1="0" x2="160" y2="150"></line><line id="gmark_5_4" x1="200" y1="0" x2="200" y2="150"></line><line id="gmark_6_4" x1="240" y1="0" x2="240" y2="150"></line><line id="gmark_7_4" x1="280" y1="0" x2="280" y2="150"></line><line id="gmark_8_4" x1="320" y1="0" x2="320" y2="150"></line><line id="gmark_9_4" x1="360" y1="0" x2="360" y2="150"></line><line id="gmark_10_4" x1="400" y1="0" x2="400" y2="150"></line><line id="gmark_11_4" x1="440" y1="0" x2="440" y2="150"></line><line id="gmark_12_4" x1="480" y1="0" x2="480" y2="150"></line><line id="gmark_13_4" x1="520" y1="0" x2="520" y2="150"></line><line id="gmark_14_4" x1="560" y1="0" x2="560" y2="150"></line><line id="gmark_15_4" x1="600" y1="0" x2="600" y2="150"></line><line id="gmark_16_4" x1="640" y1="0" x2="640" y2="150"></line><line id="gmark_17_4" x1="680" y1="0" x2="680" y2="150"></line><line id="gmark_18_4" x1="720" y1="0" x2="720" y2="150"></line><line id="gmark_19_4" x1="760" y1="0" x2="760" y2="150"></line><line id="gmark_20_4" x1="800" y1="0" x2="800" y2="150"></line><line id="gmark_21_4" x1="840" y1="0" x2="840" y2="150"></line><line id="gmark_22_4" x1="880" y1="0" x2="880" y2="150"></line><line id="gmark_23_4" x1="920" y1="0" x2="920" y2="150"></line></g><text x="460" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>TDM Philips Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_4"><g transform="translate(14)" id="wavelane_draw_0_4"></g></g><g transform="translate(0,35)" id="wavelane_1_4"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_1_4"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#ddd"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#pclk"></use><use transform="translate(260)" xlink:href="#nclk"></use><use transform="translate(280)" xlink:href="#0md"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#pclk"></use><use transform="translate(380)" xlink:href="#nclk"></use><use transform="translate(400)" xlink:href="#0md"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#ddd"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#pclk"></use><use transform="translate(500)" xlink:href="#nclk"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use><use transform="translate(560)" xlink:href="#ddd"></use><use transform="translate(580)" xlink:href="#ddd"></use><use transform="translate(600)" xlink:href="#pclk"></use><use transform="translate(620)" xlink:href="#nclk"></use><use transform="translate(640)" xlink:href="#pclk"></use><use transform="translate(660)" xlink:href="#nclk"></use><use transform="translate(680)" xlink:href="#0md"></use><use transform="translate(700)" xlink:href="#ddd"></use><use transform="translate(720)" xlink:href="#ddd"></use><use transform="translate(740)" xlink:href="#ddd"></use><use transform="translate(760)" xlink:href="#pclk"></use><use transform="translate(780)" xlink:href="#nclk"></use><use transform="translate(800)" xlink:href="#0md"></use><use transform="translate(820)" xlink:href="#ddd"></use><use transform="translate(840)" xlink:href="#ddd"></use><use transform="translate(860)" xlink:href="#ddd"></use><use transform="translate(880)" xlink:href="#pclk"></use><use transform="translate(900)" xlink:href="#nclk"></use></g></g><g transform="translate(0,65)" id="wavelane_2_4"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_2_4"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xm0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#0md"></use><use transform="translate(100)" xlink:href="#ddd"></use><use transform="translate(120)" xlink:href="#dmd"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#dm0"></use><use transform="translate(180)" xlink:href="#000"></use><use transform="translate(200)" xlink:href="#000"></use><use transform="translate(220)" xlink:href="#000"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#dmd"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#dm0"></use><use transform="translate(340)" xlink:href="#000"></use><use transform="translate(360)" xlink:href="#0md"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#dmd"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#dm1"></use><use transform="translate(460)" xlink:href="#111"></use><use transform="translate(480)" xlink:href="#1mu"></use><use transform="translate(500)" xlink:href="#uuu"></use><use transform="translate(520)" xlink:href="#umu"></use><use transform="translate(540)" xlink:href="#uuu"></use><use transform="translate(560)" xlink:href="#um1"></use><use transform="translate(580)" xlink:href="#111"></use><use transform="translate(600)" xlink:href="#111"></use><use transform="translate(620)" xlink:href="#111"></use><use transform="translate(640)" xlink:href="#1mu"></use><use transform="translate(660)" xlink:href="#uuu"></use><use transform="translate(680)" xlink:href="#umu"></use><use transform="translate(700)" xlink:href="#uuu"></use><use transform="translate(720)" xlink:href="#um1"></use><use transform="translate(740)" xlink:href="#111"></use><use transform="translate(760)" xlink:href="#1mu"></use><use transform="translate(780)" xlink:href="#uuu"></use><use transform="translate(800)" xlink:href="#umu"></use><use transform="translate(820)" xlink:href="#uuu"></use><use transform="translate(840)" xlink:href="#um0"></use><use transform="translate(860)" xlink:href="#000"></use><use transform="translate(880)" xlink:href="#000"></use><use transform="translate(900)" xlink:href="#000"></use></g></g><g transform="translate(0,95)" id="wavelane_3_4"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_4"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmx"></use><use transform="translate(60)" xlink:href="#xxx"></use><use transform="translate(80)" xlink:href="#xmv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmx-2"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xxx"></use><use transform="translate(180)" xlink:href="#xxx"></use><use transform="translate(200)" xlink:href="#xmv-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmv-2-2"></use><use transform="translate(260)" xlink:href="#vvv-2"></use><use transform="translate(280)" xlink:href="#vmx-2"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xxx"></use><use transform="translate(340)" xlink:href="#xxx"></use><use transform="translate(360)" xlink:href="#xmv-2"></use><use transform="translate(380)" xlink:href="#vvv-2"></use><use transform="translate(400)" xlink:href="#vmx-2"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xxx"></use><use transform="translate(460)" xlink:href="#xxx"></use><use transform="translate(480)" xlink:href="#xmv-2"></use><use transform="translate(500)" xlink:href="#vvv-2"></use><use transform="translate(520)" xlink:href="#vmx-2"></use><use transform="translate(540)" xlink:href="#xxx"></use><use transform="translate(560)" xlink:href="#xxx"></use><use transform="translate(580)" xlink:href="#xxx"></use><use transform="translate(600)" xlink:href="#xmv-2"></use><use transform="translate(620)" xlink:href="#vvv-2"></use><use transform="translate(640)" xlink:href="#vmv-2-2"></use><use transform="translate(660)" xlink:href="#vvv-2"></use><use transform="translate(680)" xlink:href="#vmx-2"></use><use transform="translate(700)" xlink:href="#xxx"></use><use transform="translate(720)" xlink:href="#xxx"></use><use transform="translate(740)" xlink:href="#xxx"></use><use transform="translate(760)" xlink:href="#xmv-2"></use><use transform="translate(780)" xlink:href="#vvv-2"></use><use transform="translate(800)" xlink:href="#vmx-2"></use><use transform="translate(820)" xlink:href="#xxx"></use><use transform="translate(840)" xlink:href="#xxx"></use><use transform="translate(860)" xlink:href="#xxx"></use><use transform="translate(880)" xlink:href="#xmv-2"></use><use transform="translate(900)" xlink:href="#vvv-2"></use><text x="106" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="266" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="386" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="506" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="626" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="666" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="786" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="906" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,125)" id="wavelane_4_4"><g transform="translate(14)" id="wavelane_draw_4_4"></g></g><g id="wavearcs_4"><path id="gmark_E_F" d="M 100,15 500,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,15)"><rect x="-29" y="-5" width="59.09" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Left Slots</tspan></text></g> <path id="gmark_F_G" d="M 500,15 900,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(700,15)"><rect x="-34" y="-5" width="68.88" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Right Slots</tspan></text></g> <path id="gmark_U_A" d="M 60,135 100,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,135)"><rect x="-22" y="-5" width="44.46" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>bit shift</tspan></text></g> <path id="gmark_A_B" d="M 100,135 260,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 1</tspan></text></g> <path id="gmark_B_C" d="M 260,135 420,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(340,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 2</tspan></text></g> <path id="gmark_C_D" d="M 420,135 500,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(460,135)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_D_J" d="M 500,135 660,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path> <g transform="translate(580,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n</tspan></text></g> <path id="gmark_J_L" d="M 660,135 820,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(740,135)"><rect x="-26" y="-5" width="52.16" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n+1</tspan></text></g> <path id="gmark_L_S" d="M 820,135 900,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(860,135)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_A_E" d="M 100,135 100,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 260,105 260,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_C_I" d="M 420,135 420,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_F" d="M 500,135 500,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_J_N" d="M 660,135 660,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_L_P" d="M 820,135 820,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_Q_G" d="M 900,105 900,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_S_R" d="M 900,135 900,75" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_T_U" d="M 60,75 60,135" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_4"><g transform="translate(0,35)" id="wavegap_1_4"></g><g transform="translate(0,65)" id="wavegap_2_4"></g><g transform="translate(0,95)" id="wavegap_3_4"><use transform="translate(194)" xlink:href="#gap"></use><use transform="translate(354)" xlink:href="#gap"></use><use transform="translate(474)" xlink:href="#gap"></use><use transform="translate(594)" xlink:href="#gap"></use><use transform="translate(754)" xlink:href="#gap"></use><use transform="translate(874)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_4"><g></g></g></g></svg>
- **MSB 格式** ：与 Philips 格式基本相同，但数据没有位移。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_5" height="196" width="1040" viewBox="0 0 1040 196" overflow="hidden"><g id="waves_5"><rect width="1040" height="196" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_5"><g id="gmarks_5"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_5" x1="0" y1="0" x2="0" y2="150"></line><line id="gmark_1_5" x1="40" y1="0" x2="40" y2="150"></line><line id="gmark_2_5" x1="80" y1="0" x2="80" y2="150"></line><line id="gmark_3_5" x1="120" y1="0" x2="120" y2="150"></line><line id="gmark_4_5" x1="160" y1="0" x2="160" y2="150"></line><line id="gmark_5_5" x1="200" y1="0" x2="200" y2="150"></line><line id="gmark_6_5" x1="240" y1="0" x2="240" y2="150"></line><line id="gmark_7_5" x1="280" y1="0" x2="280" y2="150"></line><line id="gmark_8_5" x1="320" y1="0" x2="320" y2="150"></line><line id="gmark_9_5" x1="360" y1="0" x2="360" y2="150"></line><line id="gmark_10_5" x1="400" y1="0" x2="400" y2="150"></line><line id="gmark_11_5" x1="440" y1="0" x2="440" y2="150"></line><line id="gmark_12_5" x1="480" y1="0" x2="480" y2="150"></line><line id="gmark_13_5" x1="520" y1="0" x2="520" y2="150"></line><line id="gmark_14_5" x1="560" y1="0" x2="560" y2="150"></line><line id="gmark_15_5" x1="600" y1="0" x2="600" y2="150"></line><line id="gmark_16_5" x1="640" y1="0" x2="640" y2="150"></line><line id="gmark_17_5" x1="680" y1="0" x2="680" y2="150"></line><line id="gmark_18_5" x1="720" y1="0" x2="720" y2="150"></line><line id="gmark_19_5" x1="760" y1="0" x2="760" y2="150"></line><line id="gmark_20_5" x1="800" y1="0" x2="800" y2="150"></line><line id="gmark_21_5" x1="840" y1="0" x2="840" y2="150"></line><line id="gmark_22_5" x1="880" y1="0" x2="880" y2="150"></line><line id="gmark_23_5" x1="920" y1="0" x2="920" y2="150"></line></g><text x="460" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>TDM MSB Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_5"><g transform="translate(14)" id="wavelane_draw_0_5"></g></g><g transform="translate(0,35)" id="wavelane_1_5"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_1_5"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#ddd"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#pclk"></use><use transform="translate(260)" xlink:href="#nclk"></use><use transform="translate(280)" xlink:href="#0md"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#pclk"></use><use transform="translate(380)" xlink:href="#nclk"></use><use transform="translate(400)" xlink:href="#0md"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#ddd"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#pclk"></use><use transform="translate(500)" xlink:href="#nclk"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use><use transform="translate(560)" xlink:href="#ddd"></use><use transform="translate(580)" xlink:href="#ddd"></use><use transform="translate(600)" xlink:href="#pclk"></use><use transform="translate(620)" xlink:href="#nclk"></use><use transform="translate(640)" xlink:href="#pclk"></use><use transform="translate(660)" xlink:href="#nclk"></use><use transform="translate(680)" xlink:href="#0md"></use><use transform="translate(700)" xlink:href="#ddd"></use><use transform="translate(720)" xlink:href="#ddd"></use><use transform="translate(740)" xlink:href="#ddd"></use><use transform="translate(760)" xlink:href="#pclk"></use><use transform="translate(780)" xlink:href="#nclk"></use><use transform="translate(800)" xlink:href="#0md"></use><use transform="translate(820)" xlink:href="#ddd"></use><use transform="translate(840)" xlink:href="#ddd"></use><use transform="translate(860)" xlink:href="#ddd"></use><use transform="translate(880)" xlink:href="#pclk"></use><use transform="translate(900)" xlink:href="#nclk"></use></g></g><g transform="translate(0,65)" id="wavelane_2_5"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_2_5"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xm0"></use><use transform="translate(60)" xlink:href="#000"></use><use transform="translate(80)" xlink:href="#0md"></use><use transform="translate(100)" xlink:href="#ddd"></use><use transform="translate(120)" xlink:href="#dmd"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#dm0"></use><use transform="translate(180)" xlink:href="#000"></use><use transform="translate(200)" xlink:href="#000"></use><use transform="translate(220)" xlink:href="#000"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#dmd"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#dm0"></use><use transform="translate(340)" xlink:href="#000"></use><use transform="translate(360)" xlink:href="#0md"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#dmd"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#dm1"></use><use transform="translate(460)" xlink:href="#111"></use><use transform="translate(480)" xlink:href="#1mu"></use><use transform="translate(500)" xlink:href="#uuu"></use><use transform="translate(520)" xlink:href="#umu"></use><use transform="translate(540)" xlink:href="#uuu"></use><use transform="translate(560)" xlink:href="#um1"></use><use transform="translate(580)" xlink:href="#111"></use><use transform="translate(600)" xlink:href="#111"></use><use transform="translate(620)" xlink:href="#111"></use><use transform="translate(640)" xlink:href="#1mu"></use><use transform="translate(660)" xlink:href="#uuu"></use><use transform="translate(680)" xlink:href="#umu"></use><use transform="translate(700)" xlink:href="#uuu"></use><use transform="translate(720)" xlink:href="#um1"></use><use transform="translate(740)" xlink:href="#111"></use><use transform="translate(760)" xlink:href="#1mu"></use><use transform="translate(780)" xlink:href="#uuu"></use><use transform="translate(800)" xlink:href="#umu"></use><use transform="translate(820)" xlink:href="#uuu"></use><use transform="translate(840)" xlink:href="#um0"></use><use transform="translate(860)" xlink:href="#000"></use></g></g><g transform="translate(0,95)" id="wavelane_3_5"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_5"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmv-2"></use><use transform="translate(60)" xlink:href="#vvv-2"></use><use transform="translate(80)" xlink:href="#vmx-2"></use><use transform="translate(100)" xlink:href="#xxx"></use><use transform="translate(120)" xlink:href="#xxx"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xmv-2"></use><use transform="translate(180)" xlink:href="#vvv-2"></use><use transform="translate(200)" xlink:href="#vmv-2-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmx-2"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vmx-2"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xxx"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xmv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmx-2"></use><use transform="translate(500)" xlink:href="#xxx"></use><use transform="translate(520)" xlink:href="#xxx"></use><use transform="translate(540)" xlink:href="#xxx"></use><use transform="translate(560)" xlink:href="#xmv-2"></use><use transform="translate(580)" xlink:href="#vvv-2"></use><use transform="translate(600)" xlink:href="#vmv-2-2"></use><use transform="translate(620)" xlink:href="#vvv-2"></use><use transform="translate(640)" xlink:href="#vmx-2"></use><use transform="translate(660)" xlink:href="#xxx"></use><use transform="translate(680)" xlink:href="#xxx"></use><use transform="translate(700)" xlink:href="#xxx"></use><use transform="translate(720)" xlink:href="#xmv-2"></use><use transform="translate(740)" xlink:href="#vvv-2"></use><use transform="translate(760)" xlink:href="#vmx-2"></use><use transform="translate(780)" xlink:href="#xxx"></use><use transform="translate(800)" xlink:href="#xxx"></use><use transform="translate(820)" xlink:href="#xxx"></use><use transform="translate(840)" xlink:href="#xmv-2"></use><use transform="translate(860)" xlink:href="#vvv-2"></use><text x="66" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="186" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="346" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="466" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="586" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="626" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="746" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="866" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,125)" id="wavelane_4_5"><g transform="translate(14)" id="wavelane_draw_4_5"></g></g><g id="wavearcs_5"><path id="gmark_E_F" d="M 60,15 460,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(260,15)"><rect x="-29" y="-5" width="59.09" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Left Slots</tspan></text></g> <path id="gmark_F_G" d="M 460,15 860,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(660,15)"><rect x="-34" y="-5" width="68.88" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Right Slots</tspan></text></g> <path id="gmark_A_B" d="M 60,135 220,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(140,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 1</tspan></text></g> <path id="gmark_B_C" d="M 220,135 380,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 2</tspan></text></g> <path id="gmark_C_D" d="M 380,135 460,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(420,135)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_D_J" d="M 460,135 620,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path> <g transform="translate(540,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n</tspan></text></g> <path id="gmark_J_L" d="M 620,135 780,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(700,135)"><rect x="-26" y="-5" width="52.16" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n+1</tspan></text></g> <path id="gmark_L_S" d="M 780,135 860,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(820,135)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_A_E" d="M 60,135 60,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 220,105 220,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_C_I" d="M 380,135 380,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_F" d="M 460,135 460,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_J_N" d="M 620,135 620,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_L_P" d="M 780,135 780,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_Q_G" d="M 860,105 860,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_S_R" d="M 860,135 860,75" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_5"><g transform="translate(0,35)" id="wavegap_1_5"></g><g transform="translate(0,65)" id="wavegap_2_5"></g><g transform="translate(0,95)" id="wavegap_3_5"><use transform="translate(154)" xlink:href="#gap"></use><use transform="translate(314)" xlink:href="#gap"></use><use transform="translate(434)" xlink:href="#gap"></use><use transform="translate(554)" xlink:href="#gap"></use><use transform="translate(714)" xlink:href="#gap"></use><use transform="translate(834)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_5"><g></g></g></g></svg>
- **PCM 短帧同步** ：数据有一个位的位移，同时 WS 信号变为脉冲，每帧持续一个 BCLK 周期。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_6" height="196" width="680" viewBox="0 0 680 196" overflow="hidden"><g id="waves_6"><rect width="680" height="196" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_6"><g id="gmarks_6"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_6" x1="0" y1="0" x2="0" y2="150"></line><line id="gmark_1_6" x1="40" y1="0" x2="40" y2="150"></line><line id="gmark_2_6" x1="80" y1="0" x2="80" y2="150"></line><line id="gmark_3_6" x1="120" y1="0" x2="120" y2="150"></line><line id="gmark_4_6" x1="160" y1="0" x2="160" y2="150"></line><line id="gmark_5_6" x1="200" y1="0" x2="200" y2="150"></line><line id="gmark_6_6" x1="240" y1="0" x2="240" y2="150"></line><line id="gmark_7_6" x1="280" y1="0" x2="280" y2="150"></line><line id="gmark_8_6" x1="320" y1="0" x2="320" y2="150"></line><line id="gmark_9_6" x1="360" y1="0" x2="360" y2="150"></line><line id="gmark_10_6" x1="400" y1="0" x2="400" y2="150"></line><line id="gmark_11_6" x1="440" y1="0" x2="440" y2="150"></line><line id="gmark_12_6" x1="480" y1="0" x2="480" y2="150"></line><line id="gmark_13_6" x1="520" y1="0" x2="520" y2="150"></line><line id="gmark_14_6" x1="560" y1="0" x2="560" y2="150"></line></g><text x="280" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>TDM PCM (short) Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_6"><g transform="translate(14)" id="wavelane_draw_0_6"></g></g><g transform="translate(0,35)" id="wavelane_1_6"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_1_6"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#ddd"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#pclk"></use><use transform="translate(260)" xlink:href="#nclk"></use><use transform="translate(280)" xlink:href="#0md"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#pclk"></use><use transform="translate(380)" xlink:href="#nclk"></use><use transform="translate(400)" xlink:href="#0md"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#ddd"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#pclk"></use><use transform="translate(500)" xlink:href="#nclk"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use></g></g><g transform="translate(0,65)" id="wavelane_2_6"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_2_6"><use xlink:href="#000"></use><use transform="translate(20)" xlink:href="#000"></use><use transform="translate(40)" xlink:href="#0m1"></use><use transform="translate(60)" xlink:href="#111"></use><use transform="translate(80)" xlink:href="#1m0"></use><use transform="translate(100)" xlink:href="#000"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#ddd"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#ddd"></use><use transform="translate(220)" xlink:href="#ddd"></use><use transform="translate(240)" xlink:href="#ddd"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#ddd"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#ddd"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#dm0"></use><use transform="translate(420)" xlink:href="#000"></use><use transform="translate(440)" xlink:href="#0m1"></use><use transform="translate(460)" xlink:href="#111"></use><use transform="translate(480)" xlink:href="#1m0"></use><use transform="translate(500)" xlink:href="#000"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use></g></g><g transform="translate(0,95)" id="wavelane_3_6"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_6"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmx"></use><use transform="translate(60)" xlink:href="#xxx"></use><use transform="translate(80)" xlink:href="#xmv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmx-2"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xxx"></use><use transform="translate(180)" xlink:href="#xxx"></use><use transform="translate(200)" xlink:href="#xmv-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmx-2"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vmx-2"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xxx"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xmv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmv-2-2"></use><use transform="translate(500)" xlink:href="#vvv-2"></use><use transform="translate(520)" xlink:href="#vmx-2"></use><use transform="translate(540)" xlink:href="#xxx"></use><text x="106" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="346" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="466" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="506" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,125)" id="wavelane_4_6"><g transform="translate(14)" id="wavelane_draw_4_6"></g></g><g id="wavearcs_6"><path id="gmark_T_E" d="M 60,15 100,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,15)"><rect x="-18" y="-5" width="37.09" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>pulse</tspan></text></g> <path id="gmark_E_G" d="M 100,15 500,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,15)"><rect x="-22" y="-5" width="44.35" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Frame</tspan></text></g> <path id="gmark_U_A" d="M 60,135 100,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,135)"><rect x="-22" y="-5" width="44.46" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>bit shift</tspan></text></g> <path id="gmark_A_B" d="M 100,135 260,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 1</tspan></text></g> <path id="gmark_B_C" d="M 260,135 340,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,135)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_C_D" d="M 340,135 500,135" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path> <g transform="translate(420,135)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n</tspan></text></g><path id="gmark_T_U" d="M 60,15 60,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_A_E" d="M 100,135 100,15" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 260,105 260,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_C_I" d="M 340,135 340,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_M" d="M 500,135 500,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_G" d="M 500,135 500,15" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_6"><g transform="translate(0,35)" id="wavegap_1_6"></g><g transform="translate(0,65)" id="wavegap_2_6"></g><g transform="translate(0,95)" id="wavegap_3_6"><use transform="translate(194)" xlink:href="#gap"></use><use transform="translate(314)" xlink:href="#gap"></use><use transform="translate(434)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_6"><g></g></g></g></svg>
- **PCM 长帧同步** ：数据有一个位的位移，同时 WS 信号将在每一帧持续一个声道的宽度。例如，如果启用了四个声道，那么 WS 的占空比将是 25%，如果启用了五个声道，则为 20%。
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" id="svgcontent_7" height="226" width="680" viewBox="0 0 680 226" overflow="hidden"><g id="waves_7"><rect width="680" height="226" style="stroke:none;fill:white"></rect><g transform="translate(100.5,46.5)" id="lanes_7"><g id="gmarks_7"><g style="stroke:#888;stroke-width:0.5;stroke-dasharray:1,3"><line id="gmark_0_7" x1="0" y1="0" x2="0" y2="180"></line><line id="gmark_1_7" x1="40" y1="0" x2="40" y2="180"></line><line id="gmark_2_7" x1="80" y1="0" x2="80" y2="180"></line><line id="gmark_3_7" x1="120" y1="0" x2="120" y2="180"></line><line id="gmark_4_7" x1="160" y1="0" x2="160" y2="180"></line><line id="gmark_5_7" x1="200" y1="0" x2="200" y2="180"></line><line id="gmark_6_7" x1="240" y1="0" x2="240" y2="180"></line><line id="gmark_7_7" x1="280" y1="0" x2="280" y2="180"></line><line id="gmark_8_7" x1="320" y1="0" x2="320" y2="180"></line><line id="gmark_9_7" x1="360" y1="0" x2="360" y2="180"></line><line id="gmark_10_7" x1="400" y1="0" x2="400" y2="180"></line><line id="gmark_11_7" x1="440" y1="0" x2="440" y2="180"></line><line id="gmark_12_7" x1="480" y1="0" x2="480" y2="180"></line><line id="gmark_13_7" x1="520" y1="0" x2="520" y2="180"></line><line id="gmark_14_7" x1="560" y1="0" x2="560" y2="180"></line></g><text x="280" y="-13" fill="#000" text-anchor="middle" xml:space="preserve"><tspan>TDM PCM (long) Timing Diagram</tspan></text></g> <g transform="translate(0,5)" id="wavelane_0_7"><g transform="translate(14)" id="wavelane_draw_0_7"></g></g><g transform="translate(0,35)" id="wavelane_1_7"><g transform="translate(14)" id="wavelane_draw_1_7"></g></g><g transform="translate(0,65)" id="wavelane_2_7"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>BCLK</tspan></text> <g id="wavelane_draw_2_7"><use xlink:href="#pclk"></use><use transform="translate(20)" xlink:href="#nclk"></use><use transform="translate(40)" xlink:href="#pclk"></use><use transform="translate(60)" xlink:href="#nclk"></use><use transform="translate(80)" xlink:href="#pclk"></use><use transform="translate(100)" xlink:href="#nclk"></use><use transform="translate(120)" xlink:href="#0md"></use><use transform="translate(140)" xlink:href="#ddd"></use><use transform="translate(160)" xlink:href="#ddd"></use><use transform="translate(180)" xlink:href="#ddd"></use><use transform="translate(200)" xlink:href="#pclk"></use><use transform="translate(220)" xlink:href="#nclk"></use><use transform="translate(240)" xlink:href="#pclk"></use><use transform="translate(260)" xlink:href="#nclk"></use><use transform="translate(280)" xlink:href="#0md"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#pclk"></use><use transform="translate(380)" xlink:href="#nclk"></use><use transform="translate(400)" xlink:href="#0md"></use><use transform="translate(420)" xlink:href="#ddd"></use><use transform="translate(440)" xlink:href="#ddd"></use><use transform="translate(460)" xlink:href="#ddd"></use><use transform="translate(480)" xlink:href="#pclk"></use><use transform="translate(500)" xlink:href="#nclk"></use><use transform="translate(520)" xlink:href="#0md"></use><use transform="translate(540)" xlink:href="#ddd"></use></g></g><g transform="translate(0,95)" id="wavelane_3_7"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>WS</tspan></text> <g transform="translate(14)" id="wavelane_draw_3_7"><use xlink:href="#000"></use><use transform="translate(20)" xlink:href="#000"></use><use transform="translate(40)" xlink:href="#0m1"></use><use transform="translate(60)" xlink:href="#111"></use><use transform="translate(80)" xlink:href="#1mu"></use><use transform="translate(100)" xlink:href="#uuu"></use><use transform="translate(120)" xlink:href="#uuu"></use><use transform="translate(140)" xlink:href="#uuu"></use><use transform="translate(160)" xlink:href="#um1"></use><use transform="translate(180)" xlink:href="#111"></use><use transform="translate(200)" xlink:href="#1m0"></use><use transform="translate(220)" xlink:href="#000"></use><use transform="translate(240)" xlink:href="#0md"></use><use transform="translate(260)" xlink:href="#ddd"></use><use transform="translate(280)" xlink:href="#ddd"></use><use transform="translate(300)" xlink:href="#ddd"></use><use transform="translate(320)" xlink:href="#ddd"></use><use transform="translate(340)" xlink:href="#ddd"></use><use transform="translate(360)" xlink:href="#ddd"></use><use transform="translate(380)" xlink:href="#ddd"></use><use transform="translate(400)" xlink:href="#dm0"></use><use transform="translate(420)" xlink:href="#000"></use><use transform="translate(440)" xlink:href="#0m1"></use><use transform="translate(460)" xlink:href="#111"></use><use transform="translate(480)" xlink:href="#1mu"></use><use transform="translate(500)" xlink:href="#uuu"></use><use transform="translate(520)" xlink:href="#uuu"></use><use transform="translate(540)" xlink:href="#uuu"></use></g></g><g transform="translate(0,125)" id="wavelane_4_7"><text x="-10" y="15" text-anchor="end" xml:space="preserve"><tspan>DIN / DOUT</tspan></text> <g transform="translate(14)" id="wavelane_draw_4_7"><use xlink:href="#xxx"></use><use transform="translate(20)" xlink:href="#xxx"></use><use transform="translate(40)" xlink:href="#xmx"></use><use transform="translate(60)" xlink:href="#xxx"></use><use transform="translate(80)" xlink:href="#xmv-2"></use><use transform="translate(100)" xlink:href="#vvv-2"></use><use transform="translate(120)" xlink:href="#vmx-2"></use><use transform="translate(140)" xlink:href="#xxx"></use><use transform="translate(160)" xlink:href="#xxx"></use><use transform="translate(180)" xlink:href="#xxx"></use><use transform="translate(200)" xlink:href="#xmv-2"></use><use transform="translate(220)" xlink:href="#vvv-2"></use><use transform="translate(240)" xlink:href="#vmx-2"></use><use transform="translate(260)" xlink:href="#xxx"></use><use transform="translate(280)" xlink:href="#xxx"></use><use transform="translate(300)" xlink:href="#xxx"></use><use transform="translate(320)" xlink:href="#xmv-2"></use><use transform="translate(340)" xlink:href="#vvv-2"></use><use transform="translate(360)" xlink:href="#vmx-2"></use><use transform="translate(380)" xlink:href="#xxx"></use><use transform="translate(400)" xlink:href="#xxx"></use><use transform="translate(420)" xlink:href="#xxx"></use><use transform="translate(440)" xlink:href="#xmv-2"></use><use transform="translate(460)" xlink:href="#vvv-2"></use><use transform="translate(480)" xlink:href="#vmv-2-2"></use><use transform="translate(500)" xlink:href="#vvv-2"></use><use transform="translate(520)" xlink:href="#vmx-2"></use><use transform="translate(540)" xlink:href="#xxx"></use><text x="106" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="226" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="346" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text> <text x="466" y="15" text-anchor="middle" xml:space="preserve"><tspan>LSB</tspan></text> <text x="506" y="15" text-anchor="middle" xml:space="preserve"><tspan>MSB</tspan></text></g></g> <g transform="translate(0,155)" id="wavelane_5_7"><g transform="translate(14)" id="wavelane_draw_5_7"></g></g><g id="wavearcs_7"><path id="gmark_T_L" d="M 60,15 220,15" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(140,15)"><rect x="-42" y="-5" width="84.39" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>one slot pulse</tspan></text></g> <path id="gmark_U_E" d="M 60,45 100,45" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(80,45)"><rect x="-22" y="-5" width="44.46" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>bit shift</tspan></text></g> <path id="gmark_E_G" d="M 100,45 500,45" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,45)"><rect x="-22" y="-5" width="44.35" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Frame</tspan></text></g> <path id="gmark_A_B" d="M 100,165 260,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(180,165)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot 1</tspan></text></g> <path id="gmark_B_C" d="M 260,165 340,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path><g transform="translate(300,165)"><rect x="-7" y="-5" width="14.21" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>...</tspan></text></g><path id="gmark_C_D" d="M 340,165 500,165" style="marker-end:url(#arrowhead);marker-start:url(#arrowtail);stroke:#0041c4;stroke-width:1;fill:none"></path> <g transform="translate(420,165)"><rect x="-17" y="-5" width="35.44" height="11" style="fill:#FFF;"></rect><text text-anchor="middle" y="3" style="font-size:11px;"><tspan>Slot n</tspan></text></g><path id="gmark_T_N" d="M 60,15 60,105" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_A_E" d="M 100,165 100,45" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_K_B" d="M 260,135 260,165" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_C_I" d="M 340,165 340,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_M" d="M 500,165 500,135" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_D_G" d="M 500,165 500,45" style="fill:none;stroke:#00F;stroke-width:1"></path><path id="gmark_L_F" d="M 220,15 220,135" style="fill:none;stroke:#00F;stroke-width:1"></path></g><g id="wavegaps_7"><g transform="translate(0,65)" id="wavegap_2_7"></g><g transform="translate(0,95)" id="wavegap_3_7"></g><g transform="translate(0,125)" id="wavegap_4_7"><use transform="translate(194)" xlink:href="#gap"></use><use transform="translate(314)" xlink:href="#gap"></use><use transform="translate(434)" xlink:href="#gap"></use></g></g><g></g></g><g id="groups_7"><g></g></g></g></svg>

## 功能概览

I2S 驱动提供以下服务：

### 资源管理

I2S 驱动中的资源可分为三个级别：

- `平台级资源` ：当前芯片中所有 I2S 控制器的资源。
- `控制器级资源` ：一个 I2S 控制器的资源。
- `通道级资源` ：一个 I2S 控制器 TX 或 RX 通道的资源。

公开的 API 都是通道级别的 API，通道句柄 可以帮助用户管理特定通道下的资源，而无需考虑其他两个级别的资源。高级别资源为私有资源，由驱动自动管理。用户可以调用 来分配通道句柄，或调用 来删除该句柄。

### 电源管理

电源管理启用（即开启 [CONFIG\_PM\_ENABLE](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/kconfig-reference.html#config-pm-enable) ）时，系统将在进入 Light-sleep 前调整或停止 I2S 时钟源，这可能会影响 I2S 信号，从而导致传输或接收的数据无效。

I2S 驱动可以获取电源管理锁，从而防止系统设置更改或时钟源被禁用。时钟源为 APB 时，锁的类型将被设置为 [`esp_pm_lock_type_t::ESP_PM_APB_FREQ_MAX`](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/power_management.html#_CPPv4N18esp_pm_lock_type_t19ESP_PM_APB_FREQ_MAXE) 。时钟源为 APLL（若支持）时，锁的类型将被设置为 [`esp_pm_lock_type_t::ESP_PM_NO_LIGHT_SLEEP`](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/power_management.html#_CPPv4N18esp_pm_lock_type_t21ESP_PM_NO_LIGHT_SLEEPE) 。用户通过 I2S 读写时（即调用 或 ），驱动程序将获取电源管理锁，并在读写完成后释放锁。

### 有限状态机

I2S 通道有三种状态，分别为 `registered（已注册）` 、 `ready（准备就绪）` 和 `running（运行中）` ，它们的关系如下图所示：

![I2S 有限状态机](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/_images/i2s_state_machine.png)

I2S 有限状态机 

图中的 `<mode>` 可用相应的 I2S 通信模式来代替，如 `std` 代表标准的双声道模式。更多关于通信模式的信息，请参考 小节。

### 数据传输

I2S 的数据传输（包括数据发送和接收）由 DMA 实现。在传输数据之前，请调用 来启用特定的通道。发送或接收的数据达到 DMA 缓冲区的大小时，将触发 `I2S_OUT_EOF` 或 `I2S_IN_SUC_EOF` 中断。注意，DMA 缓冲区的大小不等于 ，这里的一帧是指一个 WS 周期内的所有采样数据。因此， `dma_buffer_size = dma_frame_num * slot_num * slot_bit_width / 8` 。传输数据时，可以调用 来输入数据，并把数据从源缓冲区复制到 DMA TX 缓冲区等待传输完成。此过程将重复进行，直到发送的字节数达到配置的大小。接收数据时，用户可以调用函数 来等待接收包含 DMA 缓冲区地址的消息队列，从而将数据从 DMA RX 缓冲区复制到目标缓冲区。

和 都是阻塞函数，在源缓冲区的数据发送完毕前，或是整个目标缓冲区都被加载数据占用时，它们会一直保持等待状态。在等待时间达到最大阻塞时间时，返回 `ESP_ERR_TIMEOUT` 错误。要实现异步发送或接收数据，可以通过 注册回调，随即便可在回调函数中直接访问 DMA 缓冲区，无需通过这两个阻塞函数来发送或接收数据。但请注意，该回调是一个中断回调，不要在该回调中添加复杂的逻辑、进行浮点运算或调用不可重入函数。

### 配置

用户可以通过调用相应函数（即 `i2s_channel_init_std_mode()` 、 `i2s_channel_init_pdm_rx_mode()` 、 `i2s_channel_init_pdm_tx_mode()` 或 `i2s_channel_init_tdm_mode()` ）将通道初始化为特定模式。如果初始化后需要更新配置，必须先调用 以确保通道已经停止运行，然后再调用相应的 'reconfig' 函数，例如 、 和 。

### 进阶 API

为满足高质量音频需求，驱动提供了以下进阶 API：

- : 用于预加载音频数据到 I2S 内部缓存，使得 TX 通道使能后能够立即发送数据，以此降低音频初始输出延迟。
- : 用于在运行时动态微调音频速率，以匹配音频数据生产者和消费者的速度，从而防止因速率不匹配导致的中间缓存数据累积或不足。

### IRAM 安全

默认情况下，由于写入或擦除 flash 等原因导致 cache 被禁用时，I2S 中断将产生延迟，无法及时执行 EOF 中断。

在实时应用中，可通过启用 Kconfig 选项 [CONFIG\_I2S\_ISR\_IRAM\_SAFE](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/kconfig-reference.html#config-i2s-isr-iram-safe) 来避免此种情况发生，启用后：

1. 即使在 cache 被禁用的情况下，中断仍可继续运行。
2. 驱动程序将存放进 DRAM 中（以防其意外映射到 PSRAM 中）。

启用该选项可以保证 cache 禁用时的中断运行，但会相应增加 IRAM 占用。

### 线程安全

驱动程序可保证所有公开的 I2S API 的线程安全，使用时，可以直接从不同的 RTOS 任务中调用此类 API，无需额外锁保护。注意，I2S 驱动使用 mutex 锁来保证线程安全，因此不允许在 ISR 中使用这些 API。

### Kconfig 选项

- [CONFIG\_I2S\_ISR\_IRAM\_SAFE](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/kconfig-reference.html#config-i2s-isr-iram-safe) 控制默认 ISR 处理程序能否在禁用 cache 的情况下工作。更多信息可参考 。
- [CONFIG\_I2S\_ENABLE\_DEBUG\_LOG](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/kconfig-reference.html#config-i2s-enable-debug-log) 用于启用调试日志输出。启用该选项将增加固件的二进制文件大小。

## 应用实例

I2S 驱动例程请参考 [peripherals/i2s](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s) 目录。以下为每种模式的简单用法：

### 标准 TX/RX 模式的应用

- [peripherals/i2s/i2s\_codec/i2s\_es8311](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_codec/i2s_es8311) 演示了如何在 ESP32-S3 上使用 I2S ES8311 音频编解码器来播放音乐或回声，具有高性能和低功耗的多位 delta-sigma 音频 ADC 和 DAC，提供自定义音乐、调整麦克风增益和音量的选项。
- [peripherals/i2s/i2s\_basic/i2s\_std](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_basic/i2s_std) 演示了如何在 ESP32-S3 上以单工或全双工模式使用 I2S 标准模式。

不同声道的通信格式可通过以下标准模式的辅助宏来生成。如上所述，在标准模式下有三种格式，辅助宏分别为：

时钟配置的辅助宏为：

- 。

请参考 了解 STD API 的相关信息。更多细节请参考 [esp\_driver\_i2s/include/driver/i2s\_std.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_std.h) 。

#### STD TX 模式

以 16 位数据位宽为例，如果 `uint16_t` 写缓冲区中的数据如下所示：

| 数据 0 | 数据 1 | 数据 2 | 数据 3 | 数据 4 | 数据 5 | 数据 6 | 数据 7 | ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0001 | 0x0002 | 0x0003 | 0x0004 | 0x0005 | 0x0006 | 0x0007 | 0x0008 | ... |

下表展示了在不同 和 设置下线路上的真实数据。

<table><thead><tr><th><p>数据位宽</p></th><th><p>声道模式</p></th><th><p>声道掩码</p></th><th><p>WS 低电平</p></th><th><p>WS 高电平</p></th><th><p>WS 低电平</p></th><th><p>WS 高电平</p></th><th><p>WS 低电平</p></th><th><p>WS 高电平</p></th><th><p>WS 低电平</p></th><th><p>WS 高电平</p></th></tr></thead><tbody><tr><td rowspan="6"><p>16 位</p></td><td rowspan="3"><p>单声道</p></td><td><p>左</p></td><td><p>0x0001</p></td><td><p>0x0000</p></td><td><p>0x0002</p></td><td><p>0x0000</p></td><td><p>0x0003</p></td><td><p>0x0000</p></td><td><p>0x0004</p></td><td><p>0x0000</p></td></tr><tr><td><p>右</p></td><td><p>0x0000</p></td><td><p>0x0001</p></td><td><p>0x0000</p></td><td><p>0x0002</p></td><td><p>0x0000</p></td><td><p>0x0003</p></td><td><p>0x0000</p></td><td><p>0x0004</p></td></tr><tr><td><p>左右</p></td><td><p>0x0001</p></td><td><p>0x0001</p></td><td><p>0x0002</p></td><td><p>0x0002</p></td><td><p>0x0003</p></td><td><p>0x0003</p></td><td><p>0x0004</p></td><td><p>0x0004</p></td></tr><tr><td rowspan="3"><p>立体声</p></td><td><p>左</p></td><td><p>0x0001</p></td><td><p>0x0000</p></td><td><p>0x0003</p></td><td><p>0x0000</p></td><td><p>0x0005</p></td><td><p>0x0000</p></td><td><p>0x0007</p></td><td><p>0x0000</p></td></tr><tr><td><p>右</p></td><td><p>0x0000</p></td><td><p>0x0002</p></td><td><p>0x0000</p></td><td><p>0x0004</p></td><td><p>0x0000</p></td><td><p>0x0006</p></td><td><p>0x0000</p></td><td><p>0x0008</p></td></tr><tr><td><p>左右</p></td><td><p>0x0001</p></td><td><p>0x0002</p></td><td><p>0x0003</p></td><td><p>0x0004</p></td><td><p>0x0005</p></td><td><p>0x0006</p></td><td><p>0x0007</p></td><td><p>0x0008</p></td></tr></tbody></table>

> [!note] 备注
> 数据位宽为 8 位和 32 位时，缓冲区的类型最好为 `uint8_t` 和 `uint32_t` 。但需注意，数据位宽为 24 位时，数据缓冲区应该以 3 字节对齐，即每 3 个字节代表一个 24 位数据，另外，、 和写缓冲区的大小应该为 `3` 的倍数，否则线路上的数据或采样率可能会不准确。

```c
#include "driver/i2s_std.h"
#include "driver/gpio.h"

i2s_chan_handle_t tx_handle;
/* 通过辅助宏获取默认的通道配置
 * 这个辅助宏在 'i2s_common.h' 中定义，由所有 I2S 通信模式共享
 * 它可以帮助指定 I2S 角色和端口 ID */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_AUTO, I2S_ROLE_MASTER);
/* 分配新的 TX 通道并获取该通道的句柄 */
i2s_new_channel(&chan_cfg, &tx_handle, NULL);

/* 进行配置，可以通过宏生成声道配置和时钟配置
 * 这两个辅助宏在 'i2s_std.h' 中定义，只能用于 STD 模式
 * 它们可以帮助初始化或更新声道和时钟配置 */
i2s_std_config_t std_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(48000),
    .slot_cfg = I2S_STD_MSB_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_32BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .din = I2S_GPIO_UNUSED,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
/* 初始化通道 */
i2s_channel_init_std_mode(tx_handle, &std_cfg);

/* 在写入数据之前，先启用 TX 通道 */
i2s_channel_enable(tx_handle);
i2s_channel_write(tx_handle, src_buf, bytes_to_write, bytes_written, ticks_to_wait);

/* 如果需要更新声道或时钟配置
 * 需要在更新前先禁用通道 */
// i2s_channel_disable(tx_handle);
// std_cfg.slot_cfg.slot_mode = I2S_SLOT_MODE_MONO; // 默认为立体声
// i2s_channel_reconfig_std_slot(tx_handle, &std_cfg.slot_cfg);
// std_cfg.clk_cfg.sample_rate_hz = 96000;
// i2s_channel_reconfig_std_clock(tx_handle, &std_cfg.clk_cfg);

/* 删除通道之前必须先禁用通道 */
i2s_channel_disable(tx_handle);
/* 如果不再需要句柄，删除该句柄以释放通道资源 */
i2s_del_channel(tx_handle);
```

#### STD RX 模式

例如，当数据位宽为 16 时，如线路上的数据如下所示：

| WS 低电平 | WS 高电平 | WS 低电平 | WS 高电平 | WS 低电平 | WS 高电平 | WS 低电平 | WS 高电平 | ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0001 | 0x0002 | 0x0003 | 0x0004 | 0x0005 | 0x0006 | 0x0007 | 0x0008 | ... |

不同 和 配置下缓冲区中收到的数据如下所示。

<table><thead><tr><th><p>数据位宽</p></th><th><p>声道模式</p></th><th><p>声道掩码</p></th><th><p>数据 0</p></th><th><p>数据 1</p></th><th><p>数据 2</p></th><th><p>数据 3</p></th><th><p>数据 4</p></th><th><p>数据 5</p></th><th><p>数据 6</p></th><th><p>数据 7</p></th></tr></thead><tbody><tr><td rowspan="3"><p>16 位</p></td><td rowspan="2"><p>单声道</p></td><td><p>左</p></td><td><p>0x0001</p></td><td><p>0x0003</p></td><td><p>0x0005</p></td><td><p>0x0007</p></td><td><p>0x0009</p></td><td><p>0x000b</p></td><td><p>0x000d</p></td><td><p>0x000f</p></td></tr><tr><td><p>右</p></td><td><p>0x0002</p></td><td><p>0x0004</p></td><td><p>0x0006</p></td><td><p>0x0008</p></td><td><p>0x000a</p></td><td><p>0x000c</p></td><td><p>0x000e</p></td><td><p>0x0010</p></td></tr><tr><td><p>立体声</p></td><td><p>任意</p></td><td><p>0x0001</p></td><td><p>0x0002</p></td><td><p>0x0003</p></td><td><p>0x0004</p></td><td><p>0x0005</p></td><td><p>0x0006</p></td><td><p>0x0007</p></td><td><p>0x0008</p></td></tr></tbody></table>

> [!note] 备注
> 8 位、24 位和 32 位与 16 位的情况类似，接收缓冲区的数据位宽与线路上的数据位宽相等。此外需注意，数据位宽为 24 位时， 、 和接收缓冲区的大小应该为 `3` 的倍数，否则线路上的数据或采样率可能会不准确。

```c
#include "driver/i2s_std.h"
#include "driver/gpio.h"

i2s_chan_handle_t rx_handle;
/* 通过辅助宏获取默认的通道配置
 * 这个辅助宏在 'i2s_common.h' 中定义，由所有 I2S 通信模式共享
 * 它可以帮助指定 I2S 角色和端口 ID */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_AUTO, I2S_ROLE_MASTER);
/* 分配新的 TX 通道并获取该通道的句柄 */
i2s_new_channel(&chan_cfg, NULL, &rx_handle);

/* 进行配置，可以通过宏生成声道配置和时钟配置
 * 这两个辅助宏在 'i2s_std.h' 中定义，只能用于 STD 模式
 * 它们可以帮助初始化或更新声道和时钟配置 */
i2s_std_config_t std_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(48000),
    .slot_cfg = I2S_STD_MSB_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_32BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = I2S_GPIO_UNUSED,
        .din = GPIO_NUM_19,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
/* 初始化通道 */
i2s_channel_init_std_mode(rx_handle, &std_cfg);

/* 在读取数据之前，先启动 RX 通道 */
i2s_channel_enable(rx_handle);
i2s_channel_read(rx_handle, desc_buf, bytes_to_read, bytes_read, ticks_to_wait);

/* 删除通道之前必须先禁用通道 */
i2s_channel_disable(rx_handle);
/* 如果不再需要句柄，删除该句柄以释放通道资源 */
i2s_del_channel(rx_handle);
```

### PDM TX 模式的应用

- [peripherals/i2s/i2s\_basic/i2s\_pdm](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_basic/i2s_pdm) 演示了如何在 ESP32-S3 上使用 PDM TX 模式，包括必要的硬件设置和配置。

针对 TX 通道的 PDM 模式，声道配置的辅助宏为：

- `I2S_PDM_TX_SLOT_DEFAULT_CONFIG`

时钟配置的辅助宏为：

PDM TX API 的相关信息，可参考 。更多细节请参阅 [esp\_driver\_i2s/include/driver/i2s\_pdm.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_pdm.h) 。

PDM 数据位宽固定为 16 位。如果 `int16_t` 写缓冲区中的数据如下：

| 数据 0 | 数据 1 | 数据 2 | 数据 3 | 数据 4 | 数据 5 | 数据 6 | 数据 7 | ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0001 | 0x0002 | 0x0003 | 0x0004 | 0x0005 | 0x0006 | 0x0007 | 0x0008 | ... |

下表展示了不同 和 `i2s_pdm_tx_slot_config_t::slot_mask` 设置下线路上的真实数据。为方便理解，已将线路上的数据格式由 PDM 转为 PCM。

<table><thead><tr><th><p>线路模式</p></th><th><p>声道模式</p></th><th><p>线路</p></th><th><p>左</p></th><th><p>右</p></th><th><p>左</p></th><th><p>右</p></th><th><p>左</p></th><th><p>右</p></th><th><p>左</p></th><th><p>右</p></th></tr></thead><tbody><tr><td rowspan="2"><p>单线 Codec</p></td><td><p>单声道</p></td><td><p>dout</p></td><td><p>0x0001</p></td><td><p>0x0000</p></td><td><p>0x0002</p></td><td><p>0x0000</p></td><td><p>0x0003</p></td><td><p>0x0000</p></td><td><p>0x0004</p></td><td><p>0x0000</p></td></tr><tr><td><p>立体声</p></td><td><p>dout</p></td><td><p>0x0001</p></td><td><p>0x0002</p></td><td><p>0x0003</p></td><td><p>0x0004</p></td><td><p>0x0005</p></td><td><p>0x0006</p></td><td><p>0x0007</p></td><td><p>0x0008</p></td></tr><tr><td><p>单线 DAC</p></td><td><p>单声道</p></td><td><p>dout</p></td><td><p>0x0001</p></td><td><p>0x0001</p></td><td><p>0x0002</p></td><td><p>0x0002</p></td><td><p>0x0003</p></td><td><p>0x0003</p></td><td><p>0x0004</p></td><td><p>0x0004</p></td></tr><tr><td rowspan="4"><p>双线 DAC</p></td><td rowspan="2"><p>单声道</p></td><td><p>dout</p></td><td><p>0x0002</p></td><td><p>0x0002</p></td><td><p>0x0004</p></td><td><p>0x0004</p></td><td><p>0x0006</p></td><td><p>0x0006</p></td><td><p>0x0008</p></td><td><p>0x0008</p></td></tr><tr><td><p>dout2</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td><td><p>0x0000</p></td></tr><tr><td rowspan="2"><p>立体声</p></td><td><p>dout</p></td><td><p>0x0002</p></td><td><p>0x0002</p></td><td><p>0x0004</p></td><td><p>0x0004</p></td><td><p>0x0006</p></td><td><p>0x0006</p></td><td><p>0x0008</p></td><td><p>0x0008</p></td></tr><tr><td><p>dout2</p></td><td><p>0x0001</p></td><td><p>0x0001</p></td><td><p>0x0003</p></td><td><p>0x0003</p></td><td><p>0x0005</p></td><td><p>0x0005</p></td><td><p>0x0007</p></td><td><p>0x0007</p></td></tr></tbody></table>

> [!note] 备注
> PDM TX 模式有三种线路模式，分别为 `I2S_PDM_TX_ONE_LINE_CODEC` 、 `I2S_PDM_TX_ONE_LINE_DAC` 和 `I2S_PDM_TX_TWO_LINE_DAC` 。单线 Codec 用于需要时钟信号的 PDM 编解码器，PDM 编解码器可以通过时钟电平来区分左右声道。另外两种模式可通过低通滤波器直接驱动功率放大器，而无需时钟信号，所以有两条线路来区分左右声道。此外，对于单线 Codec 的单声道模式，可以通过在 GPIO 配置中设置时钟反转标志，强制将声道改变为右声道。

```c
#include "driver/i2s_pdm.h"
#include "driver/gpio.h"

/* 分配 I2S TX 通道 */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
i2s_new_channel(&chan_cfg, &tx_handle, NULL);

/* 初始化通道为 PDM TX 模式 */
i2s_pdm_tx_config_t pdm_tx_cfg = {
    .clk_cfg = I2S_PDM_TX_CLK_DEFAULT_CONFIG(36000),
    .slot_cfg = I2S_PDM_TX_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO),
    .gpio_cfg = {
        .clk = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .invert_flags = {
            .clk_inv = false,
        },
    },
};
i2s_channel_init_pdm_tx_mode(tx_handle, &pdm_tx_cfg);

...
```

### PDM RX 模式的应用

- [peripherals/i2s/i2s\_recorder](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_recorder) 演示了如何通过 I2S 外设以 PDM 数据格式用数字 MEMS 麦克风录制音频，并将其以 `.wav` 文件格式保存到 ESP32-S3 开发板上的 SD 卡中。
- [peripherals/i2s/i2s\_basic/i2s\_pdm](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_basic/i2s_pdm) 演示了如何在 ESP32-S3 上使用 PDM RX 模式，包括必要的硬件设置和配置。

针对 RX 通道的 PDM 模式，声道配置的辅助宏为：

- 该辅助宏为接收原始 PDM 数据格式提供了一些默认配置。
- 该辅助宏为接收转换后的 PCM 数据格式提供了一些默认配置。

时钟配置的辅助宏为：

PDM RX API 的相关信息，可参考 。更多细节请参阅 [esp\_driver\_i2s/include/driver/i2s\_pdm.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_pdm.h) 。

PDM 数据位宽固定为 16 位。如果线路上的数据如下所示。为方便理解，已将线路上的数据格式由 PDM 转为 PCM。

| 左 | 右 | 左 | 右 | 左 | 右 | 左 | 右 | ... |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 0x0001 | 0x0002 | 0x0003 | 0x0004 | 0x0005 | 0x0006 | 0x0007 | 0x0008 | ... |

下表展示了不同 和 设置下 'int16\_t' 缓冲区接收的数据。

<table><thead><tr><th><p>声道模式</p></th><th><p>声道掩码</p></th><th><p>数据 0</p></th><th><p>数据 1</p></th><th><p>数据 2</p></th><th><p>数据 3</p></th><th><p>数据 4</p></th><th><p>数据 5</p></th><th><p>数据 6</p></th><th><p>数据 7</p></th></tr></thead><tbody><tr><td rowspan="2"><p>单声道</p></td><td><p>左</p></td><td><p>0x0001</p></td><td><p>0x0003</p></td><td><p>0x0005</p></td><td><p>0x0007</p></td><td><p>0x0009</p></td><td><p>0x000b</p></td><td><p>0x000d</p></td><td><p>0x000f</p></td></tr><tr><td><p>右</p></td><td><p>0x0002</p></td><td><p>0x0004</p></td><td><p>0x0006</p></td><td><p>0x0008</p></td><td><p>0x000a</p></td><td><p>0x000c</p></td><td><p>0x000e</p></td><td><p>0x0010</p></td></tr><tr><td><p>立体声</p></td><td><p>左右</p></td><td><p>0x0002</p></td><td><p>0x0001</p></td><td><p>0x0004</p></td><td><p>0x0003</p></td><td><p>0x0006</p></td><td><p>0x0005</p></td><td><p>0x0008</p></td><td><p>0x0007</p></td></tr></tbody></table>

> [!note] 备注
> 在立体声模式下，右声道先被接收。如需切换缓冲区中的左右声道，可设置 `i2s_pdm_rx_gpio_config_t::invert_flags::clk_inv` 来强制反转时钟信号。
> 
> ESP32-S3 在 PDM RX 模式下最多可以支持四条数据线，每条数据线可以连接到两个 PDM MIC 的左右两个声道，这意味着 ESP32-S3 的 PDM RX 模式最多可以支持八个 PDM MIC。如需启用多条数据线，可设置 `i2s_pdm_rx_gpio_config_t::slot_mask` 中相应的位来启用相应声道，然后设置 中的数据 GPIO。

```c
#include "driver/i2s_pdm.h"
#include "driver/gpio.h"

i2s_chan_handle_t rx_handle;

/* 分配 I2S RX 通道 */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
i2s_new_channel(&chan_cfg, NULL, &rx_handle);

/* 初始化通道为 PDM RX 模式 */
i2s_pdm_rx_config_t pdm_rx_cfg = {
    .clk_cfg = I2S_PDM_RX_CLK_DEFAULT_CONFIG(36000),
    // 若不支持 PDM 转 PCM 格式转换器，请使用原始 PDM 格式
    // .slot_cfg = I2S_PDM_RX_SLOT_RAW_FMT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO),
    .slot_cfg = I2S_PDM_RX_SLOT_PCM_FMT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO),
    .gpio_cfg = {
        .clk = GPIO_NUM_5,
        .din = GPIO_NUM_19,
        .invert_flags = {
            .clk_inv = false,
        },
    },
};
i2s_channel_init_pdm_rx_mode(rx_handle, &pdm_rx_cfg);

...
```

### TDM TX/RX 模式的应用

- [peripherals/i2s/i2s\_codec/i2s\_es7210\_tdm](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_codec/i2s_es7210_tdm) 演示了如何在 ESP32-S3 上使用 I2S TDM 模式来记录连接到 ES7210 编解码器的四个麦克风，并将录制的声音以 `.wav` 格式保存到 SD 卡中。
- [peripherals/i2s/i2s\_basic/i2s\_tdm](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/i2s/i2s_basic/i2s_tdm) 演示了如何在 ESP32-S3 上以单工或全双工模式使用 TDM 模式。

可以通过以下 TDM 模式的辅助宏生成不同的声道通信格式。如上所述，TDM 模式有四种格式，它们的辅助宏分别为：

时钟配置的辅助宏为：

有关 TDM API 的信息，请参阅 。更多细节请参阅 [esp\_driver\_i2s/include/driver/i2s\_tdm.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_tdm.h) 。

> [!note] 备注
> 在为从机配置时钟时，由于硬件限制，请注意 不应小于 8，增加此字段的值可以减少从机发送数据的延迟。使用高采样率时，数据可能会延迟一个 BCLK 周期以上，这将导致数据错位。可以通过缓慢增加 的值来进行校正。
> 
> 由于 是 MCLK 基于 BCLK 的除数，增加该值也可以提高 MCLK 频率。因此，如果 MCLK 频率太高，将会无法从源时钟分频，此时时钟计算可能会失败，也就是说 不是越大越好。

#### TDM TX 模式

```c
#include "driver/i2s_tdm.h"
#include "driver/gpio.h"

/* 分配 I2S TX 通道 */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_AUTO, I2S_ROLE_MASTER);
i2s_new_channel(&chan_cfg, &tx_handle, NULL);

/* 初始化通道为 TDM 模式 */
i2s_tdm_config_t tdm_cfg = {
    .clk_cfg = I2S_TDM_CLK_DEFAULT_CONFIG(44100),
    .slot_cfg = I2S_TDM_MSB_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO,
                I2S_TDM_SLOT0 | I2S_TDM_SLOT1 | I2S_TDM_SLOT2 | I2S_TDM_SLOT3),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .din = I2S_GPIO_UNUSED,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
i2s_channel_init_tdm_mode(tx_handle, &tdm_cfg);

...
```

#### TDM RX 模式

```c
#include "driver/i2s_tdm.h"
#include "driver/gpio.h"

/* 将通道模式设置为 TDM */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_CONFIG(I2S_ROLE_MASTER, I2S_COMM_MODE_TDM, &i2s_pin);
i2s_new_channel(&chan_cfg, NULL, &rx_handle);

/* 初始化通道为 TDM 模式 */
i2s_tdm_config_t tdm_cfg = {
    .clk_cfg = I2S_TDM_CLK_DEFAULT_CONFIG(44100),
    .slot_cfg = I2S_TDM_MSB_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO,
                I2S_TDM_SLOT0 | I2S_TDM_SLOT1 | I2S_TDM_SLOT2 | I2S_TDM_SLOT3),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = I2S_GPIO_UNUSED,
        .din = GPIO_NUM_18,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
i2s_channel_init_tdm_mode(rx_handle, &tdm_cfg);
...
```

### 全双工

全双工模式可以在 I2S 端口中同时注册 TX 和 RX 通道，同时通道共享 BCLK 和 WS 信号。目前，标准和 TDM 通信模式支持以下方式的全双工通信，但不支持 PDM 全双工模式，因为 PDM 模式下 TX 和 RX 通道的时钟不同。

请注意，一个句柄只能代表一个通道，因此仍然需要对 TX 和 RX 通道逐个进行声道和时钟配置。

驱动支持两种分配全双工通道的方法：

1. 在调用 函数时，同时分配 TX 和 RX 通道两个通道。

```c
#include "driver/i2s_std.h"
#include "driver/gpio.h"

i2s_chan_handle_t tx_handle;
i2s_chan_handle_t rx_handle;

/* 分配两个 I2S 通道 */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_AUTO, I2S_ROLE_MASTER);
/* 同时分配给 TX 和 RX 通道，使其进入全双工模式。 */
i2s_new_channel(&chan_cfg, &tx_handle, &rx_handle);

/* 配置两个通道，因为在全双工模式下，TX 和 RX 通道必须相同。 */
i2s_std_config_t std_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(32000),
    .slot_cfg = I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .din = GPIO_NUM_19,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
i2s_channel_init_std_mode(tx_handle, &std_cfg);
i2s_channel_init_std_mode(rx_handle, &std_cfg);

i2s_channel_enable(tx_handle);
i2s_channel_enable(rx_handle);

...
```

2. 调用两次 函数分别分配 TX 和 RX 通道，但使用相同配置初始化 TX 和 RX 通道。

```c
#include "driver/i2s_std.h"
#include "driver/gpio.h"

i2s_chan_handle_t tx_handle;
i2s_chan_handle_t rx_handle;

/* 分配两个 I2S 通道 */
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
/* 分别分配给 TX 和 RX 通道 */
ESP_ERROR_CHECK(i2s_new_channel(&chan_cfg, &tx_handle, NULL));

/* 为两个通道设置完全相同的配置，TX 和 RX 将自动组成全双工模式 */
i2s_std_config_t std_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(32000),
    .slot_cfg = I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .din = GPIO_NUM_19,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
ESP_ERROR_CHECK(i2s_channel_init_std_mode(tx_handle, &std_cfg));
ESP_ERROR_CHECK(i2s_channel_enable(tx_handle));
// ...
ESP_ERROR_CHECK(i2s_new_channel(&chan_cfg, NULL, &rx_handle));
ESP_ERROR_CHECK(i2s_channel_init_std_mode(rx_handle, &std_cfg));
ESP_ERROR_CHECK(i2s_channel_enable(rx_handle));

...
```

### 单工模式

在单工模式下分配通道，应该为每个通道调用 。ESP32-S3 上，TX/RX 通道的时钟和 GPIO 管脚相互独立，因此可以配置为不同的模式和时钟，并且能够在单工模式下共存于同一个 I2S 端口中。对于 PDM 模式，用户可以通过在同一个 I2S 端口上注册 PDM TX 单工和 PDM RX 单工来实现 PDM 双工。但在这种情况下，PDM TX/RX 可能会使用不同的时钟，因此在配置 GPIO 管脚和时钟时需多加注意。

以下为单工模式的示例。请注意，如果 TX 和 RX 通道来自同一个控制器，则 TX 和 RX 通道的内部 MCLK 信号虽然是分开的，但输出的 MCLK 信号只能绑定到其中一个通道。如果两个通道都初始化了 MCLK，则该信号会绑定到后初始化的通道。

```c
#include "driver/i2s_std.h"
#include "driver/gpio.h"

i2s_chan_handle_t tx_handle;
i2s_chan_handle_t rx_handle;
i2s_chan_config_t chan_cfg = I2S_CHANNEL_DEFAULT_CONFIG(I2S_NUM_0, I2S_ROLE_MASTER);
ESP_ERROR_CHECK(i2s_new_channel(&chan_cfg, &tx_handle, NULL));
i2s_std_config_t std_tx_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(48000),
    .slot_cfg = I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = GPIO_NUM_0,
        .bclk = GPIO_NUM_4,
        .ws = GPIO_NUM_5,
        .dout = GPIO_NUM_18,
        .din = I2S_GPIO_UNUSED,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
/* 初始化通道 */
ESP_ERROR_CHECK(i2s_channel_init_std_mode(tx_handle, &std_tx_cfg));
ESP_ERROR_CHECK(i2s_channel_enable(tx_handle));

/* 如果没有找到其他可用的 I2S 设备，RX 通道将被注册在另一个 I2S 上
 * 并返回 ESP_ERR_NOT_FOUND */
ESP_ERROR_CHECK(i2s_new_channel(&chan_cfg, NULL, &rx_handle)); // RX 和 TX 通道都将注册在 I2S0 上，但配置可以不同
i2s_std_config_t std_rx_cfg = {
    .clk_cfg = I2S_STD_CLK_DEFAULT_CONFIG(16000),
    .slot_cfg = I2S_STD_MSB_SLOT_DEFAULT_CONFIG(I2S_DATA_BIT_WIDTH_32BIT, I2S_SLOT_MODE_STEREO),
    .gpio_cfg = {
        .mclk = I2S_GPIO_UNUSED,
        .bclk = GPIO_NUM_6,
        .ws = GPIO_NUM_7,
        .dout = I2S_GPIO_UNUSED,
        .din = GPIO_NUM_19,
        .invert_flags = {
            .mclk_inv = false,
            .bclk_inv = false,
            .ws_inv = false,
        },
    },
};
ESP_ERROR_CHECK(i2s_channel_init_std_mode(rx_handle, &std_rx_cfg));
ESP_ERROR_CHECK(i2s_channel_enable(rx_handle));
```

## 应用注意事项

### 防止数据丢失

对于需要高频采样率的应用，数据的巨大吞吐量可能会导致数据丢失。用户可以通过注册 ISR 回调函数来接收事件队列中的数据丢失事件：

> ```c
> static IRAM_ATTR bool i2s_rx_queue_overflow_callback(i2s_chan_handle_t handle, i2s_event_data_t *event, void *user_ctx)
> {
>     // 处理 RX 队列溢出事件 ...
>     return false;
> }
> 
> i2s_event_callbacks_t cbs = {
>     .on_recv = NULL,
>     .on_recv_q_ovf = i2s_rx_queue_overflow_callback,
>     .on_sent = NULL,
>     .on_send_q_ovf = NULL,
> };
> TEST_ESP_OK(i2s_channel_register_event_callback(rx_handle, &cbs, NULL));
> ```

请按照以下步骤操作，以防止数据丢失：

1. 确定中断间隔。通常来说，当发生数据丢失时，为减少中断次数，中断间隔应该越久越好。因此，在保证 DMA 缓冲区大小不超过最大值 4092 的前提下，应使 `dma_frame_num` 尽可能大。具体转换关系如下:
	```
	interrupt_interval(unit: sec) = dma_frame_num / sample_rate
	dma_buffer_size = dma_frame_num * slot_num * data_bit_width / 8 <= 4092
	```
2. 确定 `dma_desc_num` 的值。 `dma_desc_num` 由 `i2s_channel_read` 轮询周期的最大时间决定，所有接收到的数据都应该存储在两个 `i2s_channel_read` 之间。这个周期可以通过计时器或输出 GPIO 信号来计算。具体转换关系如下:
	```
	dma_desc_num > polling_cycle / interrupt_interval
	```
3. 确定接收缓冲区大小。在 `i2s_channel_read` 中提供的接收缓冲区应当能够容纳所有 DMA 缓冲区中的数据，这意味着它应该大于所有 DMA 缓冲区的总大小:
	```
	recv_buffer_size > dma_desc_num * dma_buffer_size
	```

例如，如果某个 I2S 应用的已知值包括:

```
sample_rate = 144000 Hz
data_bit_width = 32 bits
slot_num = 2
polling_cycle = 10 ms
```

那么可以按照以下公式计算出参数 `dma_frame_num` 、 `dma_desc_num` 和 `recv_buf_size`:

```
dma_frame_num * slot_num * data_bit_width / 8 = dma_buffer_size <= 4092
dma_frame_num <= 511
interrupt_interval = dma_frame_num / sample_rate = 511 / 144000 = 0.003549 s = 3.549 ms
dma_desc_num > polling_cycle / interrupt_interval = cell(10 / 3.549) = cell(2.818) = 3
recv_buffer_size > dma_desc_num * dma_buffer_size = 3 * 4092 = 12276 bytes
```

## API 参考

### 标准模式

### Header File

- [components/esp\_driver\_i2s/include/driver/i2s\_std.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_std.h)
- This header file can be included with:
	> ```c
	> #include "driver/i2s_std.h"
	> ```
- This header file is a part of the API provided by the `esp_driver_i2s` component. To declare that your component depends on `esp_driver_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_driver_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_driver_i2s
	> ```

### Functions

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_init\_std\_mode( handle, const \*std\_cfg) [](#_CPPv425i2s_channel_init_std_mode17i2s_chan_handle_tPK16i2s_std_config_t "永久链接至目标")

Initialize I2S channel to standard mode.

> [!note] 备注
> Only allowed to be called when the channel state is REGISTERED, (i.e., channel has been allocated, but not initialized) and the state will be updated to READY if initialization success, otherwise the state will return to REGISTERED.

> [!note] 备注
> When initialize the STD mode with a same configuration as another channel on a same port, these two channels can constitude as full-duplex mode automatically

参数:

- **handle** -- **\[in\]** I2S channel handler
- **std\_cfg** -- **\[in\]** Configurations for standard mode, including clock, slot and GPIO The clock configuration can be generated by the helper macro `I2S_STD_CLK_DEFAULT_CONFIG` The slot configuration can be generated by the helper macro `I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG`, `I2S_STD_PCM_SLOT_DEFAULT_CONFIG` or `I2S_STD_MSB_SLOT_DEFAULT_CONFIG`

返回:

- ESP\_OK Initialize successfully
- ESP\_ERR\_NO\_MEM No memory for storing the channel information
- ESP\_ERR\_INVALID\_ARG NULL pointer or invalid configuration
- ESP\_ERR\_INVALID\_STATE This channel is not registered

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_std\_clock( handle, const \*clk\_cfg) [](#_CPPv430i2s_channel_reconfig_std_clock17i2s_chan_handle_tPK20i2s_std_clk_config_t "永久链接至目标")

Reconfigure the I2S clock for standard mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to standard mode, i.e., `i2s_channel_init_std_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **clk\_cfg** -- **\[in\]** Standard mode clock configuration, can be generated by `I2S_STD_CLK_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not standard mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_std\_slot( handle, const \*slot\_cfg) [](#_CPPv429i2s_channel_reconfig_std_slot17i2s_chan_handle_tPK21i2s_std_slot_config_t "永久链接至目标")

Reconfigure the I2S slot for standard mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to standard mode, i.e., `i2s_channel_init_std_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **slot\_cfg** -- **\[in\]** Standard mode slot configuration, can be generated by `I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG`, `I2S_STD_PCM_SLOT_DEFAULT_CONFIG` and `I2S_STD_MSB_SLOT_DEFAULT_CONFIG`.

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_NO\_MEM No memory for DMA buffer
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not standard mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_std\_gpio( handle, const \*gpio\_cfg) [](#_CPPv429i2s_channel_reconfig_std_gpio17i2s_chan_handle_tPK21i2s_std_gpio_config_t "永久链接至目标")

Reconfigure the I2S GPIO for standard mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to standard mode, i.e., `i2s_channel_init_std_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **gpio\_cfg** -- **\[in\]** Standard mode GPIO configuration, specified by user

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not standard mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

### Structures

struct i2s\_std\_slot\_config\_t [](#_CPPv421i2s_std_slot_config_t "永久链接至目标")

I2S slot configuration for standard mode.

Public Members

data\_bit\_width [](#_CPPv4N21i2s_std_slot_config_t14data_bit_widthE "永久链接至目标")

I2S sample data bit width (valid data bits per sample)

slot\_bit\_width [](#_CPPv4N21i2s_std_slot_config_t14slot_bit_widthE "永久链接至目标")

I2S slot bit width (total bits per slot)

slot\_mode [](#_CPPv4N21i2s_std_slot_config_t9slot_modeE "永久链接至目标")

Set mono or stereo mode with I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO In TX direction, mono means the written buffer contains only one slot data and stereo means the written buffer contains both left and right data

slot\_mask [](#_CPPv4N21i2s_std_slot_config_t9slot_maskE "永久链接至目标")

Select the left, right or both slot

uint32\_t ws\_width [](#_CPPv4N21i2s_std_slot_config_t8ws_widthE "永久链接至目标")

WS signal width (i.e. the number of BCLK ticks that WS signal is high)

bool ws\_pol [](#_CPPv4N21i2s_std_slot_config_t6ws_polE "永久链接至目标")

WS signal polarity, set true to enable high lever first

bool bit\_shift [](#_CPPv4N21i2s_std_slot_config_t9bit_shiftE "永久链接至目标")

Set to enable bit shift in Philips mode

bool left\_align [](#_CPPv4N21i2s_std_slot_config_t10left_alignE "永久链接至目标")

Set to enable left alignment

bool big\_endian [](#_CPPv4N21i2s_std_slot_config_t10big_endianE "永久链接至目标")

Set to enable big endian

bool bit\_order\_lsb [](#_CPPv4N21i2s_std_slot_config_t13bit_order_lsbE "永久链接至目标")

Set to enable lsb first

struct i2s\_std\_clk\_config\_t [](#_CPPv420i2s_std_clk_config_t "永久链接至目标")

I2S clock configuration for standard mode.

Public Members

uint32\_t sample\_rate\_hz [](#_CPPv4N20i2s_std_clk_config_t14sample_rate_hzE "永久链接至目标")

I2S sample rate

clk\_src [](#_CPPv4N20i2s_std_clk_config_t7clk_srcE "永久链接至目标")

Choose clock source, see `soc_periph_i2s_clk_src_t` for the supported clock sources. selected `I2S_CLK_SRC_EXTERNAL` (if supports) to enable the external source clock input via MCLK pin,

uint32\_t ext\_clk\_freq\_hz [](#_CPPv4N20i2s_std_clk_config_t15ext_clk_freq_hzE "永久链接至目标")

External clock source frequency in Hz, only take effect when `clk_src = I2S_CLK_SRC_EXTERNAL`, otherwise this field will be ignored, Please make sure the frequency input is equal or greater than BCLK, i.e. `sample_rate_hz * slot_bits * 2`

mclk\_multiple [](#_CPPv4N20i2s_std_clk_config_t13mclk_multipleE "永久链接至目标")

The multiple of MCLK to the sample rate Default is 256 in the helper macro, it can satisfy most of cases, but please set this field a multiple of `3` (like 384) when using 24-bit data width, otherwise the sample rate might be inaccurate

uint32\_t bclk\_div [](#_CPPv4N20i2s_std_clk_config_t8bclk_divE "永久链接至目标")

The division from MCLK to BCLK, only take effect for slave role, it shouldn't be smaller than 8. Increase this field when data sent by slave lag behind

struct i2s\_std\_gpio\_config\_t [](#_CPPv421i2s_std_gpio_config_t "永久链接至目标")

I2S standard mode GPIO pins configuration.

Public Members

gpio\_num\_t mclk [](#_CPPv4N21i2s_std_gpio_config_t4mclkE "永久链接至目标")

MCK pin, output by default, input if the clock source is selected to `I2S_CLK_SRC_EXTERNAL`

gpio\_num\_t bclk [](#_CPPv4N21i2s_std_gpio_config_t4bclkE "永久链接至目标")

BCK pin, input in slave role, output in master role

gpio\_num\_t ws [](#_CPPv4N21i2s_std_gpio_config_t2wsE "永久链接至目标")

WS pin, input in slave role, output in master role

gpio\_num\_t dout [](#_CPPv4N21i2s_std_gpio_config_t4doutE "永久链接至目标")

DATA pin, output

gpio\_num\_t din [](#_CPPv4N21i2s_std_gpio_config_t3dinE "永久链接至目标")

DATA pin, input

uint32\_t mclk\_inv [](#_CPPv4N21i2s_std_gpio_config_t8mclk_invE "永久链接至目标")

Set 1 to invert the MCLK input/output

uint32\_t bclk\_inv [](#_CPPv4N21i2s_std_gpio_config_t8bclk_invE "永久链接至目标")

Set 1 to invert the BCLK input/output

uint32\_t ws\_inv [](#_CPPv4N21i2s_std_gpio_config_t6ws_invE "永久链接至目标")

Set 1 to invert the WS input/output

struct invert\_flags [](#_CPPv4N21i2s_std_gpio_config_t12invert_flagsE "永久链接至目标")

GPIO pin invert flags

struct i2s\_std\_config\_t [](#_CPPv416i2s_std_config_t "永久链接至目标")

I2S standard mode major configuration that including clock/slot/GPIO configuration.

Public Members

clk\_cfg [](#_CPPv4N16i2s_std_config_t7clk_cfgE "永久链接至目标")

Standard mode clock configuration, can be generated by macro I2S\_STD\_CLK\_DEFAULT\_CONFIG

slot\_cfg [](#_CPPv4N16i2s_std_config_t8slot_cfgE "永久链接至目标")

Standard mode slot configuration, can be generated by macros I2S\_STD\_\[mode\]\_SLOT\_DEFAULT\_CONFIG, \[mode\] can be replaced with PHILIPS/MSB/PCM

gpio\_cfg [](#_CPPv4N16i2s_std_config_t8gpio_cfgE "永久链接至目标")

Standard mode GPIO configuration, specified by user

### Macros

I2S\_STD\_PHILIPS\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_STD_PHILIPS_SLOT_DEFAULT_CONFIG "永久链接至目标")

Philips format in 2 slots.

This file is specified for I2S standard communication mode Features:

- Philips/MSB/PCM are supported in standard mode
- Fixed to 2 slots

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_STD\_PCM\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_STD_PCM_SLOT_DEFAULT_CONFIG "永久链接至目标")

PCM(short) format in 2 slots.

> [!note] 备注
> PCM(long) is same as Philips in 2 slots

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_STD\_MSB\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_STD_MSB_SLOT_DEFAULT_CONFIG "永久链接至目标")

MSB format in 2 slots.

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_STD\_CLK\_DEFAULT\_CONFIG(rate) [](#c.I2S_STD_CLK_DEFAULT_CONFIG "永久链接至目标")

I2S default standard clock configuration.

> [!note] 备注
> Please set the mclk\_multiple to I2S\_MCLK\_MULTIPLE\_384 while using 24 bits data width Otherwise the sample rate might be imprecise since the BCLK division is not a integer

参数:

- **rate** -- sample rate

### PDM 模式

### Header File

- [components/esp\_driver\_i2s/include/driver/i2s\_pdm.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_pdm.h)
- This header file can be included with:
	> ```c
	> #include "driver/i2s_pdm.h"
	> ```
- This header file is a part of the API provided by the `esp_driver_i2s` component. To declare that your component depends on `esp_driver_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_driver_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_driver_i2s
	> ```

### Functions

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_init\_pdm\_rx\_mode( handle, const \*pdm\_rx\_cfg) [](#_CPPv428i2s_channel_init_pdm_rx_mode17i2s_chan_handle_tPK19i2s_pdm_rx_config_t "永久链接至目标")

Initialize I2S channel to PDM RX mode.

> [!note] 备注
> Only allowed to be called when the channel state is REGISTERED, (i.e., channel has been allocated, but not initialized) and the state will be updated to READY if initialization success, otherwise the state will return to REGISTERED.

参数:

- **handle** -- **\[in\]** I2S RX channel handler
- **pdm\_rx\_cfg** -- **\[in\]** Configurations for PDM RX mode, including clock, slot and GPIO The clock configuration can be generated by the helper macro `I2S_PDM_RX_CLK_DEFAULT_CONFIG` The slot configuration can be generated by the helper macro `I2S_PDM_RX_SLOT_RAW_FMT_DEFAULT_CONFIG` or `I2S_PDM_RX_SLOT_PCM_FMT_DEFAULT_CONFIG`

返回:

- ESP\_OK Initialize successfully
- ESP\_ERR\_NO\_MEM No memory for storing the channel information
- ESP\_ERR\_INVALID\_ARG NULL pointer or invalid configuration
- ESP\_ERR\_INVALID\_STATE This channel is not registered

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_rx\_clock( handle, const \*clk\_cfg) [](#_CPPv433i2s_channel_reconfig_pdm_rx_clock17i2s_chan_handle_tPK23i2s_pdm_rx_clk_config_t "永久链接至目标")

Reconfigure the I2S clock for PDM RX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM RX mode, i.e., `i2s_channel_init_pdm_rx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S RX channel handler
- **clk\_cfg** -- **\[in\]** PDM RX mode clock configuration, can be generated by `I2S_PDM_RX_CLK_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_rx\_slot( handle, const \*slot\_cfg) [](#_CPPv432i2s_channel_reconfig_pdm_rx_slot17i2s_chan_handle_tPK24i2s_pdm_rx_slot_config_t "永久链接至目标")

Reconfigure the I2S slot for PDM RX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM RX mode, i.e., `i2s_channel_init_pdm_rx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S RX channel handler
- **slot\_cfg** -- **\[in\]** PDM RX mode slot configuration, can be generated by `I2S_PDM_RX_SLOT_RAW_FMT_DEFAULT_CONFIG` or `I2S_PDM_RX_SLOT_PCM_FMT_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_NO\_MEM No memory for DMA buffer
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_rx\_gpio( handle, const \*gpio\_cfg) [](#_CPPv432i2s_channel_reconfig_pdm_rx_gpio17i2s_chan_handle_tPK24i2s_pdm_rx_gpio_config_t "永久链接至目标")

Reconfigure the I2S GPIO for PDM RX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM RX mode, i.e., `i2s_channel_init_pdm_rx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S RX channel handler
- **gpio\_cfg** -- **\[in\]** PDM RX mode GPIO configuration, specified by user

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_init\_pdm\_tx\_mode( handle, const \*pdm\_tx\_cfg) [](#_CPPv428i2s_channel_init_pdm_tx_mode17i2s_chan_handle_tPK19i2s_pdm_tx_config_t "永久链接至目标")

Initialize I2S channel to PDM TX mode.

> [!note] 备注
> Only allowed to be called when the channel state is REGISTERED, (i.e., channel has been allocated, but not initialized) and the state will be updated to READY if initialization success, otherwise the state will return to REGISTERED.

参数:

- **handle** -- **\[in\]** I2S TX channel handler
- **pdm\_tx\_cfg** -- **\[in\]** Configurations for PDM TX mode, including clock, slot and GPIO The clock configuration can be generated by the helper macro `I2S_PDM_TX_CLK_DEFAULT_CONFIG` The slot configuration can be generated by the helper macro `I2S_PDM_TX_SLOT_DEFAULT_CONFIG`

返回:

- ESP\_OK Initialize successfully
- ESP\_ERR\_NO\_MEM No memory for storing the channel information
- ESP\_ERR\_INVALID\_ARG NULL pointer or invalid configuration
- ESP\_ERR\_INVALID\_STATE This channel is not registered

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_tx\_clock( handle, const \*clk\_cfg) [](#_CPPv433i2s_channel_reconfig_pdm_tx_clock17i2s_chan_handle_tPK23i2s_pdm_tx_clk_config_t "永久链接至目标")

Reconfigure the I2S clock for PDM TX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM TX mode, i.e., `i2s_channel_init_pdm_tx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S TX channel handler
- **clk\_cfg** -- **\[in\]** PDM TX mode clock configuration, can be generated by `I2S_PDM_TX_CLK_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_tx\_slot( handle, const \*slot\_cfg) [](#_CPPv432i2s_channel_reconfig_pdm_tx_slot17i2s_chan_handle_tPK24i2s_pdm_tx_slot_config_t "永久链接至目标")

Reconfigure the I2S slot for PDM TX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM TX mode, i.e., `i2s_channel_init_pdm_tx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S TX channel handler
- **slot\_cfg** -- **\[in\]** PDM TX mode slot configuration, can be generated by `I2S_PDM_TX_SLOT_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_NO\_MEM No memory for DMA buffer
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_pdm\_tx\_gpio( handle, const \*gpio\_cfg) [](#_CPPv432i2s_channel_reconfig_pdm_tx_gpio17i2s_chan_handle_tPK24i2s_pdm_tx_gpio_config_t "永久链接至目标")

Reconfigure the I2S GPIO for PDM TX mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to PDM TX mode, i.e., `i2s_channel_init_pdm_tx_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S TX channel handler
- **gpio\_cfg** -- **\[in\]** PDM TX mode GPIO configuration, specified by user

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not PDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

### Structures

struct i2s\_pdm\_rx\_slot\_config\_t [](#_CPPv424i2s_pdm_rx_slot_config_t "永久链接至目标")

I2S slot configuration for PDM RX mode.

Public Members

data\_bit\_width [](#_CPPv4N24i2s_pdm_rx_slot_config_t14data_bit_widthE "永久链接至目标")

I2S sample data bit width (valid data bits per sample), only support 16 bits for PDM mode

slot\_bit\_width [](#_CPPv4N24i2s_pdm_rx_slot_config_t14slot_bit_widthE "永久链接至目标")

I2S slot bit width (total bits per slot), only support 16 bits for PDM mode

slot\_mode [](#_CPPv4N24i2s_pdm_rx_slot_config_t9slot_modeE "永久链接至目标")

Set mono or stereo mode with I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

slot\_mask [](#_CPPv4N24i2s_pdm_rx_slot_config_t9slot_maskE "永久链接至目标")

Choose the slots to activate

data\_fmt [](#_CPPv4N24i2s_pdm_rx_slot_config_t8data_fmtE "永久链接至目标")

The data format of PDM RX mode. It determines what kind of data format is read in software. Typically, set this field to I2S\_PDM\_DATA\_FMT\_PCM when PCM2PDM filter is supported in the hardware, so that the hardware PDM2PCM filter will help to convert the raw PDM data on the line into PCM format, And then you can read PCM format data in software. Otherwise if this field is set to I2S\_PDM\_DATA\_FMT\_RAW, The data read in software are still in raw PDM format, you may need to convert the raw PDM data into PCM format manually by a software filter.

struct i2s\_pdm\_rx\_clk\_config\_t [](#_CPPv423i2s_pdm_rx_clk_config_t "永久链接至目标")

I2S clock configuration for PDM RX mode.

Public Members

uint32\_t sample\_rate\_hz [](#_CPPv4N23i2s_pdm_rx_clk_config_t14sample_rate_hzE "永久链接至目标")

I2S sample rate.

- For raw PDM mode, it typically ranges 1.024MHz ~ 6.144MHz.
- For PCM mode (PDM2PCM filter enabled), it usually ranges 16KHz ~ 48KHz

clk\_src [](#_CPPv4N23i2s_pdm_rx_clk_config_t7clk_srcE "永久链接至目标")

Choose clock source

mclk\_multiple [](#_CPPv4N23i2s_pdm_rx_clk_config_t13mclk_multipleE "永久链接至目标")

The multiple of MCLK to the sample rate

dn\_sample\_mode [](#_CPPv4N23i2s_pdm_rx_clk_config_t14dn_sample_modeE "永久链接至目标")

Down-sampling rate mode

uint32\_t bclk\_div [](#_CPPv4N23i2s_pdm_rx_clk_config_t8bclk_divE "永久链接至目标")

The division from MCLK to BCLK. The typical and minimum value is I2S\_PDM\_RX\_BCLK\_DIV\_MIN. It will be set to I2S\_PDM\_RX\_BCLK\_DIV\_MIN by default if it is smaller than I2S\_PDM\_RX\_BCLK\_DIV\_MIN

struct i2s\_pdm\_rx\_gpio\_config\_t [](#_CPPv424i2s_pdm_rx_gpio_config_t "永久链接至目标")

I2S PDM RX mode GPIO pins configuration.

Public Members

gpio\_num\_t clk [](#_CPPv4N24i2s_pdm_rx_gpio_config_t3clkE "永久链接至目标")

PDM clk pin, output

gpio\_num\_t din [](#_CPPv4N24i2s_pdm_rx_gpio_config_t3dinE "永久链接至目标")

DATA pin 0, input

gpio\_num\_t dins\[(4)\] [](#_CPPv4N24i2s_pdm_rx_gpio_config_t4dinsE "永久链接至目标")

DATA pins, input, only take effect when corresponding I2S\_PDM\_RX\_LINEx\_SLOT\_xxx is enabled in

uint32\_t clk\_inv [](#_CPPv4N24i2s_pdm_rx_gpio_config_t7clk_invE "永久链接至目标")

Set 1 to invert the clk output

struct invert\_flags [](#_CPPv4N24i2s_pdm_rx_gpio_config_t12invert_flagsE "永久链接至目标")

GPIO pin invert flags

struct i2s\_pdm\_rx\_config\_t [](#_CPPv419i2s_pdm_rx_config_t "永久链接至目标")

I2S PDM RX mode major configuration that including clock/slot/GPIO configuration.

Public Members

clk\_cfg [](#_CPPv4N19i2s_pdm_rx_config_t7clk_cfgE "永久链接至目标")

PDM RX clock configurations, can be generated by macro I2S\_PDM\_RX\_CLK\_DEFAULT\_CONFIG

slot\_cfg [](#_CPPv4N19i2s_pdm_rx_config_t8slot_cfgE "永久链接至目标")

PDM RX slot configurations, can be generated by macro I2S\_PDM\_RX\_SLOT\_RAW\_FMT\_DEFAULT\_CONFIG or I2S\_PDM\_RX\_SLOT\_PCM\_FMT\_DEFAULT\_CONFIG

gpio\_cfg [](#_CPPv4N19i2s_pdm_rx_config_t8gpio_cfgE "永久链接至目标")

PDM RX slot configurations, specified by user

struct i2s\_pdm\_tx\_slot\_config\_t [](#_CPPv424i2s_pdm_tx_slot_config_t "永久链接至目标")

I2S slot configuration for PDM TX mode.

Public Members

data\_bit\_width [](#_CPPv4N24i2s_pdm_tx_slot_config_t14data_bit_widthE "永久链接至目标")

I2S sample data bit width (valid data bits per sample), only support 16 bits for PDM mode

slot\_bit\_width [](#_CPPv4N24i2s_pdm_tx_slot_config_t14slot_bit_widthE "永久链接至目标")

I2S slot bit width (total bits per slot), only support 16 bits for PDM mode

slot\_mode [](#_CPPv4N24i2s_pdm_tx_slot_config_t9slot_modeE "永久链接至目标")

Set mono or stereo mode with I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO For PDM TX mode, mono means the data buffer only contains one slot data, Stereo means the data buffer contains two slots data

data\_fmt [](#_CPPv4N24i2s_pdm_tx_slot_config_t8data_fmtE "永久链接至目标")

The data format of PDM TX mode. It determines what kind of data format is written in software. Typically, set this field to I2S\_PDM\_DATA\_FMT\_PCM when PCM2PDM filter is supported in the hardware, so that you can write PCM format data in software, and then the hardware PCM2PDM filter will help to convert it into PDM format on the line. Otherwise if this field is set to I2S\_PDM\_DATA\_FMT\_RAW, The data written in software are supposed to be the raw PDM format.

uint32\_t sd\_prescale [](#_CPPv4N24i2s_pdm_tx_slot_config_t11sd_prescaleE "永久链接至目标")

Sigma-delta filter prescale

sd\_scale [](#_CPPv4N24i2s_pdm_tx_slot_config_t8sd_scaleE "永久链接至目标")

Sigma-delta filter scaling value

hp\_scale [](#_CPPv4N24i2s_pdm_tx_slot_config_t8hp_scaleE "永久链接至目标")

High pass filter scaling value

lp\_scale [](#_CPPv4N24i2s_pdm_tx_slot_config_t8lp_scaleE "永久链接至目标")

Low pass filter scaling value

sinc\_scale [](#_CPPv4N24i2s_pdm_tx_slot_config_t10sinc_scaleE "永久链接至目标")

Sinc filter scaling value

line\_mode [](#_CPPv4N24i2s_pdm_tx_slot_config_t9line_modeE "永久链接至目标")

PDM TX line mode, one-line codec, one-line dac, two-line dac mode can be selected

bool hp\_en [](#_CPPv4N24i2s_pdm_tx_slot_config_t5hp_enE "永久链接至目标")

High pass filter enable

float hp\_cut\_off\_freq\_hz [](#_CPPv4N24i2s_pdm_tx_slot_config_t18hp_cut_off_freq_hzE "永久链接至目标")

High pass filter cut-off frequency, range 23.3Hz ~ 185Hz, see cut-off frequency sheet above

uint32\_t sd\_dither [](#_CPPv4N24i2s_pdm_tx_slot_config_t9sd_ditherE "永久链接至目标")

Sigma-delta filter dither

uint32\_t sd\_dither2 [](#_CPPv4N24i2s_pdm_tx_slot_config_t10sd_dither2E "永久链接至目标")

Sigma-delta filter dither2

struct i2s\_pdm\_tx\_clk\_config\_t [](#_CPPv423i2s_pdm_tx_clk_config_t "永久链接至目标")

I2S clock configuration for PDM TX mode.

Public Members

uint32\_t sample\_rate\_hz [](#_CPPv4N23i2s_pdm_tx_clk_config_t14sample_rate_hzE "永久链接至目标")

I2S sample rate.

- For raw PDM mode, it typically ranges 1.024MHz ~ 6.144MHz.
- For PCM mode (PCM2PDM filter enabled), it usually ranges 16KHz ~ 48KHz

clk\_src [](#_CPPv4N23i2s_pdm_tx_clk_config_t7clk_srcE "永久链接至目标")

Choose clock source

mclk\_multiple [](#_CPPv4N23i2s_pdm_tx_clk_config_t13mclk_multipleE "永久链接至目标")

The multiple of MCLK to the sample rate

uint32\_t up\_sample\_fp [](#_CPPv4N23i2s_pdm_tx_clk_config_t12up_sample_fpE "永久链接至目标")

Up-sampling param fp

uint32\_t up\_sample\_fs [](#_CPPv4N23i2s_pdm_tx_clk_config_t12up_sample_fsE "永久链接至目标")

Up-sampling param fs, not allowed to be greater than 480

uint32\_t bclk\_div [](#_CPPv4N23i2s_pdm_tx_clk_config_t8bclk_divE "永久链接至目标")

The division from MCLK to BCLK. The minimum value is I2S\_PDM\_TX\_BCLK\_DIV\_MIN. It will be set to I2S\_PDM\_TX\_BCLK\_DIV\_MIN by default if it is smaller than I2S\_PDM\_TX\_BCLK\_DIV\_MIN

struct i2s\_pdm\_tx\_gpio\_config\_t [](#_CPPv424i2s_pdm_tx_gpio_config_t "永久链接至目标")

I2S PDM TX mode GPIO pins configuration.

Public Members

gpio\_num\_t clk [](#_CPPv4N24i2s_pdm_tx_gpio_config_t3clkE "永久链接至目标")

PDM clk pin, output

gpio\_num\_t dout [](#_CPPv4N24i2s_pdm_tx_gpio_config_t4doutE "永久链接至目标")

DATA pin, output

gpio\_num\_t dout2 [](#_CPPv4N24i2s_pdm_tx_gpio_config_t5dout2E "永久链接至目标")

The second data pin for the DAC dual-line mode, only take effect when the line mode is `I2S_PDM_TX_TWO_LINE_DAC`

uint32\_t clk\_inv [](#_CPPv4N24i2s_pdm_tx_gpio_config_t7clk_invE "永久链接至目标")

Set 1 to invert the clk output

struct invert\_flags [](#_CPPv4N24i2s_pdm_tx_gpio_config_t12invert_flagsE "永久链接至目标")

GPIO pin invert flags

struct i2s\_pdm\_tx\_config\_t [](#_CPPv419i2s_pdm_tx_config_t "永久链接至目标")

I2S PDM TX mode major configuration that including clock/slot/GPIO configuration.

Public Members

clk\_cfg [](#_CPPv4N19i2s_pdm_tx_config_t7clk_cfgE "永久链接至目标")

PDM TX clock configurations, can be generated by macro I2S\_PDM\_TX\_CLK\_DEFAULT\_CONFIG

slot\_cfg [](#_CPPv4N19i2s_pdm_tx_config_t8slot_cfgE "永久链接至目标")

PDM TX slot configurations, can be generated by macro I2S\_PDM\_TX\_SLOT\_DEFAULT\_CONFIG

gpio\_cfg [](#_CPPv4N19i2s_pdm_tx_config_t8gpio_cfgE "永久链接至目标")

PDM TX GPIO configurations, specified by user

### Macros

I2S\_PDM\_RX\_SLOT\_PCM\_FMT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_RX_SLOT_PCM_FMT_DEFAULT_CONFIG "永久链接至目标")

PDM format in 2 slots(RX). Read data in PCM format.

This file is specified for I2S PDM communication mode Features:

- Only support PDM TX/RX mode
- Fixed to 2 slots
- Data bit width only support 16 bits

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_RX\_SLOT\_RAW\_FMT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_RX_SLOT_RAW_FMT_DEFAULT_CONFIG "永久链接至目标")

PDM mode in 2 slots(RX). Read data in raw PDM format.

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_RX\_CLK\_DEFAULT\_CONFIG(rate) [](#c.I2S_PDM_RX_CLK_DEFAULT_CONFIG "永久链接至目标")

I2S default PDM RX clock configuration.

参数:

- **rate** -- sample rate

I2S\_PDM\_TX\_SLOT\_PCM\_FMT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_TX_SLOT_PCM_FMT_DEFAULT_CONFIG "永久链接至目标")

PDM style in 2 slots(TX) for codec line mode. Write PCM data.

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_TX\_SLOT\_RAW\_FMT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_TX_SLOT_RAW_FMT_DEFAULT_CONFIG "永久链接至目标")

PDM style in 2 slots(TX) for codec line mode. Write raw PDM data.

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_TX\_SLOT\_PCM\_FMT\_DAC\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_TX_SLOT_PCM_FMT_DAC_DEFAULT_CONFIG "永久链接至目标")

PDM style in 1 slots(TX) for DAC line mode.

> [!note] 备注
> The noise might be different with different configurations, this macro provides a set of configurations that have relatively high SNR (Signal Noise Ratio), you can also adjust them to fit your case.

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_TX\_SLOT\_RAW\_FMT\_DAC\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo) [](#c.I2S_PDM_TX_SLOT_RAW_FMT_DAC_DEFAULT_CONFIG "永久链接至目标")

PDM style in 1 slots(TX) for DAC line mode. Write raw PDM data.

> [!note] 备注
> The noise might be different with different configurations, this macro provides a set of configurations that have relatively high SNR (Signal Noise Ratio), you can also adjust them to fit your case.

参数:

- **bits\_per\_sample** -- I2S data bit width, only support 16 bits for PDM mode
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

I2S\_PDM\_TX\_CLK\_DEFAULT\_CONFIG(rate) [](#c.I2S_PDM_TX_CLK_DEFAULT_CONFIG "永久链接至目标")

I2S default PDM TX clock configuration for codec line mode.

> [!note] 备注
> TX PDM can only be set to the following two up-sampling rate configurations: 1: fp = 960, fs = sample\_rate\_hz / 100, in this case, Fpdm = 128\*48000 2: fp = 960, fs = 480, in this case, Fpdm = 128\*Fpcm = 128\*sample\_rate\_hz If the PDM receiver do not care the PDM serial clock, it's recommended set Fpdm = 128\*48000. Otherwise, the second configuration should be adopted.

参数:

- **rate** -- sample rate (not suggest to exceed 48000 Hz, otherwise more glitches and noise may appear)

I2S\_PDM\_TX\_CLK\_DAC\_DEFAULT\_CONFIG(rate) [](#c.I2S_PDM_TX_CLK_DAC_DEFAULT_CONFIG "永久链接至目标")

I2S default PDM TX clock configuration for DAC line mode.

> [!note] 备注
> TX PDM can only be set to the following two up-sampling rate configurations: 1: fp = 960, fs = sample\_rate\_hz / 100, in this case, Fpdm = 128\*48000 2: fp = 960, fs = 480, in this case, Fpdm = 128\*Fpcm = 128\*sample\_rate\_hz If the PDM receiver do not care the PDM serial clock, it's recommended set Fpdm = 128\*48000. Otherwise, the second configuration should be adopted.

> [!note] 备注
> The noise might be different with different configurations, this macro provides a set of configurations that have relatively high SNR (Signal Noise Ratio), you can also adjust them to fit your case.

参数:

- **rate** -- sample rate (not suggest to exceed 48000 Hz, otherwise more glitches and noise may appear)

### TDM 模式

### Header File

- [components/esp\_driver\_i2s/include/driver/i2s\_tdm.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_tdm.h)
- This header file can be included with:
	> ```c
	> #include "driver/i2s_tdm.h"
	> ```
- This header file is a part of the API provided by the `esp_driver_i2s` component. To declare that your component depends on `esp_driver_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_driver_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_driver_i2s
	> ```

### Functions

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_init\_tdm\_mode( handle, const \*tdm\_cfg) [](#_CPPv425i2s_channel_init_tdm_mode17i2s_chan_handle_tPK16i2s_tdm_config_t "永久链接至目标")

Initialize I2S channel to TDM mode.

> [!note] 备注
> Only allowed to be called when the channel state is REGISTERED, (i.e., channel has been allocated, but not initialized) and the state will be updated to READY if initialization success, otherwise the state will return to REGISTERED.

> [!note] 备注
> When initialize the TDM mode with a same configuration as another channel on a same port, these two channels can constitude as full-duplex mode automatically

参数:

- **handle** -- **\[in\]** I2S channel handler
- **tdm\_cfg** -- **\[in\]** Configurations for TDM mode, including clock, slot and GPIO The clock configuration can be generated by the helper macro `I2S_TDM_CLK_DEFAULT_CONFIG` The slot configuration can be generated by the helper macro `I2S_TDM_PHILIPS_SLOT_DEFAULT_CONFIG`, `I2S_TDM_PCM_SHORT_SLOT_DEFAULT_CONFIG`, `I2S_TDM_PCM_LONG_SLOT_DEFAULT_CONFIG` or `I2S_TDM_MSB_SLOT_DEFAULT_CONFIG`

返回:

- ESP\_OK Initialize successfully
- ESP\_ERR\_NO\_MEM No memory for storing the channel information
- ESP\_ERR\_INVALID\_ARG NULL pointer or invalid configuration
- ESP\_ERR\_INVALID\_STATE This channel is not registered

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_tdm\_clock( handle, const \*clk\_cfg) [](#_CPPv430i2s_channel_reconfig_tdm_clock17i2s_chan_handle_tPK20i2s_tdm_clk_config_t "永久链接至目标")

Reconfigure the I2S clock for TDM mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to TDM mode, i.e., `i2s_channel_init_tdm_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **clk\_cfg** -- **\[in\]** Standard mode clock configuration, can be generated by `I2S_TDM_CLK_DEFAULT_CONFIG`

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not TDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_tdm\_slot( handle, const \*slot\_cfg) [](#_CPPv429i2s_channel_reconfig_tdm_slot17i2s_chan_handle_tPK21i2s_tdm_slot_config_t "永久链接至目标")

Reconfigure the I2S slot for TDM mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to TDM mode, i.e., `i2s_channel_init_tdm_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **slot\_cfg** -- **\[in\]** Standard mode slot configuration, can be generated by `I2S_TDM_PHILIPS_SLOT_DEFAULT_CONFIG`, `I2S_TDM_PCM_SHORT_SLOT_DEFAULT_CONFIG`, `I2S_TDM_PCM_LONG_SLOT_DEFAULT_CONFIG` or `I2S_TDM_MSB_SLOT_DEFAULT_CONFIG`.

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_NO\_MEM No memory for DMA buffer
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not TDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_reconfig\_tdm\_gpio( handle, const \*gpio\_cfg) [](#_CPPv429i2s_channel_reconfig_tdm_gpio17i2s_chan_handle_tPK21i2s_tdm_gpio_config_t "永久链接至目标")

Reconfigure the I2S GPIO for TDM mode.

> [!note] 备注
> Only allowed to be called when the channel state is READY, i.e., channel has been initialized, but not started this function won't change the state. `i2s_channel_disable` should be called before calling this function if I2S has started.

> [!note] 备注
> The input channel handle has to be initialized to TDM mode, i.e., `i2s_channel_init_tdm_mode` has been called before reconfiguring

参数:

- **handle** -- **\[in\]** I2S channel handler
- **gpio\_cfg** -- **\[in\]** Standard mode GPIO configuration, specified by user

返回:

- ESP\_OK Set clock successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer, invalid configuration or not TDM mode
- ESP\_ERR\_INVALID\_STATE This channel is not initialized or not stopped

### Structures

struct i2s\_tdm\_slot\_config\_t [](#_CPPv421i2s_tdm_slot_config_t "永久链接至目标")

I2S slot configuration for TDM mode.

Public Members

data\_bit\_width [](#_CPPv4N21i2s_tdm_slot_config_t14data_bit_widthE "永久链接至目标")

I2S sample data bit width (valid data bits per sample)

slot\_bit\_width [](#_CPPv4N21i2s_tdm_slot_config_t14slot_bit_widthE "永久链接至目标")

I2S slot bit width (total bits per slot)

slot\_mode [](#_CPPv4N21i2s_tdm_slot_config_t9slot_modeE "永久链接至目标")

Set mono or stereo mode with I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO

slot\_mask [](#_CPPv4N21i2s_tdm_slot_config_t9slot_maskE "永久链接至目标")

Slot mask. Activating slots by setting 1 to corresponding bits. When the activated slots is not consecutive, those data in inactivated slots will be ignored

uint32\_t ws\_width [](#_CPPv4N21i2s_tdm_slot_config_t8ws_widthE "永久链接至目标")

WS signal width (i.e. the number of BCLK ticks that WS signal is high)

bool ws\_pol [](#_CPPv4N21i2s_tdm_slot_config_t6ws_polE "永久链接至目标")

WS signal polarity, set true to enable high lever first

bool bit\_shift [](#_CPPv4N21i2s_tdm_slot_config_t9bit_shiftE "永久链接至目标")

Set true to enable bit shift in Philips mode

bool left\_align [](#_CPPv4N21i2s_tdm_slot_config_t10left_alignE "永久链接至目标")

Set true to enable left alignment

bool big\_endian [](#_CPPv4N21i2s_tdm_slot_config_t10big_endianE "永久链接至目标")

Set true to enable big endian

bool bit\_order\_lsb [](#_CPPv4N21i2s_tdm_slot_config_t13bit_order_lsbE "永久链接至目标")

Set true to enable lsb first

bool skip\_mask [](#_CPPv4N21i2s_tdm_slot_config_t9skip_maskE "永久链接至目标")

Set true to enable skip mask. If it is enabled, only the data of the enabled channels will be sent, otherwise all data stored in DMA TX buffer will be sent

uint32\_t total\_slot [](#_CPPv4N21i2s_tdm_slot_config_t10total_slotE "永久链接至目标")

I2S total number of slots. If it is smaller than the biggest activated channel number, it will be set to this number automatically.

struct i2s\_tdm\_clk\_config\_t [](#_CPPv420i2s_tdm_clk_config_t "永久链接至目标")

I2S clock configuration for TDM mode.

Public Members

uint32\_t sample\_rate\_hz [](#_CPPv4N20i2s_tdm_clk_config_t14sample_rate_hzE "永久链接至目标")

I2S sample rate

clk\_src [](#_CPPv4N20i2s_tdm_clk_config_t7clk_srcE "永久链接至目标")

Choose clock source, see `soc_periph_i2s_clk_src_t` for the supported clock sources. selected `I2S_CLK_SRC_EXTERNAL` (if supports) to enable the external source clock inputted via MCLK pin, please make sure the frequency inputted is equal or greater than `sample_rate_hz * mclk_multiple`

uint32\_t ext\_clk\_freq\_hz [](#_CPPv4N20i2s_tdm_clk_config_t15ext_clk_freq_hzE "永久链接至目标")

External clock source frequency in Hz, only take effect when `clk_src = I2S_CLK_SRC_EXTERNAL`, otherwise this field will be ignored Please make sure the frequency inputted is equal or greater than BCLK, i.e. `sample_rate_hz * slot_bits * slot_num`

mclk\_multiple [](#_CPPv4N20i2s_tdm_clk_config_t13mclk_multipleE "永久链接至目标")

The multiple of MCLK to the sample rate, only take effect for master role

uint32\_t bclk\_div [](#_CPPv4N20i2s_tdm_clk_config_t8bclk_divE "永久链接至目标")

The division from MCLK to BCLK, only take effect for slave role, it shouldn't be smaller than 8. Increase this field when data sent by slave lag behind

struct i2s\_tdm\_gpio\_config\_t [](#_CPPv421i2s_tdm_gpio_config_t "永久链接至目标")

I2S TDM mode GPIO pins configuration.

Public Members

gpio\_num\_t mclk [](#_CPPv4N21i2s_tdm_gpio_config_t4mclkE "永久链接至目标")

MCK pin, output by default, input if the clock source is selected to `I2S_CLK_SRC_EXTERNAL`

gpio\_num\_t bclk [](#_CPPv4N21i2s_tdm_gpio_config_t4bclkE "永久链接至目标")

BCK pin, input in slave role, output in master role

gpio\_num\_t ws [](#_CPPv4N21i2s_tdm_gpio_config_t2wsE "永久链接至目标")

WS pin, input in slave role, output in master role

gpio\_num\_t dout [](#_CPPv4N21i2s_tdm_gpio_config_t4doutE "永久链接至目标")

DATA pin, output

gpio\_num\_t din [](#_CPPv4N21i2s_tdm_gpio_config_t3dinE "永久链接至目标")

DATA pin, input

uint32\_t mclk\_inv [](#_CPPv4N21i2s_tdm_gpio_config_t8mclk_invE "永久链接至目标")

Set 1 to invert the MCLK input/output

uint32\_t bclk\_inv [](#_CPPv4N21i2s_tdm_gpio_config_t8bclk_invE "永久链接至目标")

Set 1 to invert the BCLK input/output

uint32\_t ws\_inv [](#_CPPv4N21i2s_tdm_gpio_config_t6ws_invE "永久链接至目标")

Set 1 to invert the WS input/output

struct invert\_flags [](#_CPPv4N21i2s_tdm_gpio_config_t12invert_flagsE "永久链接至目标")

GPIO pin invert flags

struct i2s\_tdm\_config\_t [](#_CPPv416i2s_tdm_config_t "永久链接至目标")

I2S TDM mode major configuration that including clock/slot/GPIO configuration.

Public Members

clk\_cfg [](#_CPPv4N16i2s_tdm_config_t7clk_cfgE "永久链接至目标")

TDM mode clock configuration, can be generated by macro I2S\_TDM\_CLK\_DEFAULT\_CONFIG

slot\_cfg [](#_CPPv4N16i2s_tdm_config_t8slot_cfgE "永久链接至目标")

TDM mode slot configuration, can be generated by macros I2S\_TDM\_\[mode\]\_SLOT\_DEFAULT\_CONFIG, \[mode\] can be replaced with PHILIPS/MSB/PCM\_SHORT/PCM\_LONG

gpio\_cfg [](#_CPPv4N16i2s_tdm_config_t8gpio_cfgE "永久链接至目标")

TDM mode GPIO configuration, specified by user

### Macros

I2S\_TDM\_AUTO\_SLOT\_NUM [](#c.I2S_TDM_AUTO_SLOT_NUM "永久链接至目标")

This file is specified for I2S TDM communication mode Features:

- More than 2 slots

I2S\_TDM\_AUTO\_WS\_WIDTH [](#c.I2S_TDM_AUTO_WS_WIDTH "永久链接至目标")

I2S\_TDM\_PHILIPS\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo, mask) [](#c.I2S_TDM_PHILIPS_SLOT_DEFAULT_CONFIG "永久链接至目标")

Philips format in active slot that enabled by mask.

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO
- **mask** -- active slot mask

I2S\_TDM\_MSB\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo, mask) [](#c.I2S_TDM_MSB_SLOT_DEFAULT_CONFIG "永久链接至目标")

MSB format in active slot enabled that by mask.

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO
- **mask** -- active slot mask

I2S\_TDM\_PCM\_SHORT\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo, mask) [](#c.I2S_TDM_PCM_SHORT_SLOT_DEFAULT_CONFIG "永久链接至目标")

PCM(short) format in active slot that enabled by mask.

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO
- **mask** -- active slot mask

I2S\_TDM\_PCM\_LONG\_SLOT\_DEFAULT\_CONFIG(bits\_per\_sample, mono\_or\_stereo, mask) [](#c.I2S_TDM_PCM_LONG_SLOT_DEFAULT_CONFIG "永久链接至目标")

PCM(long) format in active slot that enabled by mask.

参数:

- **bits\_per\_sample** -- I2S data bit width
- **mono\_or\_stereo** -- I2S\_SLOT\_MODE\_MONO or I2S\_SLOT\_MODE\_STEREO
- **mask** -- active slot mask

I2S\_TDM\_CLK\_DEFAULT\_CONFIG(rate) [](#c.I2S_TDM_CLK_DEFAULT_CONFIG "永久链接至目标")

I2S default TDM clock configuration.

> [!note] 备注
> Please set the mclk\_multiple to I2S\_MCLK\_MULTIPLE\_384 while the data width in slot configuration is set to 24 bits Otherwise the sample rate might be imprecise since the BCLK division is not a integer

参数:

- **rate** -- sample rate

### I2S 驱动

### Header File

- [components/esp\_driver\_i2s/include/driver/i2s\_common.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_common.h)
- This header file can be included with:
	> ```c
	> #include "driver/i2s_common.h"
	> ```
- This header file is a part of the API provided by the `esp_driver_i2s` component. To declare that your component depends on `esp_driver_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_driver_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_driver_i2s
	> ```

### Functions

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_new\_channel(const \*chan\_cfg, \*ret\_tx\_handle, \*ret\_rx\_handle) [](#_CPPv415i2s_new_channelPK17i2s_chan_config_tP17i2s_chan_handle_tP17i2s_chan_handle_t "永久链接至目标")

Allocate new I2S channel(s)

> [!note] 备注
> The new created I2S channel handle will be REGISTERED state after it is allocated successfully.

> [!note] 备注
> When the port id in channel configuration is I2S\_NUM\_AUTO, driver will allocate I2S port automatically on one of the I2S controller, otherwise driver will try to allocate the new channel on the selected port.

> [!note] 备注
> If both tx\_handle and rx\_handle are not NULL, it means this I2S controller will work at full-duplex mode, the RX and TX channels will be allocated on a same I2S port in this case. Note that some configurations of TX/RX channel are shared on ESP32 and ESP32S2, so please make sure they are working at same condition and under same status(start/stop). Currently, full-duplex mode can't guarantee TX/RX channels write/read synchronously, they can only share the clock signals for now.

> [!note] 备注
> If tx\_handle OR rx\_handle is NULL, it means this I2S controller will work at simplex mode. For ESP32 and ESP32S2, the whole I2S controller (i.e. both RX and TX channel) will be occupied, even if only one of RX or TX channel is registered. For the other targets, another channel on this controller will still available.

参数:

- **chan\_cfg** -- **\[in\]** I2S controller channel configurations
- **ret\_tx\_handle** -- **\[out\]** I2S channel handler used for managing the sending channel(optional)
- **ret\_rx\_handle** -- **\[out\]** I2S channel handler used for managing the receiving channel(optional)

返回:

- ESP\_OK Allocate new channel(s) success
- ESP\_ERR\_NOT\_SUPPORTED The communication mode is not supported on the current chip
- ESP\_ERR\_INVALID\_ARG NULL pointer or illegal parameter in
- ESP\_ERR\_NOT\_FOUND No available I2S channel found

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_del\_channel( handle) [](#_CPPv415i2s_del_channel17i2s_chan_handle_t "永久链接至目标")

Delete the I2S channel.

> [!note] 备注
> Only allowed to be called when the I2S channel is at REGISTERED or READY state (i.e., it should stop before deleting it).

> [!note] 备注
> Resource will be free automatically if all channels in one port are deleted

参数:

**handle** -- **\[in\]** I2S channel handler

- ESP\_OK Delete successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_get\_info( handle, \*chan\_info) [](#_CPPv420i2s_channel_get_info17i2s_chan_handle_tP15i2s_chan_info_t "永久链接至目标")

Get I2S channel information.

参数:

- **handle** -- **\[in\]** I2S channel handler
- **chan\_info** -- **\[out\]** I2S channel basic information

返回:

- ESP\_OK Get I2S channel information success
- ESP\_ERR\_NOT\_FOUND The input handle doesn't match any registered I2S channels, it may not an I2S channel handle or not available any more
- ESP\_ERR\_INVALID\_ARG The input handle or chan\_info pointer is NULL

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_enable( handle) [](#_CPPv418i2s_channel_enable17i2s_chan_handle_t "永久链接至目标")

Enable the I2S channel.

> [!note] 备注
> Only allowed to be called when the channel state is READY, (i.e., channel has been initialized, but not started) the channel will enter RUNNING state once it is enabled successfully.

> [!note] 备注
> Enable the channel can start the I2S communication on hardware. It will start outputting BCLK and WS signal. For MCLK signal, it will start to output when initialization is finished

参数:

**handle** -- **\[in\]** I2S channel handler

- ESP\_OK Start successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer
- ESP\_ERR\_INVALID\_STATE This channel has not initialized or already started

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_disable( handle) [](#_CPPv419i2s_channel_disable17i2s_chan_handle_t "永久链接至目标")

Disable the I2S channel.

> [!note] 备注
> Only allowed to be called when the channel state is RUNNING, (i.e., channel has been started) the channel will enter READY state once it is disabled successfully.

> [!note] 备注
> Disable the channel can stop the I2S communication on hardware. It will stop BCLK and WS signal but not MCLK signal

参数:

**handle** -- **\[in\]** I2S channel handler

返回:

- ESP\_OK Stop successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer
- ESP\_ERR\_INVALID\_STATE This channel has not stated

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_write( handle, const void \*src, size\_t size, size\_t \*bytes\_written, uint32\_t timeout\_ms) [](#_CPPv417i2s_channel_write17i2s_chan_handle_tPKv6size_tP6size_t8uint32_t "永久链接至目标")

I2S write data.

> [!note] 备注
> Only allowed to be called when the channel state is RUNNING, (i.e., TX channel has been started and is not writing now) but the RUNNING only stands for the software state, it doesn't mean there is no the signal transporting on line.

参数:

- **handle** -- **\[in\]** I2S channel handler
- **src** -- **\[in\]** The pointer of sent data buffer
- **size** -- **\[in\]** Max data buffer length
- **bytes\_written** -- **\[out\]** Byte number that actually be sent, can be NULL if not needed
- **timeout\_ms** -- **\[in\]** Max block time

返回:

- ESP\_OK Write successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer or this handle is not TX handle
- ESP\_ERR\_TIMEOUT Writing timeout, no writing event received from ISR within ticks\_to\_wait
- ESP\_ERR\_INVALID\_STATE I2S is not ready to write

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_read( handle, void \*dest, size\_t size, size\_t \*bytes\_read, uint32\_t timeout\_ms) [](#_CPPv416i2s_channel_read17i2s_chan_handle_tPv6size_tP6size_t8uint32_t "永久链接至目标")

I2S read data.

> [!note] 备注
> Only allowed to be called when the channel state is RUNNING but the RUNNING only stands for the software state, it doesn't mean there is no the signal transporting on line.

参数:

- **handle** -- **\[in\]** I2S channel handler
- **dest** -- **\[in\]** The pointer of receiving data buffer
- **size** -- **\[in\]** Max data buffer length
- **bytes\_read** -- **\[out\]** Byte number that actually be read, can be NULL if not needed
- **timeout\_ms** -- **\[in\]** Max block time

返回:

- ESP\_OK Read successfully
- ESP\_ERR\_INVALID\_ARG NULL pointer or this handle is not RX handle
- ESP\_ERR\_TIMEOUT Reading timeout, no reading event received from ISR within ticks\_to\_wait
- ESP\_ERR\_INVALID\_STATE I2S is not ready to read

Set event callbacks for I2S channel.

> [!note] 备注
> Only allowed to be called when the channel state is REGISTERED / READY, (i.e., before channel starts)

> [!note] 备注
> User can deregister a previously registered callback by calling this function and setting the callback member in the `callbacks` structure to NULL.

> [!note] 备注
> When CONFIG\_I2S\_ISR\_IRAM\_SAFE is enabled, the callback itself and functions called by it should be placed in IRAM. The variables used in the function should be in the SRAM as well. The `user_data` should also reside in SRAM or internal RAM as well.

参数:

- **handle** -- **\[in\]** I2S channel handler
- **callbacks** -- **\[in\]** Group of callback functions
- **user\_data** -- **\[in\]** User data, which will be passed to callback functions directly

返回:

- ESP\_OK Set event callbacks successfully
- ESP\_ERR\_INVALID\_ARG Set event callbacks failed because of invalid argument
- ESP\_ERR\_INVALID\_STATE Set event callbacks failed because the current channel state is not REGISTERED or READY

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_preload\_data( tx\_handle, const void \*src, size\_t size, size\_t \*bytes\_loaded) [](#_CPPv424i2s_channel_preload_data17i2s_chan_handle_tPKv6size_tP6size_t "永久链接至目标")

Preload the data into TX DMA buffer.

> [!note] 备注
> Only allowed to be called when the channel state is READY, (i.e., channel has been initialized, but not started)

> [!note] 备注
> As the initial DMA buffer has no data inside, it will transmit the empty buffer after enabled the channel, this function is used to preload the data into the DMA buffer, so that the valid data can be transmitted immediately after the channel is enabled.

> [!note] 备注
> This function can be called multiple times before enabling the channel, the buffer that loaded later will be concatenated behind the former loaded buffer. But when all the DMA buffers have been loaded, no more data can be preload then, please check the `bytes_loaded` parameter to see how many bytes are loaded successfully, when the `bytes_loaded` is smaller than the `size`, it means the DMA buffers are full.

参数:

- **tx\_handle** -- **\[in\]** I2S TX channel handler
- **src** -- **\[in\]** The pointer of the source buffer to be loaded
- **size** -- **\[in\]** The source buffer size
- **bytes\_loaded** -- **\[out\]** The bytes that successfully been loaded into the TX DMA buffer

返回:

- ESP\_OK Load data successful
- ESP\_FAIL Failed to push the message queue
- ESP\_ERR\_INVALID\_ARG NULL pointer or not TX direction
- ESP\_ERR\_INVALID\_STATE This channel has not stated

[esp\_err\_t](https://docs.espressif.com/projects/esp-idf/zh_CN/v6.0.2/esp32s3/api-reference/system/esp_err.html#_CPPv49esp_err_t) i2s\_channel\_tune\_rate( handle, const \*tune\_cfg, \*tune\_info) [](#_CPPv421i2s_channel_tune_rate17i2s_chan_handle_tPK19i2s_tuning_config_tP17i2s_tuning_info_t "永久链接至目标")

Tune the I2S clock rate.

> [!note] 备注
> Only allowed to be called when the channel state is READY, (i.e., channel has been initialized, but not started)

> [!note] 备注
> This function is mainly to fine-tuning the mclk to match the speed of producer and consumer. So that to avoid exsaust of the memory to store the data from producer. Please take care the how different the frequency error can be tolerant by your codec, otherwise the codec might stop working if the frequency changes a lot.

参数:

- **handle** -- **\[in\]** I2S channel handler
- **tune\_cfg** -- **\[in\]** The clock tuning configuration, can be NULL if only need the current clock result
- **tune\_info** -- **\[out\]** The clock tuning information, can be NULL if not needed

返回:

- ESP\_OK Tune the clock successfully
- ESP\_ERR\_INVALID\_ARG Tune the clock failed because of the invalid argument like NULL pointer or out of range
- ESP\_ERR\_NOT\_SUPPORTED Tune the clock failed because this function does not support to tune the external clock source

### Structures

struct i2s\_event\_callbacks\_t [](#_CPPv421i2s_event_callbacks_t "永久链接至目标")

Group of I2S callbacks.

> [!note] 备注
> The callbacks are all running under ISR environment

> [!note] 备注
> When CONFIG\_I2S\_ISR\_IRAM\_SAFE is enabled, the callback itself and functions called by it should be placed in IRAM. The variables used in the function should be in the SRAM as well.

Public Members

on\_recv [](#_CPPv4N21i2s_event_callbacks_t7on_recvE "永久链接至目标")

Callback of data received event, only for RX channel The event data includes DMA buffer address and size that just finished receiving data

on\_recv\_q\_ovf [](#_CPPv4N21i2s_event_callbacks_t13on_recv_q_ovfE "永久链接至目标")

Callback of receiving queue overflowed event, only for RX channel The event data includes buffer size that has been overwritten

on\_sent [](#_CPPv4N21i2s_event_callbacks_t7on_sentE "永久链接至目标")

Callback of data sent event, only for TX channel The event data includes DMA buffer address and size that just finished sending data

on\_send\_q\_ovf [](#_CPPv4N21i2s_event_callbacks_t13on_send_q_ovfE "永久链接至目标")

Callback of sending queue overflowed event, only for TX channel The event data includes buffer size that has been overwritten

struct i2s\_chan\_config\_t [](#_CPPv417i2s_chan_config_t "永久链接至目标")

I2S controller channel configuration.

Public Members

int id [](#_CPPv4N17i2s_chan_config_t2idE "永久链接至目标")

I2S port id

role [](#_CPPv4N17i2s_chan_config_t4roleE "永久链接至目标")

I2S role, I2S\_ROLE\_MASTER or I2S\_ROLE\_SLAVE

uint32\_t dma\_desc\_num [](#_CPPv4N17i2s_chan_config_t12dma_desc_numE "永久链接至目标")

I2S DMA buffer number, it is also the number of DMA descriptor

uint32\_t dma\_frame\_num [](#_CPPv4N17i2s_chan_config_t13dma_frame_numE "永久链接至目标")

I2S frame number in one DMA buffer. One frame means one-time sample data in all slots, it should be the multiple of `3` when the data bit width is 24.

bool auto\_clear [](#_CPPv4N17i2s_chan_config_t10auto_clearE "永久链接至目标")

Alias of `auto_clear_after_cb`

bool auto\_clear\_after\_cb [](#_CPPv4N17i2s_chan_config_t19auto_clear_after_cbE "永久链接至目标")

Set to auto clear DMA TX buffer after `on_sent` callback, I2S will always send zero automatically if no data to send. So that user can assign the data to the DMA buffers directly in the callback, and the data won't be cleared after quit the callback.

bool auto\_clear\_before\_cb [](#_CPPv4N17i2s_chan_config_t20auto_clear_before_cbE "永久链接至目标")

Set to auto clear DMA TX buffer before `on_sent` callback, I2S will always send zero automatically if no data to send So that user can access data in the callback that just finished to send.

bool allow\_pd [](#_CPPv4N17i2s_chan_config_t8allow_pdE "永久链接至目标")

Set to allow power down. When this flag set, the driver will backup/restore the I2S registers before/after entering/exist sleep mode. By this approach, the system can power off I2S's power domain. This can save power, but at the expense of more RAM being consumed.

int intr\_priority [](#_CPPv4N17i2s_chan_config_t13intr_priorityE "永久链接至目标")

I2S interrupt priority, range \[0, 7\], if set to 0, the driver will try to allocate an interrupt with a relative low priority (1,2,3)

struct i2s\_chan\_info\_t [](#_CPPv415i2s_chan_info_t "永久链接至目标")

I2S channel information.

Public Members

int id [](#_CPPv4N15i2s_chan_info_t2idE "永久链接至目标")

I2S port id

role [](#_CPPv4N15i2s_chan_info_t4roleE "永久链接至目标")

I2S role, I2S\_ROLE\_MASTER or I2S\_ROLE\_SLAVE

dir [](#_CPPv4N15i2s_chan_info_t3dirE "永久链接至目标")

I2S channel direction

mode [](#_CPPv4N15i2s_chan_info_t4modeE "永久链接至目标")

I2S channel communication mode

bool is\_enabled [](#_CPPv4N15i2s_chan_info_t10is_enabledE "永久链接至目标")

I2S channel is enabled or not

pair\_chan [](#_CPPv4N15i2s_chan_info_t9pair_chanE "永久链接至目标")

I2S pair channel handle in duplex mode, always NULL in simplex mode

uint32\_t total\_dma\_buf\_size [](#_CPPv4N15i2s_chan_info_t18total_dma_buf_sizeE "永久链接至目标")

Total size of all the allocated DMA buffers

- 0 if the channel has not been initialized
- non-zero if the channel has been initialized

clk\_src [](#_CPPv4N15i2s_chan_info_t7clk_srcE "永久链接至目标")

Clock source of I2S

uint32\_t sclk\_hz [](#_CPPv4N15i2s_chan_info_t7sclk_hzE "永久链接至目标")

Source clock frequency

uint32\_t mclk\_hz [](#_CPPv4N15i2s_chan_info_t7mclk_hzE "永久链接至目标")

MCLK frequency

uint32\_t bclk\_hz [](#_CPPv4N15i2s_chan_info_t7bclk_hzE "永久链接至目标")

BCLK frequency

const void \*mode\_cfg [](#_CPPv4N15i2s_chan_info_t8mode_cfgE "永久链接至目标")

Mode configuration, it need to be casted to the corresponding type according to the communication mode

- I2S\_COMM\_MODE\_STD: i2s\_std\_config\_t\*
- I2S\_COMM\_MODE\_TDM: i2s\_tdm\_config\_t\*
- I2S\_COMM\_MODE\_PDM + I2S\_DIR\_RX: i2s\_pdm\_rx\_config\_t\*
- I2S\_COMM\_MODE\_PDM + I2S\_DIR\_TX: i2s\_pdm\_tx\_config\_t\*

### Macros

I2S\_CHANNEL\_DEFAULT\_CONFIG(i2s\_num, i2s\_role) [](#c.I2S_CHANNEL_DEFAULT_CONFIG "永久链接至目标")

get default I2S property

I2S\_GPIO\_UNUSED [](#c.I2S_GPIO_UNUSED "永久链接至目标")

Used in i2s\_gpio\_config\_t for signals which are not used

### I2S 类型

### Header File

- [components/esp\_driver\_i2s/include/driver/i2s\_types.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_driver_i2s/include/driver/i2s_types.h)
- This header file can be included with:
	> ```c
	> #include "driver/i2s_types.h"
	> ```
- This header file is a part of the API provided by the `esp_driver_i2s` component. To declare that your component depends on `esp_driver_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_driver_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_driver_i2s
	> ```

### Structures

struct lp\_i2s\_trans\_t [](#_CPPv414lp_i2s_trans_t "永久链接至目标")

LP I2S transaction type.

Public Members

void \*buffer [](#_CPPv4N14lp_i2s_trans_t6bufferE "永久链接至目标")

Pointer to buffer.

size\_t buflen [](#_CPPv4N14lp_i2s_trans_t6buflenE "永久链接至目标")

Buffer len, this should be in the multiple of 4.

size\_t received\_size [](#_CPPv4N14lp_i2s_trans_t13received_sizeE "永久链接至目标")

Received size.

struct i2s\_event\_data\_t [](#_CPPv416i2s_event_data_t "永久链接至目标")

Event structure used in I2S event queue.

Public Members

void \*dma\_buf [](#_CPPv4N16i2s_event_data_t7dma_bufE "永久链接至目标")

The first level pointer of DMA buffer that just finished sending or receiving for `on_recv` and `on_sent` callback NULL for `on_recv_q_ovf` and `on_send_q_ovf` callback

size\_t size [](#_CPPv4N16i2s_event_data_t4sizeE "永久链接至目标")

The buffer size of DMA buffer when success to send or receive, also the buffer size that dropped when queue overflow. It is related to the dma\_frame\_num and data\_bit\_width, typically it is fixed when data\_bit\_width is not changed.

struct i2s\_tuning\_config\_t [](#_CPPv419i2s_tuning_config_t "永久链接至目标")

I2S clock tuning configurations.

Public Members

tune\_mode [](#_CPPv4N19i2s_tuning_config_t9tune_modeE "永久链接至目标")

Tuning mode, which decides how to tune the MCLK with the tuning value

int32\_t tune\_mclk\_val [](#_CPPv4N19i2s_tuning_config_t13tune_mclk_valE "永久链接至目标")

Tuning value

int32\_t max\_delta\_mclk [](#_CPPv4N19i2s_tuning_config_t14max_delta_mclkE "永久链接至目标")

The maximum frequency that can be increased comparing to the initial MCLK freuqnecy

int32\_t min\_delta\_mclk [](#_CPPv4N19i2s_tuning_config_t14min_delta_mclkE "永久链接至目标")

The minimum frequency that can be decreased comparing to the initial MCLK freuqnecy

struct i2s\_tuning\_info\_t [](#_CPPv417i2s_tuning_info_t "永久链接至目标")

I2S clock tuning result.

Public Members

int32\_t curr\_mclk\_hz [](#_CPPv4N17i2s_tuning_info_t12curr_mclk_hzE "永久链接至目标")

The current MCLK frequency after tuned

int32\_t delta\_mclk\_hz [](#_CPPv4N17i2s_tuning_info_t13delta_mclk_hzE "永久链接至目标")

The current changed MCLK frequency comparing to the initial MCLK frequency

uint32\_t water\_mark [](#_CPPv4N17i2s_tuning_info_t10water_markE "永久链接至目标")

The water mark of the internal buffer, in percent

struct lp\_i2s\_evt\_data\_t [](#_CPPv417lp_i2s_evt_data_t "永久链接至目标")

Event data structure for LP I2S.

Public Members

trans [](#_CPPv4N17lp_i2s_evt_data_t5transE "永久链接至目标")

LP I2S transaction.

### Macros

I2S\_NUM\_0 [](#c.I2S_NUM_0 "永久链接至目标")

I2S controller port 0

I2S\_NUM\_1 [](#c.I2S_NUM_1 "永久链接至目标")

I2S controller port 1

I2S\_NUM\_2 [](#c.I2S_NUM_2 "永久链接至目标")

I2S controller port 2

I2S\_NUM\_AUTO [](#c.I2S_NUM_AUTO "永久链接至目标")

Select an available port automatically

### Type Definitions

typedef struct i2s\_channel\_obj\_t \*i2s\_chan\_handle\_t [](#_CPPv417i2s_chan_handle_t "永久链接至目标")

I2S channel object handle, the control unit of the I2S driver

typedef struct lp\_i2s\_channel\_obj\_t \*lp\_i2s\_chan\_handle\_t [](#_CPPv420lp_i2s_chan_handle_t "永久链接至目标")

I2S channel object handle, the control unit of the I2S driver

typedef bool (\*i2s\_isr\_callback\_t)( handle, \*event, void \*user\_ctx) [](#_CPPv418i2s_isr_callback_t "永久链接至目标")

I2S event callback.

Param handle:

**\[in\]** I2S channel handle, created from `i2s_new_channel()`

Param event:

**\[in\]** I2S event data

Param user\_ctx:

**\[in\]** User registered context, passed from `i2s_channel_register_event_callback()`

Return:

Whether a high priority task has been waken up by this callback function

typedef bool (\*lp\_i2s\_callback\_t)( handle, \*event, void \*user\_ctx) [](#_CPPv417lp_i2s_callback_t "永久链接至目标")

LP I2S event callback type.

Param handle:

**\[in\]** LP I2S channel handle

Param event:

**\[in\]** Event data

Param user\_ctx:

**\[in\]** User data

Return:

Whether a high priority task has been waken up by this callback function

### Enumerations

enum i2s\_comm\_mode\_t [](#_CPPv415i2s_comm_mode_t "永久链接至目标")

I2S controller communication mode.

*Values:*

enumerator I2S\_COMM\_MODE\_STD [](#_CPPv4N15i2s_comm_mode_t17I2S_COMM_MODE_STDE "永久链接至目标")

I2S controller using standard communication mode, support Philips/MSB/PCM format

enumerator I2S\_COMM\_MODE\_PDM [](#_CPPv4N15i2s_comm_mode_t17I2S_COMM_MODE_PDME "永久链接至目标")

I2S controller using PDM communication mode, support PDM output or input

enumerator I2S\_COMM\_MODE\_TDM [](#_CPPv4N15i2s_comm_mode_t17I2S_COMM_MODE_TDME "永久链接至目标")

I2S controller using TDM communication mode, support up to 16 slots per frame

enumerator I2S\_COMM\_MODE\_NONE [](#_CPPv4N15i2s_comm_mode_t18I2S_COMM_MODE_NONEE "永久链接至目标")

Unspecified I2S controller mode

enum i2s\_mclk\_multiple\_t [](#_CPPv419i2s_mclk_multiple_t "永久链接至目标")

The multiple of MCLK to sample rate.

> [!note] 备注
> MCLK is the minimum resolution of the I2S clock. Increasing mclk multiple can reduce the clock jitter of BCLK and WS, which is also useful for the codec that don't require MCLK but have strict requirement to BCLK. For the 24-bit slot width, please choose a multiple that can be divided by 3 (i.e. 24-bit compatible).

*Values:*

enumerator I2S\_MCLK\_MULTIPLE\_128 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_128E "永久链接至目标")

MCLK = sample\_rate \* 128

enumerator I2S\_MCLK\_MULTIPLE\_192 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_192E "永久链接至目标")

MCLK = sample\_rate \* 192 (24-bit compatible)

enumerator I2S\_MCLK\_MULTIPLE\_256 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_256E "永久链接至目标")

MCLK = sample\_rate \* 256

enumerator I2S\_MCLK\_MULTIPLE\_384 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_384E "永久链接至目标")

MCLK = sample\_rate \* 384 (24-bit compatible)

enumerator I2S\_MCLK\_MULTIPLE\_512 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_512E "永久链接至目标")

MCLK = sample\_rate \* 512

enumerator I2S\_MCLK\_MULTIPLE\_576 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_576E "永久链接至目标")

MCLK = sample\_rate \* 576 (24-bit compatible)

enumerator I2S\_MCLK\_MULTIPLE\_768 [](#_CPPv4N19i2s_mclk_multiple_t21I2S_MCLK_MULTIPLE_768E "永久链接至目标")

MCLK = sample\_rate \* 768 (24-bit compatible)

enumerator I2S\_MCLK\_MULTIPLE\_1024 [](#_CPPv4N19i2s_mclk_multiple_t22I2S_MCLK_MULTIPLE_1024E "永久链接至目标")

MCLK = sample\_rate \* 1024

enumerator I2S\_MCLK\_MULTIPLE\_1152 [](#_CPPv4N19i2s_mclk_multiple_t22I2S_MCLK_MULTIPLE_1152E "永久链接至目标")

MCLK = sample\_rate \* 1152 (24-bit compatible)

enum i2s\_tuning\_mode\_t [](#_CPPv417i2s_tuning_mode_t "永久链接至目标")

I2S clock tuning operation.

*Values:*

enumerator I2S\_TUNING\_MODE\_ADDSUB [](#_CPPv4N17i2s_tuning_mode_t22I2S_TUNING_MODE_ADDSUBE "永久链接至目标")

Add or subtract the tuning value based on the current clock

enumerator I2S\_TUNING\_MODE\_SET [](#_CPPv4N17i2s_tuning_mode_t19I2S_TUNING_MODE_SETE "永久链接至目标")

Set the tuning value to overwrite the current clock

enumerator I2S\_TUNING\_MODE\_RESET [](#_CPPv4N17i2s_tuning_mode_t21I2S_TUNING_MODE_RESETE "永久链接至目标")

Set the clock to the initial value

### Header File

- [components/esp\_hal\_i2s/include/hal/i2s\_types.h](https://github.com/espressif/esp-idf/blob/v6.0.2/components/esp_hal_i2s/include/hal/i2s_types.h)
- This header file can be included with:
	> ```c
	> #include "hal/i2s_types.h"
	> ```
- This header file is a part of the API provided by the `esp_hal_i2s` component. To declare that your component depends on `esp_hal_i2s`, add the following to your CMakeLists.txt:
	> ```
	> REQUIRES esp_hal_i2s
	> ```
	> 
	> or
	> 
	> ```
	> PRIV_REQUIRES esp_hal_i2s
	> ```

### Type Definitions

typedef int i2s\_clock\_src\_t [](#_CPPv415i2s_clock_src_t "永久链接至目标")

Define a default type to avoid compiling warnings

### Enumerations

enum i2s\_slot\_mode\_t [](#_CPPv415i2s_slot_mode_t "永久链接至目标")

I2S channel slot mode.

*Values:*

enumerator I2S\_SLOT\_MODE\_MONO [](#_CPPv4N15i2s_slot_mode_t18I2S_SLOT_MODE_MONOE "永久链接至目标")

I2S channel slot format mono, transmit same data in all slots for tx mode, only receive the data in the first slots for rx mode.

enumerator I2S\_SLOT\_MODE\_STEREO [](#_CPPv4N15i2s_slot_mode_t20I2S_SLOT_MODE_STEREOE "永久链接至目标")

I2S channel slot format stereo, transmit different data in different slots for tx mode, receive the data in all slots for rx mode.

enum i2s\_dir\_t [](#_CPPv49i2s_dir_t "永久链接至目标")

I2S channel direction.

*Values:*

enumerator I2S\_DIR\_RX [](#_CPPv4N9i2s_dir_t10I2S_DIR_RXE "永久链接至目标")

I2S channel direction RX

enumerator I2S\_DIR\_TX [](#_CPPv4N9i2s_dir_t10I2S_DIR_TXE "永久链接至目标")

I2S channel direction TX

enum i2s\_role\_t [](#_CPPv410i2s_role_t "永久链接至目标")

I2S controller role.

*Values:*

enumerator I2S\_ROLE\_MASTER [](#_CPPv4N10i2s_role_t15I2S_ROLE_MASTERE "永久链接至目标")

I2S controller master role, bclk and ws signal will be set to output

enumerator I2S\_ROLE\_SLAVE [](#_CPPv4N10i2s_role_t14I2S_ROLE_SLAVEE "永久链接至目标")

I2S controller slave role, bclk and ws signal will be set to input

enum i2s\_data\_bit\_width\_t [](#_CPPv420i2s_data_bit_width_t "永久链接至目标")

Available data bit width in one slot.

*Values:*

enumerator I2S\_DATA\_BIT\_WIDTH\_8BIT [](#_CPPv4N20i2s_data_bit_width_t23I2S_DATA_BIT_WIDTH_8BITE "永久链接至目标")

I2S channel data bit-width: 8

enumerator I2S\_DATA\_BIT\_WIDTH\_16BIT [](#_CPPv4N20i2s_data_bit_width_t24I2S_DATA_BIT_WIDTH_16BITE "永久链接至目标")

I2S channel data bit-width: 16

enumerator I2S\_DATA\_BIT\_WIDTH\_24BIT [](#_CPPv4N20i2s_data_bit_width_t24I2S_DATA_BIT_WIDTH_24BITE "永久链接至目标")

I2S channel data bit-width: 24

enumerator I2S\_DATA\_BIT\_WIDTH\_32BIT [](#_CPPv4N20i2s_data_bit_width_t24I2S_DATA_BIT_WIDTH_32BITE "永久链接至目标")

I2S channel data bit-width: 32

enum i2s\_slot\_bit\_width\_t [](#_CPPv420i2s_slot_bit_width_t "永久链接至目标")

Total slot bit width in one slot.

*Values:*

enumerator I2S\_SLOT\_BIT\_WIDTH\_AUTO [](#_CPPv4N20i2s_slot_bit_width_t23I2S_SLOT_BIT_WIDTH_AUTOE "永久链接至目标")

I2S channel slot bit-width equals to data bit-width

enumerator I2S\_SLOT\_BIT\_WIDTH\_8BIT [](#_CPPv4N20i2s_slot_bit_width_t23I2S_SLOT_BIT_WIDTH_8BITE "永久链接至目标")

I2S channel slot bit-width: 8

enumerator I2S\_SLOT\_BIT\_WIDTH\_16BIT [](#_CPPv4N20i2s_slot_bit_width_t24I2S_SLOT_BIT_WIDTH_16BITE "永久链接至目标")

I2S channel slot bit-width: 16

enumerator I2S\_SLOT\_BIT\_WIDTH\_24BIT [](#_CPPv4N20i2s_slot_bit_width_t24I2S_SLOT_BIT_WIDTH_24BITE "永久链接至目标")

I2S channel slot bit-width: 24

enumerator I2S\_SLOT\_BIT\_WIDTH\_32BIT [](#_CPPv4N20i2s_slot_bit_width_t24I2S_SLOT_BIT_WIDTH_32BITE "永久链接至目标")

I2S channel slot bit-width: 32

enum i2s\_pcm\_compress\_t [](#_CPPv418i2s_pcm_compress_t "永久链接至目标")

A/U-law decompress or compress configuration.

*Values:*

enumerator I2S\_PCM\_DISABLE [](#_CPPv4N18i2s_pcm_compress_t15I2S_PCM_DISABLEE "永久链接至目标")

Disable A/U law decompress or compress

enumerator I2S\_PCM\_A\_DECOMPRESS [](#_CPPv4N18i2s_pcm_compress_t20I2S_PCM_A_DECOMPRESSE "永久链接至目标")

A-law decompress

enumerator I2S\_PCM\_A\_COMPRESS [](#_CPPv4N18i2s_pcm_compress_t18I2S_PCM_A_COMPRESSE "永久链接至目标")

A-law compress

enumerator I2S\_PCM\_U\_DECOMPRESS [](#_CPPv4N18i2s_pcm_compress_t20I2S_PCM_U_DECOMPRESSE "永久链接至目标")

U-law decompress

enumerator I2S\_PCM\_U\_COMPRESS [](#_CPPv4N18i2s_pcm_compress_t18I2S_PCM_U_COMPRESSE "永久链接至目标")

U-law compress

enum i2s\_pdm\_data\_fmt\_t [](#_CPPv418i2s_pdm_data_fmt_t "永久链接至目标")

I2S PDM data format.

*Values:*

enumerator I2S\_PDM\_DATA\_FMT\_PCM [](#_CPPv4N18i2s_pdm_data_fmt_t20I2S_PDM_DATA_FMT_PCME "永久链接至目标")

PDM RX: Enable the hardware PDM to PCM filter to convert the inputted PDM data on the line into PCM format in software, so that the read data in software is PCM format data already, no need additional software filter. PCM data format is only available when PCM2PDM filter is supported in hardware. PDM TX: Enable the hardware PCM to PDM filter to convert the written PCM data in software into PDM format on the line, so that we only need to write the PCM data in software, no need to prepare raw PDM data in software. PCM data format is only available when PDM2PCM filter is supported in hardware.

enumerator I2S\_PDM\_DATA\_FMT\_RAW [](#_CPPv4N18i2s_pdm_data_fmt_t20I2S_PDM_DATA_FMT_RAWE "永久链接至目标")

PDM RX: Read the raw PDM data directly in software, without the hardware PDM to PCM filter. You may need a software PDM to PCM filter to convert the raw PDM data that read into PCM format. PDM TX: Write the raw PDM data directly in software, without the hardware PCM to PDM filter. You may need to prepare the raw PDM data in software to output the PDM format data on the line.

enum i2s\_pdm\_dsr\_t [](#_CPPv413i2s_pdm_dsr_t "永久链接至目标")

I2S PDM RX down-sampling mode.

*Values:*

enumerator I2S\_PDM\_DSR\_8S [](#_CPPv4N13i2s_pdm_dsr_t14I2S_PDM_DSR_8SE "永久链接至目标")

downsampling number is 8 for PDM RX mode

enumerator I2S\_PDM\_DSR\_16S [](#_CPPv4N13i2s_pdm_dsr_t15I2S_PDM_DSR_16SE "永久链接至目标")

downsampling number is 16 for PDM RX mode

enumerator I2S\_PDM\_DSR\_MAX [](#_CPPv4N13i2s_pdm_dsr_t15I2S_PDM_DSR_MAXE "永久链接至目标")

enum i2s\_pdm\_sig\_scale\_t [](#_CPPv419i2s_pdm_sig_scale_t "永久链接至目标")

pdm tx signal scaling mode

*Values:*

enumerator I2S\_PDM\_SIG\_SCALING\_DIV\_2 [](#_CPPv4N19i2s_pdm_sig_scale_t25I2S_PDM_SIG_SCALING_DIV_2E "永久链接至目标")

I2S TX PDM signal scaling: /2

enumerator I2S\_PDM\_SIG\_SCALING\_MUL\_1 [](#_CPPv4N19i2s_pdm_sig_scale_t25I2S_PDM_SIG_SCALING_MUL_1E "永久链接至目标")

I2S TX PDM signal scaling: x1

enumerator I2S\_PDM\_SIG\_SCALING\_MUL\_2 [](#_CPPv4N19i2s_pdm_sig_scale_t25I2S_PDM_SIG_SCALING_MUL_2E "永久链接至目标")

I2S TX PDM signal scaling: x2

enumerator I2S\_PDM\_SIG\_SCALING\_MUL\_4 [](#_CPPv4N19i2s_pdm_sig_scale_t25I2S_PDM_SIG_SCALING_MUL_4E "永久链接至目标")

I2S TX PDM signal scaling: x4

enum i2s\_pdm\_tx\_line\_mode\_t [](#_CPPv422i2s_pdm_tx_line_mode_t "永久链接至目标")

PDM TX line mode.

> [!note] 备注
> For the standard codec mode, PDM pins are connect to a codec which requires both clock signal and data signal For the DAC output mode, PDM data signal can be connected to a power amplifier directly with a low-pass filter, normally, DAC output mode doesn't need the clock signal.

*Values:*

enumerator I2S\_PDM\_TX\_ONE\_LINE\_CODEC [](#_CPPv4N22i2s_pdm_tx_line_mode_t25I2S_PDM_TX_ONE_LINE_CODECE "永久链接至目标")

Standard PDM format output, left and right slot data on a single line

enumerator I2S\_PDM\_TX\_ONE\_LINE\_DAC [](#_CPPv4N22i2s_pdm_tx_line_mode_t23I2S_PDM_TX_ONE_LINE_DACE "永久链接至目标")

PDM DAC format output, left or right slot data on a single line

enumerator I2S\_PDM\_TX\_TWO\_LINE\_DAC [](#_CPPv4N22i2s_pdm_tx_line_mode_t23I2S_PDM_TX_TWO_LINE_DACE "永久链接至目标")

PDM DAC format output, left and right slot data on separated lines

enum i2s\_std\_slot\_mask\_t [](#_CPPv419i2s_std_slot_mask_t "永久链接至目标")

I2S slot select in standard mode.

> [!note] 备注
> It has different meanings in tx/rx/mono/stereo mode, and it may have different behaviors on different targets For the details, please refer to the I2S API reference

*Values:*

enumerator I2S\_STD\_SLOT\_LEFT [](#_CPPv4N19i2s_std_slot_mask_t17I2S_STD_SLOT_LEFTE "永久链接至目标")

I2S transmits or receives left slot

enumerator I2S\_STD\_SLOT\_RIGHT [](#_CPPv4N19i2s_std_slot_mask_t18I2S_STD_SLOT_RIGHTE "永久链接至目标")

I2S transmits or receives right slot

enumerator I2S\_STD\_SLOT\_BOTH [](#_CPPv4N19i2s_std_slot_mask_t17I2S_STD_SLOT_BOTHE "永久链接至目标")

I2S transmits or receives both left and right slot

enum i2s\_pdm\_slot\_mask\_t [](#_CPPv419i2s_pdm_slot_mask_t "永久链接至目标")

I2S slot select in PDM mode.

*Values:*

enumerator I2S\_PDM\_SLOT\_RIGHT [](#_CPPv4N19i2s_pdm_slot_mask_t18I2S_PDM_SLOT_RIGHTE "永久链接至目标")

I2S PDM only transmits or receives the PDM device whose 'select' pin is pulled up

enumerator I2S\_PDM\_SLOT\_LEFT [](#_CPPv4N19i2s_pdm_slot_mask_t17I2S_PDM_SLOT_LEFTE "永久链接至目标")

I2S PDM only transmits or receives the PDM device whose 'select' pin is pulled down

enumerator I2S\_PDM\_SLOT\_BOTH [](#_CPPv4N19i2s_pdm_slot_mask_t17I2S_PDM_SLOT_BOTHE "永久链接至目标")

I2S PDM transmits or receives both two slots

enumerator I2S\_PDM\_RX\_LINE0\_SLOT\_RIGHT [](#_CPPv4N19i2s_pdm_slot_mask_t27I2S_PDM_RX_LINE0_SLOT_RIGHTE "永久链接至目标")

I2S PDM receives the right slot on line 0

enumerator I2S\_PDM\_RX\_LINE0\_SLOT\_LEFT [](#_CPPv4N19i2s_pdm_slot_mask_t26I2S_PDM_RX_LINE0_SLOT_LEFTE "永久链接至目标")

I2S PDM receives the left slot on line 0

enumerator I2S\_PDM\_RX\_LINE1\_SLOT\_RIGHT [](#_CPPv4N19i2s_pdm_slot_mask_t27I2S_PDM_RX_LINE1_SLOT_RIGHTE "永久链接至目标")

I2S PDM receives the right slot on line 1

enumerator I2S\_PDM\_RX\_LINE1\_SLOT\_LEFT [](#_CPPv4N19i2s_pdm_slot_mask_t26I2S_PDM_RX_LINE1_SLOT_LEFTE "永久链接至目标")

I2S PDM receives the left slot on line 1

enumerator I2S\_PDM\_RX\_LINE2\_SLOT\_RIGHT [](#_CPPv4N19i2s_pdm_slot_mask_t27I2S_PDM_RX_LINE2_SLOT_RIGHTE "永久链接至目标")

I2S PDM receives the right slot on line 2

enumerator I2S\_PDM\_RX\_LINE2\_SLOT\_LEFT [](#_CPPv4N19i2s_pdm_slot_mask_t26I2S_PDM_RX_LINE2_SLOT_LEFTE "永久链接至目标")

I2S PDM receives the left slot on line 2

enumerator I2S\_PDM\_RX\_LINE3\_SLOT\_RIGHT [](#_CPPv4N19i2s_pdm_slot_mask_t27I2S_PDM_RX_LINE3_SLOT_RIGHTE "永久链接至目标")

I2S PDM receives the right slot on line 3

enumerator I2S\_PDM\_RX\_LINE3\_SLOT\_LEFT [](#_CPPv4N19i2s_pdm_slot_mask_t26I2S_PDM_RX_LINE3_SLOT_LEFTE "永久链接至目标")

I2S PDM receives the left slot on line 3

enumerator I2S\_PDM\_LINE\_SLOT\_ALL [](#_CPPv4N19i2s_pdm_slot_mask_t21I2S_PDM_LINE_SLOT_ALLE "永久链接至目标")

I2S PDM receives all slots

enum i2s\_tdm\_slot\_mask\_t [](#_CPPv419i2s_tdm_slot_mask_t "永久链接至目标")

tdm slot number

> [!note] 备注
> Multiple slots in TDM mode. For TX module, only the active slot send the audio data, the inactive slot send a constant or will be skipped if 'skip\_msk' is set. For RX module, only receive the audio data in active slots, the data in inactive slots will be ignored. the bit map of active slot can not exceed (0x1<<total\_slot\_num). e.g: slot\_mask = (I2S\_TDM\_SLOT0 | I2S\_TDM\_SLOT3), here the active slot number is 2 and total\_slot is not supposed to be smaller than 4.

*Values:*

enumerator I2S\_TDM\_SLOT0 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT0E "永久链接至目标")

I2S slot 0 enabled

enumerator I2S\_TDM\_SLOT1 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT1E "永久链接至目标")

I2S slot 1 enabled

enumerator I2S\_TDM\_SLOT2 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT2E "永久链接至目标")

I2S slot 2 enabled

enumerator I2S\_TDM\_SLOT3 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT3E "永久链接至目标")

I2S slot 3 enabled

enumerator I2S\_TDM\_SLOT4 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT4E "永久链接至目标")

I2S slot 4 enabled

enumerator I2S\_TDM\_SLOT5 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT5E "永久链接至目标")

I2S slot 5 enabled

enumerator I2S\_TDM\_SLOT6 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT6E "永久链接至目标")

I2S slot 6 enabled

enumerator I2S\_TDM\_SLOT7 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT7E "永久链接至目标")

I2S slot 7 enabled

enumerator I2S\_TDM\_SLOT8 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT8E "永久链接至目标")

I2S slot 8 enabled

enumerator I2S\_TDM\_SLOT9 [](#_CPPv4N19i2s_tdm_slot_mask_t13I2S_TDM_SLOT9E "永久链接至目标")

I2S slot 9 enabled

enumerator I2S\_TDM\_SLOT10 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT10E "永久链接至目标")

I2S slot 10 enabled

enumerator I2S\_TDM\_SLOT11 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT11E "永久链接至目标")

I2S slot 11 enabled

enumerator I2S\_TDM\_SLOT12 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT12E "永久链接至目标")

I2S slot 12 enabled

enumerator I2S\_TDM\_SLOT13 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT13E "永久链接至目标")

I2S slot 13 enabled

enumerator I2S\_TDM\_SLOT14 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT14E "永久链接至目标")

I2S slot 14 enabled

enumerator I2S\_TDM\_SLOT15 [](#_CPPv4N19i2s_tdm_slot_mask_t14I2S_TDM_SLOT15E "永久链接至目标")

I2S slot 15 enabled

enum i2s\_etm\_event\_type\_t [](#_CPPv420i2s_etm_event_type_t "永久链接至目标")

I2S channel events that supported by the ETM module.

*Values:*

enumerator I2S\_ETM\_EVENT\_DONE [](#_CPPv4N20i2s_etm_event_type_t18I2S_ETM_EVENT_DONEE "永久链接至目标")

Trigger condition: TX: no data to send in the TX FIFO, i.e., DMA need to stop (next desc is NULL) RX: 1. If rx\_stop\_mode = 0, this event will trigger when DMA is stopped (next desc is NULL)

1. If rx\_stop\_mode = 1, this event will trigger when DMA in\_suc\_eof.
2. If rx\_stop\_mode = 2, this event will trigger when RX FIFO is full. Event that I2S TX or RX stopped

enumerator I2S\_ETM\_EVENT\_REACH\_THRESH [](#_CPPv4N20i2s_etm_event_type_t26I2S_ETM_EVENT_REACH_THRESHE "永久链接至目标")

Trigger condition: TX: the sent words(in 32-bit) number reach the threshold that configured in `etm_tx_send_word_num` RX: the received words(in 32-bit) number reach the threshold that configured in `etm_rx_receive_word_num` and `etm_rx_receive_word_num` should be smaller than the size of the DMA buffer in one `in_suc_eof` event. Event that the I2S sent or received data reached the threshold

enum i2s\_etm\_task\_type\_t [](#_CPPv419i2s_etm_task_type_t "永久链接至目标")

I2S channel tasks that supported by the ETM module.

*Values:*

enumerator I2S\_ETM\_TASK\_START [](#_CPPv4N19i2s_etm_task_type_t18I2S_ETM_TASK_STARTE "永久链接至目标")

Start the I2S channel