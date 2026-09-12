
#Lua  
#KnowledgeMap

# 🗺️ Lua 编程语言知识地图 (MOC)

> [!info] 核心概念  
> Lua 是一门强大、高效、轻量、可嵌入的脚本语言[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)。它由巴西里约热内卢天主教大学（PUC-Rio）的研究人员于1993年开发[](https://developer.huaweicloud.com/tags/200539/post_1)，以**轻量级**（解释器核心仅几百KB[](https://developer.huaweicloud.com/tags/200539/post_1)）、**高效**（基于寄存器的虚拟机[](https://developer.huaweicloud.com/tags/200539/post_1)）和**可嵌入**（设计为“胶水语言”[](https://developer.huaweicloud.com/tags/200539/post_1)）著称，被广泛应用于游戏、嵌入式系统、Nginx/OpenResty 等领域[](https://developer.huaweicloud.com/tags/200539/post_1)。

## 1. 语言基础 (Basics)

- [[Lua-初识]] (简介与特性)
- [[Lua-注释]] (`--` 单行, `--[[ ]]` 多行)[](https://developer.aliyun.com/article/1618824#1)
- [[Lua-标识符与保留字]] (字母数字下划线组成[](https://developer.aliyun.com/article/1618824#1)，含 `and`, `break`, `do`, `else` 等)[](https://cloud.tencent.com.cn/developer/article/2462940?from=15425#1#1)
- [[Lua-变量]] (动态类型[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)、全局变量与 `local` 局部变量)[](https://developer.aliyun.com/article/1618824#1)
- [[Lua-运算符]] 

## 2. 数据类型 (Data Types)

> Lua 有8种基本类型，所有值都是“一等公民”[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)。
- [[nil]] (表示无效值，在条件中为 false)[](https://developer.aliyun.com/article/1618824#1)[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)
- [[boolean]] (`true` 和 `false`，`nil` 和 `false` 为假，其他为真)[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)
- [[number]] (整数 `integer` 和浮点数 `float`，标准Lua使用64位)[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)
- [[string]] (字符串，支持 `''`, `""`, `[[]]`)[](https://cloud.tencent.com.cn/developer/article/2462940?from=15425#1#1)
- [[table]] (**核心数据结构**，关联数组，索引从1开始)[](https://developer.huaweicloud.com/tags/200539/post_1)
- [[function]] (函数，由C或Lua编写)
- [[thread]] (协程，协同程序的执行体)[](https://developer.aliyun.com/article/1618824#1)
- [[userdata]] (用户自定义数据，用于存储C/C++数据)[](https://developer.aliyun.com/article/1618824#1)

## 3. 控制结构 (Control Structures)

- [[Lua-条件判断]] (`if`, `elseif`, `else`, `end`)[](https://developer.huaweicloud.com/tags/200539/post_1)
- [[Lua-循环]] (`while`, `for`, `repeat ... until`)[](https://developer.huaweicloud.com/tags/200539/post_1)

## 4. 函数 (Function)

- [[Lua-函数定义]] (`function ... end`)[](https://developer.aliyun.com/article/1618824#1)
- [[Lua-函数参数]] (固定参[](https://developer.aliyun.com/article/1618824#1)、可变参 `...`[](https://developer.aliyun.com/article/1618824#1))
- [[Lua-函数返回值]] (可返回多个值)[](https://developer.aliyun.com/article/1618824#1)
- [[Lua-一等公民]] (函数可赋值给变量、作为参数传递、作为返回值)[](https://cloud.baidu.com/article/2968834)[](https://developer.huaweicloud.com/tags/200539/post_1)
- [[Lua-闭包]] (函数与其外部环境的组合)[](https://developer.huaweicloud.com/tags/200539/post_1)

## 5. 高级特性 (Advanced)

- [[Lua-元表 (Metatable)]] (改变表的行为，如 `__add`[](https://my.oschina.net/emacs_9178136/blog/18119152#1)、`__index`[](https://my.oschina.net/emacs_9178136/blog/18119152#1))
- [[Lua-元方法 (Metamethod)]] (与元表配合，实现运算符重载、面向对象等)[](https://my.oschina.net/emacs_9178136/blog/18119152#1)
- [[Lua-协程 (Coroutine)]] (轻量级协作式多任务)[](https://my.oschina.net/emacs_9178136/blog/18119152#1)
- [[Lua-模块 (Module)]] (代码组织与封装，`require`)[](https://my.oschina.net/emacs_9178136/blog/18119152#1)
- [[Lua-垃圾回收 (GC)]] (自动内存管理)[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)[](https://developer.huaweicloud.com/tags/200539/post_1)

## 6. 交互与生态 (Interop & Eco)

- [[Lua-C API]] (嵌入Lua到C/C++程序[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)[](https://developer.huaweicloud.com/tags/200539/post_1)，通过**Lua栈**交换数据[](https://developer.huaweicloud.com/tags/200539/post_1))
- [[Lua-独立程序]] (`lua` 解释器，用于交互式或批量执行)[](https://raw.githubusercontent.com/atom-l/lua5.4-manual-zh/master/README.md#1)
- [[Lua-应用场景]] (游戏脚本、嵌入式系统、Web服务器扩展如OpenResty)