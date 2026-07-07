#C 
#理论/CS 


```c
// 1. 指针函数：返回 int*，只是函数，不是指针 
int* func1(int a); 
// 2. 函数指针：p 是指针，存同格式函数的地址 
int (*p)(int a);
```




```c
// 实现 open 逻辑 
int codec_open(const audio_codec_data_if_t *h, void *data_cfg, int cfg_size) { 
// h 就是结构体自身指针，可以拿上下文 
	printf("打开音频接口\n"); 
	return 0; 
} 

// 实现 is_open 
bool codec_is_open(const audio_codec_data_if_t *h) 
{ return true; } 

// 实现 close 
int codec_close(const audio_codec_data_if_t *h) { 
	printf("关闭音频接口\n"); 
	return 0; 
	}
```

