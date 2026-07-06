

### i2c
esp32的i2c有两个控制器

```c
/**

 * @brief I2C port number.

 */

typedef int i2c_port_num_t;
```

这个port就是选择方式：
- `I2C_NUM_0`
- `I2c_NUM_1`

各个外设的CLK宏定义在`clk_tree_defs.h

i2s
```c
typedef struct {
    int                 id;                 /*!< I2S port id */
    i2s_role_t          role;               /*!< I2S role, I2S_ROLE_MASTER or I2S_ROLE_SLAVE */
    /* DMA configurations */
    uint32_t            dma_desc_num;       /*!< I2S DMA buffer number, it is also the number of DMA descriptor */
    uint32_t            dma_frame_num;      /*!< I2S frame number in one DMA buffer. One frame means one-time sample data in all slots,
                                             *   it should be the multiple of `3` when the data bit width is 24.

                                             */
    union {

        bool            auto_clear;         /*!< Alias of `auto_clear_after_cb` */

        bool            auto_clear_after_cb; /*!< Set to auto clear DMA TX buffer after `on_sent` callback, I2S will always send zero automatically if no data to send.

                                             *   So that user can assign the data to the DMA buffers directly in the callback, and the data won't be cleared after quit the callback.

                                             */

    };

    bool                auto_clear_before_cb; /*!< Set to auto clear DMA TX buffer before `on_sent` callback, I2S will always send zero automatically if no data to send

                                             *   So that user can access data in the callback that just finished to send.

                                             */

    bool                allow_pd;           /*!< Set to allow power down. When this flag set, the driver will backup/restore the I2S registers before/after entering/exist sleep mode.

                                             * By this approach, the system can power off I2S's power domain.

                                             * This can save power, but at the expense of more RAM being consumed.

                                             */

    int                 intr_priority;      /*!< I2S interrupt priority, range [0, 7], if set to 0, the driver will try to allocate an interrupt with a relative low priority (1,2,3) */

} i2s_chan_config_t;
```


### i2s
```c
typedef struct {

    int16_t  mclk;

    int16_t  bclk;

    int16_t  ws;

    int16_t  dout;

    int16_t  din;

} codec_i2s_pin_t;
```

```
┌─────────────────┐           ┌──────────────────────────┐
│       ESP       │           │          ES8311          │
│                 │           │                          │
│       I2S_MCK_IO├──────────►│PIN2-MCLK                 │
│                 │           │                          │           ┌─────────┐
│       I2S_BCK_IO├──────────►│PIN6-BCLK       PIN12-OUTP├───────────┤         │
│                 │           │                          │           │ EARPHONE│
│        I2S_WS_IO├──────────►│PIN8-LRCK       PIN13-OUTN├───────────┤         │
│                 │           │                          │           └─────────┘
│        I2S_DO_IO├──────────►│PIN9-SDIN                 │
│                 │           │                          │
│        I2S_DI_IO│◄──────────┤PIN7-SDOUT                │
│                 │           │                          │           ┌─────────┐
│                 │           │               PIN18-MIC1P├───────────┤         │
│       I2C_SCL_IO├──────────►│PIN1 -CCLK                │           │  MIC    │
│                 │           │               PIN17-MIC1N├───────────┤         │
│       I2C_SDA_IO│◄─────────►│PIN19-CDATA               │           └─────────┘
│                 │           │                          │
│          VCC 3.3├───────────┤VCC                       │
│                 │           │                          │
│              GND├───────────┤GND                       │
└─────────────────┘           └──────────────────────────┘
```

## 各字段说明

|字段|全称|方向（主→从）|说明|
|---|---|---|---|
|`mclk`|**M**aster **Cl**oc**k**|主设备 → 从设备|主时钟，为整个音频系统提供基准时序，通常为采样率的 256 倍（如 256 × 48kHz = 12.288MHz）|
|`bclk`|**B**it **Cl**oc**k**|主设备 → 从设备|位时钟，每一位音频数据一个时钟脉冲，频率 = 采样率 × 通道数 × 位深|
|`ws`|**W**ord **S**elect|主设备 → 从设备|帧同步信号，切换左右声道，频率 = 采样率（如 48kHz），高电平=右声道，低电平=左声道|
|`dout`|**D**ata **Out**|主设备（ESP32）→ 从设备（ES8311）|播放路径：ESP32 **发送**音频数据给 ES8311（DAC）|
|`din`|**D**ata **In**|从设备（ES8311）→ 主设备（ESP32）|录音路径：ES8311 **发送**音频数据给 ESP32（ADC）|


```c
typedef struct {

    uint8_t port;      /*!< I2S port, this port need pre-installed by other modules */

    void   *rx_handle; /*!< I2S rx handle, need provide on IDF 5.x */

    void   *tx_handle; /*!< I2S tx handle, need provide on IDF 5.x */

    int     clk_src;   /*!< I2S clock source, need converted from `i2s_clock_src_t`. If set to 0 will use default clock source */

} audio_codec_i2s_cfg_t;
```

