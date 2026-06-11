#实操/开发/嵌入式/STM32 
对于STM32的三种低功耗模式，这里不再赘述。参考[[STM32-低功耗]]

---

### 实现代码

>清除唤醒位
```c
PWR->CR |= PWR_CR_CWUF;
```

> [!NOTE] 注意
> 我们看PA0的引脚定义：
> ![[SCH_ZET6开发板_1_2024-07-01.pdf#page=2&rect=111,801,312,821&color=yellow|SCH_ZET6开发板_1_2024-07-01, p.2]]
> 由图可知，如果我们在这之前使用PA0输出了低电平且接了上拉电阻，那么在切换WKUP模式的过程中，硬件会被上拉电阻拉起，就会触发WKUP。所以我们要事先清除掉唤醒位


>操作[[RM0008中文参考手册.pdf#page=45&selection=电源控制/状态寄存器|电源控制/状态寄存器]]，使能WKUP引脚
```c
PWR->CSR |= PWR_CSR_EWUP;
```
将`WKUP`引脚用于唤醒，详见：
![[RM0008中文参考手册.pdf#page=45&rect=154,533,535,598&color=yellow]]


>配置待机模式
```c
PWR->CR |= PWR_CR_PDDS;
```
[[RM0008中文参考手册.pdf#page=44&selection=PDDS|PDDS（掉电深睡眠）]]位用于配置CPU进入深睡眠之后的模式选择，即：停机模式/待机模式


>CPU进入深睡眠模式
```c
SCB->SCR |= SCB_SCR_SLEEPDEEP;
```
在SCB_SCR寄存器中配置CPU，使其进入深睡眠模式


>进入待机模式
```c
__WFI();
```