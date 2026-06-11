#实操/开发/嵌入式/STM32 

> [!NOTE] 电路设计
>LED1电路 
>![[EXTI4.png]]
![[EXTI1.png]]
KEY电路
![[EXTI2.png]]
![[EXTI3.png]]
> Contents


---
需求：利用外部中断检测按键KEY3，当按键按下，翻转LED1显示。

```c
#include "key.h"
#include "led.h"

// Key3 - PF10  
void Int_Key_Init(void){
    // RCC使能
    RCC->APB2ENR |= (RCC_APB2ENR_IOPFEN | RCC_APB2ENR_AFIOEN) ;
    // PF10 下拉输入: MODE-00 CNF-10
    GPIOF->CRH &=~ (GPIO_CRH_MODE10 | GPIO_CRH_CNF10_0);
    GPIOF->CRH |= GPIO_CRH_CNF10_1 ;
    GPIOF->ODR &=~ GPIO_ODR_ODR10 ;
    // AFIO(AFIO引脚复用选择)将GPIOA~GPIOG总共112根线分成了16组
    // 每组7根线; A0~G0为一组; A1~G1为一组....
    // AFIO有四个寄存器: AFIO_EXTICR1~EXTICR4
    // 每一个CR寄存器配置了四个通道(一个通道7根线)
    // 当前使用的是PF10；0-3 4-7 8-11 12-15；10属于8-11，也就是CR3
    // 10对应的4位置0
    AFIO->EXTICR[2] &=~ AFIO_EXTICR3_EXTI10 ;
    // 10对应的4位 0101
    AFIO->EXTICR[2] |= AFIO_EXTICR3_EXTI10_PF;
    // EXTI配置
    // 配置上升沿触发
    EXTI->RTSR |= EXTI_RTSR_TR10 ;
    // 配置开放10号通道
    EXTI->IMR |= EXTI_IMR_MR10 ;
    // NVIC设置
    // 1) 设置4位全部都是抢占优先级
    NVIC_SetPriorityGrouping(3);
    // 2) 设置当前中断的优先级
    NVIC_SetPriority(EXTI15_10_IRQn,3);  // 40 - EXTI15_10_IRQn
    //NVIC_EncodePriority(uint32_t PriorityGroup, uint32_t PreemptPriority, uint32_t SubPriority);
    // 3) NVIC使能
    NVIC_EnableIRQ(EXTI15_10_IRQn);

}
__weak void Int_Key_CallBack(void){}
// STM32中，中断服务函数名称是固定好的
void EXTI15_10_IRQHandler() {
    // 清除中断标志位
    EXTI->PR |= EXTI_PR_PR10 ;
    Int_Key_CallBack();
}
```