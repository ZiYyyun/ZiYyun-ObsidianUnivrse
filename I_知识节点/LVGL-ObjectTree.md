#LVGL 

有关`lv_object_tree.h`的API在[lv_obj_tree.h — LVGL documentation](https://lvgl.io/docs/open/9.1/API/core/lv_obj_tree)

---



> [!NOTE] Questions
> 1. 为什么 LVGL 一定要有 Object Tree？
> 2. parent 到底是什么？
> 3. children 是怎么管理的？
> 4. 为什么位置都是相对于 parent？
> 5. 为什么删除 parent，所有子对象都会消失？

### 基本概念

```c
lv_obj_t *btn = lv_button_create(parent);
```
> [!QUESTION] btn是真的被创建在了parent上吗？

准确来讲：其实不是。
> `btn` 只是被挂载到了`parent`的子列表(chlidren)里面，如下：

```
parent
│
├── child1
├── child2
├── child3
└── btn
```

> **每个节点(Object)都只有一个父(Parent)，但是可以有很多孩子(Children)**,这就是对象树(Object Tree)




### Display

需要注意的是，`display`并不是`Object`，只是`display`管理当前`screen`。所以以后我们切换界面：

```
Display                           Display                

↓                                    ↓

Screen1                           Screen2
```








```
                 Display
                     │
             Active Screen
                     │
        ┌────────────┴────────────┐
        │                         │
    Container                 Keyboard
        │
   ┌────┴────┐
   │         │
 Button    Label
   │
 Image
```


