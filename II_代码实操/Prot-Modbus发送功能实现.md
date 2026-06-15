#实操/开发/嵌入式/STM32/项目/步进电机 
#modbus 
#Prot 

我们参考[[]]中有关Modbus指令的详细讲解：
需要实现的功能如下：
- 读取线圈状态 (Read Coils)
- 读取离散输入 (Read Discrete Inputs)
- 读取保持寄存器 (Read Holding Registers)
- 读取输入寄存器 (Read Input Registers)
- 写单个线圈 (Write Single Coil)
- 写单个保持寄存器 (Write Single Register)
- 写多个线圈 (Write Multiple Coils)
- 写多个寄存器 (Write Multiple Registers)



## Modbus 主机驱动（发送端实现）

### 底层串口发送
==static==
	static void Modbus_Send(uint8_t *cmd, uint16_t len)
```c
    HAL_UART_Transmit(&huart2, cmd, len, 1000);

    printf("TX : ");

    for (uint16_t i = 0; i < len; i++)
        printf("%02X ", cmd[i]);

    printf("\r\n");
```

- 调用 HAL 库阻塞式串口发送函数，通过 USART2 发出完整的 Modbus 指令帧
- 附带调试打印逻辑，将发送内容按十六进制逐字节打印，方便抓包调试

### 固定 8 字节指令组帧发送
==static==
	static void Modbus_SendFrame(uint8_t id, ModbusFunctionCode_t func, uint16_t addr, uint16_t data)
```c
    uint8_t cmd[8];

    cmd[0] = id;
    cmd[1] = func;

    cmd[2] = addr >> 8;
    cmd[3] = addr;

    cmd[4] = data >> 8;
    cmd[5] = data;

    uint16_t crc = usMBCRC16(cmd, 6);

    cmd[6] = crc;
    cmd[7] = crc >> 8;

    Modbus_Send(cmd, 8);
```

- 用于组装长度固定为 8 字节的 Modbus 指令，适配读指令、写单个线圈 / 寄存器等功能码
- 帧结构按协议顺序填充：从站地址 → 功能码 → 起始地址高 / 低字节 → 数据高 / 低字节
- 对前 6 字节数据计算 CRC16 校验值，校验值**低字节在前、高字节在后**填充到帧尾
- 组帧完成后调用底层发送函数发出数据

### 读指令通用接口

	void Modbus_Read(uint8_t id, ModbusFunctionCode_t func, uint16_t addr, uint16_tnumber)
```c
    Modbus_SendFrame(id,
                     func,
                     addr,
                     number);
```

- 通用读指令封装，支持读线圈、读离散输入、读保持寄存器、读输入寄存器等功能码
- 入参：从站 ID、功能码、起始地址、读取数量，内部直接复用 8 字节组帧函数

### 写单个线圈 / 单个保持寄存器

	void Modbus_WriteSingle(uint8_t id, ModbusFunctionCode_t func,uint16_t addr, uint16_t value)

> 取低八位值
```c
    if (func == MODBUS_FUNC_WRITE_COIL)
    {
        value = value ? 0xFF00 : 0x0000;
    }    
```

>调用发送函数
```c    
    Modbus_SendFrame(id,
                     func,
                     addr,
                     value);
```

- 兼容两种功能码：写单个线圈（0x05）、写单个保持寄存器（0x06）
- 写线圈时遵循协议规范：线圈置通写入 `0xFF00`，线圈置断写入 `0x0000`
- 数值转换完成后，统一调用 8 字节组帧发送函数


### 写多个线圈（0xF）

	void Modbus_WriteCoils(uint8_t id, uint16_t addr, uint16_t num, uint8_t *coil)


> [!NOTE] Tip
> - 函数的后两个参数，第一个穿一个数组，第二个穿数组的长度
> - num表示要写入的线圈数量  因为这个函数是写多个线圈，而我们不知道有多少个 Modbus一股脑全塞进来，所以我们需要判断


