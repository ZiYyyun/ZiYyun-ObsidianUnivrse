
### 
复制软件中生成好的字模 新建一个头文件。

```c
extern uint8_t title[][32];

const uint8_t title[][32] = 
{
	/* 这里粘贴字模 */
}
```


### 引用字模文件

- 在对应的驱动文件中include新建好的字模文件

### 修改驱动代码

```c
void Inf_OLED_ShowChinese(uint8_t x, uint8_t y, uint8_t num, uint8_t size1, uint8_t mode)
{
    uint8_t m, temp;
    uint8_t x0 = x, y0 = y;
    uint16_t i, size3 = (size1 / 8 + ((size1 % 8) ? 1 : 0)) * size1;
    for (i = 0; i < size3; i++)
    {
        if (size1 == 16)
/*-------------------这个位置-----------------------------*/
            temp = title[num][i]; //--------------这个位置------
        else
            return;
        for (m = 0; m < 8; m++)
        {
            if (temp & 0x01)
                Inf_OLED_DrawPoint(x, y, mode);
            else
                Inf_OLED_DrawPoint(x, y, !mode);
            temp >>= 1;
            y++;
        }
        x++;
        if ((x - x0) == size1)
        {
            x = x0;
            y0 = y0 + 8;
        }
        y = y0;
    }
}
```

在对应的驱动文件中找到`ShowChinese`函数，找到关键数组，替换变量名