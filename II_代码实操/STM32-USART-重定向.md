

```c
include "stdio.h"
```

### 重定向printf
```c
int fputc(int ch, FILE *f) 
{ 
    HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 0xffff); 
    return ch; 
}
```

### 重定向scanf
```c
int fgetc(FILE *f) 
{ 
	uint8_t ch = 0; 
	HAL_UART_Receive(&huart1, &ch, 1, 0xffff); 
	return ch; 
}
```