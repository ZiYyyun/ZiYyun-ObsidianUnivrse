

### assert断言











### 使用枚举统一状态码
```c
typedef enum {
    OK = 0,
    ERR_INVALID_ARG = -1,
    ERR_IO = -2,
    ERR_NO_MEMORY = -3,
    ERR_PARSE = -4,
} Status;
```

```c
Status parse_config(const char *path);
Status load_file(const char *path, char **out_buf);
Status init_device(void);
```

> 调用函数时：
```c
Status st;

st = init_device();
if (st != OK) {
    return st;
}

st = load_file("config.txt", &buf);
if (st != OK) {
    return st;
}

st = parse_config(buf);
if (st != OK) {
    return st;
}
```


### 宏定义减少重复

```c
#define RETURN_IF_ERROR(expr)       \
    do {                            \
        Status _st = (expr);        \
        if (_st != OK) {            \
            return _st;             \
        }                           \
    } while (0)
```

```c
Status run(void)
{
    char *buf = NULL;
    RETURN_IF_ERROR(init_device());
    RETURN_IF_ERROR(load_file("config.txt", &buf));
    RETURN_IF_ERROR(parse_config(buf));
    free(buf);
    return OK;
}
```
