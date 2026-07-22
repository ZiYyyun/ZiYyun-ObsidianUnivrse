#LVGL 


```
                    LVGL
                      │
 ┌────────────────────┼────────────────────┐
 │                    │                    │
 Core               Widget             Rendering
 │                    │                    │
 │                    │                    │
 Object Tree          Label              Draw
 Parent               Button             Layer
 Child                Image              Buffer
 Screen               Switch             GPU
 Display              Dropdown           Dirty Area
 │
 │
 ├──────────── Layout
 │                │
 │         None
 │         Flex
 │         Grid
 │
 ├──────────── Style
 │                │
 │         Color
 │         Border
 │         Radius
 │         Padding
 │         Font
 │         Shadow
 │         State
 │
 ├──────────── Event
 │                │
 │         Click
 │         Press
 │         Scroll
 │         Bubble
 │
 ├──────────── Animation
 │                │
 │         Timeline
 │         Transition
 │
 ├──────────── Input Device
 │                │
 │         Touch
 │         Encoder
 │         Keyboard
 │
 └──────────── Display Driver
                  │
           Flush
           Tick
           DMA
           Double Buffer
```

### API函数结构

在大多数情况下，LVGL小部件的API函数结构如下：

- `lv_ + <widget_name> + create(parent)`
- `lv_ + <widget_name> + set + <property>(widget, value)`
- `lv_ + <widget_name> + get + <property>(widget)`
- `lv_ + <widget_name> + add + <property>(widget)`

### 基本属性
- Position 职位
- Size
- Parent 父親
- Styles 样式
- Event callbacks 事件回调
- Flags like _Clickable_, _Scrollable_, etc. 旗帜如_可点击_、_可滚动_等。
- Etc.

You can set/get these attributes with `lv_obj_set_...` and `lv_obj_get_...` functions. For example:  
可以使用 `lv_obj_set` 和 `lv_obj_get` 函数来设置/获取这些属性。例如：

```c
/* Set basic widget attributes */
lv_obj_set_size(btn1, 100, 50);   /* Set a button's size */
lv_obj_set_pos(btn1, 20, 30);     /* Set a button's position */
```

