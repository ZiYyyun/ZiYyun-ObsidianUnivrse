#理论/嵌入式/STM32 


![[Pasted image 20240917113728.png]]
```c stm32f103x8.h
typedef struct
{
  __IO uint32_t CR;
  __IO uint32_t CFGR;
  __IO uint32_t CIR;
  __IO uint32_t APB2RSTR;
  __IO uint32_t APB1RSTR;
  __IO uint32_t AHBENR;
  __IO uint32_t APB2ENR;
  __IO uint32_t APB1ENR;
  __IO uint32_t BDCR
  __IO uint32_t CSR;
} RCC_TypeDef;
```

| 寄存器          | 描述                                    | 功能                                         |
| ------------ | ------------------------------------- | ------------------------------------------ |
| **CR**       | Clock Control Register                | 控制 HSE（外部高速时钟）、HSI（内部高速时钟）和 PLL 的使能        |
| **CFGR**     | Clock Configuration Register          | 选择系统时钟源（HSI、HSE、PLL）、配置 AHB 和 APB 总线的时钟分频器 |
| **CIR**      | Clock Interrupt Register              | 时钟稳定后产生的中断、PLL 锁定中断、振荡器故障中断等               |
| **APB2RSTR** | APB2 Peripheral Reset Register        | 复位 GPIOA、USART1、ADC1 等外设                   |
| **APB1RSTR** | APB1 Peripheral Reset Register        | 复位 TIM2、USART2 等低速外设                       |
| **AHBENR**   | AHB Peripheral Clock Enable Register  | 使能 DMA 控制器时钟                               |
| **APB2ENR**  | APB2 Peripheral Clock Enable Register | 使能 GPIOA、GPIOB、USART1、ADC1 等外设的时钟          |
| **APB1ENR**  | APB1 Peripheral Clock Enable Register | 使能 TIM2、USART2、I2C1 等外设的时钟                 |
| **BDCR**     | Backup Domain Control Register        | 使能 LSE（外部低速时钟）、选择 RTC 时钟源                  |
| **CSR**      | Control and Status Register           | 复位标志清除、LSI 时钟使能等                           |

### 时钟控制寄存器（RCC_CR）

> [!NOTE] Title
> ![[Pasted image 20240917202503.png]]![[Pasted image 20240917202606.png]]


### 时钟配置寄存器（RCC_CFGR）

> [!NOTE] CFGR
> ![[Pasted image 20240917202729.png]]![[Pasted image 20240917202746.png]]![[Pasted image 20240917202758.png]]


### 时钟中断寄存器（RCC_CIR）

> [!NOTE] Title
> ![[Pasted image 20240918211302.png]]![[Pasted image 20240918211325.png]]![[Pasted image 20240918211405.png]]![[Pasted image 20240918211420.png]]


### APB2外设时钟使能寄存器（RCC_APB2RSTR）

> [!NOTE] Title
> ![[Pasted image 20240918211458.png]]![[Pasted image 20240918211510.png]]![[Pasted image 20240918211614.png]]![[Pasted image 20240918211631.png]]

### APB1外设复位寄存器（RCC_APB1RSTR）

> [!NOTE] Title
> ![[Pasted image 20240918211922.png]]![[Pasted image 20240918212010.png]]![[Pasted image 20240918212028.png]]![[Pasted image 20240918212040.png]]


### AHB外设时钟使能寄存器（RCC_AHBENR）

> [!NOTE] Title
> ![[Pasted image 20240918212247.png]]![[Pasted image 20240918212258.png]]![[Pasted image 20240918212323.png]]


### APB2外设时钟使能寄存器（RCC_APB2ENR）

> [!NOTE] Title
> ![[Pasted image 20240918212342.png]]![[Pasted image 20240918212409.png]]![[Pasted image 20240918212422.png]]


### APB1外设时钟使能寄存器（RCC_APB1ENR）

> [!NOTE] Title
> ![[Pasted image 20240918212757.png]]![[Pasted image 20240918212813.png]]![[Pasted image 20240918212826.png]]![[Pasted image 20240918212846.png]]


### 备份域控制寄存器（RCC_BDCR）

> [!NOTE] Title
> ![[Pasted image 20240918213011.png]]![[Pasted image 20240918213022.png]]


### 控制/状态寄存器（RCC_CSR）

> [!NOTE] Title
> ![[Pasted image 20240918213045.png]]![[Pasted image 20240918213059.png]]![[Pasted image 20240918213109.png]]





![[Pasted image 20240917133549.png]]
