#理论/嵌入式/FreeRTOS 
#FreeRTOS 

### 任务优先级（Task Priority）的基本法则

> [!warning] SysTick的优先级
> 在CubeMX中，我们不需要改动SysTick的优先级。因为FreeRTOS需要SysTick和PendSV处于最低优先级（15）
> 而且：**任何调用了 FreeRTOS API（如释放信号量、发送消息队列）的中断，其优先级数值必须 `≥` `configMAX_SYSCALL_INTERRUPT_PRIORITY`（即优先级不能高于 5，只能填 5~15 之间的数字）。**

#### 核心法则：数字越大，优先级越高！
- **最低优先级**：永远是 **`0`**。系统的**空闲任务（Idle Task）** 固定且永远运行在优先级 0。
- **最高优先级**：由 `FreeRTOSConfig.h` 中的宏 `configMAX_PRIORITIES` 决定。有效范围是 `0` 到 `configMAX_PRIORITIES - 1`。

#### 调度的两大铁律
- **铁律 1（抢占式）**：调度器**永远且只运行**处于“就绪态”的**最高优先级**任务。只要高优先级任务醒来，低优先级任务立刻被“踢”下 CPU。
- **铁律 2（时间片轮转）**：如果有**多个相同优先级**的任务同时处于就绪态，FreeRTOS 会在每次 SysTick（系统滴答）中断时，让它们轮流执行（前提是 `configUSE_TIME_SLICING` 设为 1）。

#### 动态修改任务优先级
>优先级不是创建时就锁死的，可以在运行中动态修改。
```c
// 将 xTaskHandle 任务的优先级修改为 5
vTaskPrioritySet(xTaskHandle, 5); 

// 获取某任务当前的优先级
UBaseType_t currentPrio = uxTaskPriorityGet(xTaskHandle); 
```

- **实战场景**：比如一个“日志上传任务”平时是低优先级（1），当用户按下“紧急导出”按键时，通过 `vTaskPrioritySet` 将其临时提升到高优先级（5），传完后再降回来。

### 中断优先级（Interrupt Priority）

必须把 **“硬件中断优先级”** 和 **“FreeRTOS 任务优先级”** 彻底分开来看。

#### 极其反直觉的 Cortex-M 硬件规则

在 ARM Cortex-M（如 STM32）的 NVIC（嵌套向量中断控制器）中，硬件中断优先级的规则是：  
👉 **数字越小，优先级越高！（0 是最高优先级，255 是最低）**  
_(这与 FreeRTOS 任务优先级完全相反！)_








### 