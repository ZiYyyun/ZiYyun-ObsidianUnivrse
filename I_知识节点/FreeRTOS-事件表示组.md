#理论/嵌入式/FreeRTOS    #FreeRTOS 
这个是多个条件同时满足的解决方案。
事件标志组就像是一个共享的标志牌集合，每个标志位都代表一种特定的状态或事件。任务可以等待或设置这些标志位，从而实现任务之间的协同工作。
其实这个东西和`xTaskNotify`系列函数差不多，前者是单一条件，而事件组是多个条件

---

### 事件位和事件组
>事件位用于指示事件是否发生。 事件位通常称为事件标志。
>事件组就是一组事件位。 事件组中的事件位通过位编号来引用。

### 主要函数

| 函数                            | 描述               |
| ----------------------------- | ---------------- |
| xEventGroupCreate()           | 使用动态方式创建事件标志组    |
| xEventGroupCreateStatic()     | 使用静态方式创建事件标志组    |
| xEventGroupClearBits()        | 清零事件标志位          |
| xEventGroupClearBitsFromISR() | 在中断中清零事件标志位      |
| xEventGroupSetBits()          | 设置事件标志位          |
| xEventGroupSetBitsFromISR()   | 在中断中设置事件标志位      |
| xEventGroupWaitBits()         | 等待事件标志位          |
| xEventGroupSync()             | 设置事件标志位，并等待事件标志位 |
