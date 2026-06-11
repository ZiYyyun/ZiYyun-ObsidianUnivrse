#实操/开发/嵌入式/STM32/项目/牛马定位器

### Debug定义
```h
#define
```


### I2C从机读写标识
```h
#define DEVICE_WRITE 0x4E
#define DEVICE_READ  0x4F
```

### 寄存器地址


```c
#define CHIP_ID 0x01
#define USER_SET 0xC3
#define STEP_CNT_L 0xC4
#define STEP_CNT_M 0xC5
#define STEP_CNT_H 0xC6
```

### user_set寄存器功能位

![[DS3553-编写手册.pdf#page=9&rect=32,529,473,660|ds3553, p.9]]
```c
#define PWR_MOD (1 << 7)
#define SEN_DIS (1 << 6)
#define RAISE_EN (1 << 5)
#define PLUSE_EN (1 << 4)
#define NOISE_DIS (1 << 3)
#define CLEAR_EN (1 << 2)
#define PEDO_1 (1 << 1)
#define PEDO_0 (1 << 0)
```

### 定义枚举

```c
typedef enum
{
    DS3553_RESET = 0,
    DS3553_SET
} DS3553_STATE;

```
方便写0，写1操作

