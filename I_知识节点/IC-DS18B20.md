> [!总结]
  DS18B20 是基于单[[Prot-1-wire]]的数字温度传感器，51 单片机通过 GPIO 模拟 1-Wire 时序，就能完成温度读取，核心就 2 件事：搞准 1-Wire 的 4 个基础时序，记住 DS18B20 的标准操作流程。

### 工作流程

> [!NOTE] Title
> ![[Pasted image 20260420190502.png]]

1. MUC向18B20发送测温命令
2. MCU向18B20发送读取温度命令
3. 18B20向MCU发送寄存器中保存的温度值


### 主要功能函数
> 工作流程
- 初始化函数
- 写命令
- 读命令
- 匹配



#### 写数据
```c
void Dri_Write_SendByte(u8 byte){
for (i = 0; i < 8; i++){
  u8 i = 0;
  
  DQ = 0;
  
  if(byte & 0x01){
   DQ = 1;
   
   byte >>= 1;
  }
 }
}
```