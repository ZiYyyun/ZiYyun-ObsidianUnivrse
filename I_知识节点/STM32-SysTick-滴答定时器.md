#理论/嵌入式/STM32 
系统定时器（SysTick系统）是属于CM3内核，内嵌在NVIC中。

系统定时器是一个24bit的向下**递减**的计数器，计数器每计数一次的时间为1 / SYSCLK，一般我们设置系统时钟SYSCLK（与AHB相同）等于72M。当重装载数值寄存器的值递减到0的时候，系统定时器就产生一次中断，以此循环往复。

---

### 相关寄存器
有4个寄存器与systick相关。
```c core_cm3.h
typedef struct
{
  __IO uint32_t CTRL;     /*!< Offset: 0x00  SysTick Control and Status Register */
  __IO uint32_t LOAD;     /*!< Offset: 0x04  SysTick Reload Value Register       */
  __IO uint32_t VAL;      /*!< Offset: 0x08  SysTick Current Value Register      */
  __I  uint32_t CALIB;    /*!< Offset: 0x0C  SysTick Calibration Register        */
} SysTick_Type;
```

#### 控制和状态寄存器（CTRL）

![[SysTick_CTRL.png]]
> 关于**CLKSOURCE**位，当0时，时钟频率是AHB/8；当1时，时钟频率是AHB。

#### 重装载寄存器（LOAD）

![[SysTick_LOAD.png]]

#### 当前数值寄存器（VAL）

![[SysTick_VAL.png]]


#### 校准数值寄存器（CALIB）
很少用到，略


