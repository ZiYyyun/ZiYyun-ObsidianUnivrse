#实操/开发/嵌入式  #modbus  #Prot #blog 

本笔记记录 Modbus 协议的实操落地,基于 FreeModbus 协议栈在 STM32 上移植与使用。理论部分见 [[I_知识节点/Prot-Modbus]]。

---

## 📋 常用功能码速查

| 功能码 | 操作 | 数据类型 |
|--------|------|----------|
| 0x01 | 读线圈 | 位 |
| 0x02 | 读离散输入 | 位 |
| 0x03 | 读保持寄存器 | 字 |
| 0x04 | 读输入寄存器 | 字 |
| 0x05 | 写单个线圈 | 位 |
| 0x06 | 写单个寄存器 | 字 |
| 0x0F | 写多个线圈 | 位 |
| 0x10 | 写多个寄存器 | 字 |



## 🧩 移植流程

```
FreeModbus 源码获取
      ↓
拷贝 Modbus/ 与 port/ 到工程
      ↓
配置 include path
      ↓
移植 portserial.c (串口驱动)
      ↓
移植 porttimer.c (定时器驱动)
      ↓
移植 port.c (寄存器回调)
      ↓
去除断言依赖 -DNDEBUG
      ↓
eMBInit + eMBEnable 启动
```

> 详细移植步骤见 [[II_代码实操/Prot-Modbus移植]]



## 🚀 启动模板

```c
#include "mb.h"
#include "mbport.h"

// 寄存器定义
#define REG_INPUT_SIZE   10
#define REG_HOLD_SIZE    10
#define REG_COILS_SIZE   10
#define REG_DISC_SIZE    10

uint16_t REG_INPUT_BUF[REG_INPUT_SIZE]  = {0};
uint16_t REG_HOLD_BUF[REG_HOLD_SIZE]   = {0};
uint8_t  REG_COILS_BUF[REG_COILS_SIZE] = {0};
uint8_t  REG_DISC_BUF[REG_DISC_SIZE]   = {0};

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_USART2_UART_Init();
    MX_TIM3_Init();

    // eMBInit(模式, 从机地址, 端口, 波特率, 校验)
    eMBInit(MB_RTU, 0x01, 0, 9600, MB_PAR_NONE);
    eMBEnable();

    while (1)
    {
        eMBPoll();   // 必须周期性调用
    }
}
```



## 📨 收发回调框架

四种寄存器对应四个回调,实现见 [[II_代码实操/Prot-Modbus移植#port文件]]:

| 回调函数 | 对应功能码 | 操作 |
|---------|-----------|------|
| `eMBRegInputCB` | 0x04 | 读输入寄存器 |
| `eMBRegHoldingCB` | 0x03/0x06/0x10 | 读写保持寄存器 |
| `eMBRegCoilsCB` | 0x01/0x05/0x0F | 读写线圈 |
| `eMBRegDiscreteCB` | 0x02 | 读离散输入 |



## 🔧 发送功能实现

> 主动发送 Modbus 请求帧,见 [[II_代码实操/Prot-Modbus发送功能实现]]



## ⚙️ 关键配置

### 串口参数

| 参数 | 推荐值 |
|------|--------|
| 波特率 | 9600 / 115200 |
| 数据位 | 8 |
| 停止位 | 1 / 2(奇偶校验时) |
| 校验 | None / Even / Odd |

### 3.5 字符超时

```
T3.5 = 3.5 × (11 bit) / 波特率
9600   → T3.5 ≈ 4.0 ms
115200 → T3.5 ≈ 0.33 ms (协议规定最小 1.75 ms)
```



## 🐛 调试技巧

- [ ] 用串口助手抓包,对照功能码确认帧结构
- [ ] 检查从机地址是否冲突
- [ ] 定时器中断频率是否满足 3.5 字符超时
- [ ] `eMBPoll()` 是否在主循环中持续调用
- [ ] CRC 校验失败 → 检查串口波特率/校验位配置



## 📎 相关笔记

- 理论: [[I_知识节点/Prot-Modbus]]
- 移植: [[II_代码实操/Prot-Modbus移植]]
- 发送: [[II_代码实操/Prot-Modbus发送功能实现]]
- 项目应用: [[五、项目仓库/步进电机/App_MotorValue]]
- ADAM-4150 控制: [[II_代码实操/基于Modbus_HEX指令的ADAM-4150的串口控制]]
