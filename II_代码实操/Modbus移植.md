#实操/开发/嵌入式/STM32/项目  #modbus

本案例在 #实操/开发/嵌入式/STM32/项目/步进电机 项目的电机端进行FreeModbus移植。

### 仓库
[cwalter-at/freemodbus: BSD licensed MODBUS RTU/ASCII and TCP slave](https://github.com/cwalter-at/freemodbus)
克隆好仓库之后，把`Modbus`文件夹和对应案例中`port`文件夹下的所有内容拷贝到项目中，并配置好include path。

### 驱动移植
#### 串口驱动编写

修改`portserial.c`

> 串口使能功能实现

修改：
	void vMBPortSerialEnable(BOOL xRxEnable, BOOL xTxEnable)
```c
    if (xRxEnable)
    {
        __HAL_UART_ENABLE_IT(&huart2, UART_IT_RXNE);
    } else {
        __HAL_UART_DISENABLE_IT(&huart2, UART_IT_TXE);
    }

    if (xTxEnable)
    {
        __HAL_UART_ENABLE_IT(&huart2, UART_IT_TXE);
    }else {
        __HAL_UART_DISENABLE_IT(&huart2, UART_IT_TXE);
    }
```
注意到：这个函数有两个形参：`xRxEnable`和`xTxEnable`，且这个函数是用于启用收发功能的，所以我们判断置一后，便可以开启串口中断使能标识，反之亦然。

> 串口数据发送

	BOOL xMBPortSerialPutByte(CHAR ucByte)
```c
    USART2->DR = ucByte & 0xFF;
    return TRUE;
```
这个函数用来把形参中的数据传给串口DR寄存器，通过串口发出去


> [!NOTE] Tip
> 这里模`oxFF`表示只取低八位数据：
> ```
> 00010010 00110100
> &
> 00000000 11111111
> ```
> 因为DR寄存器在头文件里被定义为`uint32_t`，然而真正一次串口只能发送`uint8_t`的数据


> 串口数据接收

	BOOL xMBPortSerialGetByte(CHAR *pucByte)
```c
    *pucByte = (CHAR)(USART2->DR) & 0xFF;
    return TRUE;
```
获取接收到的数据

> 中断服务程序

	void USART2_IRQHandler(void)
```c
    if (__HAL_UART_GET_FLAG(&huart2, UART_FLAG_TXE) != RESET) // 如果：中断发送标志被置位
    {
        prvvUARTTxReadyISR();                          // 执行：发送中断服务程序
        __HAL_UART_CLEAR_FLAG(&huart2, UART_FLAG_TXE); // 然后：清除中断发送标志
    }
else if (__HAL_UART_GET_FLAG(&huart2, UART_FLAG_RXNE) != RESET)
    {
        prvvUARTRxISR();
        __HAL_UART_CLEAR_FLAG(&huart2, UART_FLAG_RXNE);
    }

    HAL_UART_IRQHandler(&huart2); // 调用HAL库的UART中断服务程序
```

如果串口中断服务函数被调用，就执行modbus的中断服务函数。收发同理。




#### 定时器驱动编写
修改`porttimer.c`

> 定时器初始化

	BOOL xMBPortTimersInit (USHORT usTim1Timerout50us)

```c
    return TRUE;
```

该函数为 Modbus 定时器初始化接口，此处仅做占位返回，定时器实际配置由 STM32 HAL 库工程完成。

> 定时器使能

	inline void vMBPortTimersEnable()
```c
    /* Enable the timer with the timeout passed to xMBPortTimersInit( ) */
    __HAL_TIM_SET_COUNTER(&htim3, 0);
    HAL_TIM_Base_Start_IT(&htim3);
```

先将定时器计数器值清零，再开启定时器并使能中断，为 Modbus 帧超时计时做准备。

> 定时器关闭

	inline void vMBPortTimersDisable()
```c
    /* Disable any pending timers. */
    HAL_TIM_Base_Stop_IT(&htim3);
```

关闭定时器并关闭对应中断，停止 Modbus 超时计时。

> 定时器中断回调函数

	static void prvvTIMERExpiredISR(void)
```c
{
    (void)pxMBPortCBTimerExpired();
}
```

定时器计时溢出后调用该函数，向上层 Modbus 协议栈发送定时器超时通知。

> 定时器中断服务程序

	void TIM3_IRQHandler(void)

```c
{
    if (__HAL_TIM_GET_FLAG(&htim3, TIM_FLAG_UPDATE) != RESET)
    {
        __HAL_TIM_CLEAR_IT(&htim3, TIM_IT_UPDATE);
        prvvTIMERExpiredISR();
    }

    HAL_TIM_IRQHandler(&htim3);
}
```

检测定时器更新中断标志，若标志置位则清除中断标志，并执行定时器超时回调函数，最后调用 HAL 库标准定时器中断处理函数。






#### port文件

	port.h
```c
// 定义十路输入寄存器的大小
#define REG_INPUT_SIZE 10
// 声明输入寄存器缓冲区，用于存储十路输入寄存器的值
extern uint16_t REG_INPUT_BUF[REG_INPUT_SIZE];

// 定义十路保持寄存器的大小
#define REG_HOLD_SIZE 10
// 声明保持寄存器缓冲区，用于存储十路保持寄存器的值
extern uint16_t REG_HOLD_BUF[REG_HOLD_SIZE];

// 定义十路线圈的大小
#define REG_COILS_SIZE 10
// 声明线圈缓冲区，并初始化，用于存储十路线圈的状态
extern uint8_t REG_COILS_BUF[REG_COILS_SIZE];

// 定义十路离散量的大小
#define REG_DISC_SIZE 10
// 声明离散量缓冲区，并初始化，用于存储十路离散量的状态
extern uint8_t REG_DISC_BUF[REG_DISC_SIZE];

```

port.c
```c
#include "mb.h"
#include "mbport.h"

// 准备使用[2]标示当前转速
//  声明输入寄存器缓冲区，用于存储十路输入寄存器的值
uint16_t REG_INPUT_BUF[REG_INPUT_SIZE]= {0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

// 准备使用[2]标示目标转速
//  声明保持寄存器缓冲区，用于存储十路保持寄存器的值
uint16_t REG_HOLD_BUF[REG_HOLD_SIZE] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

// 准备使用[2]标示电机的启动和停止
//  定义十路线圈的大小
uint8_t REG_COILS_BUF[REG_COILS_SIZE] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0};

// 准备使用[2]标示电机的方向
//  声明离散量缓冲区，并初始化，用于存储十路离散量的状态
uint8_t REG_DISC_BUF[REG_DISC_SIZE] = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0};
/**
 * @brief CMD4命令处理回调函数
 * 
 * 该函数用于处理MODBUS协议中的CMD4命令，即读取输入寄存器。
 * 它将指定地址范围内的输入寄存器的值复制到缓冲区中。
 *
 * @param pucRegBuffer 指向用于存储寄存器值的缓冲区的指针。
 * @param usAddress 要读取的起始寄存器地址。
 * @param usNRegs 要读取的寄存器数量。
 *
 * @return 返回执行结果的错误代码。
 */
eMBErrorCode eMBRegInputCB(UCHAR *pucRegBuffer, USHORT usAddress, USHORT usNRegs)
{
    // 计算寄存器索引，从0开始
    USHORT usRegIndex = usAddress - 1;

    // 非法检测：检查访问范围是否超出寄存器缓冲区大小
    if ((usRegIndex + usNRegs) > REG_INPUT_SIZE)
    {
        return MB_ENOREG;
    }

    // 循环读取寄存器值并写入缓冲区
    while (usNRegs > 0)
    {
        // 将寄存器的高8位写入缓冲区
        *pucRegBuffer++ = (unsigned char)(REG_INPUT_BUF[usRegIndex] >> 8);
        // 将寄存器的低8位写入缓冲区
        *pucRegBuffer++ = (unsigned char)(REG_INPUT_BUF[usRegIndex] & 0xFF);
        usRegIndex++;
        usNRegs--;
    }

    return MB_ENOERR;
}

/**
 * @brief CMD6、3、16命令处理回调函数
 * 
 * 该函数用于处理Modbus协议中的 Holding Registers 读写请求。
 * 它根据请求的模式（读或写）对指定的寄存器进行相应的操作。
 *
 * @param pucRegBuffer 寄存器数据缓冲区，用于读取或写入寄存器数据。
 * @param usAddress 请求访问的起始寄存器地址。
 * @param usNRegs 请求访问的寄存器数量。
 * @param eMode 访问模式，可以是 MB_REG_WRITE（写寄存器）或 MB_REG_READ（读寄存器）。
 *
 * @return 返回执行结果，如果成功则返回 MB_ENOERR，否则返回相应的错误代码。
 */
eMBErrorCode eMBRegHoldingCB(UCHAR *pucRegBuffer, USHORT usAddress, USHORT usNRegs, eMBRegisterMode eMode)
{
    // 计算寄存器索引，Modbus 地址从 1 开始，数组索引从 0 开始，因此需要减 1。
    USHORT usRegIndex = usAddress - 1;

    // 非法检测：检查访问范围是否超出寄存器缓冲区大小。
    if ((usRegIndex + usNRegs) > REG_HOLD_SIZE)
    {
        return MB_ENOREG;
    }

    // 写寄存器
    if (eMode == MB_REG_WRITE)
    {
        // 循环将每个寄存器的数据从缓冲区写入到寄存器中。
        while (usNRegs > 0)
        {
            REG_HOLD_BUF[usRegIndex] = (pucRegBuffer[0] << 8) | pucRegBuffer[1];
            pucRegBuffer += 2;
            usRegIndex++;
            usNRegs--;
        }
    }
    // 读寄存器
    else
    {
        // 循环将每个寄存器的数据从寄存器中读取到缓冲区。
        while (usNRegs > 0)
        {
            *pucRegBuffer++ = (unsigned char)(REG_HOLD_BUF[usRegIndex] >> 8);
            *pucRegBuffer++ = (unsigned char)(REG_HOLD_BUF[usRegIndex] & 0xFF);
            usRegIndex++;
            usNRegs--;
        }
    }

    return MB_ENOERR;
}

/**
 * @brief CMD1、5、15命令处理回调函数
 * 
 * 该函数用于处理Modbus协议中的CMD1、CMD5和CMD15命令。
 * 它主要负责读取或写入寄存器中的位数据。
 *
 * @param pucRegBuffer 指向寄存器缓冲区的指针，用于读取或写入数据。
 * @param usAddress 要操作的寄存器起始地址。
 * @param usNCoils 要操作的位数。
 * @param eMode 操作模式，可以是读或写。
 *
 * @return 返回操作结果，如果成功则返回MB_ENOERR，否则返回相应的错误代码。
 */
eMBErrorCode eMBRegCoilsCB(UCHAR *pucRegBuffer, USHORT usAddress, USHORT usNCoils, eMBRegisterMode eMode)
{
    // 计算寄存器索引
    USHORT usRegIndex = usAddress - 1;
    // 用于位操作的变量
    UCHAR ucBits = 0;
    // 用于存储位状态的变量
    UCHAR ucState = 0;
    // 用于循环操作的变量
    UCHAR ucLoops = 0;

    
    // 非法检测：检查访问范围是否超出寄存器缓冲区大小。
    if ((usRegIndex + usNCoils) > REG_COILS_SIZE)
    {
        return MB_ENOREG;
    }

    // 根据操作模式执行相应的操作
    if (eMode == MB_REG_WRITE)
    {
        // 计算需要循环的次数
        ucLoops = (usNCoils - 1) / 8 + 1;
        // 写操作
        while (ucLoops != 0)
        {
            // 获取当前寄存器的状态
            ucState = *pucRegBuffer++;
            // 位操作
            ucBits = 0;
            // 遍历每个位
            while (usNCoils != 0 && ucBits < 8)
            {
                // 将状态写入寄存器缓冲区
                REG_COILS_BUF[usRegIndex++] = (ucState >> ucBits) & 0X01;
                // 更新剩余位数
                usNCoils--;
                // 更新位索引
                ucBits++;
            }
            // 更新循环次数
            ucLoops--;
        }
    }
    else
    {
        // 计算需要循环的次数
        ucLoops = (usNCoils - 1) / 8 + 1;
        // 读操作
        while (ucLoops != 0)
        {
            // 初始化状态变量
            ucState = 0;
            // 位操作
            ucBits = 0;
            // 遍历每个位
            while (usNCoils != 0 && ucBits < 8)
            {
                // 根据寄存器缓冲区的状态更新状态变量
                if (REG_COILS_BUF[usRegIndex])
                {
                    ucState |= (1 << ucBits);
                }
                // 更新剩余位数
                usNCoils--;
                // 更新寄存器索引
                usRegIndex++;
                // 更新位索引
                ucBits++;
            }
            // 将状态写入寄存器缓冲区
            *pucRegBuffer++ = ucState;
            // 更新循环次数
            ucLoops--;
        }
    }

    return MB_ENOERR;
}

/**
 * @brief CMD2命令处理回调函数
 * 
 * 该函数用于处理MODBUS协议中的CMD2命令，主要负责读取离散输入寄存器的值。
 *
 * @param pucRegBuffer 指向存放寄存器数据的缓冲区。
 * @param usAddress 寄存器的起始地址。
 * @param usNDiscrete 要读取的离散输入寄存器的数量。
 *
 * @return 返回执行结果，如果成功则返回MB_ENOERR，否则返回相应的错误代码。
 */
eMBErrorCode eMBRegDiscreteCB(UCHAR *pucRegBuffer, USHORT usAddress, USHORT usNDiscrete)
{
    // 计算寄存器索引，从0开始
    USHORT usRegIndex = usAddress - 1;
    // 用于处理位操作的变量
    UCHAR ucBits = 0;
    // 用于存储当前寄存器状态的变量
    UCHAR ucState = 0;
    // 用于控制循环次数的变量
    UCHAR ucLoops = 0;

    // 非法检测：检查访问范围是否超出寄存器缓冲区大小
    if ((usRegIndex + usNDiscrete) > REG_DISC_SIZE)
    {
        return MB_ENOREG;
    }

    // 计算需要循环的次数，每次循环处理最多8个离散输入
    ucLoops = (usNDiscrete - 1) / 8 + 1;
    // 循环读取离散输入寄存器的值
    while (ucLoops != 0)
    {
        ucState = 0;
        ucBits = 0;
        // 读取每个寄存器的值，并将其状态更新到ucState变量中
        while (usNDiscrete != 0 && ucBits < 8)
        {
            if (REG_DISC_BUF[usRegIndex])
            {
                ucState |= (1 << ucBits);
            }
            usNDiscrete--;
            usRegIndex++;
            ucBits++;
        }
        // 将读取到的状态值存入缓冲区中
        *pucRegBuffer++ = ucState;
        ucLoops--;
    }

    // 模拟离散量输入被改变，这里简单地将每个寄存器的状态取反
    for (usRegIndex = 0; usRegIndex < REG_DISC_SIZE; usRegIndex++)
    {
        REG_DISC_BUF[usRegIndex] = !REG_DISC_BUF[usRegIndex];
    }

    return MB_ENOERR;
}

/**
 * @brief 将 Modbus 库错误码映射为 Modbus 异常码
 * 
 * @param error Modbus 库错误码（eMBErrorCode）
 * @return 对应的 Modbus 异常码
 */
uint8_t mapErrorToException(eMBErrorCode error)
{
    switch (error)
    {
        case MB_ENOREG:     // 非法寄存器地址
            return 0x02;    // 非法数据地址
        case MB_EINVAL:     // 非法参数
            return 0x03;    // 非法数据值
        case MB_ENORES:     // 资源不足
            return 0x04;    // 从站设备故障
        case MB_ETIMEDOUT:  // 超时
            return 0x06;    // 从站设备忙
        case MB_EPORTERR:   // 端口错误
            return 0x04;    // 从站设备故障
        case MB_ENOERR:     // 无错误
            return 0x00;    // 无异常
        default:            // 未知错误
            return 0x04;    // 从站设备故障
    }
}


```



### 去除断言依赖
编译时代码会报错，在keil的Misc Controls里加上
```
-DNDEBUG
```

![[Pasted image 20260614161356.png]]