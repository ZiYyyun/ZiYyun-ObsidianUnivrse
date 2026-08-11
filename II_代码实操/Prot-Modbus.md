#实操/开发/嵌入式/Linux  #modbus  #Prot

本笔记记录 Modbus 协议在全志 V3s (Linux) 下的实操落地,基于 libmodbus 库。理论部分见 [[I_知识节点/Prot-Modbus]]。

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



## 🧩 libmodbus 简介

> 📦 开源仓库:[stephane/libmodbus](https://github.com/stephane/libmodbus)
> 支持 Modbus RTU(串口) 与 Modbus TCP(网络) 两种模式,Linux/Unix 友好。

### 两种模式对比

| 模式 | 后端 | 适用场景 |
|------|------|----------|
| RTU | `/dev/ttyS*` 串口 | 与本地从机直连 |
| TCP | socket 网络 | 与远程设备/Broker 通信 |



## 🛠️ 交叉编译

V3s 工具链:`arm-linux-gnueabihf-gcc`

```bash
# 1. 下载源码
git clone https://github.com/stephane/libmodbus.git
cd libmodbus

# 2. 生成 configure
./autogen.sh

# 3. 交叉编译配置
./configure \
    --host=arm-linux-gnueabihf \
    --prefix=/opt/libmodbus-v3s \
    CC=arm-linux-gnueabihf-gcc

# 4. 编译安装
make -j4
make install

# 5. 部署到板子
# 库文件:    /opt/libmodbus-v3s/lib/libmodbus.so*
# 头文件:    /opt/libmodbus-v3s/include/modbus/
# 拷贝 .so 到板子 /usr/lib/
```

> ⚠️ 运行时记得 `export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH` 或配置 `ldconfig`



## 🚀 RTU 模式模板

```c
#include <stdio.h>
#include <modbus.h>

int main(void)
{
    modbus_t *ctx;
    uint16_t  hold_regs[10];
    int       rc;

    // 1. 创建 RTU 上下文 (/dev/ttyS1, 9600, 8N1)
    ctx = modbus_new_rtu("/dev/ttyS1", 9600, 'N', 8, 1);
    if (ctx == NULL) {
        fprintf(stderr, "Failed to create ctx\n");
        return -1;
    }

    // 2. 设置从机地址
    modbus_set_slave(ctx, 0x01);

    // 3. 设置超时(秒+微秒)
    modbus_set_response_timeout(ctx, 1, 0);

    // 4. 打开串口
    if (modbus_connect(ctx) == -1) {
        fprintf(stderr, "Connect failed: %s\n", modbus_strerror(errno));
        modbus_free(ctx);
        return -1;
    }

    // 5. 读保持寄存器 (功能码 0x03)
    rc = modbus_read_registers(ctx, 0, 10, hold_regs);
    if (rc == -1) {
        fprintf(stderr, "Read failed: %s\n", modbus_strerror(errno));
    } else {
        printf("Read %d regs OK\n", rc);
    }

    // 6. 写单个寄存器 (功能码 0x06)
    modbus_write_register(ctx, 0, 0x1234);

    // 7. 关闭与释放
    modbus_close(ctx);
    modbus_free(ctx);
    return 0;
}
```

### Makefile

```makefile
CROSS   = arm-linux-gnueabihf-
CC      = $(CROSS)gcc
CFLAGS  = -I/opt/libmodbus-v3s/include
LDFLAGS = -L/opt/libmodbus-v3s/lib -lmodbus

TARGET  = modbus_rtu_demo
SRC     = main.c

$(TARGET): $(SRC)
	$(CC) $(CFLAGS) $< -o $@ $(LDFLAGS)

clean:
	rm -f $(TARGET)
```



## 🌐 TCP 模式模板

```c
#include <stdio.h>
#include <modbus.h>

int main(void)
{
    modbus_t *ctx;
    uint16_t  regs[10];

    // 1. 创建 TCP 上下文 (IP, 端口)
    ctx = modbus_new_tcp("192.168.1.100", 502);
    if (ctx == NULL) {
        fprintf(stderr, "Failed to create ctx\n");
        return -1;
    }

    // 2. 超时配置
    modbus_set_response_timeout(ctx, 1, 0);

    // 3. 连接
    if (modbus_connect(ctx) == -1) {
        fprintf(stderr, "Connect failed: %s\n", modbus_strerror(errno));
        modbus_free(ctx);
        return -1;
    }

    // 4. 读写操作 (与 RTU 完全相同)
    modbus_read_registers(ctx, 0, 10, regs);
    modbus_write_register(ctx, 0, 0x1234);

    // 5. 释放
    modbus_close(ctx);
    modbus_free(ctx);
    return 0;
}
```



## 🔧 常用 API 速查

| 函数 | 功能 |
|------|------|
| `modbus_new_rtu` | 创建 RTU 上下文(设备/波特率/校验/数据位/停止位) |
| `modbus_new_tcp` | 创建 TCP 上下文(IP/端口) |
| `modbus_set_slave` | 设置从机地址 |
| `modbus_set_response_timeout` | 设置响应超时 |
| `modbus_connect` | 打开串口 / 建立 TCP 连接 |
| `modbus_read_bits` | 读线圈 (0x01) |
| `modbus_read_input_bits` | 读离散输入 (0x02) |
| `modbus_read_registers` | 读保持寄存器 (0x03) |
| `modbus_read_input_registers` | 读输入寄存器 (0x04) |
| `modbus_write_bit` | 写单线圈 (0x05) |
| `modbus_write_register` | 写单寄存器 (0x06) |
| `modbus_write_bits` | 写多线圈 (0x0F) |
| `modbus_write_registers` | 写多寄存器 (0x10) |
| `modbus_close` | 关闭连接 |
| `modbus_free` | 释放上下文内存 |
| `modbus_strerror` | 错误码转字符串 |



## ⚙️ 串口配置要点

V3s 上用 `/dev/ttyS*`,如果出现权限问题:

```bash
# 1. 查看串口设备
ls /dev/ttyS*

# 2. 赋权限
chmod 666 /dev/ttyS1

# 3. 检查是否被其他程序占用
fuser /dev/ttyS1
```

> 📌 485 方向控制:V3s 通常配合 GPIO 切换收发方向,需要在读写前拉高/拉低 RS485 的 DE/RE 引脚。



## 🐛 调试技巧

- [ ] `dmesg | grep tty` 确认串口节点是否注册成功
- [ ] 用 PC 端 Modbus Poll 作为从机验证主机程序
- [ ] 检查波特率、校验位是否与从机一致
- [ ] RTU 模式下 T3.5 超时由 libmodbus 自动处理
- [ ] TCP 模式注意防火墙是否放行 502 端口
- [ ] 多线程访问同一个 `modbus_t*` 需要加锁



## 📎 相关笔记

- 理论: [[I_知识节点/Prot-Modbus]]
- 单片机移植版: [[II_代码实操/Prot-Modbus移植]]
- 通信协议汇总: [[I_知识节点/Prot-Modbus]]
