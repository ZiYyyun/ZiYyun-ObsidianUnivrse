#实操/开发/嵌入式/STM32 



### 初始化流程
1.时钟使能
- GPIOB 模块时钟使能
 - I2C2 模块时钟使能
2.GPIO模式配置
- PB10、PB11 都设置为复用开漏输出
3.I2C 配置
- 选择I2C模式或SMBus模式
- 设置I2C的时钟频率, 最大36MHz
- 选择为标准模式（0:标准模式; 1:快速模式）
- 设置标准模式下传输速度为100KHz
- 设置允许的SCL的最大上升时间
- I2C2模块使能


### 实现函数
```c
void Dri_I2C_Init(void);
void Dri_I2C_Start(void);
void Dri_I2C_Stop(void);
void Dri_I2C_SendAddr(uint8_t addr);
void Dri_I2C_SendByte(uint8_t byte);
uint8_t Dri_I2C_ReceiveByte(void);
void Dri_I2C_SendAck(uint8_t ack);
```


### 初始化
#### 配置GPIO与SMBUS模式
`SCL` 与 `SDA` 对应的引脚分别是`GPIOB10` `GPIOB11`，当前是IIC的硬件实现，故引脚应配置为复用开漏输出
```c
	//使能APB1和APB2总线
    RCC->APB2ENR |= RCC_APB2ENR_IOPBEN ;
    RCC->APB1ENR |= RCC_APB1ENR_I2C2EN;
	//配置引脚输出模式
    GPIOB->CRH |= (GPIO_CRH_MODE10 | GPIO_CRH_MODE11 | GPIO_CRH_CNF10 | GPIO_CRH_CNF11 );
    // 设置当前硬件用于I2C , 默认就是用于I2C
    I2C2->CR1 &=~ I2C_CR1_SMBUS ;   //寄存器位于I2C_CR1_SMBUS位。用于配置I2C模式/SMBUS模式
```

#### 配置I2C频率及传输模式
```c
    // 设置FREQ[5:0]：I2C模块时钟频率  - 100100 - 36MHz
    I2C2->CR2 |= 0x24 ;
    // 设置标准传输模式 - 默认就是标准
    I2C2->CCR &=~ I2C_CCR_FS ;
    // 标准速率是：100kbit/s , 因此传输1bit需要10us。标准模式高电平占50%，因此高电平时长就是5us。
    I2C2->CCR |= 180 ;
```


### 接收数据

其实对于硬件I2C模式来说，收发操作很简单，在硬件I2C模式中，已经帮我们省去了很多步骤，当**接收到数据之后**，我们只负责直接从`DR`寄存器中读取就好了。
```c
byte = I2C2->DR;
```
但是我们要如何确认可以接收完成了呢？
依靠I2C状态寄存器`SR1`的标志位`RXNE(Receive Not Empty)`标志位
```c
while((I2C2->CR1 & I2C_SR1_RXNE) && timeout) { timeout--; }
```
别忘了加入超时防卡死

### I2C 起始信号

> 功能：发送 I2C 协议规定的起始信号（SCL 高电平时 SDA 拉低），通知从机开始通信

硬件自动生成时序，软件仅需触发并等待完成运行

```c
void Dri_I2C_Start(void){
    u16 timeout = UINT16_MAX;
    // 向硬件发送起始信号指令，置位START位
    I2C2->CR1 |= I2C2_CR1_START;
    // 等待起始信号生成完成：检测SR1的SB(起始标志)位置1
    while (!(I2C2->SR1 & I2C_SR1_SB) && timeout) { timeout--; }    
}
```

核心：START 位触发硬件生成信号，**SB=1**代表起始信号发送完成

### I2C 停止信号

> 功能：发送 I2C 协议规定的停止信号（SCL 高电平时 SDA 拉高），结束本次通信

硬件自动完成时序，无需软件控制引脚

```c
void Dri_I2C_Stop(void){
    // 置位STOP位，硬件自动生成停止信号
    I2C2->CR1 |= I2C_CR1_STOP ;
}
```

核心：仅需操作`CR1`的 STOP 位，发送完成后硬件自动释放总线
### I2C 发送从机地址
> 功能：发送 7 位设备地址 + 读写位，等待从机应答，是 I2C 通信的必备步骤


```c
void Dri_I2C_SendAddr(uint8_t addr){
    // 将地址写入DR寄存器，硬件自动发送
    I2C2->DR = addr ;
    // 等待地址发送完成+从机应答：检测ADDR标志位置1
    uint16_t timeout = UINT16_MAX ;
    while( !( I2C2->SR1 & I2C_SR1_ADDR ) && timeout){
        timeout--;
    }
    // 未超时则读取SR2，硬件自动清除ADDR标志位（必须操作）
    if(timeout){
        I2C2->SR2;
    }
}
```

核心：

1. **ADDR=1** = 地址发送完成 + 收到从机 ACK
2. 必须读`SR2`清标志，否则 I2C 总线会卡死

### I2C 发送单字节数据

> 功能：向 I2C 从机发送 1 个字节数据，硬件自动完成时序与移位

```c
void Dri_I2C_SendByte(uint8_t byte){
    uint16_t timeout = UINT16_MAX ;
    // 等待发送寄存器为空：TXE=1代表可以写入新数据
    while( ((I2C2->SR1 & I2C_SR1_TXE )==0) && timeout){
        timeout--;
    }
    // 将数据写入DR寄存器，硬件自动发送
    I2C2->DR = byte ;
    // 等待数据发送完成：BTF=1代表字节发送完毕
    timeout = UINT16_MAX;
    while( (( I2C2->SR1 & I2C_SR1_BTF )==0) && timeout){
        timeout--;
    }
}
```

核心：

1. **TXE**：数据寄存器空标志，写数据前必须等待
2. **BTF**：字节发送完成标志，发送后必须等待

### I2C 发送应答信号

> 功能：配置接收数据后的应答位（ACK/NACK），通知从机是否继续接收


```c
void Dri_I2C_SendAck(uint8_t ack){
    // 配置规则：1-NACK  0-ACK（与硬件默认定义相反）
    if(ack){
        // 关闭应答，发送NACK
        I2C2->CR1 &=~ I2C_CR1_ACK ;
    }else{
        // 开启应答，发送ACK
        I2C2->CR1 |= I2C_CR1_ACK ;
    }
}
```

核心：

1. **ACK 位**：控制硬件是否返回应答信号
2. 接收最后 1 个字节时发 NACK，中间字节发 ACK