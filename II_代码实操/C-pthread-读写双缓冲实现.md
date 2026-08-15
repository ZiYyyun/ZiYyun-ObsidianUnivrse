#C  #pthread 



### 结构体声明
```c
typedef struct {
    char* buf;          // 👉 实际存数据的内存块（比如：[L][L][D][A][T][A]...）
                        //    L=Length(2字节), D=Data
    uint16_t size;      // 👉 这个盘子总共能装多少字节（总容量）
    uint16_t used_len;  // 👉 目前已经用了多少字节（从0开始增长）
} SubBuffer;
```


```c
typedef struct {
    SubBuffer* buf_arr[2];   // 👉 两个盘子：buf_arr[0] 和 buf_arr[1]
                             //    其中一个当前是“读盘”，另一个是“写盘”
    uint8_t read_index;      // 👉 告诉你：现在哪个盘子是“读盘”？
                             //    比如 read_index = 0 → buf_arr[0] 是读盘
    uint8_t write_index;     // 👉 告诉你：现在哪个盘子是“写盘”？
                             //    比如 write_index = 1 → buf_arr[1] 是写盘
                             //    ⚠️ 注意：read_index 和 write_index 一定不同！
    pthread_mutex_t readLock;   // 👉 “读锁”：多人想同时读？排队！
    pthread_mutex_t writeLock;  // 👉 “写锁”：多人想同时写？排队！
                                //    （防止读写冲突、写写冲突）
} DoubleBuffer;
```


```
+-----------------------------+
|      DoubleBuffer           |
|                             |
|  buf_arr[0] ────────┐       |     ← read_index = 0 → 这个是【读缓冲】
|                     ▼       |
|              +--------------+
|              | SubBuffer    |
|              | buf: [......]| ← 数据存在这里（已用 used_len 字节）
|              | size: 1024   |
|              | used_len: 200|
|              +--------------+
|                             |
|  buf_arr[1] ────────┐       |     ← write_index = 1 → 这个是【写缓冲】
|                     ▼       |
|              +--------------+
|              | SubBuffer    |
|              | buf: [......]| ← 新数据正在往这里写
|              | size: 1024   |
|              | used_len: 50 |
|              +--------------+
|                             |
|  readLock: 🔒 (保护读操作)     |
|  writeLock: 🔒 (保护写操作)    |
+-----------------------------+
```


