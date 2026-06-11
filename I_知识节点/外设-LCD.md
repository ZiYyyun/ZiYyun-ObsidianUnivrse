#理论/开发/嵌入式 



> [!NOTE] 引脚定义
> ![[SCH_ZET6开发板_1_2024-07-01.pdf#page=14&rect=335,258,508,423&color=yellow|SCH_ZET6开发板_1_2024-07-01, p.14]]
> - D0-D15是16位数据总线接口。分别接FSMC的D0-D15。
> - RST是LCD复位引脚，低电平复位。接LCD-RST（PG15）。
> - RD是读控制引脚，上升沿时读数据。接FSMC-NOE（PD4）。
> - WR是写控制引脚，上升沿时写数据。接FSMC-NWE（PD5）。
> - RS是数据或命令选择引脚RS=1写数据，RS=0写命令。接FSMC-A10（PG0）。
> - CS是片选引脚，低电平有效。接FSMC-NE4（PG12）。
> - LEDA是背光电源（3.0V-3.4V）引脚。
> - LEDK是背光亮度控制引脚。通过LCD-BG（PB0）来驱动MOS管Q5的导通电流。可以通过给LCD-BG输出PWM波来控制背光的亮度。占空比越大，背光就会越亮。
> - YD、XL、YU、XR是触摸屏控制引脚








