#实操/开发/嵌入式  
#esp32 

参考：[ESP32官方例程-tjpgd](https://github.com/espressif/esp-idf/tree/v6.0.2/examples/peripherals/lcd/tjpgd)


### 步骤概述

```
                    app_main()
                        │
                        ▼
         gpio_config()                    ← 配置LCD背光GPIO
                        │
                        ▼
         spi_bus_initialize()             ← 初始化SPI总线
                        │
                        ▼
     esp_lcd_new_panel_io_spi()           ← 创建LCD通信接口(IO)
                        │
                        ▼
    esp_lcd_new_panel_st7789()            ← 创建ST7789驱动对象
                        │
                        ▼
        esp_lcd_panel_reset()             ← LCD硬件复位
                        │
                        ▼
         esp_lcd_panel_init()             ← 初始化LCD寄存器
                        │
                        ▼
     esp_lcd_panel_disp_on_off(true)      ← 开启显示
                        │
                        ▼
      esp_lcd_panel_invert_color()        ← 是否颜色反转
                        │
                        ▼
        esp_lcd_panel_swap_xy()           ← 是否交换XY
                        │
                        ▼
        gpio_set_level(BK_LIGHT)          ← 打开背光
                        │
                        ▼
       heap_caps_malloc() ×2              ← 创建DMA双缓冲
                        │
                        ▼
                 while(1)
                        │
                        ▼
         display_pretty_colors()
                        │
                        ▼
      pretty_effect_calc_lines()          ← CPU计算图像
                        │
                        ▼
    esp_lcd_panel_draw_bitmap()           ← DMA发送到LCD
                        │
                        └───────循环──────►
```


### 函数解析





#### GPIO配置
> 配置GPIO
```c
esp_err_t gpio_config(const gpio_config_t *pGPIOConfig)
```
#### spi相关
> 初始化SPI总线
```c
esp_err_t spi_bus_initialize(
	spi_host_device_t host_id, 
	const spi_bus_config_t *bus_config, 
	spi_dma_chan_t dma_chan)
```

#### LCD相关函数
> 创建LCD通信接口(IO)
```c
esp_err_t esp_lcd_new_panel_io_spi(	//[!Describe]该函数 esp_lcd_new_panel_io_spi 用于创建一个用于SPI接口的LCD面板IO句柄。它接受SPI总线句柄、IO配置参数以及一个指向返回句柄的指针作为参数。
	esp_lcd_spi_bus_handle_t bus, 
	const esp_lcd_panel_io_spi_config_t *io_config,                         esp_lcd_panel_io_handle_t *ret_io)

```

> 创建ST7789驱动对象

```c
esp_err_t esp_lcd_new_panel_st7789(
	const esp_lcd_panel_io_handle_t io, 
	const esp_lcd_panel_dev_config_t *panel_dev_config,
    esp_lcd_panel_handle_t *ret_panel)
```

> LCD硬件复位

```c
esp_err_t esp_lcd_panel_reset(esp_lcd_panel_handle_t panel)
```


> 初始化lcd
```c
esp_err_t esp_lcd_panel_init(esp_lcd_panel_handle_t panel)
```


> 开启LCD
```c
esp_err_t esp_lcd_panel_disp_on_off(
	esp_lcd_panel_handle_t panel, 
	bool on_off)
```

> 翻转颜色

```c
esp_err_t esp_lcd_panel_invert_color(
	esp_lcd_panel_handle_t panel, 
	bool invert_color_data)
```


> 是否交换xy
```c
esp_err_t esp_lcd_panel_swap_xy(
	esp_lcd_panel_handle_t panel, 
	bool swap_axes)
```


> 设置GPIO输出登记
```c
esp_err_t gpio_set_level(gpio_num_t gpio_num, uint32_t level)
```

> 渲染图像并显示
```c
static void display_pretty_colors(esp_lcd_panel_handle_t panel_handle)
```