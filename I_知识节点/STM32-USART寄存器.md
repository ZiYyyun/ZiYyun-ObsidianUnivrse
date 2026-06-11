#理论/嵌入式/STM32 


```c stm32f10x.h
typedef struct
{
  __IO uint16_t SR;
  uint16_t  RESERVED0;
  __IO uint16_t DR;
  uint16_t  RESERVED1;
  __IO uint16_t BRR;
  uint16_t  RESERVED2;
  __IO uint16_t CR1;
  uint16_t  RESERVED3;
  __IO uint16_t CR2;
  uint16_t  RESERVED4;
  __IO uint16_t CR3;
  uint16_t  RESERVED5;
  __IO uint16_t GTPR;
  uint16_t  RESERVED6;
} USART_TypeDef;
```

#### 配置步骤
1. 将USART_CR1寄存器的UE置1来激活USART。
2. 编程USART_CR1的M位定义字长
3. 在USART_CR2中编写停止位的个数
4. 如果需多缓冲器通信，选择USART_CR3中的DMA使能位(DMAR)。按多缓冲器通信所要求的配置DMA寄存器。
6. 利用波特率寄存器USART_BRR选择希望的波特率。
7. 设置USART_CR1的RE位。激活接收器，使它开始寻找起始位。

![[Pasted image 20241113175350.png]]
![[Pasted image 20241113175340.png]]![[Pasted image 20241113175322.png]]

| 寄存器      | 描述  | 功能  |
| -------- | --- | --- |
| **SR**   |     |     |
| **DR**   |     |     |
| **BRR**  |     |     |
| **CR1**  |     |     |
| **CR2**  |     |     |
| **CR3**  |     |     |
| **GTPR** |     |     |

