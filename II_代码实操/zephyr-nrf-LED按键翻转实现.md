#zephyr
#nordic

```c
#include <zephyr/kernel.h>
#include <zephyr/drivers/gpio.h>
/* 获取设备树节点 */
#define LED_NODE DT_ALIAS(led0)
#define BUTTON_NODE DT_ALIAS(sw0)
```
这里宏定义的`DT_ALIAS()`，在`devicestree.h`中有定义：

> [!NOTE]
> ```c
> #define DT_ALIAS(alias) DT_CAT(DT_N_ALIAS_, alias)
> ```

可以理解为从设备树中获取对应的参数(但是这个dt_cat底层我不知道咋写的)

> 定义LED引脚
```c
static const struct gpio_dt_spec led =
    GPIO_DT_SPEC_GET(LED_NODE, gpios);

static const struct gpio_dt_spec button =
    GPIO_DT_SPEC_GET(BUTTON_NODE, gpios);
```
翻阅`GPIO_DT_SPEC_GET();`函数，看到有三个参数：

> [!NOTE]
> ```c
> #define GPIO_DT_SPEC_GET_BY_IDX(node_id, prop, idx)                \
>     {                                      \
>         .port = DEVICE_DT_GET(DT_GPIO_CTLR_BY_IDX(node_id, prop, idx)),\
>         .pin = DT_GPIO_PIN_BY_IDX(node_id, prop, idx),             \
>         .dt_flags = DT_GPIO_FLAGS_BY_IDX(node_id, prop, idx),          \
>     }
> ```

> [!NOTE]
> 其中，`dt_flags`其实指的就是设备树中的标志位，比如：
> ```c
>                 led0: led_0 {
>                         gpios = <&gpio1 2 GPIO_ACTIVE_HIGH>;
>                         label = "Status LED 0";
>                 };
> ```
> 中的`GPIO_ACTIVE_HIGH`



```c
static struct gpio_callback button_cb_data;
```

> 按键中断处理
```c
void button_pressed(
        const struct device *dev,
        struct gpio_callback *cb,
        uint32_t pins)
{
    gpio_pin_toggle_dt(&led);
}

```

> [!NOTE] device结构体（节选）
> ```c
> struct device {
>     /** Name of the device instance */
>     const char *name;
>     /** Address of device instance config information */
>     const void *config;
>     /** Address of the API structure exposed by the device instance */
>     const void *api;
>     /** Address of the common device state */
>     struct device_state *state;
>     /** Address of the device instance private data */
>     void *data;
>     /** Device operations */
>     struct device_ops ops;
>     /** Device flags */
>     device_flags_t flags;
> ```

可以看到注释： 指针变量`*data` 存放的是设备的地址



```c
int main(void)
{
    int ret;
    if(!gpio_is_ready_dt(&led))
    {
        return -1;
    }

    if(!gpio_is_ready_dt(&button))
    {
        return -1;
    }

    ret = gpio_pin_configure_dt(
                &led,
                GPIO_OUTPUT_INACTIVE);

    if(ret)
    {
        return -1;
    }
```


> 配置gpio
```c

    ret = gpio_pin_configure_dt(
                &button,
                GPIO_INPUT);

    if(ret)
    {
        return -1;
    }
```


> 设置中断触发方式
```c
    ret = gpio_pin_interrupt_configure_dt(
                &button,
                GPIO_INT_EDGE_TO_ACTIVE);

    if(ret)
    {
        return -1;
    }
```

> 初始化回调函数
```c
    gpio_init_callback(
            &button_cb_data,
            button_pressed,
            BIT(button.pin));
```

> 注册回调函数
```c
    gpio_add_callback(
            button.port,
            &button_cb_data);

    while(1)
    {
        k_sleep(K_FOREVER);
    }
}
```