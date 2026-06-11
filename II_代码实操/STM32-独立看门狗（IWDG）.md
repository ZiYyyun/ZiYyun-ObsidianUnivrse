#实操/开发/嵌入式/STM32 


### 初始化
	void Dri_IWDG_Init(void)

>启动看门狗

参考：[[I_知识节点/STM32-独立看门狗（IWDG）#键寄存器|STM32-独立看门狗（IWDG）]]
```c
IWDG->KR |= 0xCCCC;
IWDG->KR |= 0x5555;
```
1. 使能IWDG看门狗
2. 允许访问PR和RLR寄存器

>配置预分频寄存器

```c
IWDG->PR &= ~IWDG_PR_PR;
IWDG->PR |= IWDG_PR_PR_2;
```
配置预分频因子为64



>设置喂狗时间
```c
IWDG->RLR |= 2499;
```
**设置看门狗的溢出时间为 4s**，上面我们配置了分频因子为64，已知独立看门狗时钟LSI=40KHz，分频因子64，重装载值2499：
计数频率：40000 / 64 = 625 Hz
溢出时间：(2499+1) / 625 = 4 s

>刷新看门狗（喂狗）
```c
Dri_IWDG_Refresh();
```
  
### 刷新（喂狗）
	void Dri_IWDG_Refresh(void)

>喂狗
```c
IWDG->KR = 0xAAAA;
```
喂狗的代码很简单，往IWDG_KR里写`0xAAAA`