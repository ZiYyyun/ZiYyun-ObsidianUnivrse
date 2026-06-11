#日常/日记 


<%* // 自动获取当前日记的日期（支持补写旧日记）
const noteDate = moment(tp.file.title, 'YYYY-MM-DD');
const prevDate = noteDate.clone().subtract(1, 'days').format('YYYY-MM-DD');
const nextDate = noteDate.clone().add(1, 'days').format('YYYY-MM-DD');
const tomorrowDate = noteDate.clone().add(1, 'days').format('MM月DD日');
_%>

# <% noteDate.format('YYYY年MM月DD日') %>（<% noteDate.format('dddd') %>）

## 🔗 日期导航
| << [[<% prevDate %>|前一天]] | **今日** | [[<% nextDate %>|后一天]] >> |

---

## 📝 今日待办任务
> 🎯 优先完成这3件最重要的事（MIT）
1. [ ] <% tp.system.prompt("今日首要任务", "") %>
2. [ ] 
3. [ ] 

---
### 其他待办

- [ ] 
- [ ] 
- [ ] 
- [ ] 

---

## 📚 今日学习内容
| 学习主题 | 来源/关联笔记 | 耗时 | 核心收获 |
| -------- | ------------ | ---- | -------- |
| {{cursor}} | [[ ]] |  |  |
|  |  |  |  |
|  |  |  |  |

> 补充说明：可在「来源/关联笔记」中链接到你的课程笔记、书籍笔记或文章链接

---

## 📄 今日创建的笔记
> 自动列出你在这一天创建的所有笔记，无需手动记录！

```dataview
LIST
WHERE file.ctime >= date("<% noteDate.format('YYYY-MM-DD') %>") 
  AND file.ctime < date("<% nextDate %>")
  AND !contains(file.folder, "_templates") // 排除模板文件夹
SORT file.ctime DESC
```
