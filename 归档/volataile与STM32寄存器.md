
我们清楚有关GPIO的定义在这个结构体里：
[[STM32-GPIO寄存器#核心定义]]

比如：
```c
__IO uint32_t CRL;
```

这个`__IO`在`core_cm3.h`中有宏定义：

```c
#ifdef __cplusplus
  #define     __I     volatile                /*!< defines 'read only' permissions      */
#else
  #define     __I     volatile const          /*!< defines 'read only' permissions      */
#endif
#define     __O     volatile                  /*!< defines 'write only' permissions     */
#define     __IO    volatile                  /*!< defines 'read / write' permissions   */
```

