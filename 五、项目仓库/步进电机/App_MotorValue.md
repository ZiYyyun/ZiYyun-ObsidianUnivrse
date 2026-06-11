
[^1]
### 角度与脉冲转换函数

```c
float PulseToAngle(float pulse)  
{  
	return pulse * 360.0f / MOTOR_PULSE_PER_REV;  
}
```
`360.0f`表示一圈是三百六十度，`MOTOR_PULSE_PER_REV`表示一圈需要的脉冲次数
这两个变量计算出来：一次脉冲旋转的度数 
于是这个结果再✖️pulse（当前传参的脉冲次数）便会得出当前角度

```c
float AngleToPulse(float angle)  
{  
return angle * MOTOR_PULSE_PER_REV / 360.0f;  
}
```
一圈所需脉冲数➗️360度得：一度对应的脉冲次数
最后在✖️角度，得出：传入的角度的脉冲次数


### 转速与脉冲转换函数

```c
float PulseSpeedToRPM(float PulseSpeed)
{
	return PulseSpeed * 60.0f / MOTOR_PULSE_PER_REV;
}
```
- PulseSpeed：脉冲次数（每秒）
- RPM：转速（每秒）
传入的参数PulseSpeed✖️60秒得出Pulse
再除以一圈所需脉冲数等于：RPM（每秒所需转圈数）


```c
float RPMToPulseSpeed(float RPM)
{
	return RPM * MOTOR_PULSE_PER_REV / 60.0f;
}
```
- RPM是圈数（每秒）✖️ 每圈所需脉冲数 = 总共脉冲数
- 再除以时间得出脉冲速度（pulsespeed）

[^1]: 注：本驱动由于原版可读性太差 文中已是GPT重置版
