#实操/开发/嵌入式/STM32 

![[RM0008中文参考手册.pdf#page=310&rect=96,572,481,688&color=yellow|RM0008中文参考手册, p.310]]

### 初始化
	void Dri_RTC_Init(void)
初始化 RTC 外设，使用外部低速晶振（LSE）作为时钟源，并配置 1 秒计数。

>使能备份域时钟和电源接口时钟
```c
RCC->APB1ENR |= (RCC_APB1ENR_BKPEN | RCC_APB1ENR_PWREN);
```

>取消备份域写保护
```c
PWR->CR |= PWR_CR_DBP;
```

>使能 RTC 时钟
```c
RCC->BDCR |= RCC_BDCR_RTCEN;
```

>启动 LSE
```c
RCC->BDCR |= RCC_BDCR_LSEON;
while(!(RCC->BDCR & RCC_BDCR_LSERDY));
```

>选择 LSE 作为 RTC 时钟源
```c
RCC->BDCR |= RCC_BDCR_RTCSEL_LSE;
```

>等待上一次操作完成
```c
while(!(RTC->CRL & RTC_CRL_RTOFF));
```

>进入配置模式
```c
RTC->CRL |= RTC_CRL_CNF;
```

>设置预分频值：产生 1 Hz 时钟
```c
RTC->PRLH = 0;
RTC->PRLL = 32768 - 1;
```
有关分频的讲解，参照[[STM32-RTC#]]


> [!info]
> LSE 频率为 32.768 kHz，RTC 计数器期望 1 秒加 1，因此分频系数应为 32768 – 1。
> `PRLH` 存储高 4 位（实际分频值高 4 位始终为 0），`PRLL` 存储低 16 位。

>退出配置模式
```c
RTC->CRL &= ~RTC_CRL_CNF;
```

>等待本次操作完成
```c
while(!(RTC->CRL & RTC_CRL_RTOFF));
```

> [!attention]
> 操作 RTC 寄存器前必须进入配置模式（`CNF = 1`），完成后再退出。进入和退出都需要检查 `RTOFF` 位确保前一操作结束。

---

### 设置时间
	void Dri_RTC_SetDateTime(DateTimeTypeDef dt)
将日期时间结构体转换为 UNIX 时间戳，并写入 RTC 计数器。

>构造 `tm` 结构体，转换为 UNIX 时间戳
```c
struct tm t1 = {0};
t1.tm_year = dt.year - 1900;
t1.tm_mon  = dt.month - 1;
t1.tm_mday = dt.day;
t1.tm_hour = dt.hour;
t1.tm_min  = dt.minute;
t1.tm_sec  = dt.second;

uint32_t times = mktime(&t1);
```

>`mktime` 将本地时间 `tm` 转换为自 1970-01-01 00:00:00 起算的秒数（UNIX 时间戳）。

>等待上一次操作完成
```c
while(!(RTC->CRL & RTC_CRL_RTOFF));
```

>进入配置模式
```c
RTC->CRL |= RTC_CRL_CNF;
```

>等待秒标志，确保与当前时间同步
```c
while(!(RTC->CRL & RTC_CRL_SECF));
```

>写入计数值
```c
RTC->CNTL = times;
RTC->CNTH = (times >> 16);
```

> [!info]
> `RTC->CNTH` 存储计数器的高 16 位，`RTC->CNTL` 存储低 16 位，组合成完整的 32 位计数值。

>退出配置模式
```c
RTC->CRL &= ~RTC_CRL_CNF;
```

>等待本次操作完成
```c
while(!(RTC->CRL & RTC_CRL_RTOFF));
```

---

### 获取时间
	void Dri_RTC_GetDateTime(DateTimeTypeDef *dt)
从 RTC 计数器读取当前时间戳，转换为日期时间结构体。

>等待影子寄存器同步
```c
while(!(RTC->CRL & RTC_CRL_RSF));
```

>读取计数器值
```c
uint32_t times = (RTC->CNTH << 16) | RTC->CNTL;
```

>`RSF` 为寄存器同步标志，硬件完成影子寄存器更新后置 1，读取前必须等待以确保时间一致性。

>将时间戳转换为日期时间
```c
struct tm *tms = localtime(&times);
dt->year   = tms->tm_year + 1900;
dt->month  = tms->tm_mon + 1;
dt->day    = tms->tm_mday;
dt->hour   = tms->tm_hour;
dt->minute = tms->tm_min;
dt->second = tms->tm_sec;
```

> `localtime` 将 UNIX 时间戳转换回本地时间 `tm` 结构，年份需 `+1900`，月份需 `+1` 恢复常规表示。