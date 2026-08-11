#实操/开发/嵌入式/Linux  #MQTT  #Prot

本笔记记录 MQTT 协议在全志 V3s (Linux) 下的实操落地,基于 Mosquitto 客户端库 / Paho MQTT-C。理论部分见 [[I_知识节点/Prot-MQTT]]。

---

## 📋 核心角色与概念

| 角色     | 说明                              |
| ------ | ------------------------------- |
| Client | 终端设备(V3s 板子)                    |
| Broker | 服务器(Mosquitto/EMQX 等,可跑在 PC/云端) |
| Topic  | 通信主题,层级用 `/` 分隔                 |



> 常用 Topic 示例:`motor/speed`、`motor/temp`、`led/state`


## 🧩 库选型对比

| 库 | 语言 | 特点 |
|----|------|------|
| Mosquitto (libmosquitto) | C | Eclipse 官方,API 简洁,Linux 友好 ✅ 推荐 |
| Paho MQTT-C | C | Eclipse 官方,功能全面,跨平台 |
| paho-mqtt | Python | 简单易用,适合快速验证 |

本笔记以 **libmosquitto** 为主。



## 🛠️ 交叉编译

V3s 工具链:`arm-linux-gnueabihf-gcc`

### 编译 Mosquitto 库

```bash
# 1. 下载源码
git clone https://github.com/eclipse/mosquitto.git
cd mosquitto

# 2. 修改 config.mk,指定交叉编译
#    WITH_SRV := no       # 不依赖 c-ares
#    WITH_DOCS := no      # 不编译文档

# 3. 交叉编译
make CC=arm-linux-gnueabihf-gcc \
     CXX=arm-linux-gnueabihf-g++ \
     AR=arm-linux-gnueabihf-ar \
     -j4

# 4. 部署到板子(关键文件)
# lib/libmosquitto.so*   →  /usr/lib/
# lib/cpp/libmosquittopp.so* (若用 C++)
# 头文件 mosquitto.h     →  /usr/include/
# 客户端程序 mosquitto_pub/sub (可选)
```

> ⚠️ 运行时记得 `ldconfig` 刷新动态库缓存

### 测试 Broker(PC 端)

```bash
# PC 上快速起一个 Broker
mosquitto -p 1883

# 另开终端订阅
mosquitto_sub -h 127.0.0.1 -t "test/topic" -v

# 发布测试
mosquitto_pub -h 127.0.0.1 -t "test/topic" -m "hello"
```



## 🚀 操作流程

```
网络初始化(ETH/WiFi 就绪)
      ↓
mosquitto_new (创建客户端实例)
      ↓
mosquitto_connect (连接 Broker)
      ↓
mosquitto_connect_callback_set (注册连接回调)
      ↓
mosquitto_message_callback_set (注册消息回调)
      ↓
mosquitto_subscribe (订阅主题)
      ↓
mosquitto_loop_start (后台线程收包)
      ↓
mosquitto_publish (按需发布)
      ↓
mosquitto_loop_stop + mosquitto_destroy (清理)
```



## 💻 客户端模板

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <mosquitto.h>

#define BROKER_HOST  "192.168.1.100"
#define BROKER_PORT  1883
#define CLIENT_ID    "v3s_client_01"

/* 连接回调 */
static void on_connect(struct mosquitto *m, void *obj, int rc)
{
    if (rc == 0) {
        printf("[MQTT] Connected OK\n");
        mosquitto_subscribe(m, NULL, "motor/cmd", 0);
    } else {
        printf("[MQTT] Connect failed: %s\n", mosquitto_connack_string(rc));
    }
}

/* 消息回调 */
static void on_message(struct mosquitto *m, void *obj,
                       const struct mosquitto_message *msg)
{
    printf("[MQTT] Topic: %s  Payload: %.*s\n",
           msg->topic, msg->payloadlen, (char *)msg->payload);
}

