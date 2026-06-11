
![[Pasted image 20240812142034.png]]
![[Pasted image 20240812142046.png]]

### 实现代码

> 代码编译并烧录后，LED1与LED2同时闪烁


```c
#include <ioCC2530.h>

#define LED1 P1_0        //P1.0端口控制LED1发光二极管
#define LED2 P1_1
#define unint unsigned int

void Init_GPIO()
{
    // 设置 gpio
    P1SEL &= ~(0x01 | 0x02);
    // 设置 输出
    P1DIR |= (0x01 | 0x02);
    // P1端口下拉
    P1 &= (0x01 | 0x02);
}

void delay(unint z)
{
    for (unint i = 0; i < z; i++)
    {
        for (unint j = 0; j < 500; j++);
    }
}

void main(void) {
    
    Init_GPIO();
    
    while (1) 
    {
        LED1 = 0;   // 熄灭LED1发光二极管
        LED2 = 0;
        delay(1000);
        LED2 = 1;
        LED1 = 1;       // 点亮LED1发光二极管
        delay(1000);
    }
}
```

### Init_GPIO函数

#### - 设置`P1.0`和`P1.1`为通用IO口
```c
P1SEL &= ~(0x01 | 0x02);// P1SEL的第0位和第1位清零，分别对应P1.0和P1.1
```
- `0x01` 是二进制 `00000001`，对应P1.0。
- `0x02` 是二进制 `00000010`，对应P1.1。
- `~(0x01 | 0x02)` 结果为 `11111100`，表示清零P1SEL的第0位和第1位，设置P1.0和P1.1为通用IO口。
#### - 设置`P1.0`和`P1.1`为输出模式
```c
P1DIR |= (0x01 | 0x02);// P1DIR的第0位和第1位置位，分别对应P1.0和P1.1
```

#### - 将`P1.0`和`P1.1`端口下拉
```c
P1 &= ~(0x01 | 0x02); // 将P1的第0位和第1位清零，使P1.0和P1.1输出低电平
```