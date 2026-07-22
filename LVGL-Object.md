#LVGL 


> [!NOTE] 本节目标
> Object是什么？
> 为什么所有控件都是Object？
> 为什么传parent？
> 为什么返回lv_obj_t？
> Object里面保存什么？


### 基本概念

假设我们要实现一个画面：
```
+-------------------------+
|        Weather          |
|                         |
|      ☀️  28 °C         |
|                         |
|      [ Refresh ]        |
+-------------------------+
```

在这个画面中，所包含的组件有：
- Screen
- Label
- Image
- Button
但是在LVGL的世界里，它们一律被抽象为`Object`

