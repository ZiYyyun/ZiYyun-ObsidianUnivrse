#理论/嵌入式/FreeRTOS 
使用**队列**、**信号量**或**事件组**进行任务间通信需要通过中间对象， 发送任务写入通信对象，接收任务从通信对象读取。使用**任务通知**进行任务间通信，发送任务直接向接收任务发送通知，而无需中间对象

---


### 相关宏
```c
configUSE_TASK_NOTIFICATIONS	//是否启动任务通知功能，默认值是1
configTASK_NOTIFICATION_ARRAY_ENTRIES	//定义每个任务的任务通知数组大小，即每个任务支持的通知条目数，默认值是1
```

### 相关函数

| 函数                           | 功能                      |
| ---------------------------- | ----------------------- |
| xTaskNotify()                | 发送通知 带通知值               |
| xTaskNotifyAndQuery()        | 发送通知，带有通知值并且保留接收任务的原通知值 |
| xTaskNotifyGive()            | 发送通知，不带通知值              |
| xTaskNotifyFromISR()         |                         |
| xTaskNotifyAndQueryFromISR() |                         |
| vTaskNotifyGiveFromISR()     |                         |
| ulTaskNotifyTake()           | 获取任务通知，可选退出函数时对通知置清零或减1 |
| xTaskNotifyWait()            | 获取任务通知，可获取通知值和清除通知值的指定位 |

> [!NOTE] Tips
> - 对于TaskNotify这个函数，如果多次计数，那么它的返回值就是计数次数
> - 带有ISR的函数专用于中断回调函数内，其余则不需要





```c
uint32_t ulTaskGenericNotifyTake(UBaseType_t uxIndexToWait,
                                 BaseType_t xClearCountOnExit,
                                 TickType_t xTicksToWait)
{
    uint32_t ulReturn;
    configASSERT(uxIndexToWait < configTASK_NOTIFICATION_ARRAY_ENTRIES);
    taskENTER_CRITICAL();
    {
        /* Only block if the notification count is not already non-zero. */
        if (pxCurrentTCB->ulNotifiedValue[uxIndexToWait] == 0UL)
        {
            /* Mark this task as waiting for a notification. */
            pxCurrentTCB->ucNotifyState[uxIndexToWait] = taskWAITING_NOTIFICATION;
            if (xTicksToWait > (TickType_t)0)
            {
                prvAddCurrentTaskToDelayedList(xTicksToWait, pdTRUE);
                traceTASK_NOTIFY_TAKE_BLOCK(uxIndexToWait);
                /* All ports are written to allow a yield in a critical
                 * section (some will yield immediately, others wait until the
                 * critical section exits) - but it is not something that
                 * application code should ever do. */
                portYIELD_WITHIN_API();
            }
            else
            {
                mtCOVERAGE_TEST_MARKER();
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
    }
    taskEXIT_CRITICAL();
    taskENTER_CRITICAL();
    {
        traceTASK_NOTIFY_TAKE(uxIndexToWait);
        ulReturn = pxCurrentTCB->ulNotifiedValue[uxIndexToWait];
        if (ulReturn != 0UL)
        {
            if (xClearCountOnExit != pdFALSE)
            {
                pxCurrentTCB->ulNotifiedValue[uxIndexToWait] = 0UL;
            }
            else
            {
                pxCurrentTCB->ulNotifiedValue[uxIndexToWait] = ulReturn - (uint32_t)1;
            }
        }
        else
        {
            mtCOVERAGE_TEST_MARKER();
        }
        pxCurrentTCB->ucNotifyState[uxIndexToWait] = taskNOT_WAITING_NOTIFICATION;
    }
    taskEXIT_CRITICAL();
    return ulReturn;
}
```