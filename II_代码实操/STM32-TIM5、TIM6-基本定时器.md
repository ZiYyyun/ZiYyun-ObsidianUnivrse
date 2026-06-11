

### 初始化操作
```c
void Dri_Tim6_Init(void){
    //1.RCC使能
    RCC->APB1ENR |= RCC_APB1ENR_TIM6EN ;

    //2.自动重装载的预装载使能
    // 1-需要缓冲。不过目前后续不需要修改ARR的值，因此有无缓冲不重要
    TIM6->CR1 |= TIM_CR1_ARPE ;

    //3.OPM - 禁用单脉冲模式
    TIM6->CR1 &=~ TIM_CR1_OPM ;

    //4.DIER - DMA和Interrupt使能寄存器
    //当前要中断    UIE:更新中断使能
    TIM6->DIER |= TIM_DIER_UIE ; 

    //5.时基单元设置： CNT-计数器 ； PSC - 预分频器 ； ARR - 自动重装载寄存器
    TIM6->CNT = 0 ;
    // 时钟源频率是72MHz；我打算使用100us为单位的周期，因此预分频是7200-1
    TIM6->PSC = 7200-1;
    // 我打算产生1s的时长 一次是100us ， 那么需要多少个100us？
    // 需要10000个 100us，因此ARR是：10000-1
    TIM6->ARR = 10000-1;

    // NVIC设置
    NVIC_SetPriorityGrouping(3);
    NVIC_SetPriority(TIM6_IRQn,3);
    NVIC_EnableIRQ(TIM6_IRQn);
}
```






