#理论/嵌入式/FreeRTOS   #FreeRTOS 


```
toggle执行
↓
阻塞500 Tick
↓
任务进入Ready
↓
调度器决定是否运行
↓
再次toggle
```

因为：

```
时间到≠立即运行
```

这句话以后学习优先级时特别重要。




### 任务的状态

> [!NOTE] Idole Task
> 如果当前没有任务执行，系统会有一个默认任务：
> **Idole Task**，它的优先级是0（最低），如果当前所有任务都阻塞，那么`Idole Task`会运行


### 运行态（Running）


### 就绪态（Ready）




### 阻塞态（Suspend）
> 阻塞态发生于我们调用了`vTaskDelay()`或`vTaskSuspend()`函数。

当vTaskDelay执行完毕后，系统调度器会自动把这个任务划为调度范围内
但是`vTaskSuspend()`函数不会，需要使用`vTaskResume()`来手动恢复


#### 时间问题
如果我们写：
```c
vTaskDelay(500);
```
那么这个500指的就是500个`Tick`，并不等于500ms。
当我们需要500ms的准确时间时，要写：
```c
vTaskDelay(pdMS_TO_TICKS(500));
```


> [!NOTE] SysTick和Tick
> 需要值得注意的是：虽然很多情况下1Tick就等于一次SysTick，但是它们本质不同：
> Tick是内核的时间单位，SysTick是产生Tick的一种方式。



### Q&A 
前面我们知道了，FreeRTOS中空闲任务`Idole Task`的优先级是0,那么疑问来了：
如果我们定义一个任务A，然后设置它的优先级为0，当任务A阻塞结束后变成了Ready态时，系统调度器会批准谁运行？

> 答案是：任务A

因为此时`Idle`正在运行，`TaskA`刚刚由阻塞态变运行态，调度器会重新检查有没有可运行的任务，发现：

| 任务    | 状态    |
| ----- | ----- |
| TaskA | Ready |
| Idle  | Ready |
然后判断：`TaskA`不是`Idle`任务，于是FreeRTOS会优先执行用户任务。
`Idle`只是一个临时工。