#实操/开发/嵌入式/STM32/项目/牛马定位器 

### RCC
![[Pasted image 20260521102302.png]]

![[Pasted image 20260521102353.png]]


### SYS
![[Pasted image 20260521101915.png]]
打开SYS-Debug：Serial Wire

### GPIO配置
![[Pasted image 20260521101607.png]]
配置`PB5`引脚，由[[SCH_牛马定位器_原理图.pdf#page=1&selection=DS3553-CS|DS3553-原理图]]可知：该引脚为DS3553的CS引脚。
配置引脚默认高电平，参考DS3553的[[DS3553-编写手册.pdf#page=6&selection=通信条件：CS 拉低。|通信条件：CS 拉低。]]

### I2C
![[Pasted image 20260521101517.png]]


### USART
![[Pasted image 20260521102011.png]]
配置USART1为异步模式。



### W5500驱动配置（Gateway端）
参考原理图，W5500的六个引脚分布如下：
![[SCH_ZET6开发板_1_2024-07-01.pdf#page=9&rect=67,109,257,180|SCH_ZET6开发板_1_2024-07-01, p.9]]

需要注意的是，PD3作为W5500的片选引脚，需要被配置为：默认高电平