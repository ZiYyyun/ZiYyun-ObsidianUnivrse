#实操/开发/嵌入式  #MQTT  #Prot

本笔记记录 MQTT 协议的实操落地,基于 W5500 + STM32 作为客户端连接 Broker。理论部分见 [[I_知识节点/Prot-MQTT]]。

---

## 📋 核心角色与概念

| 角色 | 说明 |
|------|------|
| Client | 终端设备(单片机) |
| Broker | 服务器(Mosquitto/EMQX 等) |
| Topic | 通信主题,层级用 `/` 分隔 |

> 常用 Topic 示例:`motor/speed`、`motor/temp`、`led/state`



## 🚀 操作流程

```
W5500 初始化(phy link)
      ↓
TCP 连接 Broker
      ↓
MQTTClientInit
      ↓
MQTTConnect(建立会话)
      ↓
MQTTSubscribe(订阅主题)
      ↓
MQTTYield(循环收包)
      ↓
收到消息 → 回调处理
```



## 🧩 启动模板

```c
#include "MQTTClient.h"

MQTTClient  client;
Network     network;
unsigned char sendbuf[128], readbuf[128];

void messageArrived(MessageData *data)
{
    // 处理收到的订阅消息
    // data->message->payload  载荷
    // data->topicName         主题
}

int main(void)
{
    HAL_Init();
    SystemClock_Config();
    MX_SPI1_Init();         // W5500 挂在 SPI 上
    MX_USART1_UART_Init();

    W5500_Init();           // PHY 初始化
    W5500_SetDHCP();        // 获取 IP

    // 1. 网络层初始化 + TCP 连接
    NetworkInit(&network);
    NetworkConnect(&network, "broker.example.com", 1883);

    // 2. MQTT 客户端初始化
    MQTTClientInit(&client, &network, 1000, sendbuf, sizeof(sendbuf), readbuf, sizeof(readbuf));

    // 3. 建立连接
    MQTTPacket_connectData opts = MQTTPacket_connectData_initializer;
    opts.clientID.cstring  = "stm32_client";
    opts.keepAliveInterval = 60;
    opts.cleansession      = 1;
    MQTTConnect(&client, &opts);

    // 4. 订阅
    MQTTSubscribe(&client, "motor/cmd", QOS0, messageArrived);

    // 5. 主循环收包
    while (1)
    {
        MQTTYield(&client, 1000);
    }
}
```



## 📨 发布消息

```c
MQTTMessage msg;
msg.payload     = (void *)"hello";
msg.payloadlen  = 5;
msg.qos         = QOS0;
msg.retained    = 0;
msg.dup         = 0;

MQTTPublish(&client, "motor/status", &msg);
```



## ⚙️ QoS 服务等级

| QoS | 含义 | 适用场景 |
|-----|------|----------|
| 0 | 至多一次,不保证到达 | 高频遥测数据 |
| 1 | 至少一次,可能重复 | 普通控制指令 |
| 2 | 恰好一次,开销大 | 计费/关键指令 |



## 🔧 关键函数速查

| 函数 | 功能 |
|------|------|
| `NetworkInit` | 初始化网络层(绑定 W5500 socket) |
| `NetworkConnect` | 建立 TCP 连接到 Broker |
| `MQTTClientInit` | 初始化客户端,绑定收发缓冲 |
| `MQTTConnect` | 发起 MQTT 连接握手 |
| `MQTTSubscribe` | 订阅主题,注册回调 |
| `MQTTUnsubscribe` | 取消订阅 |
| `MQTTPublish` | 发布消息到指定主题 |
| `MQTTYield` | 循环收包,需周期调用 |
| `MQTTDisconnect` | 断开连接 |



## 🐛 调试技巧

- [ ] W5500 物理链路是否 LINK UP
- [ ] Broker IP/端口是否可达(用 PC 的 `mosquitto_sub` 验证)
- [ ] `MQTTYield` 调用频率是否 < `keepAliveInterval`
- [ ] 发送缓冲区大小是否足够容纳最大包
- [ ] Topic 层级分隔符 `/` 是否正确
- [ ] 中文载荷注意 UTF-8 编码



## 📎 相关笔记

- 理论: [[I_知识节点/Prot-MQTT]]
- 实例: [[II_代码实操/Prot-MQTT实例]]
- W5500 驱动: [[II_代码实操/IC-W5500驱动移植]]
- QS100 NB-IoT: [[II_代码实操/IC-QS100-NBIoT]]
