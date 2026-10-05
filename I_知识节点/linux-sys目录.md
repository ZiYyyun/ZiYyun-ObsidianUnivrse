#Linux  #理论


### sysfs

```
/sys
├── block/    # 块设备：emmc、sd卡、mmcblk、loop等（ARM最常见mmcblk）
├── bus/      # 总线：i2c、spi、usb、platform、pci、amba（ARM特有AMBA总线）
├── class/    # 设备类：led、gpio、net、tty、rtc、input、power_supply
├── dev/      # 设备号映射目录，字符/块设备号索引
├── devices/  # 完整设备树层次，最重要！所有硬件设备都在这里（对应设备树节点）
├── firmware/ # 固件信息：devicetree(设备树dtb)、efi、arm相关固件
├── fs/       # 文件系统信息
├── kernel/   # 内核参数、debug、cgroup
├── module/   # 加载的内核模块（驱动ko）
├── power/    # 电源管理：休眠、wakelock、suspend，嵌入式ARM高频使用
```

#### bus
```
/sys/bus/
├── amba/         # ARM AMBA总线，AHB/APB外设，SOC内部外设
├── platform/     # platform平台设备，绝大多数片上外设：uart、gpio、timer
├── i2c/          # I2C设备，触摸、rtc、传感器
├── spi/          # SPI外设，flash、显示屏
├── usb/
├── mmc/          # SD/EMMC
```

#### class
```
/sys/class/
├── gpio/         # gpiochip，控制GPIO输出电平，嵌入式必用
├── led/          # LED子系统，控制板载LED
├── rtc/          # 实时时钟
├── net/          # 网卡eth/wlan
├── tty/          # 串口ttyS/ttyUSB
├── input/        # 按键、触摸屏
├── power_supply/ # 电池、充电管理
├── thermal/      # 温度传感器、温控
├── watchdog/     # 看门狗
```


> [!NOTE] ARM 独有的特色节点（x86 基本没有）
> 1. `/sys/bus/amba`：ARM AHB/APB 总线
> 2. `/sys/firmware/devicetree`：设备树
> 3. `/sys/class/gpio` 大量片上 gpiochip
> 4. `/sys/devices/platform/soc` soc 内部外设树
> 5. 很多 SOC 会在 `/sys/class/thermal` 导出 CPU 温度


