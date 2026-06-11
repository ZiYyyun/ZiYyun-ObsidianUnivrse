#实操/开发/嵌入式/STM32 


#### 初始化
> 参考：这里直接初始化IIC
```c
Dri_I2C_Init();
```
#### 写位
> 参考：[[IC-M24C02-EEPROM#]]
```c
    //1.发送起始信号
    Dri_I2C_Start();
    //2.发送设备地址
    Dri_I2C_SendByte(DEV);
    //3.等待ack
    Dri_I2C_ReceiveAck();
    //4.发送字地址
    Dri_I2C_SendByte(addr);
    //5.等待ack
    Dri_I2C_ReceiveAck();
    //6.发送数据字节
    Dri_I2C_SendByte(byte);
    //7.等待ack
    Dri_I2C_ReceiveAck();
    //8.停止信号
    Dri_I2C_Stop();

    //9.进入内部写周期
    Delay_ms(5);
```
#### 读数据
> 参考：[[IC-M24C02-EEPROM#读操作]]
```c
    //1.起始信号
    Dri_I2C_Start();
    //2.发送设备地址
    Dri_I2C_SendByte(DEV);
    //3.接收ack
    Dri_I2C_ReceiveAck();
    //4.发送字地址
    Dri_I2C_SendByte(addr);
    //5.接收ack
    Dri_I2C_ReceiveAck();
    //6.起始信号
    Dri_I2C_Start();
    //7.发送设备地址
    Dri_I2C_SendByte(DEV+1);
    //8.接收ack
    Dri_I2C_ReceiveAck();
    //9.接收数据字节
    uint8_t byte = Dri_I2C_ReceiveByte();
    //10.发送NACK
    Dri_I2C_SendAck(NACK);
    //11.停止信号
    Dri_I2C_Stop();

    return byte ;
```
#### 页写（连续写入）
> 参考：[[IC-M24C02-EEPROM#页写]]
```c
    Dri_I2C_Start();
    Dri_I2C_SendByte(DEV);
    Dri_I2C_ReceiveAck();
    Dri_I2C_SendByte(addr);
    Dri_I2C_ReceiveAck();
    for (uint8_t i = 0; i < len; i++){
        Dri_I2C_SendByte(bytes[i]);
        Dri_I2C_ReceiveAck();
    }
    Dri_I2C_Stop();

    Delay_ms(5);
```
#### 连续读取
> 参考：[[IC-M24C02-EEPROM#连续读（批量读取多个字节）]]
```c
    Dri_I2C_Start();
    Dri_I2C_SendByte(DEV);
    Dri_I2C_ReceiveAck();
    Dri_I2C_SendByte(addr);
    Dri_I2C_ReceiveAck();
    Dri_I2C_Start();
    Dri_I2C_SendByte(DEV+1);
    Dri_I2C_ReceiveAck();
    for (uint8_t i = 0; i < len; i++){
        bytes[i] = Dri_I2C_ReceiveByte();
        Dri_I2C_SendAck(i==(len-1));
    }
    Dri_I2C_Stop();
```