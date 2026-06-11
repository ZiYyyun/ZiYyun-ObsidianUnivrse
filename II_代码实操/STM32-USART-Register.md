

### 初始化
步骤如下：
- 开启 GPIOA 时钟 + USART1 时钟
> 注意：`USART1`在`APB2`总线，`USART2`、`USART3`在`APB1`总线
- 配置 PA9、PA10 引脚模式（复用功能）
- 配置 USART 控制寄存器：数据位、停止位、收发使能
> 默认参数（通用标准）：8 位数据 + 1 位停止位 + 无校验 + 异步模式
- 配置 **BRR 波特率寄存器**（最核心）
- 使能 USART1 外设

### 函数结构
需要实现的函数如下：
```c
void Dri_USART1_Init(void);                           //初始化USART
void Dri_USART1_SendChar(uint8_t ch);                 //发送字符
void Dri_USART1_SendStr(uint8_t *str , uint8_t len);  //发送字符串
```


### 开始编写
#### 初始化USART

- 使能`APB2ENR`
```c
RCC->APB2ENR |= RCC_APB2ENR_USART1EN;
```

- 使能`GPIOA`
```c
RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
```

- `PA9`、`PA10`分别对应`TX` `RX`，配置PA9复用推挽输出，PA10浮空输入，
对应: PA9 : MODE=11 CNF=01
```c usart.c
GPIOA->CRH |= (GPIO_CRH_MODE9 | GPIO_CRH_CNF9_1 | GPIO_CRH_CNF10_0);
GPIOA->CRH &= ~(GPIO_CRH_MODE10 | GPIO_CRH_CNF9_0 | GPIO_CRH_CNF10_1);
```

- 配置字长和停止位
> [[STM32-USART#字长]]  [[STM32-USART#通讯的停止位]]
```c
USART1->CR1 &= ~USART_CR1_M;        //字长
USART1->CR1 &= ~USART_CR2_STOP;     //停止位
```

- 使能`USART1`
```c
USART1->CR1 |= USART_CR1_TE;       //使能usart1
USART1->CR1 |= USART_CR1_UE;       //使能usart1发送
```