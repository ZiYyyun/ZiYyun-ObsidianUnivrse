![[Pasted image 20240823145617.png]]

### 文字取模
```c
{0x00,0x40,0x42,0x44,0x58,0x40,0x40,0x7F,0x40,0x40,0x50,0x48,0xC6,0x00,0x00,0x00},
{0x00,0x40,0x44,0x44,0x44,0x44,0x44,0x44,0x44,0x44,0x44,0x44,0xFF,0x00,0x00,0x00},/*"当",0*/

{0x08,0x08,0xE8,0x29,0x2E,0x28,0xE8,0x08,0x08,0xC8,0x0C,0x0B,0xE8,0x08,0x08,0x00},
{0x00,0x00,0xFF,0x09,0x49,0x89,0x7F,0x00,0x00,0x0F,0x40,0x80,0x7F,0x00,0x00,0x00},/*"前",1*/

{0x00,0x00,0x00,0xFE,0x82,0x82,0x82,0x82,0x82,0x82,0x82,0xFE,0x00,0x00,0x00,0x00},
{0x00,0x00,0x00,0xFF,0x40,0x40,0x40,0x40,0x40,0x40,0x40,0xFF,0x00,0x00,0x00,0x00},/*"日",2*/

{0x00,0x04,0xFF,0x24,0x24,0x24,0xFF,0x04,0x00,0xFE,0x22,0x22,0x22,0xFE,0x00,0x00},
{0x88,0x48,0x2F,0x09,0x09,0x19,0xAF,0x48,0x30,0x0F,0x02,0x42,0x82,0x7F,0x00,0x00},/*"期",3*/

{0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00},
{0x00,0x00,0x36,0x36,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00},/*"：",4*/

{0x00,0x20,0x18,0xC7,0x44,0x44,0x44,0x44,0xFC,0x44,0x44,0x44,0x44,0x04,0x00,0x00},
{0x04,0x04,0x04,0x07,0x04,0x04,0x04,0x04,0xFF,0x04,0x04,0x04,0x04,0x04,0x04,0x00},/*"年",5*/

{0x00,0x00,0x00,0xFE,0x22,0x22,0x22,0x22,0x22,0x22,0x22,0x22,0xFE,0x00,0x00,0x00},
{0x80,0x40,0x30,0x0F,0x02,0x02,0x02,0x02,0x02,0x02,0x42,0x82,0x7F,0x00,0x00,0x00},/*"月",6*/

{0x00,0x00,0x00,0xFE,0x82,0x82,0x82,0x82,0x82,0x82,0x82,0xFE,0x00,0x00,0x00,0x00},
{0x00,0x00,0x00,0xFF,0x40,0x40,0x40,0x40,0x40,0x40,0x40,0xFF,0x00,0x00,0x00,0x00},/*"日",7*/
```

