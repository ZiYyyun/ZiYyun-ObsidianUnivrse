#理论/嵌入式/STM32 


[[#^eec454|HAL库实现]]
[[#^a071a1|寄存器实现]]

>寄存器实现
^a071a1
---

*注意，因为SPI是全双工的通信，可以同时收发，所以SPI其实没有单独的收、发函数。如果需要接收，就发一个0x00就好了*

> [!NOTE] 原理图
> 
> ![[SCH_ZET6开发板_1_2024-07-01.pdf#page=5&rect=70,328,246,392&color=yellow|SCH_ZET6开发板_1_2024-07-01, p.5]]
> 由原理图可见，我们需要配置 `PC13` `PA5` `PA6` `PA7`这四个引脚，由[[Prot-SPI]]可知，四个引脚需要分别被配置为：

> 宏定义 *方便操作引脚高低位*
```c
#define CS_LOW  (GPIOC->ODR &=~ GPIO_ODR_ODR13)
#define CS_HIGH  (GPIOC->ODR |= GPIO_ODR_ODR13)
#define SCK_LOW (GPIOA->ODR &=~ GPIO_ODR_ODR5)
#define SCK_HIGH (GPIOA->ODR |= GPIO_ODR_ODR5)
#define MOSI_LOW (GPIOA->ODR &=~ GPIO_ODR_ODR7)
#define MOSI_HIGH (GPIOA->ODR |= GPIO_ODR_ODR7)
#define MISO_READ (GPIOA->IDR & GPIO_IDR_IDR6)
```
### 初始化SPI
	Dri_Spi_Init(void)
> 使能时钟和GPIO引脚配置，不再赘述            
```c
    RCC->APB2ENR |= (RCC_APB2ENR_IOPCEN | RCC_APB2ENR_IOPAEN) ;

    //2.工作模式设置
    GPIOA->CRL |= (GPIO_CRL_MODE5 | GPIO_CRL_MODE7);
    GPIOA->CRL &=~(GPIO_CRL_CNF5 | GPIO_CRL_CNF7) ;

    GPIOC->CRH |=  GPIO_CRH_MODE13;
    GPIOC->CRH &=~ GPIO_CRH_CNF13 ;

    GPIOA->CRL &=~ (GPIO_CRL_MODE6 | GPIO_CRL_CNF6_1) ;
    GPIOA->CRL |= GPIO_CRL_CNF6_0 ;
```

> 拉低`CS`同时拉高`SCK`
```c
    CS_HIGH ;
    SCK_LOW ;
```

### 开始
	Dri_SPI_Start(void)

### 停止
	Dri_SPI_Stop(void)

### 交换数据
	Dri_SPI_SwarByte(u8 byte)
> 定义变量

```c
u8 receiveByte = 0;
```

```c
for(u8 i = 0; i < 8; i++){
	
```

```c
for(u8 i = 0; i < 8; i++){
(byte & 0x80) ? MOSI_HIGH : MOSI_LOW;
```
这个0x80的转换成二进制是 1000 0000。结合这个&就是取出Byte的最高位。如果有数据，就把`MOSI`拉高

>拉高`SCK`，准备接收/发送
```c
SCK_HIGH;
```

>把接收到的数据左移一位
```c
receiveByte <<= 1;
```

>把数据赋给`receiveByte`
```c
if(MISO_READ){ receiveByte |= 0x01; }
```

>拉低`SCK`，结束收发
```c
SCK_LOW;
```

>左移`Byte`
```c
byte <<= 1;
}
```


>HAL库实现
^eec454
---

