#实操/开发/嵌入式/STM32


> [!NOTE] 需求描述
> ![[Pasted image 20260422192033.png]]
> 需要LED1、LED2、LED3实现流水灯，其分别被链接在`PA0` `PA1` `PA8`引脚上

### LED 操作
```c led.h
#ifndef __LED_H__
#define __LED_H__

#include "stm32f10x.h"

#define LED1 GPIO_ODR_ODR0
#define LED2 GPIO_ODR_ODR1
#define LED3 GPIO_ODR_ODR8

void Int_Led_Init(void);                              //初始化LED
void Int_Led_On(uint16_t led);                        //开灯函数
void Int_Led_Off(uint16_t led);                       //关灯
void Int_Led_Toggle(uint16_t led);                    //翻转LED
void Int_Led_OnAll(uint16_t leds[] , uint8_t size);   //开启所有灯
void Int_Led_OffAll(uint16_t leds[] , uint8_t size);  //关闭所有

#endif /* __LED_H__ */
```

#### led.c
- 目标配置:通用[[STM32-GPIO输入输出模式#推挽式输出(GPIO_Mode_Out_PP)]]
- 对应寄存器配置: `GPIOA->CRL`: 11    `MODE`: 00

使用寄存器配置GPIO，参考[[STM32-寄存器点灯]]步骤来进行.

```c led.c
#include "led.h"
void Int_Led_Init(void){
    RCC->APB2ENR |= RCC_APB2ENR_IOPAEN ;                //操作RCC的APB2ENR寄存器，使能APB2ENR总线
    // 目标配置：通用推挽输出
    GPIOA->CRL |= (GPIO_CRL_MODE0 | GPIO_CRL_MODE1) ;   //操作GPIOA的CRL(低八位)寄存器，同时对MODE的第0、1位置1
    GPIOA->CRL &= ~(GPIO_CRL_CNF0 | GPIO_CRL_CNF1);     //操作GPIOA的CRL寄存器，对CNF的0、1位置0
    
    //由于LED3在PA8，由GPIOx_CRH寄存器控制
    GPIOA->CRH |= GPIO_CRH_MODE8;                       //操作GPIOA的CRH(高八位)寄存器，对MODE置1
    GPIOA->CRH &= ~GPIO_CRH_CNF8 ;                      //操作GPIOA的CRH(高八位)寄存器，对CNF置0
}
```


接下来三个函数对传进来的形参进行简单的置0、置1操作
```c

void Int_Led_On(uint16_t led){
    GPIOA->ODR &=~ led;
}

void Int_Led_Off(uint16_t led){
    GPIOA->ODR |= led;
}

void Int_Led_Toggle(uint16_t led){
    GPIOA->ODR ^= led;
}

```

分别实现点亮和熄灭所有灯。通过一个for循环实现。接受的形参比较有意思，在`main.c`中可知，需要一个数组来存放三个LED的地址
```c
void Int_Led_OnAll(uint16_t leds[] , uint8_t size){
    for (uint8_t i = 0; i < size; i++){
        Int_Led_On(leds[i]);
    }
}

void Int_Led_OffAll(uint16_t leds[] , uint8_t size){
    for (uint8_t i = 0; i < size; i++){
        Int_Led_Off(leds[i]);
    }
}
```

#### main.c

```c
#include "led.h"
#include "delay.h"

int main(){

    uint16_t leds[3]={LED1,LED2,LED3};

    Int_Led_Init();                         //调用函数，初始化所需GPIO

    while(1){

        for (uint8_t i = 0; i < 3; i++){ 
            Int_Led_OffAll(leds,3);
            Int_Led_On(leds[i]);
            Delay_ms(300);
        }
    }
}
```

使用一个数组来存放3个LED的地址，其中`LED1`、`LED2`、`LED3`在`led.h`中已经使用宏定义声明