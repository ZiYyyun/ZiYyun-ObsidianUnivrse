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

