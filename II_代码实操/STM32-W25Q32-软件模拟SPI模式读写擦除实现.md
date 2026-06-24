#实操/开发/嵌入式/STM32 




### 初始化
	void Dri_W25Q32_Init()
这个函数没啥好说的，直接初始化`IIC`就行
```c
Dri_IIC_Init();
```

### 空闲确认
	void Dri_W25Q32_WaitNotBusy(void)
```c
Dri_SPI_Start();
Dri_SPI_SwapByte();
```

>发送读取状态指令
```c
Dri_SPI_SwapByte(0x05);
```

>读取状态
```c
while( Dri_SPI_SwapByte(0x00) & 0x01 );
```
> [!PDF]
> [[winbond-W25Q32JVSSIQ_datasheet_WJ323642.pdf#page=27&selection=The BUSY bit is a 1 during the Write Status Register cycle and a 0 when the cycle is finished and ready to accept other instructions again.|C179173_NOR+FLASH_W25Q32JVSSIQ_规格书_WJ323642 第27页]]
> 为什么要读`0x01`这个位呢？因为手册里有写，这个是`BUSY`位
> The BUSY bit is a 1 during the Write Status Register cycle and a 0 when the cycle is finished and ready to accept other instructions again. 
> 

>结束通信
```c
Dri_SPI_Stop();
```


### 读取ID
	Int_W25Q32_ReadId(uint8_t, uint16_t *did)

```c
Int_W25Q32_WaitNotBusy();
Dri_SPI_Start();
```

>发送功能位，读取ID
```c
Dri_SPI_SwapByte(0x9F);
```
参考芯片手册，`0x9F`位代表读取ID



### 扇区擦除
	void Dri_SPI_SectorErase(u32 addr)

> [!重要] Q：为什么addr要声明为32位
> 因为`W25Q32`的功能位最低是24位，但是C语言只有 u8 u16 u32 和 u64，没有 u24，所以只能选择u32

参考 数据手册 sectorErase的功能位是`0x20`
>同上，略
```c
    Int_W25Q32_WaitNotBusy();
    Int_W25Q32_WriteEnable();
    Dri_SPI_Start();
    Dri_SPI_SwapByte(0x20);
```

>清除
```c
    u8 block = (addr >> 16) & 0x00FF;
    Dri_SPI_SwapByte(block);
```


这段代码什么意思呢？拆开来看，先看 addr >> 16 
```c
addr      22222222 11111111 11111111 11111111
目标：              ||||||||
```
也就是说我们这行代码是为了清除我标注的这八位，所以先把addr进行右移16位，得到：
```c
addr      00000000 00000000 22222222 11111111
目标：                                ||||||||
```
此时我们再进行与运算，因为`0xFF`的值为





### 写数据




### 读数据




