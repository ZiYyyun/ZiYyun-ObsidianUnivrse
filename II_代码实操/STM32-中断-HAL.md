#实操/开发/嵌入式/STM32 
在裸机开发（没有操作系统）中，中断就像是单片机的 **“条件反射”**。当外部事件（比如按键按下、定时器到期、串口收到数据）发生时，CPU 会立刻停下手里正在干的活（主循环 `while(1)`），跑去执行一段特定的代码（中断服务函数），执行完再回去接着干原来的活。

---

### 配置CubeMX
把需要中断的引脚配置为“外部中断模式”，并开启对应的NVIC

### 编写中断入口函数
>当硬件触发中断时，CPU 会跳转到一个固定的入口函数。这个函数通常放在 `stm32f1xx_it.c`（中断服务文件）中。它的唯一工作就是**清理中断标志位**，然后把活儿交给 HAL 库的回调函数。

```c
void EXTI0_IRQHandler(void) 
{ 
	HAL_GPIO_EXTI_IRQHandler(KEY1_Pin);
}
```

### 通用处理函数
**在 `stm32f1xx_hal_gpio.c` 中**
```c
void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin) { 
	if (__HAL_GPIO_EXTI_GET_IT(GPIO_Pin) != 0x00U) { 
		__HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
		HAL_GPIO_EXTI_Callback(GPIO_Pin); 
	} 
}
```
### 中断回调函数

```c
// 重写 HAL 库的回调函数 
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin) { // 判断是不是 KEY1 触发的中断 
if (GPIO_Pin == KEY1_Pin) { // 【核心逻辑】：按键按下了，翻转 LED！ 
		HAL_GPIO_TogglePin(LED1_GPIO_Port, LED1_Pin); 
	} 
}
```
这个函数定义在`stm32f1xx_hal_gpio.c`中，这是个虚函数，当我们需要实现它时，需要在`main.c`中重写


> [!WARNING] 注意
> 在中断回调函数里，绝对不能有延时和阻塞操作（比如用循环等待串口发送），一定要快进快出
> _原因：`HAL_Delay()` 本身是靠 SysTick 中断来计数的。你在外部中断里死等，SysTick 中断进不来，时间永远不走，单片机直接死锁（死机）。_