> 定义变量
```c
    uint8_t byte_num = (num + 7) / 8;
    uint16_t total = 9 + byte_num;
    uint8_t *cmd = pvPortMalloc(total);
```
- `coil` 传入线圈状态数组首地址，`num` 表示要写入的线圈总数量
- `total`计算出整个指令的字节数（除了数据之外有9字节这是确定的）
- `byte_num`表示要写入的数据的字节数  因为这个函数是写多个线圈，而我们不知道有多少个 Modbus一股脑全塞进来，所以我们需要判断
- 使用 `pvPortMalloc` 动态分配帧缓冲区，避免固定数组溢出或空间浪费

>放入已知值
```c
    cmd[0] = id;
    cmd[1] = MODBUS_FUNC_WRITE_COILS;
    cmd[2] = addr >> 8;
    cmd[3] = addr;
    cmd[4] = num >> 8;
    cmd[5] = num;
    cmd[6] = byte_num;
```
- 帧头依次填充：从站地址、功能码、起始地址、线圈总数、数据字节数


> 写入各个线圈

双层循环完成位拼装：外层按==字节==遍历，内层按==位==逐个填充线圈状态
```c
    for (uint8_t i = 0; i < byte_num; i++)
    {
        cmd[7 + i] = 0;    //先把要操作的地方清零
        for (uint8_t j = 0; j < 8; j++)
        {
            uint16_t index = i * 8 + j;    //index表示第几个线圈 比如i=0就是0+j=0-7的线圈，i=1就是8-15的线圈，以此类推
            if (index < num)
            {
                if (coil[index])
                {
                    cmd[7 + i] |= (1 << j);
                }
            }
        }
    }
```
- 第一个for循环的`byte_num`就是要循环的次数 比如只有两个字节，那我们大循环两次（一次一个字节），小循环8次(一次一位，八次一个字节)就够了
- 先把要操作的地方清零
- `index`表示第几个线圈 比如i=0就是0+j=0-7的线圈，i=1就是8-15的线圈，以此类推  `index < num` 判断：防止最后一个字节不满 8 位时，越界访问线圈数组
- `index = i * 8 + j`：计算当前处理的线圈对应数组下标，实现按位映射
- `if`用于判断如果这个线圈还没写完（因为最后一个字节可能不满8位，所以我们需要判断一下）


> 添加crc校验码
```c
    uint16_t crc = usMBCRC16(cmd, total - 2);

    cmd[total - 2] = crc;
    cmd[total - 1] = crc >> 8;

    Modbus_Send(cmd, total);
    vPortFree(cmd);
```
- 计算 CRC 校验并填充帧尾，发送完成后调用 `vPortFree` 释放动态内存


> 写多个保持寄存器（功能码 0x10）

	void Modbus_WriteRegisters(uint8_t id, uint16_t addr, uint16_t *reg, uint16_t reg_num)
```c
    uint16_t byte_num = reg_num * 2;
    uint16_t total = 9 + byte_num;
    uint8_t *cmd = pvPortMalloc(total);

    cmd[0] = id;
    cmd[1] = MODBUS_FUNC_WRITE_REGISTERS;
    cmd[2] = addr >> 8;
    cmd[3] = addr;
    cmd[4] = reg_num >> 8;
    cmd[5] = reg_num;
    cmd[6] = byte_num;

    for (uint16_t i = 0; i < reg_num; i++)
    {
        cmd[7 + i * 2] = reg[i] >> 8;
        cmd[8 + i * 2] = reg[i];
    }

    uint16_t crc = usMBCRC16(cmd, total - 2);

    cmd[total - 2] = crc;
    cmd[total - 1] = crc >> 8;

    Modbus_Send(cmd, total);
    vPortFree(cmd);
```

- `byte_num = reg_num * 2`：每个保持寄存器占 2 字节，计算数据段总长度
- `total = 9 + byte_num`：整帧总长度 = 9 字节固定帧头 + 数据段长度
- 动态分配内存存放完整指令帧，适配任意数量的寄存器写入
- 帧头依次填充：从站地址、功能码、起始地址、寄存器总数、数据字节数
- 循环填充每个寄存器数值，遵循 Modbus 大端序：先写高 8 位，后写低 8 位
- 计算 CRC 校验值填充帧尾，发送完成后释放动态内存