```c
/**
  ****************************************************************************
  * File Name          : main.c
  * Description        : Main program body
  ******************************************************************************
  */
#include <string.h>
#include "board.h"
#include "hal_key.h"
#include "tim-board.h"
#include "timer_handles.h"

#include "usart1-board.h"
#include "flash.h"

//flash地址
#define nian_FLASH  0x0800f300
#define yue_FLASH	  0x0800f400
#define ri_FLASH		0x0800f500

//2021 年
//10 月 11 日
uint32_t nian=2021,yue=10,ri=11,y = 0;

uint32_t y_cache,tu = 0;
//y_cache：存储y的缓存用于清除 <
//tu：切换运行图 0 为初始界面 1 为设置图 2 为修改图

void Init() {
    BoardInitMcu();
    BoardInitPeriph();
    keys_init();//按键初始化
    setTimer2Callback(Time2Handler);
    Tim2McuInit(1);//定时器初始化，设置定时中断1ms中断一次
}

//初始运行图显示当前日期（年、月、日)。
void Initial_operation_diagram()
{
	//当前日期：
	OLED_ShowCHinese(0,0,50);//当
	OLED_ShowCHinese(16,0,51);//前
	OLED_ShowCHinese(16+16,0,52);//日
	OLED_ShowCHinese(16+16+16,0,53);//期
	OLED_ShowCHinese(16+16+16+16,0,54);//：
	//OLED_ShowCHinese(uint8_t x,uint8_t y,uint8_t no);

	OLED_ShowNum(0,3,nian,4,16);
	//OLED_ShowNum(uint8_t x,uint8_t y,uint32_t num,uint8_t len,uint8_t size);
	OLED_ShowCHinese(5*8,3,55);//年
	
	OLED_ShowNum(0,6,yue,2,16);
	OLED_ShowCHinese(3*8,6,56);//月
	OLED_ShowNum(3*8+16+8,6,ri,2,16);
	OLED_ShowCHinese(3*8+16+4*8,6,57);//日
}

//显示设置图
void Setup_Diagram()
{
	OLED_ShowCHinese(0,0,55);//年
	OLED_ShowCHinese(16,0,54);//：
	OLED_ShowNum(16+16+8+8,0,nian,4,16);
	
	OLED_ShowCHinese(0,3,56);//月
	OLED_ShowCHinese(16,3,54);//：
	OLED_ShowNum(16+16+8+8 +8+8,3,yue,2,16);
	
	OLED_ShowCHinese(0,6,57);//日
	OLED_ShowCHinese(16,6,54);//：
	OLED_ShowNum(16+16+8+8 +8+8,6,ri,2,16);
	
}

//按键
void Key()
{
	if(isKey4Pressed())
	{
		resetKey4();
		OLED_Clear();//屏幕清除
		Setup_Diagram();//设置图
		//显示三角型
		tu++;if(tu > 2)tu = 1;//切换图
		WriteFlashTest(nian_FLASH,nian);
		WriteFlashTest(yue_FLASH,yue);
		WriteFlashTest(ri_FLASH,ri);
		//WriteFlashTest(uint32_t Addr,uint32_t WriteFlashData);//写入
	}
	
	if(isKey2Pressed())//上移动用 -
	{
		resetKey2();
		if(tu == 1)
		{
			y_cache = y;
			y = y - 3;
			if(y > 6)  y = 6;// 确保 y 不会小于 0
		}
		if(tu == 2 && y == 0) nian++;
		if(tu == 2 && y == 3)	yue++;
		if(tu == 2 && y == 6) ri++;
		if(yue > 12)yue = 1;
		if(ri > 30)ri = 1;
	}
	if(isKey3Pressed())//下移动用 +
	{
		resetKey3();
		if(tu == 1)
		{
			y_cache = y;
			y = y + 3;
			if(y > 6) y = 0;// 确保 y 不会大于 6
		}
		if(tu == 2 && y == 0) nian--;
		if(tu == 2 && y == 3)	yue--;
		if(tu == 2 && y == 6) ri--;
		if(yue > 12 || yue == 0 )yue = 12;
		if(ri > 30 || ri == 0)ri = 30;
	}
	if(tu == 1)
	{
		OLED_ShowString(16+16 +8+8 +4*8 +4*8 - 2*8,y_cache,(uint8_t *)"   ");//清除三角
		OLED_ShowString(16+16 +8+8 +4*8 +4*8 - 2*8,y,(uint8_t *)"<  ");//显示三角
		GpioWrite(&Led2,1);//关闭*符号，板上的LED2 灯熄灭
	}
	if(tu == 2)
	{
		OLED_ShowString(16+16 +8+8 +4*8 +4*8 - 2*8,y,(uint8_t *)"< *");//显示*
		GpioWrite(&Led2,0);//在当前设置项开启*符号，板上的LED2 灯点亮，
		Setup_Diagram();//刷新数据
	}
}

//串口接收
void ck()
{
	if(Get_F_USART1_RX_FINISH())
	{
		//存储成功的返回.
		uint8_t ck_data_true[]  = {0xFB, 0x01, 0xFE};
		//存储失败的返回
		uint8_t ck_data_false[] = {0xFB, 0x00, 0xFE};
		bool USART1_TX_Flag = false;//发送成功失败标志位
		if(USART1_RX_BUF[0] == 0xFB)
		{
			//年
			if(USART1_RX_BUF[1] == 0x01 && USART1_RX_BUF[4] == 0xFE)
			{
				nian = USART1_RX_BUF[2] * 100;//千位百位
				nian = nian + USART1_RX_BUF[3];//十位个位
				USART1_TX_Flag = true;//成功
				//USART1_SendStr(uint8_t *Data, uint16_t length);
			}
			//月
			else if(USART1_RX_BUF[1] == 0x02 && USART1_RX_BUF[3] == 0xFE)
			{
				if(USART1_RX_BUF[2] < 0x0D && USART1_RX_BUF[2] > 0x00)
				{
					yue = USART1_RX_BUF[2];//修改月
					USART1_TX_Flag = true;//成功
				}
			}
			//日
			else if(USART1_RX_BUF[1] == 0x03 && USART1_RX_BUF[3] == 0xFE)
			{
				if(USART1_RX_BUF[2] < 0x1F && USART1_RX_BUF[2] > 0x00)
				{
					ri = USART1_RX_BUF[2];//修改日
					USART1_TX_Flag = true;//成功
				}
			}
			if(USART1_TX_Flag)USART1_SendStr(ck_data_true,3);//发送成功
			else USART1_SendStr(ck_data_false,3);//发送失败
			
			USART1_ReceiveClr();//串口1相关寄存器和标志位清空
			if(tu == 0) Initial_operation_diagram();//刷新数据
			if(tu == 1 || tu == 2)Setup_Diagram();//刷新数据
			WriteFlashTest(nian_FLASH,nian);
			WriteFlashTest(yue_FLASH,yue);
			WriteFlashTest(ri_FLASH,ri);
		}
	}
}

/**
 * Main application entry point.
 */
int main( void )
{
    Init();
		OLED_Init();//OLED屏幕初始化
		USART1_Init(115200);//串口初始化
		nian = PrintFlashTest(nian_FLASH);
		yue = PrintFlashTest(yue_FLASH);
		yue = PrintFlashTest(yue_FLASH);
		//uint32_t PrintFlashTest(uint32_t FLASH_ADDR);//读取	
		Initial_operation_diagram();//初始运行图显示当前日期（年、月、日)。
    while( 1 )   
    {
			Key();//按键
			ck();//串口
    }
}

```