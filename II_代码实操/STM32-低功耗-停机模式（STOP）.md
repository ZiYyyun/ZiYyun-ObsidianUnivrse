#实操/开发/嵌入式/STM32 
对于STM32的三种低功耗模式，这里不再赘述。参考[[STM32-低功耗]]

---

### 实现代码

>CPU进入深睡眠模式
```c
SCB->SCR |= SCB_SCR_SLEEPDEEP;
```
在SCB_SCR寄存器中配置CPU，使其进入深睡眠模式


>开启PWR时钟
```c
RCC->APB1ENR |= RCC_APB1ENR_PWREN;
```


>PDDS位置0
```c
PWR->CR &= ~PWR_CR_PDDS;
```
[[RM0008中文参考手册.pdf#page=44&selection=PDDS|PDDS]]功能位用于切换 停机（STOP）/待机（Standby） 模式，这里选择停机


>开启电压调节器，使其在停机模式下处于低功耗模式
```c
PWR->CR |= PWR_CR_LPDS;
```


>进入睡眠模式
```c
__WIF();
```






