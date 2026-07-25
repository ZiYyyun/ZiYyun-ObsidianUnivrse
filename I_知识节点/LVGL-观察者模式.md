#LVGL 

```
            speedSubject
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
  SpeedLabel   AvgLabel   HomeLabel
```

你创建一个 **`lv_subject_t`**（主题），它就是一个"数据盒子"，里面装着 int、string、color 等值。然后你让 UI 控件（label、slider、bar……）**订阅**这个 subject，成为它的"观察者"。从此以后，**只要你改了 subject 里的数据，所有订阅了它的控件会自动刷新显示**；反过来，用户在 slider 上拖了一下，subject 的值也会自动更新，其他绑定了同一个 subject 的控件也跟着联动。整个过程你不需要写任何"手动同步"的代码。

---


> [!QUESTION] 它解决了什么问题？

> 没有观察者模式时：

```c
// 温度变了，你得手动更新每一个相关控件 
lv_label_set_text(temp_label, "25°C"); 
lv_bar_set_value(temp_bar, 25, LV_ANIM_OFF); 
lv_arc_set_value(temp_arc, 25); // 每加一个控件，就要多写一行……
```

> 有了观察者模式之后：

```c
// 1. 创建 subject 
lv_subject_t temp_subject; 
lv_subject_init_int(&temp_subject, 25); // 2. 控件订阅（一次绑定，永久生效） 
lv_label_bind_text(temp_label, &temp_subject); 
lv_bar_bind_value(temp_bar, &temp_subject); lv_arc_bind_value(temp_arc, &temp_subject); // 3. 以后只需要改数据，界面全自动更新 
lv_subject_set_int(&temp_subject, 30); // 三个控件同时刷新！
```


> [!NOTE] 核心价值
> **解耦合。** 数据层（业务逻辑）不需要知道有哪些 UI 控件在显示它，UI 控件也不需要知道数据从哪来。你加控件、删控件、换控件，业务代码一行不用改。这就是经典的 **MVC / 模型-视图分离** 思想在嵌入式 GUI 中的落地。
