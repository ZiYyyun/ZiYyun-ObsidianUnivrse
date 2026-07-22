#理论/开发/嵌入式 
#LVGL


```
LVGL
│
├──────── 1. Core（核心）
│           │
│           ├── Object
│           ├── Object Tree
│           ├── Class
│           ├── Display
│           ├── Screen
│           ├── Group
│           └── Timer
│
├──────── 2. Layout（布局）
│           │
│           ├── None
│           ├── Flex
│           ├── Grid
│           ├── Align
│           ├── Position
│           ├── Size
│           ├── Padding
│           ├── Margin（实际上LVGL没有真正CSS意义的margin）
│           └── Scroll
│
├──────── 3. Style（样式）
│           │
│           ├── Style
│           ├── Property
│           ├── State
│           ├── Part
│           ├── Theme
│           ├── Transition
│           └── Animation
│
├──────── 4. Widget（控件）
│           │
│           ├── Label
│           ├── Button
│           ├── Image
│           ├── Arc
│           ├── Slider
│           ├── Switch
│           ├── Bar
│           ├── Chart
│           ├── Dropdown
│           ├── Roller
│           ├── TextArea
│           ├── Keyboard
│           ├── Menu
│           ├── List
│           └── ...
│
├──────── 5. Event（事件）
│           │
│           ├── Event Code
│           ├── Callback
│           ├── Bubble
│           ├── Trickle
│           └── User Data
│
├──────── 6. Draw（绘制）
│           │
│           ├── Draw Task
│           ├── Layer
│           ├── Renderer
│           ├── Mask
│           ├── Blend
│           └── Image Decoder
│
├──────── 7. Display（显示驱动）
│           │
│           ├── Flush
│           ├── Buffer
│           ├── Double Buffer
│           ├── DMA
│           ├── Tick
│           └── Refresh
│
├──────── 8. Input Device（输入设备）
│           │
│           ├── Touch
│           ├── Encoder
│           ├── Mouse
│           ├── Keyboard
│           └── Gesture
│
├──────── 9. Data Binding（数据绑定）
│           │
│           ├── Subject
│           ├── Observer
│           ├── Bind Text
│           ├── Bind Value
│           └── Two-way Binding
│
└──────── 10. Resource（资源）
            │
            ├── Font
            ├── Image
            ├── FileSystem
            ├── PNG/JPG Decoder
            └── Cache
```

# LVGL 布局

---




> 如果拿前端与之对比：

| Web 前端      | LVGL                                      |
| ----------- | ----------------------------------------- |
| HTML        | `lv_obj_create()`、`lv_btn_create()` 等创建对象 |
| DOM Tree    | `lv_obj_t` 对象树                            |
| `<div>`     | `lv_obj`                                  |
| `<button>`  | `lv_btn`                                  |
| `<span>`    | `lv_label`                                |
| CSS         | `lv_obj_set_style_xxx()`                  |
| Flex        | `lv_obj_set_flex_flow()`                  |
| Grid        | `lv_obj_set_grid_dsc_array()`             |
| JavaScript  | 事件回调（`lv_obj_add_event_cb()`）             |
| `onclick`   | `LV_EVENT_CLICKED`                        |
| `innerText` | `lv_label_set_text()`                     |
| 浏览器渲染       | LVGL Draw Engine                          |
| GPU         | LCD 驱动（SPI/RGB/MIPI 等）                    |


**LVGL = 面向对象 + Widget树 + Flex/Grid布局**
**HTML = DOM树 + CSS盒模型 + 浏览器排版引擎**

*LVGL 学的是**对象如何摆放** ,而前端更多是**盒子如何计算大小***


## 开发逻辑

```
创建对象
      ↓
设置父子关系
      ↓
设置尺寸和位置
      ↓
设置样式
      ↓
绑定事件
```

> 比如

```c
lv_obj_t *btn = lv_button_create(parent);      //[!DESCRIBE]创建btn对象
lv_obj_set_size(btn, 100, 40);
lv_obj_center(btn);

lv_obj_t *label = lv_label_create(btn);
lv_label_set_text(label, "OK");
lv_obj_center(label);
```

## 布局方向

> 三个基本标志位

|标志|含义|类比 CSS|
|---|---|---|
|`ROW` / `COLUMN`|排列方向：水平 / 垂直|`flex-direction`|
|`WRAP`|超出容器边界时自动换行（或换列）|`flex-wrap: wrap`|
|`REVERSE`|反转排列顺序|`flex-direction: *-reverse`|

### 水平方向

```
ROW（水平，不换行）:
┌──────────────────────┐
│ [1] [2] [3] [4] [5] →│ 超出部分被截断
└──────────────────────┘

ROW_WRAP（水平 + 换行）: 
┌──────────────┐
│ [1] [2] [3]  │
│ [4] [5] [6]  │ 自动折到下一行 
│ [7] [8]      │ 
└──────────────┘
```



### 垂直方向

```
COLUMN（垂直，不换列）:
┌──────┐
│ [1]  │
│ [2]  │
│ [3]  │ ↓ 超出部分被截断
│ [4]  │
│ [5]  │
└──────┘

COLUMN_WRAP（垂直 + 换列）:
┌──────────────────┐
│ [1]  [4]  [7]    │
│ [2]  [5]  [8]    │ 自动折到下一列
│ [3]  [6]         │
└──────────────────┘
```


## 盒子模型

### 内边距
> 在 LVGL 的“盒子模型”中，**Pad（内边距）是指“控件自身的边界”与“其内部内容（或子控件）”之间的距离。**

内边距可以满足控件或者元素：

```
【文字内容示例】

  没设置 Pad (pad=0)           设置了 Pad (pad=15)
  ┌──────────────┐             ┌──────────────────────┐
  │点击我        │             │                      │
  └──────────────┘             │   ┌──────────────┐   │
                               │   │  点击我      │   │
                               │   └──────────────┘   │
                               │                      │
                               └──────────────────────┘
  文字紧贴边缘，很丑           文字四周有 15px 留白，好看


【子控件示例】

  没设置 Pad (pad=0)           设置了 Pad (pad=15)
  ┌──────────────────┐         ┌──────────────────────┐
  │ ┌────┐           │         │                      │
  │ │子件│           │         │   ┌────┐             │
  │ └────┘           │         │   │子件│             │
  │                  │         │   └────┘             │
  │                  │         │                      │
  └──────────────────┘         └──────────────────────┘
  子控件紧贴父容器左上角       子控件距离父容器边缘有 15px
```


## 边框


## 外边距


## Flex 布局

### 主轴对齐

### 交叉轴对齐

### 轨道对齐


## Grid 布局

### 网格描述

### 单元格放置

### 单元格对齐


## 尺寸与位置

### 尺寸设置

### 位置设置

### 自适应尺寸


## 对齐方式

### 自身对齐

### 相对对齐

