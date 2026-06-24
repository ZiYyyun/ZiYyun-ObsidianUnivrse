#理论/开发/嵌入式 


### CRC8经典校验函数

```c
/******************************************************************************
 * @brief  CRC8计算
 * @param  data   数据数组
 * @param  len    数据长度
 * @return CRC值
 ******************************************************************************/
uint8_t CRC8_Calculate(uint8_t *data, uint16_t len)
{
    uint8_t crc = 0;                 // CRC初值
    while (len--)                    // 还有数据没处理
    {
        crc ^= *data++;              // 当前数据与CRC异或
        for (uint8_t i = 0; i < 8; i++) // 一个字节有8位
        {
            if (crc & 0x80)          // 判断最高位是不是1
            {
                crc = (crc << 1) ^ 0x07;
            }
            else
            {
                crc <<= 1;
            }
        }
    }
    return crc;
}
```