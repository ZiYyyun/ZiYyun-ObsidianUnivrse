

### 硬件选型

> [!QUESTION] 主控

主控是ESP32-S3：
- 最高主频：240M
- RAM：512KB
- Flash：8MB
提供蓝牙、WiFi功能

> [!QUESTION] 音频

芯片：ES8311
控制信号通信：I2C
音频数据通信：I2S

- 采样格式：PCM
- 采样率：16000Hz
- 采样精度：16位

> [!QUESTION] 功率放大器

- 型号：NS4150B

> [!QUESTION] IPS屏幕

- 型号：ST7789V
- 分辨率：320x240
- 调用库：lvgl移植


> [!NOTE] 项目串讲

### 运行过程
#### 激活
1. 通过蓝牙配网，连接WiFi
2. 向官网发送http激活请求，官网返回http响应数据，解析响应数据判断是否有激活码
		- 如果需要激活，激活码会显示在屏幕上
		- 如果没有激活码，表示已经激活
#### 语音交互

```
麦克风-> ES8311 -> Task1 -> SR语音识别 -> Task2 -> 环形缓冲区 -> Task3 -> 音频编码 -> 环形缓冲区 -> Task4 -> WebSocket -> 服务器
```

```
扬声器 <- ES8311 <- 音频解码 <- Task5 <- 环形缓冲区 <- 服务器
```

#### 设备状态（枚举）
- IDLE
- SPEAKING
- LISTING

#### Task2
1. 唤醒词检测回调
- 设备状态 = IDLE
		连接websocket
		发送hello
- 设备状态 = SPEAKING
		发送种植
	发送唤醒词唤醒小智
- 状态变化回调
		






我的项目是基于 ESP32-S3 的智能语音聊天机器人，主要实现了蓝牙配网、语音交互、LCD显示AI回复以及IoT设备控制。

硬件平台采用 ESP32-S3 双核处理器：

- 主频240MHz
- Flash 8MB
- SRAM 512KB
- 外扩PSRAM用于缓存大量数据

外围设备：
### 音频部分

采用：

ES8311 音频Codec + 功放
其中：
- I2C负责配置ES8311寄存器
- I2S负责PCM音频数据传输

### 显示部分
采用：
ST7789 LCD
- 通过 SPI通信
- 使用LVGL负责UI绘制。

软件基于 ESP-IDF + FreeRTOS

采用多任务设计：
包括：
- 音频采集任务
- 语音识别任务
- 音频编码任务
- WebSocket通信任务
- 音频播放任务
- LVGL显示任务

整体数据流程：
```
麦克风
 |
ES8311
 |
I2S
 |
SR语音识别
 |
PCM缓存
 |
Opus编码
 |
WebSocket
 |
服务器AI模型
 |
WebSocket返回
 |
Opus解码
 |
ES8311
 |
扬声器
```


#### 设备状态

```
IDLE
 |
 | 唤醒词
 ↓
LISTENING
 |
 | 上传语音
 ↓
SPEAKING
 |
 | AI回复
 ↓
IDLE
```