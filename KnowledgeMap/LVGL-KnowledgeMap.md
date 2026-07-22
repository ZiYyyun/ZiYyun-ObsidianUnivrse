#LVGL 
#KnowledgeMap


# 🗺️ LVGL 知识地图 (MOC)

> [!info] 核心概念
> LVGL (Light and Versatile Graphics Library) 是一个开源的嵌入式图形库。

## 1. Core（核心）
- [[LVGL-初识]]
- [[LVGL-Object]] (lv_obj)
- [[Object Tree]] (对象树与父子关系)
- [[LVGL Class]] (控件类与继承)
- [[LVGL Display]] (显示接口)
- [[Screen]] (屏幕与默认屏幕)
- [[Group]] (焦点组与键盘导航)
- [[LVGL Timer]] (定时器与任务调度)

## 2. Layout（布局）
- [[None Layout]] (绝对定位)
- [[Flex Layout]] (弹性布局)
- [[Grid Layout]] (网格布局)
- [[Align]] (对齐方式)
- [[Position & Size]] (位置与尺寸计算)
- [[Padding]] (内边距)
- *注：LVGL 没有 CSS 意义的 Margin，通过父容器 Padding 或子对象 Position 实现。*
- [[Scroll]] (滚动与滚动条)

## 3. Style（样式）
- [[LVGL Style]] (样式对象)
- [[Style Property]] (样式属性)
- [[State]] (状态：默认、按下、聚焦等)
- [[Part]] (部件：主体、滚动条、指示器等)
- [[Theme]] (主题与默认样式)
- [[Transition]] (状态过渡动画)
- [[Animation]] (独立动画)

... (以此类推，将你的所有节点加上 `[[ ]]`)