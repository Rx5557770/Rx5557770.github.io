
---
title: lua
published: 2025-10-23
description: lua制作你的脚本集！
image: ''
tags: ['code', 'lua']
category: 代码日常
draft: false
lang: zh-CN
---
    #### 注释

```lua
print("这段代码会运行")
--print("我被注释掉了，所以不会运行")
--[[
    我是多行注释
    不管我写多少行
    都不会影响代码运行
]]
```

#### 赋值

```lua
a, b, c = 0, 1
print(a,b,c)
--输出0   1   nil

a, b = a+1, b+1, b+2
print(a,b)
--输出1   2

a, b, c = 0
print(a,b,c)
--输出0   nil   nil
```