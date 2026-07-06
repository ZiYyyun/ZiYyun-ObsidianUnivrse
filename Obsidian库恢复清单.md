# Obsidian库恢复清单

> 生成时间：2026-07-05
> 背景：重装系统后从GitHub克隆恢复，插件和PDF丢失
> 原因：.gitignore 错误过滤了 `.obsidian/` 和 `*.pdf`

---

## 一、第三方插件清单

### ✅ 已确认使用（笔记中有明确语法痕迹）

| 插件名称 | 插件ID | 用途 | 确认依据 |
|---------|--------|------|---------|
| Obsidian Git | obsidian-git | GitHub备份同步 | 已安装，community-plugins.json中有记录 |
| Dataview | dataview | 数据查询视图 | 19个文件使用```dataview代码块，含table/list视图 |
| Templater | templater-obsidian | 高级模板脚本 | 日记模板使用<%* %>脚本语法，含moment日期处理 |
| Tasks | obsidian-tasks-plugin | 任务管理查询 | HomePage模板使用```tasks代码块查询待办 |

### ⚠️ 可能安装过但未大量使用（近期对话提及）

| 插件名称 | 用途 | 状态 |
|---------|------|------|
| Codeblock Customizer | 代码块注释标注、美化 | 笔记中未找到// [!语法，可能刚装未用 |
| Ace Code Editor | 代码文件嵌入高亮 | 笔记中未找到![[xxx.c]]嵌入，可能刚装未用 |
| Better Code Block | 代码块美化 | 同上，可能研究中未实际使用 |

### 📌 核心插件（已启用）

从 core-plugins.json 确认：
- daily-notes（日记）
- templates（模板）
- bases（原生数据库视图，对应 .base 文件）
- canvas（画布）
- properties（属性）
- file-recovery（文件恢复）
- sync（同步）
- graph（关系图谱）
- backlink（反向链接）
- outline（大纲）
- 等等...

---

## 二、缺失的PDF文件清单

### 🔧 芯片/模块Datasheet

| 文件名 | 描述 | 关联笔记 | 重要程度 |
|--------|------|---------|---------|
| TI-BQ769x0_DataSheet.pdf | TI BQ769x0系列电池管理芯片数据手册 | IC-TI_BQ769x0.md、IC-TI_BQ76920驱动编写.md | ⭐⭐⭐⭐⭐ |
| RM0008中文参考手册.pdf | STM32F10x系列中文参考手册 | STM32-ADC.md、STM32-RTC.md、STM32-低功耗.md等 | ⭐⭐⭐⭐⭐ |
| QS-100模块AT命令手册_V1.0.pdf | QS-100 NB-IoT模块AT命令手册 | IC-QS100-NB-IoT.md | ⭐⭐⭐⭐ |
| DS3553-编写手册.pdf | DS3553芯片编写手册 | 全局宏定义配置--config.h.md | ⭐⭐⭐ |

### 📐 原理图/开发板资料

| 文件名 | 描述 | 关联笔记 | 重要程度 |
|--------|------|---------|---------|
| SCH_ZET6开发板_1_2024-07-01.pdf | STM32F103ZET6开发板原理图 | STM32-ADC.md、STM32-SPI软件模拟实现.md、外设-LCD.md等 | ⭐⭐⭐⭐⭐ |
| SCH_直流步进电机控制板_2025-04-14.pdf | 直流步进电机控制板原理图 | 步进电机/编码器.md | ⭐⭐⭐ |

---

## 三、图片附件情况

### ✅ 已保留的图片（24张）

存放在 `附件/` 目录下，主要是：
- STM32外设寄存器截图（EXTI、SysTick、ADC、RTC等）
- 通信协议示意图（IIC、串口、1-Wire等）
- 开发板原理图截图
- 其他教学配图

> 原因：.gitignore 只忽略了 `Attachments/` 目录，而你的图片存在 `附件/` 目录下，所以图片都保住了

---

## 四、.base 数据库文件

### ✅ 已保留的数据库（4个）

这些是 Obsidian 原生 Bases 功能的数据库文件，不是第三方插件：

| 文件名 | 用途 |
|--------|------|
| ESP32.base | ESP32标签笔记的表格视图 |
| FreeRTOS.base | FreeRTOS标签笔记的表格视图 |
| STM32索引-基础.base | STM32相关笔记的多视图数据库 |
| 牛马定位器.base | 牛马定位器项目的数据库 |
| 通讯协议汇总.base | 通讯协议相关笔记的数据库 |

> 说明：Bases 是 Obsidian 1.7+ 版本的原生功能，对应 core-plugins.json 中的 `"bases": true`

---

## 五、恢复建议

### 1. 修正 .gitignore（必做）

当前 .gitignore 错误过滤了：
```
.obsidian/      # 导致插件和配置全丢
*.pdf           # 导致所有PDF丢失
Attachments/    # 附件目录过滤
```

**替换为安全配置：**
```gitignore
# 仅忽略缓存与设备专属布局
.obsidian/cache
.obsidian/workspace.json
.obsidian/workspace-mobile.json

# 系统垃圾文件
.DS_Store
Thumbs.db

# 可选：Obsidian本地回收站
.trash/
```

### 2. 插件恢复优先级

1. **第一优先级（核心工作流）**：obsidian-git、dataview、templater
2. **第二优先级（辅助功能）**：obsidian-tasks-plugin
3. **第三优先级（按需安装）**：Codeblock Customizer、Ace Code Editor、Better Code Block

### 3. PDF恢复建议

按重要程度从高到低重新下载：
1. RM0008中文参考手册.pdf - ST官方网站或正点原子/野火资料站
2. SCH_ZET6开发板原理图.pdf - 对应开发板商家资料
3. TI-BQ769x0_DataSheet.pdf - TI官网
4. QS-100模块AT命令手册.pdf - 模块厂商官网
5. 其他原理图和手册

---

## 六、目录结构总览

```
obsidian-/
├── .obsidian/          # 配置目录（之前被gitignore过滤了）
│   └── plugins/        # 插件目录（目前只有obsidian-git）
├── I_知识节点/         # 理论知识笔记
├── II_代码实操/        # 实践代码笔记
├── III_资源仓库/       # 资源汇总
│   └── 参考手册/       # 参考资料图片
├── IV_基建工具/        # 模板、工具类笔记
├── 五、项目仓库/        # 项目笔记
├── Daily/              # 日记
├── 附件/               # 图片附件（已保留）
├── 归档/               # 归档笔记
├── 索引/               # 索引导航
├── *.base              # 数据库文件（已保留）
└── .gitignore          # 需要修正
```
