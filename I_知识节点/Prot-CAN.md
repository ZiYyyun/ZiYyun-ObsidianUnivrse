#can #Prot  #理论/嵌入式/STM32 




### 数据格式

| 类型    | Identifier | IDE | RTR | DATA | 用途   |
| ----- | ---------- | --- | --- | ---- | ---- |
| 标准数据帧 | 11位        | 0   | 0   | 有    | 发送数据 |
| 扩展数据帧 | 29位        | 1   | 0   | 有    | 发送数据 |
| 标准远程帧 | 11位        | 0   | 1   | 无    | 请求数据 |
| 扩展远程帧 | 29位        | 1   | 1   | 无    | 请求数据 |


> 其实这四种帧除了Identifier和RTR这两个有区别，别的都一样，其中`Identifier`由`IDE`来决定是标准还是扩展

> ID可以决定优先级，ID越小，优先级越高

#### 数据帧
```
标有下划线的数据位，由stm32硬件自动完成
┌────┬────────┬────┬────┬────┬────┬──────────┬─-───────┬─--──┬─--──┐
│SOF │ Identifier  │RTR │IDE │ r0 │DLC │ DATA│   CRC   │ ACK │ EOF │
└────┴────────┴────┴────┴────┴────┴──────────┴─-───────┴──--─┴───--┘
------                                        ----------------------
```

- `SOF(start of frame)`：开始位，只有1bit
- `Identifier`:是什么数据（自己定义的） 11bit
- `RTR(remote transmit request)`：只有两个值，要么0（Data Frame）要么1（Remote Frame）
- `IDE(Identifier Extension)`：0/1 -- 决定标准帧/扩展帧
- `DLC(Data Length Code)`：数据的长度 byte
- `Data`：数据
- `CRC`：循环冗余校验。由控制器自动完成
- `Ack`：确认位
- `EOF(End of Frame)`：结束

#### 远程帧
```
┌────┬────────┬────┬────┬────┬────┬──────┬────┬────┐
│SOF │ Std ID │RTR │IDE │ r0 │DLC │ CRC  │ACK │EOF │        
└────┴────────┴────┴────┴────┴────┴──────┴────┴────┘

┌────┬──────────────┬────┬────┬────┬────┬──────┬────┬────┐
│SOF │ Ext ID(29bit)│RTR │IDE │r1/r0│DLC│ CRC  │ACK │EOF │
└────┴──────────────┴────┴────┴────┴────┴──────┴────┴────┘
```


> [!NOTE] Note
> ID也可以决定优先级，ID越小优先级越高
> CAN有两种ID：11bit的标准帧和29bit的扩展帧（汽车常用）



### 配置流程

```
初始化CAN

设置波特率

配置过滤器

启动CAN

打开RX中断

收到消息

读取FIFO

解析Data
```

### 主要函数
#### 发送数据
> 发送函数
```c
HAL_StatusTypeDef HAL_CAN_AddTxMessage(CAN_HandleTypeDef *hcan, const CAN_TxHeaderTypeDef *pHeader, const uint8_t aData[], uint32_t* pTxMailbox)
```

```
CPU
 │
 │ HAL_CAN_AddTxMessage()
 ▼
Mailbox(发送邮箱)
 │
 ▼
CAN控制器
 │
 ▼
CAN总线
```


前面提到，一个完整的Frame需要配置的那几个参数，在pHeader里实现：

```c
    CAN_TxHeaderTypeDef txHeader;         
    txHeader.IDE = CAN_ID_STD;            //HAL库宏定义
    txHeader.RTR = CAN_RTR_DATA;          //HAL库宏定义
    txHeader.StdId = stdID;               //自己传
    txHeader.DLC = len;                   //自己传
```

`pTxMailbox`指的是


#### 接收数据

接收函数与发送的逻辑完全不一样，在接收数据时，主控可能在做其他事情，此时就算中断也还是会丢数据，因为CAN总线的所以需要一个缓冲区，这个缓冲区便是`FIFO(First in First Out)`
```
┌──────────┐
│ Frame1   │
├──────────┤
│ Frame2   │
├──────────┤
│ Frame3   │
└──────────┘
```
特别的是，STM32提供两个FIFO：`FIFO0` 和 `FIFO1`。过滤器将会决定数据接收到哪个。


> [!NOTE] FIFO的内部
> FIFO内部储存了一个完整的`Frame`，包括`ID``DLC``RTR``IDE``DATA`和`CRC`。等到CPU取出来之后，HAL库会帮我们拆成`CAN_RxHeaderTypeDef`和一个`uint8_t RxData[8]`



> 接收函数
```c
HAL_StatusTypeDef HAL_CAN_GetRxMessage(CAN_HandleTypeDef *hcan, uint32_t RxFifo, CAN_RxHeaderTypeDef *pHeader, uint8_t aData[])
```
这个函数总共做了四件事情：
1. 找到`FIFO`的第一帧
2. 把`Header(ID RTR IDE DLC)`拆出来，放进`CAN_RxHeaderTypeDef`类型的这个结构体变量里
3. 把`DATA`复制出来，放到数组里
4. 删除`FIFO`第一帧，因为遵循**先进先出**




#### 状态判断函数

| 函数原型                                                                       | 功能说明                         | 返回值         | 使用场景                    |
| -------------------------------------------------------------------------- | ---------------------------- | ----------- | ----------------------- |
| FlagStatus CAN_GetFlagStatus(CAN_TypeDef* CANx, uint16_t CAN_FLAG)         | 获取任意 CAN 标志位（接收 / 发送 / 错误通用） | SET/RESET   | 查询 FIFO 是否有数据、发送完成、总线错误 |
| ITStatus CAN_GetITStatus(CAN_TypeDef* CANx, uint16_t CAN_IT)               | 判断中断标志是否置 1（中断服务函数专用）        | SET/RESET   | IRQ 里区分是接收中断、错误中断还是发送中断 |
| uint8_t CAN_GetFIFOStatus(CAN_TypeDef* CANx, uint8_t FIFO_Flag)            | 查询 FIFO0/FIFO1 当前缓存帧数        | 0/1/2/3     | 判断 FIFO 存了几帧报文，预判溢出     |
| FlagStatus CAN_GetFIFOOverflowStatus(CAN_TypeDef* CANx, uint8_t FIFO_Flag) | 查询 FIFO 溢出标志                 | SET = 溢出丢帧  | 检测高速报文覆盖旧数据             |
| FlagStatus CAN_GetFIFOFullStatus(CAN_TypeDef* CANx, uint8_t FIFO_Flag)     | 查询 FIFO 已满（3 帧）              | SET=FIFO 塞满 | 提前预警即将溢出                |



### CAN的中断

