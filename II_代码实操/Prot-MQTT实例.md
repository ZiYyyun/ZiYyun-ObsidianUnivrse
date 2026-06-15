#理论/开发/嵌入式  #Prot  

本案例使单片机作为客户端来进行MQTT功能实现，其中所需软件库来源于W5500官方移植




### 操作流程
```
W5500初始化
↓
TCP连接Broker
↓
MQTTClient初始化
↓
MQTTConnect
↓
MQTTSubscribe
↓
MQTTYield(循环收包)
↓
收到消息回调
```