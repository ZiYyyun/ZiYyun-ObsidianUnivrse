#IC      #TI        #BQ76920

#实操/开发/嵌入式/STM32/项目/BMS 

芯片中文描述：3 至 5 节电池锂离子和锂磷酸盐电池监视器
[BQ76920 数据表、产品信息和支持 | 德州仪器 TI.com.cn](https://www.ti.com.cn/product/cn/BQ76920)

---


### 核心参数
| 参数               | 值                             |
| ---------------- | ----------------------------- |
| 封装               | TSSOP-20                      |
| 串联电芯数            | 3~5节                          |
| 工作电压范围           | 6V ~ 36V (BAT引脚)              |
| 工作温度             | -40°C ~ 85°C                  |
| 通信接口             | I2C（地址 `0x18`，7-bit）          |
| I2C速率            | ≤400kHz                       |
| 电芯电压测量精度         | ±5mV                          |
| 内部LDO输出 (REGOUT) | 3.3V / 最大5mA                  |
| 库仑计数器            | 16-bit ΔΣ ADC                 |
| 硬件保护             | OV/UV/OCD/SCD                 |
| 均衡方式             | 内部MOS + 外部电阻（被动均衡），电流极小       |
| 温度通道             | 内部die温度 + 最多3个外部NTC (TS1/TS2) |


### 低功耗&模式
#### SHIP模式

进入ship模式后，所有寄存器的值将会被重置
> [!PDF] SHIP模式进入方法
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=28&selection=• Write #1: [SHUT_A] = 0, [SHUT_B] = 1 • Write #2: [SHUT_A] = 1, [SHUT_B] = 0|TI-BQ769x0_DataSheet 第28页]]
> To enter SHIP mode from NORMAL mode, the `[SHUT_A]` and `[SHUT_B]` bits in the `SYS_CTRL1` register must be written with specific patterns **across two consecutive writes**:
> ```
> • Write #1: [SHUT_A] = 0, [SHUT_B] = 1
> • Write #2: [SHUT_A] = 1, [SHUT_B] = 0
> ```

根据手册描述，我们需要往`SYS_CTRL1`寄存器的那两个位写入特定的值，并连续执行两次


>唤醒：

TS1引脚拉高 ≥2ms

#### Normal模式

#### Sleep模式
> 进入方法：
- `SYS_CTRL1(0x04)` 的 `SLEEP` 位 = 1

- ADC停止工作，库仑计可继续运行（需 `CC_EN=1`）
- **唤醒**：ALERT触发 或 TS1拉高

### 数据读写操作


> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=3&selection=Device Comparison Table|TI-BQ769x0_DataSheet 第3页]]
> I2C从机地址在第三页有记录

| 操作类型 | 第一个数据字节的 CRC     | 后续数据字节的 CRC |
| ---- | ---------------- | ----------- |
| 块写   | 从地址 + 寄存器地址 + 数据 | 仅数据字节       |
BQ769x0 实现标准的 100 kHz I2C 接口，作为从设备运行，7 位设备地址由出厂编程。所有信息都通过读写相应寄存器来传输。手册原文强调：


> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=18&selection=. Block reads and writes, buffered by an 8-bit CRC code per byte, ensure a fast and robust transmission of data.|TI-BQ769x0_DataSheet 第18页]]
> 
> Block reads and writes, buffered by an 8-bit CRC code per byte, ensure a fast and robust transmission of data.
> 

即块读写以每字节 8 位 CRC 作为==缓冲校验==，所以我们读到的数据长这样：
```
[数据0][CRC0] [数据1][CRC1] [数据2][CRC2]
```



> [!NOTE] 重复起始（Repeated Start）
> ![[TI-BQ769x0_DataSheet.pdf#page=27&rect=104,332,511,505&color=yellow|TI-BQ769x0_DataSheet, p.27]]
> 这个图片上半部分为主机发送，下半部分为从机发送返回数据
> ```
> 主机：起始信号->从机地址->寄存器地址->起始信号->从机地址
> 从机：    			                 ACK->数据->CRC->STOP
> ```
> ==起始信号之后的数据需要进行CRC校验==


#### 读操作

> 手册中的相关描述

> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=27&selection=• In a single-byte read transaction, the CRC is calculated after the second start and uses the slave address and data byte. • In a block read transaction, the CRC for the first data byte is calculated after the second start and uses the slave address and data byte. The CRC for subsequent data bytes is calculated over the data byte only.|TI-BQ769x0_DataSheet 第27页]]
> 
> • In a single-byte read transaction, the CRC is calculated after the second start and uses the slave address and data byte. 
> • In a block read transaction, the CRC for the first data byte is calculated after the second start and uses the slave address and data byte. The CRC for subsequent data bytes is calculated over the data byte only.

