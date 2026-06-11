#理论/嵌入式/STM32 


> [!重点] 推挽和开漏的区别
>同样都是代码写「输出高电平」：
> 1. **推挽模式**
> 2. MCU 内部主动发力 → 引脚稳稳输出 `3.3V`
> 3. **开漏模式**
> 4. MCU 内部直接摆烂断开 → 引脚啥电压都没有
> 5. **电平完全由你外部接的上拉电阻决定**

---

### GPIO的基本结构
- `Vdd`: 高电平(3.3V)
- `Vss`: 低电平(0V/接地)
![[Pasted image 20240910143610.png]]
```c
typedef enum
{
    GPIO_Mode_AIN = 0x0,           // 模拟输入
    GPIO_Mode_IN_FLOATING = 0x04,  // 浮空输入
    GPIO_Mode_IPD = 0x28,          // 下拉输入
    GPIO_Mode_IPU = 0x48,          // 上拉输入
    GPIO_Mode_Out_OD = 0x14,       // 开漏输出
    GPIO_Mode_Out_PP = 0x10,       // 推挽输出
    GPIO_Mode_AF_OD = 0x1C,        // 复用开漏输出
    GPIO_Mode_AF_PP = 0x18         // 复用推挽输出
} GPIOMode_TypeDef;
```
### GPIO的模式

| 模式类型                               | 配置描述                        |
| ---------------------------------- | --------------------------- |
| **浮空输入 (`GPIO_Mode_IN_FLOATING`)** | 引脚电平是真实的外部连接器件电压，电平有不确定性    |
| **上拉输入 (`GPIO_Mode_IPU`)**         | 默认通过电阻上拉到VCC，不接外部器件时可以读出高电平 |
| **下拉输入 (`GPIO_Mode_IPD`)**         | 默认通过电阻下拉到GND，不接外部器件时可以读出低电平 |
| **模拟输入 (`GPIO_Mode_AIN`)**         | 将外部信号直接传输到数模转换通道上           |
| **开漏输出 (`GPIO_Mode_Out_OD`)**      | 只能输出低电平，高电平由电阻上拉决定          |
| **推挽式输出 (`GPIO_Mode_Out_PP`)**     | 可以输出强高和强低，通常使用该功能控制LED      |
| **开漏复用功能 (`GPIO_Mode_AF_OD`)**     | 用于外设功能使用，如I2C的SCL和SDA线      |
| **推挽式复用功能 (`GPIO_Mode_AF_PP`)**    | 用于外设功能使用，如UART的TX和RX线       |
配置引脚
```c
GPIO_InitStruct.Pin = GPIO_PIN_1; 
```
配置模式
```c
GPIO_InitStruct.Mode = GPIO_MODE_INPUT;  \\上拉输入
GPIO_InitStruct.Modr = GPIO_MODE_OUTPUT_OD;  \\开漏输出
GPIO_InitStruct.Modr = GPIO_MODE_OUTPUT_PP;  \\推挽输出
GPIO_InitStruct.Modr = GPIO_MODE_ANALOG;
```
配置上拉/下拉/复用
```c
GPIO_InitStruct.Pull = GPIO_PULLUP; 
```
初始化GPIO
```c
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
```
 - **输入模式**
---
#### 浮空输入（GPIO_Mode_IN_FLOATING）

> [!NOTE] 浮空输入
>  浮空就是逻辑器件与引脚即不接高电平，也不接低电平。由于逻辑器件的内部结构和外部引脚所接的器件决定电平状态。一般实际运用时，引脚不建议悬空，易受干扰。通俗讲就是浮空就是浮在空中，就相当于此端口在默认情况下什么都不接，呈高阻态，这种设置在数据传输时用的比较多。浮空最大的特点就是电压的不确定性，它可能是0V，页可能是VCC，还可能是介于两者之间的某个值（最有可能） 浮空一般用来做ADC输入用，这样可以减少上下拉电阻对结果的影响。![[Pasted image 20240910145728.png]]

 

#### 上拉输入 (GPIO_Mode_IPU)

> [!NOTE] 上拉输入
> 上拉就是把点位拉高，比如拉到Vcc。上拉就是将不确定的信号通过一个电阻嵌位在高电平。电阻同时起到限流的作用。弱强只是上拉电阻的阻值不同，没有什么严格区分。
![[Pasted image 20240910145844.png]]

#### 下拉输入 (GPIO_Mode_IPD)

> [!NOTE] 下拉输入
> 下拉就是把点位拉低，比如拉到GND。下拉就是将不确定的信号通过一个电阻嵌位在低电平。电阻同时起到限流的作用。弱强只是下拉电阻的阻值不同，没有什么严格区分![[Pasted image 20240910150021.png]]
#### 模拟输入 (GPIO_Mode_AIN)

> [!NOTE] 模拟输入
> 模拟输入是指传统方式的输入，数字输入是输入PCM数字信号，即0,1的二进制数字信号，通过数模转换，转换成模拟信号，经前级放大进入功率放大器，功率放大器还是模拟的![[Pasted image 20240910150153.png]]



 - **输出模式**
---

#### 开漏输出(GPIO_Mode_Out_OD)

> [!NOTE] 开漏输出
> IO输出0接GND，IO输出1，悬空，需要外接上拉电阻，才能实现输出高电平。当输出为1时，IO口的状态由上拉电阻拉高电平，但由于是开漏输出模式，这样IO口也就可以由外部电路改变为低电平或不变。可以读IO输入电平变化，实现C51的IO双向功能。只能输出强低电平。![[Pasted image 20240910150238.png]]
#### 推挽式输出(GPIO_Mode_Out_PP)

> [!NOTE] 推挽输出
> IO输出0-接GND， IO输出1 -接VCC。这是使用最多的了。控制LED基本都是使用这种模式。
可以输出强高低电平，连接外部数字器件![[Pasted image 20240910150454.png]]

#### 开漏复用功能(GPIO_Mode_AF_OD)

> [!NOTE] 开漏复用
> 用于外设使用![[Pasted image 20240910150408.png]]

#### 推挽式复用功能(GPIO_Mode_AF_PP)

> [!NOTE] 推挽复用
> 用于外设使用![[Pasted image 20240910150724.png]]


### 推挽和开漏

> [!NOTE] GPIO接法
> 当引脚被写入`1`时: `Vdd`导通且`Vss`断路，`IO`引脚输出低电平
> 当引脚被写入`0`时: `Vdd`断路且`Vss`导通，`IO`引脚输出高电平
> ** 写`1`导上面 写`0`导下面 **
> ![[Pasted image 20240910143442.png]]

