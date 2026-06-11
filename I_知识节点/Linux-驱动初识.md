### 驱动的分类
> Linux将驱动分为 **字符设备** 、 **网络设备** 、 **块设备** 三类
- 字符设备：必须以串行顺序依次进行访问，如 鼠标
- 网络设备：按照任意顺序进行访问， 如 硬盘
- 块设备：面向数据包的接收和发送
### Linux驱动的编译
|特性|编译进内核|内核模块|
|---|---|---|
|加载方式|自动加载|手动加载（insmod/modprobe）|
|内存占用|增加内核体积|按需占用内存|
|调试灵活性|需重启内核|可动态加载/卸载|
|适用场景|关键驱动|非必需驱动|
#### Linux内核模块的编译
```shell makefile
obj-m += helloworld.o
KDIR:=/home/topeet/liunx-kernel
PWD?$=(shell pwd)
all:
	make -C $(KDIR) M=$(PWD) modules
clean:
	rm -f *.ko *.o *.mod.o *.symvers *.order
```