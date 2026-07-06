#esp32 #理论/开发/嵌入式 


[按键 - - — ESP-IoT-Solution latest 文档](https://docs.espressif.com/projects/esp-iot-solution/zh_CN/latest/input_device/button.html)

---



## 一、GPIO 基础概念

- **物理 GPIO 管脚数量**（因芯片而异）
    - ESP32：34 个（GPIO0~19、21~23、25~27、32~39）
    - ESP32-S3：45 个（GPIO0~21、26~48）
    - ESP8684：21 个（GPIO0~20）
    - ESP32-C6：31 个（GPIO0~30）
    - ESP32-H2：19 个（GPIO0~5、8~14、22~27）
- **管脚功能模式**：通用 GPIO 功能 或 直连外设功能

---

## 二、核心硬件模块

### 1. IO MUX

- 为每个 GPIO 管脚提供一个配置寄存器 `IO_MUX_GPIOn_REG`
- 两种配置：连接 GPIO 交换矩阵 / 旁路 GPIO 交换矩阵（直连）
- 高速信号（SPI、JTAG、UART 等）可旁路 GPIO 交换矩阵直通


---

## 三、焊盘（PAD）内部结构

| 信号  | 说明      |
| --- | ------- |
| IE  | 输入使能    |
| OE  | 输出使能    |
| WPU | 内部弱上拉电阻 |
|WPD|内部弱下拉电阻|



---

## 四、GPIO 配置（ESP-IDF API）

### 配置结构体 `gpio_config_t`

```c
typedef struct {
    uint64_t pin_bit_mask;//[!pin_bit_mask]引脚掩码，用于区分引脚
    gpio_mode_t mode;//[!mode]I/O模式
    gpio_pullup_t pull_up_en;//[!pull_up_en]内部上啦使能
    gpio_pulldown_t pull_down_en;//[!pull_down_en]内部下拉使能
    gpio_int_type_t intr_type;//[!intr_type]中断触发模式
    gpio_hys_ctrl_mode_t hys_ctrl_mode;//[!hys_ctrl_mode]gpio中断使能
#endif
} gpio_config_t;
```


> [!NOTE] Tip
> 需要注意的是，ESP对数据类型要求比较严格。
> 如结构体所示，`pin_bit_mask`被声明成64位，是因为ESP32有40多个gpio，`uint32_t`放不下
> 所以我们传参时：
> ```c
> .pin_bit_mask = xULL << GPIO_NUM_x, //[!x]这里的x替换为引脚号
> ```



---

### 中断功能

#### 中断触发模式

以下宏定义在结构体中的`.intr_type`赋值

| 模式                     | 说明     |
| ---------------------- | ------ |
| `GPIO_INTR_DISABLE`    | 禁用     |
| `GPIO_INTR_POSEDGE`    | 上升沿触发  |
| `GPIO_INTR_NEGEDGE`    | 下降沿触发  |
| `GPIO_INTR_ANYEDGE`    | 任意边沿触发 |
| `GPIO_INTR_LOW_LEVEL`  | 低电平触发  |
| `GPIO_INTR_HIGH_LEVEL` | 高电平触发  |

#### 中断配置流程

```c
gpio_config()//[!DESCRIBE]配置GPIO
↓
gpio_install_isr_service()//[!DESCRIBE]安装ISR服务
↓
gpio_isr_handler_add()//[!DESCRIBE]注册ISR函数
↓
GPIO变化
↓
ISR执行
```


---

### 常用函数

| API                          | 功能              | 类比 STM32 HAL                     |
| ---------------------------- | --------------- | -------------------------------- |
| `gpio_config()`              | 初始化 GPIO        | `HAL_GPIO_Init()`                |
| `gpio_set_level()`           | 设置输出电平          | `HAL_GPIO_WritePin()`            |
| `gpio_get_level()`           | 读取输入电平          | `HAL_GPIO_ReadPin()`             |
| `gpio_reset_pin()`           | 恢复为默认状态         | 无直接对应                            |
| `gpio_set_direction()`       | 修改引脚方向          | 修改 `Mode` 后重新初始化                 |
| `gpio_set_pull_mode()`       | 修改上下拉           | 修改 `Pull` 后重新初始化                 |
| `gpio_install_isr_service()` | 安装 GPIO 中断服务    | 配置 EXTI/NVIC                     |
| `gpio_isr_handler_add()`     | 为某个 GPIO 注册中断回调 | `HAL_GPIO_EXTI_Callback()` 的注册思路 |

---

### 特殊功能

- **输入常量**：无需绑定 GPIO 管脚，可使外设读取恒定高/低电平
- **信号取反**：置位 `GPIO_FUNCy_IN_INV_SEL`
- **开漏输出**：通过 `GPIO_PINx_PAD_DRIVER` 寄存器选择
- **LP GPIO 矩阵**（ESP32-P4、ESP32-C5 等）：支持低功耗域的 GPIO 交换矩阵 


---

### 注意事项（以 ESP32 为例）

- **GPIO34~39** 仅为输入，无软件上拉/下拉功能
- **GPIO6~11、16~17** 通常连接内部 Flash/PSRAM，不可随意使用
- **GPIO36、39** 使用 ADC 或 Wi-Fi/蓝牙睡眠模式时，避免使用中断
- Strapping 管脚需注意启动时序