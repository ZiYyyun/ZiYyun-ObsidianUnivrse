#Prot 

MQTT（Message Queuing Telemetry Transport）本质上是一个基于发布-订阅模式（Publish/Subscribe）的轻量级通信协议.专门为低宽带高延时网络设计

---



### 核心概念
MQTT就三个角色：
- `Client`：终端
- `Broker`：服务器
- `Topic`：通讯

> [!NOTE] Topic
> MQTT没有IP地址概念，靠Topic通信，比如：
> - motor/speed
> - motor/temp
> - led/state
> 



### 主要函数

| 函数名               | 核心功能                                 |
| ----------------- | ------------------------------------ |
| `MQTTClientInit`  | 初始化客户端结构体，绑定网络栈、收发缓冲区与命令超时配置         |
| `MQTTConnect`     | 向 Broker 发起 MQTT 连接，阻塞等待连接响应         |
| `MQTTSubscribe`   | 订阅指定主题，注册消息接收回调函数                    |
| `MQTTUnsubscribe` | 取消订阅指定主题                             |
| `MQTTPublish`     | 向指定主题发布消息，完整支持 QoS 0/1/2             |
| `MQTTDisconnect`  | 主动断开与 Broker 的 MQTT 连接               |
| `MQTTYield`       | 单次轮询处理收包与保活心跳，适用于无 OS 的裸机轮询模式        |
| `MQTTStartTask`   | 开启`MQTT_TASK`宏时可用，启动独立线程运行 MQTT 后台循环 |