int main(void)
{
    struct mosquitto *m;
    int               rc;

    /* 1. 初始化库 */
    mosquitto_lib_init();

    /* 2. 创建客户端实例 */
    m = mosquitto_new(CLIENT_ID, true, NULL);
    if (!m) {
        fprintf(stderr, "Failed to create client\n");
        return -1;
    }

    /* 3. 注册回调 */
    mosquitto_connect_callback_set(m, on_connect);
    mosquitto_message_callback_set(m, on_message);

    /* 4. 设置用户名密码(如 Broker 需要认证) */
    // mosquitto_username_pw_set(m, "user", "pass");

    /* 5. 连接 Broker */
    rc = mosquitto_connect(m, BROKER_HOST, BROKER_PORT, 60);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Connect failed: %s\n", mosquitto_strerror(rc));
        mosquitto_destroy(m);
        mosquitto_lib_cleanup();
        return -1;
    }

    /* 6. 启动后台收包线程(非阻塞) */
    rc = mosquitto_loop_start(m);
    if (rc != MOSQ_ERR_SUCCESS) {
        fprintf(stderr, "Loop start failed: %s\n", mosquitto_strerror(rc));
    }

    /* 7. 主循环:按需发布 */
    while (1) {
        char payload[] = "hello from v3s";
        mosquitto_publish(m, NULL, "motor/status",
                          strlen(payload), payload, 0, false);
        sleep(5);
    }

    /* 8. 清理 */
    mosquitto_loop_stop(m, true);
    mosquitto_disconnect(m);
    mosquitto_destroy(m);
    mosquitto_lib_cleanup();
    return 0;
}
```

### Makefile

```makefile
CROSS   = arm-linux-gnueabihf-
CC      = $(CROSS)gcc
CFLAGS  = -I/opt/mosquitto-v3s/include
LDFLAGS = -L/opt/mosquitto-v3s/lib -lmosquitto -lpthread

TARGET  = mqtt_demo
SRC     = main.c

$(TARGET): $(SRC)
	$(CC) $(CFLAGS) $< -o $@ $(LDFLAGS)

clean:
	rm -f $(TARGET)
```



## 📨 发布与订阅 API

| 函数 | 功能 |
|------|------|
| `mosquitto_publish` | 发布消息到指定主题 |
| `mosquitto_subscribe` | 订阅主题 |
| `mosquitto_unsubscribe` | 取消订阅 |
| `mosquitto_message_callback_set` | 注册消息到达回调 |
| `mosquitto_connect_callback_set` | 注册连接状态回调 |
| `mosquitto_disconnect_callback_set` | 注册断开回调 |



## ⚙️ QoS 服务等级

| QoS | 含义 | 适用场景 |
|-----|------|----------|
| 0 | 至多一次,不保证到达 | 高频遥测数据 |
| 1 | 至少一次,可能重复 | 普通控制指令 |
| 2 | 恰好一次,开销大 | 计费/关键指令 |

> 📌 QoS 2 需要 4 次握手,V3s 资源有限时建议用 QoS 1 兼顾可靠与开销。



## 🔧 核心 API 速查

| 函数 | 功能 |
|------|------|
| `mosquitto_lib_init` | 初始化库(进程级,只调用一次) |
| `mosquitto_new` | 创建客户端实例 |
| `mosquitto_connect` | 连接 Broker(指定 IP/端口/心跳) |
| `mosquitto_connect_async` | 异步连接(配合 loop_start) |
| `mosquitto_disconnect` | 断开连接 |
| `mosquitto_loop_start` | 启动后台线程收包 |
| `mosquitto_loop_stop` | 停止后台线程 |
| `mosquitto_loop_forever` | 阻塞式收包(单线程模式) |
| `mosquitto_publish` | 发布消息 |
| `mosquitto_subscribe` | 订阅主题 |
| `mosquitto_destroy` | 释放客户端实例 |
| `mosquitto_lib_cleanup` | 清理库资源 |
| `mosquitto_strerror` | 错误码转字符串 |



## 🐛 调试技巧

- [ ] 网络层:`ping <broker_ip>` 确认可达
- [ ] 端口:`telnet <broker_ip> 1883` 测试 TCP 连通
- [ ] 用 PC 端 `mosquitto_sub` 监听板子发布的主题
- [ ] 检查 `keepalive` 是否合理(默认 60s)
- [ ] 断线重连:可注册 disconnect 回调 + `mosquitto_reconnect`
- [ ] 中文载荷注意 UTF-8 编码
- [ ] 多线程发布需要加锁(同一 `mosquitto_t*` 非线程安全)



## 📎 相关笔记

- 理论: [[I_知识节点/Prot-MQTT]]
- 单片机+W5500 实例: [[II_代码实操/Prot-MQTT实例]]
- V3s 网络配置: [[II_代码实操/V3s-网络配置]]
