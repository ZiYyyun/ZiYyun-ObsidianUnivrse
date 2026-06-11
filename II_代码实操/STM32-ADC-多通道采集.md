
[[#^32d46e|寄存器方法实现]]
[[#^df862a|HAL库方法实现]]


> 寄存器方法实现
^32d46e
---

### 初始化
	void Dri_ADC_Init(void)

> 老生常谈，开启外设时钟
```c
RCC->APB2ENR |= (RCC_APB2ENR_ADC1EN | RCC_APB2ENR_IOPCEN) ;
```
我们由 [[I_知识节点/STM32-ADC#引脚定义&分布|原理图]]  可知，上述代码同时使能ADC1和GPIOC的时钟，即把ADC1配置在GPIOC上运行

>设置ADC预分频
```c
RCC->CFGR |= RCC_CFGR_ADCPRE_DIV16;
```
在RCC_CFGR寄存器中，[[III_资源仓库/参考手册/1-STM32F10x-中文参考手册_最新.pdf#page=62&selection=ADCPRE|ADCPRE]]功能位是专门用来控制ADC分频的

>配置GPIO为[[RM0008中文参考手册.pdf#page=114&selection=模拟输入|模拟输入]]
```c
GPIOC->CRL &=~ (GPIO_CRL_MODE0 | GPIO_CRL_CNF0);
```

>开启扫描模式：
```c
ADC1->CR1 |= ADC_CR1_SCAN;
```

>配置转换触发方式：
```c
ADC1->CR2 |= (ADC_CR2_EXTTRIG | ADC_CR2_EXTSEL);
```
`EXTTRIG`:[[RM0008中文参考手册.pdf#page=173&selection=该位由软件设置和清除，用于开启或禁止可以启动规则通道组转换的外部触发事件|该位由软件设置和清除，用于开启或禁止可以启动规则通道组转换的外部触发事件]]
`EXTSEL`:全部置1，即：111。配置为[[RM0008中文参考手册.pdf#page=173&selection=SWSTART|SWSTART]]模式

> [!NOTE] 为什么EXTTRING要置1？
> 
> ![[RM0008中文参考手册.pdf#page=156&rect=189,203,380,327&color=yellow]]
> 如图所示，`EXTTRING`是个与门（有0则0），置了1之后，`EXTSEL`的配置才有意义

>设置转换结果对齐方式
```c
ADC1->CR2 &=~ ADC_CR2_ALIGN ;
```

>开启[[RM0008中文参考手册.pdf#page=174&selection=连续转换|连续转换]]模式和[[RM0008中文参考手册.pdf#page=174&selection=直接存储器访问模式|直接存储器访问模式]]
```c
ADC1->CR2 |= ADC_CR2_CONT;
ADC1->CR2 |= ADC_CR2_DMA;
```
当前有三个通道（`IN10`, `IN11`, `IN12`）需要转换，所以需要开启`DMA`，方便直接写入存储器

>设置三个通道的采样周期
```c
ADC1->SMPR1 &=~ (ADC_SMPR1_SMP10 | ADC_SMPR1_SMP11 | ADC_SMPR1_SMP12 );
ADC1->SMPR1 |= (ADC_SMPR1_SMP10_1 | ADC_SMPR1_SMP11_1 | ADC_SMPR1_SMP12_1);
```

>写入转换规则
```c
ADC1->SQR3 &=~ ADC_SQR3_SQ1 ;                 //清零对应位
ADC1->SQR3 |= (10<<0);

ADC1->SQR3 &=~ ADC_SQR3_SQ2 ;                 //清零对应位
ADC1->SQR3 |= (11<<5);

ADC1->SQR3 &=~ ADC_SQR3_SQ3 ;                 //清零对应位
ADC1->SQR3 |= (12<<10);
```
这里的位运算是什么意思呢？参考 [[STM32-ADC#多通道转换|这里]] 的描述——在`SQR3`中，有`SQ1[4:0]`到`SQ6[4:0]`这六个功能位，其中数字就代表着顺序
*比如我们看`SQ1`，如果我们想要哪个通道排在第一位，我们就往`SQ1`里面写，同理，`SQ2`就是第二次......*
那么顺序的操作说完了，怎么写，写什么呢？比如我想让`Channel12`放在转换的第二位，那么我可以： *直接把12这个数字写进`SQ2`，即 ADC1->SQR3 |= (12 << 5); *

>设置转换通道数量
```c
ADC1->SQR1 &=  ~ADC_SQR1_L;
ADC1->SQR1 |= ADC_SQR1_L_1;
```
最后我们在 功能位里写一下我们的转换数量（即有几个通道要转换）

**关于DMA的设置参考[[STM32-DMA&USRT-Register]]，重复内容略**

>DMA设置非存储器到存储器模式
```c
DMA1_Channel1->CCR &= ~DMA_CCR1_MEM2MEM;
```

>优先级设置
```c
DMA1_Channel1->CCR |= DMA_CCR1_PL;
```

>存储器和外设数据宽度
```c
DMA1_Channel1->CCR &=~ (DMA_CCR1_MSIZE_1 | DMA_CCR1_PSIZE_1);
DMA1_Channel1->CCR |= (DMA_CCR1_MSIZE_0 | DMA_CCR1_PSIZE_0) ;
```

>存储器和外设自增配置：存储器自增 外设自增
```c
DMA1_Channel1->CCR |= DMA_CCR1_MINC;
DMA1_Channel1->CCR &= ~DMA_CCR1_PINC;
```

>循环模式
```c
DMA1_Channel1->CCR |= DMA_CCR1_CIRC ;
```

>数据传输方向
```c
DMA1_Channel1->CCR &=~ DMA_CCR1_DIR ;
```
### 开始转换
	void Dri_ADC_StartConvert(void)
参考手册中的描述：
> [!PDF|yellow] [[RM0008中文参考手册.pdf#page=157&selection=108,0,148,1&color=yellow|1-STM32F10x-中文参考手册_最新, p.157]]
> > 11.3.1 ADC开关控制通过设置ADC_CR2寄存器的ADON位可给ADC上电。当第一次设置ADON位时，它将ADC从断电状态下唤醒。 ADC上电延迟一段时间后(t STAB)，再次设置ADON位时开始进行转换。通过清除ADON位可以停止转换，并将ADC置于断电模式。在这个模式中，ADC几乎不耗电(仅几个μA)
> 
> 

>启动转换
```c
ADC1->CR2 |= ADC_CR2_ADON;
```
那我们可以得知，当我们需要初始化ADC时，首先要操作`ADC->CR2`的 [[RM0008中文参考手册.pdf#page=175&selection=ADON|ADON]] 位。当我们需要开始转换时，应再次设置ADON位。关闭的话，则清除ADON位即可。

>操作[[RM0008中文参考手册.pdf#page=174&selection=CAL|CAL]]位，启动硬件自动校准：
```c
ADC1->CR2 |= ADC_CR2_CAL;
```

>等待硬件自动校准完成：
```c
while(ADC1->CR2 & ADC_CR2_CAL);
```
需注意的是，[[RM0008中文参考手册.pdf#page=174&selection=该位由软件设置以开始校准，并在校准结束时由硬件清除。|该位由软件设置以开始校准，并在校准结束时由硬件清除。]]

>通过swstart启动转换：
```c
ADC1->CR2 |= ADC_CR2_SWSTART ;
```

>等待转换完成：
```c
while(!(ADC1->SR & ADC_SR_EOC));
```
当转换完成后，  寄存器的[[RM0008中文参考手册.pdf#page=170&selection=EOC|EOC]]位会被硬件置1



> HAL库方法实现
^df862a
---

### CubeMX配置

配置ADC通道数量，以及各个通道
![[Pasted image 20260511143131.png]]

配置DMA
![[Pasted image 20260511143248.png]]


```c main.c

```