通过手册中的描述，读操作可以分为两类：
- 单次读取
- 块读取
区别就在于进行CRC校验时，所参与的数据：块读取或单次读取的首个字节的CRC校验，要求从机地址+数据字节

| 首字读 | 从地址 + 数据字节 |
| --- | ---------- |

>[!WARNING] 为什么读操作不校验数据
>只有写操作才会对寄存器地址进行校验
>因为对于读操作来说，寄存器地址是第一次起始时发送，不在第二次起始之后的范围
>然而写操作不需要重复起始

#### 写操作

> 手册

 > [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=27&selection=• In a single-byte write transaction, the CRC is calculated over the slave address, register address, and data. • In a block write transaction, the CRC for the first data byte is calculated over the slave address, register address, and data. The CRC for subsequent data bytes is calculated over the data byte only.|TI-BQ769x0_DataSheet 第27页]]
> 
> • In a single-byte write transaction, the CRC is calculated over the **slave address, register address, and data**. 
> • In a block write transaction, the CRC for the first data byte is calculated over the **slave address, register address, and data**. The CRC for subsequent data bytes is calculated over the data byte only.

写操作与读操作类似，块写入或单次写入的首个字节需要对从机地址、==寄存器地址==、数据进行CRC校验


| 首字写 | 从地址+寄存器地址+数据字节 |
| --- | -------------- |



### 常用换换算公式

> [!NOTE] 增益值和偏移量
> BQ769内部有许多的ADC，这些ADC寄存器并不直接返回电压值，只是ADC转换后的数字，所以公式里有`GAIN`和`OFFSET`
> 有点像 `y = kx + b`
> > `Gain`负责缩放， `Offset`负责修正。
> > 
> 这里的`Gain(增益值)`和`Offset(偏移量)`就是工厂为每颗芯片写入的校正值

#### 增益值和偏移量
> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=38&selection=ADCGAIN2:0 (Bits 7–5, address 0x59): ADC GAIN offset lower 3 LSB ADCGAIN<4:0> is a production-trimmed value for the ADC transfer function, in units of μV/LSB. The range is 365 μV/LSB to 396 μV/LSB, in steps of 1 μV/LSB, and may be calculated as follows: GAIN = 365 μV/LSB + (ADCGAIN<4:0>in decimal) × (1 μV/LSB) Alternately, a conversion table is provided below|TI-BQ769x0_DataSheet 第38页]]
> 
> ```
> GAIN = 365 μV/LSB + (ADCGAIN<4:0>in decimal) × (1 μV/LSB) 
> ```
偏移量在`ADCOFFSET(0x51)`寄存器> [!PDF]


#### 电压
> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=19&selection=V(cell) = GAIN x ADC(cell) + OFFSET |TI-BQ769x0_DataSheet 第19页]]
> 
> ```
> V(cell) = GAIN x ADC(cell) + OFFSET 
> ```

#### 电流

> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=21&selection=CC Reading (in μV) = [16-bit 2’s Complement Value] × (8.44 μV/LSB) |TI-BQ769x0_DataSheet 第21页]]
> 
> ```
> CC Reading (in μV) = [16-bit 2’s Complement Value] × (8.44 μV/LSB) 
> ```

#### 温度
> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=21&selection=CC Reading (in μV) = [16-bit 2’s Complement Value] × (8.44 μV/LSB)|TI-BQ769x0_DataSheet 第21页]]
> 
> ```
> CC Reading (in μV) = [16-bit 2’s Complement Value] × (8.44 μV/LSB)
> ```




### 均衡管理

> [!NOTE] 注意
> BQ769x0是电池管理AFE芯片，电压采集、过压/欠压保护这些核心功能，都依赖内置的14位ADC实现：
> 官方要求：配置保护、电压采集相关寄存器前，‌**最好显式开启ADC**‌。虽然芯片从SHIP模式唤醒到NORMAL模式后，这个位会默认置1，但实际调试中经常出现默认配置不生效、电压读数不稳定的问题，因此需要手动确认开启。

