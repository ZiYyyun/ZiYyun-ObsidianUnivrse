#实操/开发/嵌入式/STM32/项目/牛马定位器  #IC 
### 低功耗模式
参考[[IC-AT6558R-卫星定位#低功耗模式]]
[[III_资源仓库/原理图/牛马定位器_原理图.pdf#page=1&selection=GPS-EN|GPS-EN]]引脚在PB3，所以我们通过控制PB3引脚的高低电平来实现低功耗模式的开关

>开启低功耗（即引脚拉低）

	void Int_AT6558R_EnterLP(void)
```c
HAL_GPIO_WritePin(GPS_EN_GPIO_Port, GPS_EN_Pin, GPIO_PIN_RESET);
```

>关闭低功耗（引脚拉高）

	void Int_AT6558R_LeaveLP(void)
```c
HAL_GPIO_WritePin(GPS_EN_GPIO_Port, GPS_EN_Pin, GPIO_PIN_SET);
```

---

### 全局变量与宏
```c
#define MAXSIZE 1024
uint8_t gps_buffers[MAXSIZE];
uint16_t gps_receive_length;
```

> [!info] Q：为什么 MAXSIZE 是 1024？
> AT6558R 输出的 NMEA 语句单条最长约 80 字节，但采用空闲中断 + 循环接收时，缓存需要容纳连续多帧数据。1024 是兼顾 RAM 空间和一次中断接收量的经验值，防止溢出。

---

### 初始化
	void Int_AT6558R_Init(void)
初始化 AT6558R，发送测试字符串，配置输出语句和频率，并启动 UART 空闲中断接收。
参考：[[IC-AT-6558R-卫星定位#初始化]]
>通过 UART1 打印初始化测试信息
```c
HAL_UART_Transmit(&huart1, (uint8_t *)"init test OK\r\n", 13, 500);
```

>使能 GPS 模块（拉高 EN 引脚）

参考：[[IC-AT-6558R-卫星定位#使能]]
```c
HAL_GPIO_WritePin(GPS_EN_GPIO_Port, GPS_EN_Pin, AT6558R_SET);
```

>配置只输出 RMC 语句（PCAS02）和输出频率为 2Hz（PCAS04）
```c
Int_AT6558R_SendCustomMessage("PCAS02,500");
Int_AT6558R_SendCustomMessage("PCAS04,2");
```

> [!note]
> `PCAS02` 设置 NMEA 输出语句组合，`500` 代表只输出 `$GPRMC`；`PCAS04` 设置输出频率，`2` 表示 2Hz。

>启动 UART2 空闲中断接收，循环直至成功
```c
HAL_StatusTypeDef status = HAL_ERROR;
while (status != HAL_OK)
{
    status = HAL_UARTEx_ReceiveToIdle_IT(&huart2, gps_buffers, MAXSIZE);
}
```

> [!attention]
> 必须通过空闲中断持续接收 GPS 模块的串口数据，保证后续能解析。

---

### 发送自定义配置指令
	static void Int_AT6558R_SendCustomMessage(uint8_t *cmd)
将 AT6558R 私有协议指令（如 PCAS02,500）封装为带校验的完整帧，通过 UART1 发送。

>计算异或校验值（从指令首字节开始依次异或）
```c
uint8_t custom_array[20] = {0};
uint8_t ch_0 = cmd[0];
for (uint8_t i = 1; cmd[i] != '\0'; i++)
{
    ch_0 ^= cmd[i];
}
```

>拼接帧格式 "$cmd*XX"，XX 为校验值十六进制大写
```c
sprintf(custom_array, "$%s*%02X", cmd, ch_0);
```

>通过 UART1 发送
```c
HAL_UART_Transmit(&huart1, custom_array, strlen((char *)custom_array), 1000);
```

> [!info]
> 指令格式参照 AT6558R 手册，例如发送 `PCAS04,2` 会被封装为 `$PCAS04,2*XX\r\n`。

---

### 中断回调
	void Int_AT6558R_CallBack(uint16_t gps_length)
UART 空闲中断回调，记录接收长度并重新启动接收。

>保存本次接收长度
```c
gps_receive_length = gps_length;
```

>再次调用接收接口，保证持续接收
```c
HAL_StatusTypeDef status = HAL_ERROR;
while (status != HAL_OK)
{
    status = HAL_UARTEx_ReceiveToIdle_IT(&huart2, gps_buffers, MAXSIZE);
}
```

---

### 获取 GPS 数据
	void Int_AT6558R_GetGPS(uint8_t receive_buffers[], uint16_t *length)
将中断缓冲区中的 GPS 原始数据拷贝到用户缓冲区，并清空内部缓存。

>初始化传出参数，清空用户缓冲区
```c
*length = 0;
memset(receive_buffers, 0, strlen((char *)receive_buffers));
```

>如果内部缓冲区有数据，则拷贝
```c
if (gps_receive_length > 0)
{
    memcpy(receive_buffers, gps_buffers, strlen((char *)gps_buffers));
    *length = gps_receive_length;

    // 清空内部缓冲区和长度，准备下一次接收
    gps_receive_length = 0;
    memset(gps_buffers, 0, strlen((char *)gps_buffers));
}
```

> [!warning]
> 调用该方法后内部数据会被清空，必须在回调产生后尽快取出，否则数据可能丢失。