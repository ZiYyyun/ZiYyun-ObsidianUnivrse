#理论/嵌入式/FreeRTOS 


> [!warning] 注意
> **普通任务代码** 和 **中断服务函数（ISR）代码** 遵循**两套完全不同的规则**。  
> 在FreeRTOS中，中断也要求**快进快出**且中断期间影响调度器运行——中断期间，所有同优先级和低优先级的任务全都无法运行

### 第一铁律：中断里绝对不能阻塞

**问题**：`vTaskDelay(100)` 能否在中断里调用？  
**答案**：**绝对不能！**

**原因剖析**：
- `vTaskDelay()` 内部会修改任务状态、修改阻塞链表、触发任务调度。
- 中断（ISR）**根本不是任务**，没有任务控制块（TCB）。
- 在 ISR 中触发调度或阻塞会导致系统崩溃。
- **结论**：`中断里不能阻塞` 是 FreeRTOS 的第一原则。

### 2. 两套 API 函数体系
FreeRTOS 为任务和中断分别提供了两套 API，中断专用 API 均带有 **`FromISR`** 后缀。

| 功能        | 任务级 API (Task Context) | 中断级 API (ISR Context)      |
| --------- | ---------------------- | -------------------------- |
| **释放信号量** | `xSemaphoreGive()`     | `xSemaphoreGiveFromISR()`  |
| **发送队列**  | `xQueueSend()`         | `xQueueSendFromISR()`      |
| **任务通知**  | `vTaskNotifyGive()`    | `vTaskNotifyGiveFromISR()` |

> [!note] 为什么要有 `FromISR` 版本？  
> 参考[[STM32-中断-HAL#中断回调函数]]，因为中断有特殊要求：
> **不能阻塞、不能等待、执行必须极快**。
> `FromISR` 版本去除了所有可能导致阻塞的逻辑，专为 ISR 优化。在中断中调用普通版本（如 `xQueueSend()`）是极其危险的行为。

### 经典架构：中断与任务的分工

**设计原则**：**中断（ISR）只负责“通知/搬运”，任务（Task）负责“处理业务”。**
#### ❌ 错误做法（在 ISR 中处理业务）
```c
// 按键中断里做复杂操作
void EXTI_IRQHandler(void) {
    printf("按键按下");      // 耗时！
    HAL_GPIO_TogglePin(...); // 业务逻辑！
}
// UART中断里解析数据 (如 AT6558R GPS)
void USART_IRQHandler(void) {
    // 解析NMEA、计算经纬度、字符串处理... 严重超时！
}
```

#### ✅ 正确做法（ISR 快速退出，交由任务处理）
```
【按键场景】
EXTI 触发 → 进入 IRQHandler → 发送任务通知 → 立即退出 ISR
Key_Task (阻塞等待) → 收到通知 → 处理按键业务逻辑

【GPS UART 场景】
USART 触发 → 进入 IRQHandler → 接收1个字节 → 通知 GPS 任务 → 立即退出 ISR
GPS_Task (阻塞等待) → 收到通知 → 拼接字符串 → 解析 GGA/RMC 协议
```
**优势**：ISR 极短，系统响应更快，不会丢失后续中断。

### 4. 最佳实践：任务通知机制 (Task Notification)
在“中断通知任务”的场景中，最推荐使用的是 **任务通知**，它比信号量更快、更省内存。

**黄金组合函数**：
- **ISR 中发送**：`vTaskNotifyGiveFromISR()`
- **任务中接收**：`ulTaskNotifyTake()`

```c
// 1. 中断服务函数中 (ISR)
void USART1_IRQHandler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    // ... 清除中断标志位 ...
    vTaskNotifyGiveFromISR(GPSTaskHandle, &xHigherPriorityTaskWoken);
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken); // 必要时触发上下文切换
}

// 2. 任务代码中 (Task)
void GPS_Task(void *pvParameters) {
    while(1) {
        // 阻塞等待中断通知，超时时间设为 portMAX_DELAY
        ulTaskNotifyTake(pdTRUE, portMAX_DELAY); 
        // ... 执行 GPS 数据解析业务 ...
    }
}
```

> [!tip] 总结  
> `vTaskNotifyGiveFromISR` + `ulTaskNotifyTake` 是 FreeRTOS 中**中断通知任务最快的方法**，在实际开发中经常用来替代二值信号量。

