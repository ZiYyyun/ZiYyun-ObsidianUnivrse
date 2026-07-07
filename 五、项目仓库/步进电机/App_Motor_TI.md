#实操/开发/嵌入式/STM32/项目/步进电机

### 判断运动轨迹

在这里提到的“速度”指的就是`Pulse`
```c
  
accelerationDistance =  
(  
Motor_GetMaxPulseSpeed() *  
Motor_GetMaxPulseSpeed()  
- 
Motor_GetCurrentPulseSpeed() *  
Motor_GetCurrentPulseSpeed()  
)  
/  
(  
2.0f *  
Motor_GetAcceleration()  
);  
  
if (accelerationDistance <  
fabsf(Motor_GetTargetPulse()) / 2.0f)  
{  
isTrapezoidProfile = 1;  
}  
else  
{  
isTrapezoidProfile = 0;  
}
```
这里第一部分：( v1^2 - v0^2 ) / 2a = s(位移)
这里的v1是我们预设好的最大速度（超过这个速度电机堵转）

这个公式得出s的意思是：我们要加速到最大速度，还差多少个pulse。（减速同理）

```c
     /\
    /  \
   /    \
```
如图，一个完整地路段存在加速和减速两个部分。所以我们把s✖️2，得出这段路程总共需要的加速减速时间 那么如果我们目标速度没达到的话，会发现这个差小于目标，反之它们的差就是中间那一段平稳的轨迹


### 更新速度

```c
    float newSpeed = 0;

    /*
     * 加速区
     */
    if (Motor_GetCurrentPulse()
        < accelerationDistance)
    {
        /*
         * v²=v0²+2as
         */

        newSpeed =
            sqrtf(
                Motor_GetCurrentPulseSpeed() *
                Motor_GetCurrentPulseSpeed()

                +

                2.0f *
                Motor_GetAcceleration() *
                1.0f
            );
    }

    /*
     * 匀速区
     */
    else if (
        isTrapezoidProfile

        &&

        Motor_GetCurrentPulse()
        <
        fabsf(Motor_GetTargetPulse())
        -
        accelerationDistance
    )
    {
        newSpeed =
            Motor_GetMaxPulseSpeed();
    }
    /*
     * 减速区
     */
    else
    {
        /*
         * v²=v0²-2as
         */
        newSpeed =
            sqrtf(
                Motor_GetCurrentPulseSpeed() *
                Motor_GetCurrentPulseSpeed()
                -
                2.0f *
                Motor_GetAcceleration() *
                1.0f
            );
    }

    /*
     * 最小速度限制
     */
    if (newSpeed <
        Motor_GetMinPulseSpeed())
    {
        newSpeed =
            Motor_GetMinPulseSpeed();
    }

    /*
     * 最大速度限制
     */
    if (newSpeed >
        Motor_GetMaxPulseSpeed())
    {
        newSpeed =
            Motor_GetMaxPulseSpeed();
    }

    /*
     * 保存当前速度
     */
    Motor_SetCurrentPulseSpeed(
        newSpeed);

    /*
     * 更新ARR
     *
     * pulse/s
     * ↓
     * us
     * ↓
     * ARR
     */
    __HAL_TIM_SET_AUTORELOAD(
        &htim1,
        Motor_GetPulsePeriodUS());
```