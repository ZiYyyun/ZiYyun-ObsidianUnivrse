#理论/开发/嵌入式 
#LVGL

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


### 开发逻辑

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