#理论/嵌入式 #Prot  #SPI

> SPI（Serial Peripheral Interface）是一种同步串行通信协议，用于主设备与外设之间的高速数据传输。它由四条信号线构成


> [!NOTE] 注意
> SPI的CS（片选）引脚通常为**低电平有效**，所以我们在配置SPI的时候默认把CS电平设置为高



#### 功能引脚

| 名称   | 描述                  | 用途       |
| ---- | ------------------- | -------- |
| MISO | Master In Slave Out | 主设备接收数据  |
| MOSI | Master Out Slave In | 主设备发送数据  |
| SCK  | Serial Clock        | 控制数据传输节奏 |
| SS   | Slave Select        | 指示特定的从设备 |
 > 其中`MOSI`、`MISO`、`SCK`分别链接至所有设备，每个`NSS`连接一个设备。需要和某个设备通讯时，拉低对应的NSS引脚即可实现
 
```c
 typedef struct {
  SPI_TypeDef                *Instance;  // SPI 实例（如 SPI1、SPI2）
  SPI_InitTypeDef            Init;       // 初始化配置
  uint8_t                    *pTxBuffPtr;// 发送缓冲区指针
  uint16_t                   TxXferSize; // 发送数据大小
  // ...（其他字段）
} SPI_HandleTypeDef;

// SPI_InitTypeDef 关键参数：
typedef struct {
  uint32_t Mode;               // 主/从模式 SPI_MODE_MASTER 或 SPI_MODE_SLAVE
  uint32_t Direction;          // 数据传输方向（全双工/半双工等）
  uint32_t DataSize;           // 数据长度：SPI_DATASIZE_8BIT 或 16BIT
  uint32_t CLKPolarity;        // CPOL
  uint32_t CLKPhase;           // CPHA
  uint32_t NSS;                // 片选模式（硬件自动/软件控制）
  uint32_t BaudRatePrescaler;  // 时钟分频（决定 SCK 频率）
  uint32_t FirstBit;           // 数据传输顺序（MSB 或 LSB 先行）
} SPI_InitTypeDef;
```

#### 核心参数
##### 主从模式
>主模式下，SCK引脚产生串行时钟。从模式下，SCK引脚只负责接收


##### 时钟极性

##### 时钟相位

##### 波特率

##### 片选控制模式



