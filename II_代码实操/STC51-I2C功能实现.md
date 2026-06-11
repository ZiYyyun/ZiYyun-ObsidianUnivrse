#实操/开发/嵌入式/51

> [!NOTE] 函数结构
> 参考 IIC时序图 写出IIC通讯信号
> ![[IIC函数封装.png]]

### I2C初始化
参考:通讯时序图[[Prot-IIC#起始]]
```c
void Dri_IIC_Start()
{
	SCL = 1;
	SDA = 1;
	SDA = 0;
	SCL = 0;
}
```

### 发送字节

参考:通讯时序图[[Prot-IIC#发送一个字节]]

> 注:一个`Byte(字节)`是8`bit(比特)`

```c
void Dri_IIC_SendByte(u8 byte)
{
	u8 i;
	//高位先行（大部分情况下的芯片）
	for (i = 0; i < 8; i++) {
		SDA = (byte & (0x80 >> i)) == 0 ? 0 : 1;    //给SDA赋值：判断形参byte的第i位是否为0，若是则0，若不是则赋1
	}
	SCL = 1;                                        //拉高SCL，把数据发出去
	SCL = 0;                                        //拉低SCL，准备发下一位
}
```

注：`0x80`即`1000 0000`，为了判断这个字节的最高位是不是`1`

### 发送确认信号

```c
void Dri_IIC_SendAck(bit ack)
{
    SDA = ack;                                      //把传进来的形参赋给SDA
    SCL = 1;                                        
    SCL = 0;
}
```
发送确认信号时，要求`SCL`应为高电平,参考[[Prot-IIC#确认信号]]

### 接收确认信号

```c
void Dri_IIC_ReceiveACK()
{
	bit ack = 0;
	SDA = 1;                       //释放SDA控制
	SCL = 1;                       //拉高SCL，准备接收
	ack = SDA;
	SCL = 0;
	return ack;
}
```



### 接收一个字节

```c
u8 Dri_IIC_ReceiveByte()
{
    u8 byte = 0, i = ;
    SDA = 1;                                    //释放SDA的控制权
    for (i = 0; i < 8; i++) {
        byte <<= 1;                             //左移byte
        SCL = 1;                                //拉高SCL，采集SDA数据
        if(SDA == 1) { byte |= 0x01; }          //如果SDA是1，则给byte最低位赋1
        SCL = 0;                                //拉低SCL，采集完成
    }
    return byte;
}
```
> Q：为什么SDA赋1就释放了控制权？
> A：因为IIC内部为开漏输出模式，且电路板上有个上拉电阻。当SDA等于1时，内部MOS管关断，引脚悬空，电平完全取决于外部上拉电阻。



### 发送停止信号

参考：[[Prot-IIC#结束]]
```c
void Dri_IIC_Stop()

{
    SDA = 0;
    SCL = 1;
    SDA = 1;
}
```

