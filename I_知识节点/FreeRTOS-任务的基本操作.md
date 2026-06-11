#理论/嵌入式/FreeRTOS 


### 相关函数
任务的创建和删除本质上就是调用这些函数：

| API函数               | 描述       |
| ------------------- | -------- |
| xTaskCreate()       | 动态方式创建任务 |
| xTaskCreateStatic() | 静态方式创建任务 |
| vTaskDelete()       | 删除任务     |


### 创建
```c
xTaskCreate(
    TaskFunction_t pvTaskCode,      //指向任务的指针
    const char * const pcName,      //任务命名
    uint16_t usStackDepth,          //任务堆栈大小（默认单位4字节）
    void *pvParameters,             //传递给任务的参数
    UBaseType_t uxPriority,         //任务优先级（范围configMAX_PRIORITIES - 1）
    TaskHandle_t *pxCreatedTask     //任务句柄
);
```


### 删除


### 挂起



> [!NOTE] 注意
> 如果函数在Task内部调用且传参为`NULL`则表示挂起自己


### 恢复

