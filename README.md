# 📚 Obsidian 知识宇宙

> 💡 嵌入式开发者的个人知识库 · 基于 Obsidian + Git 持续维护

## 🎯 简介

这是一个围绕 **嵌入式开发** 为主线的 Obsidian 知识库,涵盖 STM32、ESP32、FreeRTOS、LVGL、通信协议、IC 驱动等核心方向,辅以 Rust / C++ / Qt 拓展,搭配每日笔记和项目实战沉淀。

## 📁 目录结构

```
obsidian-/
├── 📝 Daily/              # 每日笔记 + 练习
├── 🧠 I_知识节点/         # 理论知识(STM32/FreeRTOS/LVGL/Prot/IC...)
├── 💻 II_代码实操/         # 实践代码(寄存器/HAL 双版本)
├── 📦 III_资源仓库/       # Datasheet、剪藏、参考手册
├── 🛠️ IV_基建工具/        # 模板、画布、首页
├── 🚀 五、项目仓库/       # 步进电机、牛马定位器、BMS、Ai小智...
├── 🗂️ 索引/              # 主索引导航
├── 📦 归档/               # 历史归档
├── 🖼️ 附件/              # 图片附件
└── 📊 *.base             # 原生数据库视图
```

<details>
<summary>📁 笔记示例</summary>

| 主题 | 笔记 |
|------|------|
| STM32 寄存器 | `I_知识节点/STM32-GPIO寄存器.md` |
| FreeRTOS | `I_知识节点/FreeRTOS-任务通知.md` |
| 通信协议 | `I_知识节点/Prot-Modbus.md` |
| IC 驱动 | `II_代码实操/IC-ES8311.md` |
| 项目实战 | `五、项目仓库/步进电机/App_Motor_TI.md` |
| LVGL 图形库 | `I_知识节点/LVGL-布局.md` |

</details>

## 🗂️ 索引入口

| 入口                        | 说明           |
| ------------------------- | ------------ |
| 📌 [嵌入式-索引](索引/嵌入式-索引.md) | 主索引入口,嵌入各子索引 |
| ❓ [Q&A](索引/Q&A.md)        | 问答速查         |

## 📊 数据库视图(Bases)

原生 Bases 功能,提供笔记的表格化视图:

- `STM32索引-基础.base` — STM32 笔记多视图
- `FreeRTOS.base` — RTOS 任务/队列/信号量
- `ESP32.base` — ESP32 笔记汇总
- `通讯协议汇总.base` — 协议对照
- `IC.base` — 芯片驱动库
- `LVGL.base` — 图形库
- `牛马定位器.base` — 项目笔记

## 🔧 使用方式

### 📥 拉取最新

```bash
git pull
```

### 📤 提交更新

```bash
git add .
git commit -m "feat: 简述本次更新"
git push
```

### ⚠️ 注意事项

- 🔒 请勿直接推送到 `main` 分支,建议走 PR
- 📝 日记自动生成,模板见 `IV_基建工具/DailyNotes-template.md`

## 🧰 插件清单

### 🧩 第三方插件

| 功能 | 插件 |
|------|------|
| 🔄 Git 版本同步 | Obsidian Git |
| ☁️ 坚果云备份 | Nutstore Sync |
| 💾 本地备份 | Local Backup |
| 📊 数据查询视图 | Dataview |
| 🎨 代码块美化 | Codeblock Customizer |
| 📕 PDF 增强 | PDF Plus |

## 📜 License

个人学习笔记,仅供自用。



---

<div align="center">

⭐ 如果这个知识库对你有帮助,欢迎 Star

📝 持续更新中 · Maintained with 💚

</div>
