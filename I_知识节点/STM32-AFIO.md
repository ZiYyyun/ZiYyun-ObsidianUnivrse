AFIO(Alternate Function Input/Output 复用输入输出)
AFIO是stm32上的众多片上外设之一，专门用来执行“复用功能的重映射”

---
*注意： 对寄存器AFIO_EVCR，AFIO_MAPR和AFIO_EXTICRX进行读写操作前，应当首先打开AFIO*
*的时钟。参考第6.3.7节APB2外设时钟使能寄存器(RCC_APB2ENR)。*

### 外部中断配置控制器(AFIO_EXTICR)
STM32F1 的 `AFIO_EXTICR` 共 4 个寄存器，每个管 **4 条 EXTI 线**，对应关系如下：

| 寄存器            | 负责的 EXTI 线      | 对应的 GPIO 引脚号    |
| :------------- | :-------------- | :-------------- |
| `AFIO_EXTICR1` | EXTI0 ~ EXTI3   | Pin 0 ~ Pin 3   |
| `AFIO_EXTICR2` | EXTI4 ~ EXTI7   | Pin 4 ~ Pin 7   |
| `AFIO_EXTICR3` | EXTI8 ~ EXTI11  | Pin 8 ~ Pin 11  |
| `AFIO_EXTICR4` | EXTI12 ~ EXTI15 | Pin 12 ~ Pin 15 |
以`PF10`引脚为例，代码应写为：
```c
AFIO_EXTICR[2] -> |= AFIO_EXTICR3_EXTI10_PF; 
```
因为AFIO_EXTICR是从零开始的。
