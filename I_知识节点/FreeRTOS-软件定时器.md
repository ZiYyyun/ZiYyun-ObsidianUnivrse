#理论/嵌入式/FreeRTOS    #FreeRTOS 
FreeRTOS 中的软件定时器是一种不占用硬件定时器资源，而是由内核守护任务（Daemon Task）基于系统滴答（SysTick）统一调度，在指定时间到达后自动执行回调函数的轻量级定时机制

---



### 相关宏定义
```c
/* 软件定时器相关定义 */
/* 1: 使能软件定时器, 默认: 0。使能后需指定下面3个 */
#define configUSE_TIMERS 1
/* 定义软件定时器任务的优先级 */
#define configTIMER_TASK_PRIORITY (configMAX_PRIORITIES - 1)
/* 定义软件定时器命令队列的长度（能缓存几个Timer命令）*/
#define configTIMER_QUEUE_LENGTH 5
/* 定义软件定时器任务的栈空间大小*/
#define configTIMER_TASK_STACK_DEPTH (configMINIMAL_STACK_SIZE * 2)
```

### 相关函数

| 函数                          | 描述             |
| --------------------------- | -------------- |
| xTimerCreate()              | 动态方式创建软件定时器    |
| xTimerCreateStatic()        | 静态方式创建软件定时器    |
| xTimerStart()               | 开启软件定时器定时      |
| xTimerStartFromISR()        | 在中断中开启软件定时器定时  |
| xTimerStop()                | 停止软件定时器定时      |
| xTimerStopFromISR()         | 在中断中停止软件定时器定时  |
| xTimerReset()               | 复位软件定时器定时      |
| xTimerResetFromISR()        | 在中断中复位软件定时器定时  |
| xTimerChangePeriod()        | 更改软件定时器的定时超时时间 |
| xTimerChangePeriodFromISR() | 在中断中更改定时超时时间   |


### 创建函数

```c
TimerHandle_t xTimerCreate(
    const char * const pcTimerName,                 //给任务起的名字
    TickType_t xTimerPeriodInTicks,                 //周期
    UBaseType_t uxAutoReload,                       //是否循环
    void * pvTimerID,                               //TimerID（唯一）
    TimerCallbackFunction_t pxCallbackFunction      //回调函数
);
```


>回调函数：

	void LedTimerCallback(TimerHandle_t xTimer)
```c
{  
	HAL_GPIO_TogglePin(  
		LED1_GPIO_Port,  
		LED1_Pin  
	);  
}
```

### 启动定时器
当我们创建好定时器任务时，需要调用`xTimerStart()`函数来启动定时器任务

```c
xTimerStart(
    LedTimer,
    portMAX_DELAY
);
```

