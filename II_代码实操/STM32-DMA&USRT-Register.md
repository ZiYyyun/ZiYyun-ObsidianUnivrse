目标：编写DMA驱动，实现初始化、发送和接收功能
涉及寄存器：参考[[STM32-DMA#寄存器分类]]


> [!NOTE] 关键
> - **DMA 通道与外设映射**：需查阅 STM32 参考手册的 “DMA 请求映射” 表，确保外设（如 USART1_TX/RX）与 DMA 通道正确对应。
> - **数据宽度一致性**：`MSIZE`和`PSIZE`需与外设（如 USART 是 8 位数据）匹配，否则可能导致数据错位。
> - **传输完成后处理**：若需重新开启 DMA 传输，需先失能通道（`EN=0`），再重新配置`CNDTR`等寄存器。
> - **传输完成后处理**：若需重新开启 DMA 传输，需先失能通道（`EN=0`），再重新配置`CNDTR`等寄存器。
> - **中断优先级**：若使用 DMA 中断，需在 NVIC 中使能对应通道的中断，并设置合适的优先级。


### 初始化函数

使能RCC时钟，DMA挂在AHBENR总线上
```c
RCC->AHBENR |= RCC_AHBENR_DMA1EN;
```


```c
    DMA1_Channel4->CCR &= ~DMA_CCR4_MEM2MEM;
    DMA1_Channel4->CCR |= DMA_CCR4_PL;
    DMA1_Channel4->CCR &= ~(DMA_CCR4_MSIZE | DMA_CCR4_PSIZE);
    DMA1_Channel4->CCR |= DMA_CCR4_MINC;
    DMA1_Channel4->CCR &= ~DMA_CCR4_PINC;
    DMA1_Channel4->CCR &= ~DMA_CCR4_CIRC;
    DMA1_Channel4->CCR |= DMA_CCR4_DIR;
```
配置`CCR`寄存器，
1. 存储器到存储器模式
2. 通道优先级
3. 储存器数据宽度 外设数据宽度
4. 存储地址增量模式
5. 外设地址增量模式
6. 循环模式
7. 数据传输方向
### 发送函数
	void Dri_DMA1_Transmit(uint32_t srcAddr, uint32_t destAddr, uint16_t len)
```c
    USART1->CR3 |= USART_CR3_DMAT;
    DMA1_Channel4->CMAR = srcAddr;
    DMA1_Channel4->CPAR = destAddr;
    DMA1_Channel4->CNDTR = len;

    DMA1_Channel4->CCR |= DMA_CCR4_EN;
```
1. 配置`CR3`寄存器，使能`USART1`DMA发送
2. 配置`DMA1_Channel4`的`CCR`寄存器，开启通道
我们的发送函数总共需要三个参数，`srcAddr`、`destAddr`和`len`，分别对应了：**存储器地址、外设地址和字长**。

### 接收函数
	void Dri_DMA1_Receive(uint32_t srcAddr , uint32_t destAddr , uint16_t len)
接收函数与发送函数类似，只是配置`USART`时需使能接收，另外注意接收和发送的引脚不一样，对应Channel也不一样。
```c
    // USART的DMA传输默认是关闭的。需要使能
    USART1->CR3 |= USART_CR3_DMAR ;
    // 设置存储器地址
    DMA1_Channel5->CMAR = destAddr ;
    // 设置外设地址
    DMA1_Channel5->CPAR = srcAddr ;
    // 设置传输长度
    DMA1_Channel5->CNDTR = len ;
    // 一定要提前准备好数据才能使能
    DMA1_Channel5->CCR |= DMA_CCR5_EN ;
```

### 中断服务函数
	void DMA1_Channel5_IRQHandler()
中断服务函数是为了在 DMA 传输完成时，由硬件自动触发中断，及时清零中断标志避免重复触发，并通过设置标志位通知主程序传输已完成，让主程序无需持续轮询，提高系统效率。
```c
int receiveFlag = 0 ;
void DMA1_Channel5_IRQHandler(){
    if(DMA1->ISR & DMA_ISR_TCIF5){      //检查通道位5接收完成标志
        DMA1->IFCR |= DMA_IFCR_CGIF5 ;  //清除通道5的所有中断标志
        receiveFlag = 1 ;               //标记接收完成
    }
}
```
