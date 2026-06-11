


`PA0`与`PA1`引脚分别对应`TIM5`的`CH1`和`CH2`
![[Pasted image 20260506205917.png]]


### 初始化TIM5
#### 配置时钟及GPIO
时钟配置参考 [[STM32-时钟树]]，不再赘述
```c
	RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
	RCC->APB1ENR |= RCC_APB1ENR_TIM5EN;
	//GPIO:复用推挽输出  ---------     MODE:11     CNF:10
	GPIOA->CRL |= (GPIO_CRL_MODE1 | GPIO_CRL_CNF1_1);
	GPIOA->CRL &= ~GPIO_CRL_CNF0_0;
```
其中，TIM5的两个引脚需配置为[[STM32-GPIO输入输出模式#推挽式复用功能(GPIO_Mode_AF_PP)]]

#### 配置TIM计数以及脉冲模式
参考：[[STM32-TIM-定时器#单脉冲模式（OPM）]]
```c
	TIM5->CR1 &= ~TIM_CR1_DIR;
	TIM5->CR1 &= ~TIM_CR1_OPM;
```


#### 配置输出模式
参考：[[STM32-TIM-定时器#捕获/比较模式寄存器（TIMx_CCMR）#输出模式]]
```c
    //将CH2设置为输出比较模式 - CC2S   00-输出
    TIM5->CCMR1 &=~ TIM_CCMR1_CC2S ;
    //将CH2通道的输出比较模式设置为PWM1模式 - 110
    TIM5->CCMR1 |= (TIM_CCMR1_OC2M_2 | TIM_CCMR1_OC2M_1);
    TIM5->CCMR1 &=~ TIM_CCMR1_OC2M_0 ;
```

#### 设置通道极性
参考：[[STM32-TIM-定时器#捕获/比较使能寄存器(TIMx_CCER)]]
```c
    //设置CH2的通道极性 - 默认有效电平是高电平
    TIM5->CCER &=~ TIM_CCER_CC2P ;
```

#### 时基单元配置（时基三要素）
参考[[STM32-TIM-定时器#时基单元]]
```c
    //ARPE：自动重装载预装载允许位
    TIM5->CR1 &=~ TIM_CR1_ARPE ;
    //时基三要素
    TIM5->CNT = 0 ;
    TIM5->PSC = 7200-1;
    TIM5->ARR = 100-1;
```


#### 使能定时器输出与时基单元
参考：[[STM32-TIM-定时器#捕获/比较使能寄存器(TIMx_CCER)]]
参考：[[STM32-TIM-定时器#控制寄存器1（CR1）]]

```c
    // tim5的CH2的输出比较使能
    TIM5->CCER |= TIM_CCER_CC2E ;
    // tim5时基单元使能
    TIM5->CR1 |= TIM_CR1_CEN ;
```


### 主函数实现呼吸灯

```c
int main(void)
{
    Int_Led_Init();
    Dri_Tim5_Init();
    Dri_Tim5_Start();

    uint8_t duty = 10, dir = 0;
    while (1)
    {
        TIM5->CCR2 = duty = dir ? --duty : ++duty;
        Delay_ms(10);
        dir = (duty >= 99) ? 1 : (duty == 0) ? 0 : dir;
    }
}
```