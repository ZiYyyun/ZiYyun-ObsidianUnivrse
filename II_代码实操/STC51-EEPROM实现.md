#实操/开发/嵌入式/51 


> [!NOTE] 开发目标
> 实现EEPROM的读写
> ![[EEPROM部分原理图.png]]
> 


### 写函数
```c
bit Int_EEPROM_WriteBytes(u8 address, u8 *bytes, u8 len)
{
    bit ack = 0;
    u8 page_remain;

    while (len > 0)
    {
        // 计算当前页所剩的存储单元个数
        page_remain = PAGE_SIZE - address % PAGE_SIZE;

        // 判断当前页空间是否足够
        if (len > page_remain) // 当前页空间不够，需要分多个页写入 
        {
            // 先写入当前页
            ack |= Int_EEPROM_WriteIntoPage(address, bytes, page_remain);
            // 地址后移
            address += page_remain;
            // bytes指针后移
            bytes += page_remain;
            // len减去已经写入的个数
            len -= page_remain;
            // 延时3ms，EEPROM的写周期
            Delayms(3);
        }
        else // 当前页空间足够，只需写入到一页中 
        {
            ack |= Int_EEPROM_WriteIntoPage(address, bytes, len);
            len = 0;
        }
    }
    return ack;
}
```
根据IIC通讯流程，每发送一个字节前需要传入两个参数以及两次接收`ACK`，IIC外设调用流程参考[[IC-EEPROM#操作]]

### 读函数

```c
bit Int_EEPROM_ReadByte(u8 addr, u8 *bytes, u8 len){
    u8 i,len;
    Dri_IIC_Start();
    Dri_IIC_ReceiveACK();
    Dri_IIC_SendByte(addr);
    Dri_IIC_ReceiveACK();

    //发送起始信号
    Dri_IIC_Start();
    Dri_IIC_SendByte(0xA1);
    Dri_IIC_ReceiveACK();

    for (i = 0; i < len; i++)
    {
        Dri_IIC_SendByte(*bytes);
        Dri_IIC_SendACK(i == (len-1) ? 1 : 0);  // 当i自增到最后一次时，i就等于len-1(因为i是从0开始的)
    }
}
```


### 将数据写入到一个页中
```c
static bit Int_EEPROM_WriteIntoPage(u8 address, u8 *bytes, u8 len)
{
    u8 i;
    bit ack = 0;

    // 发送起始信号
    Dri_IIC_Start();

    // 发送从设备地址和写标识并接收应答信号
    Dri_IIC_SendByte(DEV_ADDR);
    ack |= Dri_IIC_ReceiveACK();

    // 发送EEPROM内部的字地址(存储单元的地址)并接收应答信号
    Dri_IIC_SendByte(address);
    ack |= Dri_IIC_ReceiveACK();

    // 循环，逐个字节发送
    for (i = 0; i < len; i++)
    {
        Dri_IIC_SendByte(bytes[i]);
        ack |= Dri_IIC_ReceiveACK();
    }
    // 发送结束信号
    Dri_IIC_Stop();
    return ack;
}
```