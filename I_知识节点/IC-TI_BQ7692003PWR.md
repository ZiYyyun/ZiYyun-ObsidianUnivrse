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

### 寄存器

#### 系统状态与告警（只读 ×1）

|寄存器|说明|
|---|---|
|`SYS_STAT`|核心状态寄存器。反映芯片是否就绪，以及是否触发过压（OV）、欠压（UV）、短路放电（SCD）、过流放电（OCD）等告警。|

---

#### 系统控制与配置（读写 ×3）

|寄存器|说明|
|---|---|
|`SYS_CTRL1`|控制 ADC 采样、温度传感器选择、负载检测，以及手动关闭充放电 MOSFET（SHUT_A / SHUT_B）。|
|`SYS_CTRL2`|充放电 MOSFET 软件开关（DSG_ON / CHG_ON），以及库仑计的使能与单次触发开关。|
|`CC_CFG`|芯片内部配置寄存器，**必须写入固定值 0x19**，写错可能导致芯片无法正常工作。|

---

#### 保护阈值与延迟配置（读写 ×5）

|寄存器|说明|
|---|---|
|`PROTECT1`|配置短路放电（SCD）触发阈值及触发后延迟时间。|
|`PROTECT2`|配置过流放电（OCD）触发阈值及触发后延迟时间。|
|`PROTECT3`|配置欠压（UV）和过压（OV）保护前的延迟过滤时间，防止瞬间波动误报。|
|`OV_TRIP`|单体电芯过压保护阈值，超过此值触发告警。|
|`UV_TRIP`|单体电芯欠压保护阈值，低于此值触发告警。|

---

#### 电池均衡控制（读写 ×3）

|寄存器|说明|
|---|---|
|`CELLBAL1` / `CELLBAL2` / `CELLBAL3`|每个 Bit 对应一颗电芯，写 1 即开启该节电芯的被动均衡放电，拉平整组电压。|

---

#### 单体电芯电压（只读，共 15 组）

|寄存器|说明|
|---|---|
|`VC1_HI` / `VC1_LO` ~ `VC15_HI` / `VC15_LO`|每节电芯占 2 个寄存器：HI 存高 6 位，LO 存低 8 位，拼接后即为该节电芯实时电压。最多支持 15 串。|

---

#### 总电压 / 温度 / 电流（只读）

|寄存器|说明|
|---|---|
|`BAT_HI` / `BAT_LO`|整组电池总电压，高低字节组合。|
|`TS1_HI` / `TS1_LO`、`TS2_HI` / `TS2_LO`|第 1、2 路温度传感器采样值，高低字节组合。|
|`TS3_HI` / `TS3_LO`|仅 BQ76940 型号存在，用于第 3 路温度监控（该型号串数更多，需要额外测温点）。|
|`CC_HI` / `CC_LO`|实时充放电电流（库仑计采样值），16 位高低字节组合，配合公式可换算为安培。|

---

#### ADC 校准与补偿（读写 ×3）

| 寄存器                                     | 说明                                                                   |
| --------------------------------------- | -------------------------------------------------------------------- |
| `ADCCGAIN1` / `ADCCGAIN2` / `ADCOFFSET` | 用于补偿芯片内部 ADC 的器件误差。系统初始化时须写入计算好的增益和偏移值，写入后不参与日常循环读取，但直接影响电压/电流的读数精度。 |

---


### 低功耗&模式
#### SHIP模式
> 进入方法：
- 扣电池 
- 特定的`IIC`命令
SHIP模式可以把所有寄存器重置

>唤醒：

TS1引脚拉高 ≥2ms

#### Normal模式

#### Sleep模式
> 进入方法：
- `SYS_CTRL1(0x04)` 的 `SLEEP` 位 = 1

- ADC停止工作，库仑计可继续运行（需 `CC_EN=1`）
- **唤醒**：ALERT触发 或 TS1拉高






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
