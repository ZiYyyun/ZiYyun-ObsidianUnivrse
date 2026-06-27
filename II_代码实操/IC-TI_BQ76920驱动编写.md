#IC #TI 
#实操/开发/嵌入式/STM32/项目/BMS 

### 读数据
	uint8_t Int_BQ769_ReadRegister(uint16_t mem_addr, uint8_t *buffer, uint16_t sizes)

>申请数组
```c
uint8_t *tmp_array = pvPortMalloc(sizes * 2);
    if (tmp_array == NULL)
    {
        COM_DEBUG_LN("malloc失败");
        return 0;
    }
```


> I2C读数据操作
```c    
    taskENTER_CRITICAL();
    HAL_StatusTypeDef ret = HAL_I2C_Mem_Read(&hi2c2, 0x10, mem_addr, I2C_MEMADD_SIZE_8BIT, tmp_array, sizes * 2, 1000);
    taskEXIT_CRITICAL();

    if (ret != HAL_OK)
    {

        COM_DEBUG_LN("I2C读取失败");
        vPortFree(tmp_array);
        return 0;
    }
```
查询数据手册，得到读寄存器的功能码[[TODO]]

> [!Warning] 获取到的数据流
> 由[[IC-TI_BQ769x0#读寄存器]]可知，读取到的数据如下：
> ```
> [数据0][CRC0] [数据1][CRC1] [数据2][CRC2]
> ```


>CRC校验操作

>校验第一组数据

这里的`sizes`表示有几个字节，每个字节就是一个数组
```c  
    for (uint8_t i = 0; i < sizes; i++)
    {
        // 进行第一个CRC校验
        if (i == 0)  //第一组
        {
            // 1.计算出CRC数值与人家从设备返回的CRC比较
            uint8_t crc_array[2] = {0x10 + 1, tmp_array[0]};      //把I2C设备读地址和收到的数据一起校验
            uint8_t crc = crc8(crc_array, 2);                     //送去校验，得出结果存到crc
            // CRC校验失败
            if (crc != tmp_array[1])
            {
                vPortFree(tmp_array);
                tmp_array = NULL;
                COM_DEBUG_LN("CRC校验失败");
                return 0;
            }
        }
        
```
第一组比较特殊，因为[[TODO]]芯片规定：第一组的I2C数据的CRC是由`I2C功能位`和`接收到的数据`同时校验得出。所以第一组和其余数据分开来算

>校验其他数据
```c
        else     //第二组(i == 1)
        {
            // 进行后续的CRC校验
            uint8_t crc = crc8(&tmp_array[i * 2], 1);   //对第一组之后的数据进行校验（因为芯片规定，仅第一组需要带上I2C设备地址校验）
            if (crc != tmp_array[i * 2 + 1])            
            {
                vPortFree(tmp_array);
                tmp_array = NULL;
                COM_DEBUG_LN("CRC校验失败");
                return 0;
            }
        }
```
这里就不像第一组那么复杂了，直接拿着数据计算就好
`i*2`就能得到每个数据所对应的下标，`i*2+1`：CRC校验码紧随其后，所以+1


>校验完成
```c
        // data CRC  data1 CRC  data2 CRC
        // CRC校验通过-添加到buffer数组里面
        buffer[i] = tmp_array[2 * i];
    }
    // 切记释放内存,切记释放内存,栈溢出了!!!!
    vPortFree(tmp_array);
    tmp_array = NULL;
    return 1;
```
计算完成后，把数据放到`buffer`里，最后释放内存


### 获取Gain和Offset
	static void App_Battery_GetGainAndOffset(void)

> 读取Gain
```c
static void App_Battery_GetGainAndOffset(void)
{
    uint8_t adc_gain1_val = 0;
    uint8_t adc_gain2_val = 0;
    //--------------------------------------------------
    // 读取ADCGAIN1寄存器
    //--------------------------------------------------
    if (!Int_BQ769_ReadRegister(BQ_ADCGAIN1, &adc_gain1_val, 1))
    {
        COM_DEBUG_LN("Read ADCGAIN1 Failed");
        return;
    }
    //--------------------------------------------------
    // 读取ADCGAIN2寄存器
    //--------------------------------------------------
    if (!Int_BQ769_ReadRegister(BQ_ADCGAIN2, &adc_gain2_val, 1))
    {
        COM_DEBUG_LN("Read ADCGAIN2 Failed");
        return;
    }
```

>计算Gain
```c
    gain = (365 + (adc_gain2_val >> 5) | ((adc_gain1_val & 0x0C) << 1)) / 1000.0;
```

这里参考手册中的计算公式：
> [!PDF]
> [[III_资源仓库/Data_SHOUCE/TI-BQ769x0_DataSheet.pdf#page=38&selection=GAIN = 365 μV/LSB + (ADCGAIN<4:0>in decimal) × (1 μV/LSB)|TI-BQ769x0_DataSheet 第38页]]
> 
> GAIN = 365 μV/LSB + (ADCGAIN<4:0>in decimal) × (1 μV/LSB)

我们需要拿到的是`ADCGAIN<4:0>`，目前获取到了`ADCGAIN<2:0>` 和 `ADCGAIN<4:3>` ，
![[TI-BQ769x0_DataSheet.pdf#page=31&rect=55,707,557,721&color=yellow|TI-BQ769x0_DataSheet, p.31]]
![[TI-BQ769x0_DataSheet.pdf#page=31&rect=54,604,558,644&color=yellow|TI-BQ769x0_DataSheet, p.31]]
因为我们I2C读的是8Bit数据，所以位移之后需要拼接一下，即
- 把`ADCGAIN2`右移5位
- 把`ADCGAIN1`右移2位
然后根据公式计算，获得`Gain`值

> 读取Offset
```c
    if (!Int_BQ769_ReadRegister(BQ_ADCOFFSET, (uint8_t *)&offset, 1))
    {
        COM_DEBUG_LN("Read ADCOFFSET Failed");
        return;
    }
    COM_DEBUG_LN("Gain = %.3f mV/LSB", gain);
    COM_DEBUG_LN("Offset = %d mV", offset);
}
```

### 获取电压和电流
#### 获取各个电池的电压
	void App_Battery_GetV(void)

```c
    uint8_t temp_array[BATTERY_NUM * 2] = {0};
    if (Int_BQ769_ReadRegister(BQ_VC1_HI, temp_array, BATTERY_NUM * 2) == 0)
    {
        printf("getting volita crc faild");
        return;
    }

    for (size_t i = 0; i < BATTERY_NUM; i++)
    {
        battery_array[i] = (gain * (temp_array[2 * i + 1] | temp_array[2 * i] << 8) + offset) / 1000.0;
        printf("battery NO.%d,V:%fv\r\n", i + 1, battery_array[i]);
    }
```
我们声明8位的数组，用于存储数据，通过I2C读取`VC`寄存器获取ADC值，然后通过公式转换为电压。
需要注意的芯片的寄存器，`VC1_HI`在前，`VC1_LO`在后，所以我们需要位移并拼接一下，才能得到完整的一节ADC值：
> 把HI左移8位，再把LO放到低位

> Q：把一个8位的数组左移8位，不会变成0吗？
> A：[[寄存器操作技巧--位运算#整数提升]]

#### 获取总电压
	void App_Battery_GetTotalV(void)

```c
    uint8_t temp_array[BATTERY_NUM * 2] = {0};
    if (Int_BQ769_ReadRegister(BQ_BAT_HI, temp_array, BATTERY_NUM * 2) == 0)
    {
        printf("getting volita crc faild");
        return;
    }
  
    for (size_t i = 0; i < BATTERY_NUM; i++)
    {
        battery_array[i] = (gain * (temp_array[1] | temp_array[0] << 8) + offset) / 1000.0;
        printf("battery total,V:%fv\r\n", i + 1, battery_array[i]);
    }
}
```

#### 获取温度
	void App_battery_GetTemp(void)

```c
    // 读取温度寄存器
    uint8_t tmp_array[2] = {0};
    if (Int_BQ769_ReadRegister(BQ_TS1_HI, tmp_array, 2) == 0)
    {
        printf("读取温度寄存器CRC校验失败\r\n");
        return;
    }
    // ts1引脚电压
    float ts1_vtsx = ((tmp_array[1] | tmp_array[0] << 8) * 382) / 1000000.0; // 转换成V
    // 热敏电阻的阻值
    float thermistor_rtx = (10000 * ts1_vtsx) / (3.3 - ts1_vtsx);
    // 获取电池组温度
    battery_temp = Com_BQ769_GetTemperatureByRes((int)thermistor_rtx);
    printf("电池组温度:%d", battery_temp);
```

#### 获取电流
	void App_Battery_GetCurrentA(void)

```c
    uint8_t tmp_array[2] = {2};
    if (Int_BQ769_ReadRegister(BQ_CC_HI, tmp_array, 2) == 0)
    {
        printf("读取电流CRC校验失败\r\n");
        return;
    }
    // 获取ADC数值,不要负数
    int16_t adc_val = tmp_array[1] | (tmp_array[0] << 8);
    if (adc_val < 0)
    {
        adc_val = -adc_val;
    }
    // 计算出电流大小
    battery_A = ((adc_val * 8.44) / 1000.0) / 5;
    printf("电流大小:%f\r\n", battery_A);
}
```
### 其它功能配置
	static void App_Battery_OtherConfig(void)

> `Sys_CTRL1`寄存器
```c
    // 使用外部NTC测量电池温度
    bq_register.SysCtrl1.SysCtrl1Bit.TEMP_SEL = 1;
    // 打开ADC
    bq_register.SysCtrl1.SysCtrl1Bit.ADC_EN = 1;
    Int_BQ769_WriteRegister( BQ_SYS_CTRL1, bq_register.SysCtrl1.SysCtrl1Byte);
```


> `SYS_CTRL2` 
```c
    // 开启库仑计（Charge Counter）
    bq_register.SysCtrl2.SysCtrl2Bit.CC_EN = 1;
    // 允许充电MOS导通
    bq_register.SysCtrl2.SysCtrl2Bit.CHG_ON = 1;
    // 允许放电MOS导通
    bq_register.SysCtrl2.SysCtrl2Bit.DSG_ON = 1;
    Int_BQ769_WriteRegister( BQ_SYS_CTRL2, bq_register.SysCtrl2.SysCtrl2Byte);
```

> `Protect1` 短路保护配置
```c
    // 电流采样电阻配置
    // 1 -> 5mΩ
    // 0 -> 20mΩ
    bq_register.Protect1.Protect1Bit.RSNS = 1;
    // 短路保护阈值
    bq_register.Protect1.Protect1Bit.SCD_THRESH = 0x7;
    // 短路保护延时
    bq_register.Protect1.Protect1Bit.SCD_DELAY = BMS_SCD_DELAY_400us;
    Int_BQ769_WriteRegister( BQ_PROTECT1, bq_register.Protect1.Protect1Byte);
```


> `Protect2` 放电过流保护
```c
    // 放电过流阈值
    bq_register.Protect2.Protect2Bit.OCD_THRESH = 0xF;
    // 放电过流延时
    bq_register.Protect2.Protect2Bit.OCD_DELAY = BMS_OCD_DELAY_1280ms;
    Int_BQ769_WriteRegister( BQ_PROTECT2, bq_register.Protect2.Protect2Byte);
```


> `Protect3` 过压欠压保护延时
```c
    // 欠压保护延时
    bq_register.Protect3.Protect3Bit.UV_DELAY = BMS_UV_DELAY_16s;
    // 过压保护延时
    bq_register.Protect3.Protect3Bit.OV_DELAY = BMS_OV_DELAY_8s;
    Int_BQ769_WriteRegister( BQ_PROTECT3, bq_register.Protect3.Protect3Byte);
```


> 配置过压保护阈值（OV）
```c
    // TI数据手册公式：
    // Register = ((Voltage ×1000) - Offset) / Gain
    // 最终寄存器只保存高8位，因此右移4位
    uint16_t ov = (OV_PROTECT * 1000.0f - offset) / gain;
    uint8_t ov_result = (ov >> 4) & 0xFF;
```




> 配置欠压保护阈值（UV）
```c
    uint16_t uv = (UV_PROTECT * 1000.0f - offset) / gain;
    uint8_t uv_result = (uv >> 4) & 0xFF;
```


> 写入OV、UV寄存器
```c
    Int_BQ769_WriteRegister(BQ_OV_TRIP, ov_result);
    Int_BQ769_WriteRegister(BQ_UV_TRIP, uv_result);
```


> 配置库仑计
```c
    Int_BQ769_WriteRegister(BQ_CC_CFG, 0x19);
}
```



## 流程图
```
                App_Battery_Init()
                        │
        ┌───────────────┴────────────────┐
        │                                │
        ▼                                ▼
 Int_BQ769_Init()              读取 Gain / Offset
 (唤醒芯片、进入工作模式)                   │
        │                                │
        └───────────────┬────────────────┘
                        ▼
               配置系统控制寄存器
          ┌─────────────────────────┐
          │ SYS_CTRL1               │
          │ • 开启ADC                │
          │ • 外部NTC测温            │
          └─────────────────────────┘
                        │
                        ▼
          ┌─────────────────────────┐
          │ SYS_CTRL2               │
          │ • 开启库仑计(CC)          │
          │ • 开启充电MOS            │
          │ • 开启放电MOS            │
          └─────────────────────────┘
                        │
                        ▼
          ┌─────────────────────────┐
          │ PROTECT1~3              │
          │ • 短路保护(SCD)          │
          │ • 放电过流(OCD)          │
          │ • 过压/欠压延时           │
          └─────────────────────────┘
                        │
                        ▼
          ┌─────────────────────────┐
          │ OV_TRIP / UV_TRIP       │
          │ 写入保护阈值              │
          └─────────────────────────┘
                        │
                        ▼
               BQ769开始正常工作
```