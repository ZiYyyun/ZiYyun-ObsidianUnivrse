
# 第二步：n-> 是什么意思

很多初学者卡在这里。

你先看普通结构体。

```
Network mqtt_network;
```

访问成员：

```
mqtt_network.my_socket = 0;
```

---

如果是指针：

```
Network *n;
```

那么：

```
(*n).my_socket = 0;
```

意思：

```
先找到n指向的结构体再访问成员
```

---

因为太常用。

C语言专门设计了：

```
->
```

运算符。

所以：

```
(*n).my_socket
```

等价于：

```
n->my_socket
```

---

因此：

```
n->my_socket = sn;
```

翻译：

```
把n指向结构体里的my_socket赋值为sn
```

---

实际上：

```
mqtt_network.my_socket = 0;
```



