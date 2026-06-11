#实操/开发/嵌入式/STM32 

### 实现目标
利用`SysTick`系统定时器的中断，每隔一秒让`LED`闪烁一次。

### 硬件设计


> [!NOTE] 硬件电路设计
> ![[STM32-LED123.png]]
> 其中，`LED1` `LED2` `LED3`分别对应`PA0` `PA1` `PA8`


#### 配置LED引脚
参考：[[STM32-GPIO输入输出模式#推挽式输出(GPIO_Mode_Out_PP)]]配置LED对应引脚为推挽输出，不再赘述。
```c
#include "led.h"
void Int_Led_Init(void){
	RCC->APB2ENR |= RCC_APB2ENR_IOPAEN;
	
	GPIOA->CRL |= GPIO_CRL_MODE0;
	GPIOA->CRL &= ~GPIO_CRL_CNF0;
	
	GPIOA->CRL |= GPIO_CRL_MODE1;
	GPIOA->CRL &= ~GPIO_CRL_CNF1;
	
	GPIOA->ODR |= GPIO_ODR_ODR0; //
}

void Int_Led_Toggle(void){
	GPIOA->ODR ^= GPIO_ODR_ODR0;
}
```

#### 初始化SysTick
[[STM32-SysTick-滴答定时器]]

```c
#include "systick.h"
#include "stm32f10x.h"
#include "led.h"
void Dri_Systick_Init(void)
{
    SysTick->CTRL |= SysTick_CTRL_CLKSOURCE;  //选择时钟源
	SysTick->CTRL |= SysTick_CTRL_TICKINT;    //中断使能
    SysTick->LOAD = 72000 - 1;                //设置重装载值
	SysTick->CTRL |= SysTick_CTRL_ENABLE;     //计时器使能
	
		NVIC_SetPriorityGrouping(3);          //配置中断优先级分组
		NVIC_SetPriority(SysTick_IRQn, 3);    //配置中断优先级
}
```


其中的`SysTick_Handler()`中断服务函数比较特殊，这个函数的名字不能改变，当`SysTick`定时器被开启后，定时一到，CPU会直接跳去执行`SysTick_Handler()`
```c
void SysTick_Handler(){
	static u16 count = 0;
	count++;  //用于统计1ms中触发中断的次数
	if( count % 1000 == 0){count = 0; Int_Led_Toggle();}
}
```

> 其中，`static`关键字不可或缺，区别如下
> - 不加 `static`：每次进函数，count 都会重置为 0，无法计数；
> - 加 `static`：变量只会初始化一次，值会一直保存，实现累计计数。

