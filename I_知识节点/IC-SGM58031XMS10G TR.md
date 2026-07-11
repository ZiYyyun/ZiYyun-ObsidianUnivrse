#IC 

[SGM58031XMS10G/TR中文资料_最新报价_数据手册下载_SGMICRO(圣邦微)-模数转换芯片ADC-立创商城](https://item.szlcsc.com/734851.html?alichlgref=https%3A%2F%2Fwww.bing.com%2F)

是TI ads1115的国产平替
是一款低功耗、16位、精密的ΔΣ模数转换器（ADC）。它的供电电压范围为3V至5.5V。包含片上参考电压源和振荡器。它具有一个I²C兼容接口，并且可以选择四个I²C从地址。滤波器的数据速率高达960SPS。具有片上可编程增益放大器（PGA），可从电源提供低至±256mV的输入范围。输入多路复用器支持4路单端输入或2路差分输入配置。有绿色MSOP-10和TDFN-3×3-10L封装。工作环境温度范围为-40℃至 + 125℃。
[[IC-SGMICRO-SGM58031XMS10G-TR.pdf]]

---

### 引脚定义


![[IC-SGMICRO-SGM58031XMS10G-TR.pdf#page=3&rect=46,548,579,729&color=yellow|IC-SGMICRO-SGM58031XMS10G-TR, p.3]]


| 引脚            | 功能                  | 描述                                  |
| ------------- | ------------------- | ----------------------------------- |
| `ADDR`        | I2C设备地址选择           | /                                   |
| `ALERT/RDY`   | 中断/转换完成通知脚          | - ADC转换完成时，该引脚拉低<br>- 电压阈值报警        |
| `AIN`         | 模拟输入通道              | /                                   |
| `SCL/SDA`     | I2C时钟/数据线           | /                                   |
| `Vdd`         | 电源正极                | 为芯片供电                               |
| `AIN3/VREFIN` | 模拟输入通道 3 / 外部参考电压输入 | 芯片默认使用内部2.048V基准，此时这个脚就当作普通的AIN3通道。 |

### 寄存器地址

| Address | Register                                             | 描述                  |
| ------- | ---------------------------------------------------- | ------------------- |
| 0x0     | [[IC-SGM58031XMS10G TR#转换寄存器\| Conversion Register]] | 存储 ADC 转换结果的 16 位数据 |
| 0x01    | Config Register                                      |                     |
| 0x02    | Low_Thresh Register                                  |                     |
| 0x03    | High_Thresh Register                                 |                     |
| 0x04    | Config1 Register                                     |                     |
| 0x05    | Chip_ID Register                                     |                     |
| 0x06    | GN_Trim1 Register for EXT_REF                        |                     |

#### 设备地址
> [!PDF|yellow] [[IC-SGMICRO-SGM58031XMS10G-TR.pdf#page=18&selection=41,0,47,36&color=yellow|IC-SGMICRO-SGM58031XMS10G-TR, p.18]]
> > The SGM58031 has a separate address setting pin ADDR, which can be connected to GND, VDD, SDA and SCL. Table 8 shows the four available addresses.
> 
> 
- 如果 ADDR 接 GND，地址为 `1001000` (即 7位地址 `0x48`)。
- 如果 ADDR 接 VDD，地址为 `1001001` (即 `0x49`)。



#### 转换寄存器

![[IC-SGMICRO-SGM58031XMS10G-TR.pdf#page=19&rect=41,382,556,432&color=yellow|IC-SGMICRO-SGM58031XMS10G-TR, p.19]]

16位的ADC转换结果在这个寄存器存放
#### 配置寄存器

![[IC-SGMICRO-SGM58031XMS10G-TR.pdf#page=20&rect=41,213,556,674&color=yellow|IC-SGMICRO-SGM58031XMS10G-TR, p.20]]



### 通讯时序
通讯时序在手册的P12
> ([[IC-SGMICRO-SGM58031XMS10G-TR.pdf#page=12&selection=14,0,14,32&color=yellow|IC-SGMICRO-SGM58031XMS10G-TR, p.12]])
> DETAILED DESCRIPTION (continued)

第一页是读操作，第二页是写操作。因为[[Prot-IIC]]协议规定，读操作需要`Repeat Start`