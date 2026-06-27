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

### 引脚定义
| 引脚  | 名称     | 说明                                  |
| --- | ------ | ----------------------------------- |
| 1   | DSG    | 放电MOS驱动输出（低有效，接NMOS栅极）              |
| 2   | CHG    | 充电MOS驱动输出（低有效，接NMOS栅极）              |
| 3   | VSS    | 电源地（GND）                            |
| 4   | SDA    | I2C数据线（需外接10kΩ上拉）                   |
| 5   | SCL    | I2C时钟线（需外接10kΩ上拉）                   |
| 6   | TS1    | 温度检测1输入（NTC热敏电阻），SHIP模式唤醒信号（拉高≥2ms） |
| 7   | CAP1   | 电源滤波电容引脚（接0.1μF电容到VSS）              |
| 8   | REGOUT | 3.3V LDO输出（最大5mA，可给MCU供电）           |
| 9   | REGSRC | LDO输入源（接BAT）                        |
| 10  | BAT    | 电池组总正极输入（6V~36V）                    |
| 11  | NC     | 未连接（悬空）                             |
| 12  | VC5    | 电芯5正极采样（5节电池时用）                     |
| 13  | VC4    | 电芯4正极采                              |


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
我们需要通过IIC来进行寄存器的读写操作，那么我们需要知道两个参数：
- 芯片I2C读/写设备地址
- 芯片寄存器的地址
它们分别对应I2C参数中的`DevAddress`与`MemAddress`

> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=3&selection=Device Comparison Table|TI-BQ769x0_DataSheet 第3页]]
> 
> ## Device Comparison Table
> 







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

