#理论/嵌入式/FreeRTOS 
前面学到了`xTaskNotifyTake()`一系列函数，它们只能实现“通知”的效果，但是不能传递数据。于是我们就有Queue消息队列

---


> [!NOTE] Queue与TaskNotify
> 当我们需要告诉某个任务“来活了”时，我们需要调用到`TaskNotifyGive()`函数，与`QueueReceive()`不同的是，`Notify`只会告诉任务“来活了”，却不能告诉任务“什么活”

### 相关配置
```c
#define configUSE_QUEUE_SETS 1
#define configQUEUE_REGISTRY_SIZE 8
#define configSUPPORT_DYNAMIC_ALLOCATION 1
#define configSUPPORT_STATIC_ALLOCATION 1
```

### 相关函数
| 函数                                            | 备注                                   |
| :-------------------------------------------- | :----------------------------------- |
| `xQueueCreate(length, itemSize)`              | 返回句柄；length为最大条目数，itemSize为每个条目的字节大小 |
| `xQueueSend(handle, &data, ticksToWait)`      | 任务级调用，支持阻塞等待                         |
| `xQueueReceive(handle, &buffer, ticksToWait)` | 接收成功后==数据会从队列中移除==                   |
| `xQueuePeek(handle, &buffer, ticksToWait)`    | 不移除数据，仅复制出来查看                        |
| `xQueueSendFromISR(...)`                      | 需传入 `pxHigherPriorityTaskWoken` 参数   |
| `xQueueReceiveFromISR(...)`                   | 同上                                   |
| `uxQueueMessagesWaiting(handle)`              | 用于调试或流控                              |


### 概述
>Queue的内部长这样：
```text
┌───────────┐
│ A │ B │ C │
└───────────┘
```
其遵循**先进先出**（FIFO）的原则。
Queue支持存入任意数据类型，如`int` `uint` `struct`等
### 关键函数
数据嘛，无非就是增删改查
#### 创建
```c
QueueHandle_t xQueueCreate(
    UBaseType_t uxQueueLength,
    UBaseType_t uxItemSize
);
```
- `uxQueueLength`：最大能存多少个元素
- `uxItemSize`：每个元素多少字节
#### 发送

```c
BaseType_t xQueueSend(
    QueueHandle_t xQueue,
    const void * pvItemToQueue,
    TickType_t xTicksToWait
);
```
- `xQueue`：发送给哪个队列
- `pvItemToQueue`：数据地址
- `xTicksToWait`：队列已满时等待时长

#### 接收

```c
BaseType_t xQueueGenericReceive( 
	QueueHandle_t xQueue,
	void * const pvBuffer,
	TickType_t xTicksToWait,
	const BaseType_t xJustPeek 
)
```


### 中断

这个函数的源码稍显复杂
```c
BaseType_t xQueueGenericSendFromISR ( 
	QueueHandle_t xQueue, 
	const void * const pvItemToQueue, 
	BaseType_t * const pxHigherPriorityTaskWoken, 
	const BaseType_t xCopyPosition 
)
```

```c
#define xQueueSendFromISR( 
	xQueue, 
	pvItemToQueue, 
	pxHigherPriorityTaskWoken 
	)
```
其实我们看这个宏定义就可以确定——传参只有三个。其余传参基本同上，不再赘述

>常用模板
```c
BaseType_t xHigherPriorityTaskWoken = pdFALSE;

xQueueSendFromISR(
    UARTQueue,       //
    &data,
    &xHigherPriorityTaskWoken
);

portYIELD_FROM_ISR(
    xHigherPriorityTaskWoken
);
```
需要注意一下，中断中的写法有一些不太一样。尤其是这个`xHigherPriorityTaskWoken`它用来表示“当前是否有更高优先级的中断任务被触发”