#实操/开发/嵌入式/STM32


### 实现函数
```c
void Dri_I2C_Init(void);
void Dri_I2C_Start(void);
void Dri_I2C_Stop(void);
void Dri_I2C_SendByte(uint8_t byte);
uint8_t Dri_I2C_ReceiveByte(void);
uint8_t Dri_I2C_ReceiveAck(void);
void Dri_I2C_SendAck(uint8_t ack);
```


### 初始化
#### 宏定义

```c i2c.c
#define SCL_LOW     (GPIOB->ODR &=~ GPIO_ODR_ODR10)
#define SCL_HIGH    (GPIOB->ODR |=  GPIO_ODR_ODR10)
#define SDA_LOW     (GPIOB->ODR &=~ GPIO_ODR_ODR11)
#define SDA_HIGH    (GPIOB->ODR |=  GPIO_ODR_ODR11)
#define READ_SDA    (GPIOB->IDR & GPIO_IDR_IDR11)
#define I2C_DELAY   Delay_us(10)
```
#### 初始化
> 参考：[[Prot-IIC#初始化操作（软件模拟）]]
```c i2c.c
void Dri_I2C_Init(void){
    RCC->APB2ENR |= RCC_APB2ENR_IOPBEN ;
    GPIOB->CRH |= (GPIO_CRH_MODE10 | GPIO_CRH_MODE11 | GPIO_CRH_CNF10_0 | GPIO_CRH_CNF11_0 );
    GPIOB->CRH &=~(GPIO_CRH_CNF10_1 | GPIO_CRH_CNF11_1) ;
    SDA_HIGH ;
    SCL_HIGH ;
    I2C_DELAY;
}
```
> 在尚硅谷STM32开发板中，IIC对应引脚为`PB10`与`PB11`，由于IIC协议需要，引脚需被配置为[[STM32-GPIO输入输出模式#开漏输出(GPIO_Mode_Out_OD)]]模式

#### 开始
> 参考：[[Prot-IIC#起始]]
```c
void Dri_I2C_Start(void){
    SCL_HIGH;
    SDA_HIGH;
    I2C_DELAY;
    SDA_LOW;

    SCL_LOW;
    I2C_DELAY;
}
```

#### 停止
> 参考：[[Prot-IIC#结束]]
```c
void Dri_I2C_Stop(void){
    SDA_LOW;
    I2C_DELAY;
    SCL_HIGH;
    I2C_DELAY;
    SDA_HIGH;
    I2C_DELAY;
}
```

#### 发送字节
> 参考：[[Prot-IIC#发送]]
```c
void Dri_I2C_SendByte(uint8_t byte){
    for (uint8_t i = 0; i < 8; i++){
        ((byte & 0x80)==0) ? SDA_LOW : SDA_HIGH ;
        I2C_DELAY;
        SCL_HIGH;
        I2C_DELAY;

        SCL_LOW;
        I2C_DELAY;
        byte<<=1;
    }
}
```


#### 接收字节
> 参考：[[Prot-IIC#读取数据]]
```c
uint8_t Dri_I2C_ReceiveByte(void){
    uint8_t byte = 0 ;

    SDA_HIGH ;
    I2C_DELAY ;

    for (uint8_t i = 0; i < 8; i++){
        byte <<= 1 ;

        SCL_HIGH;
        I2C_DELAY;

        if(READ_SDA){
            byte |= 0x01;
        }

        SCL_LOW;
        I2C_DELAY ;
    }
    return byte ;
}
```


#### 接收确认（ACK）信号
> 参考：[[Prot-IIC#确认信号]]
```c
uint8_t Dri_I2C_ReceiveAck(){
    uint8_t ack = 0 ;
    SDA_HIGH ;
    I2C_DELAY;

    SCL_HIGH;
    I2C_DELAY;

    ack = READ_SDA ? 1 : 0 ;

    SCL_LOW ;
    I2C_DELAY ;

    return ack ;
}
```



#### 发送确认（ACK）信号
> 参考：[[II_代码实操/STM32-IIC-Register#发送确认（ACK）信号]]
```c
void Dri_I2C_SendAck(uint8_t ack){
    ack ? SDA_HIGH : SDA_LOW ;
    SCL_HIGH ;
    I2C_DELAY ;

    SCL_LOW;
    I2C_DELAY ;
}
```