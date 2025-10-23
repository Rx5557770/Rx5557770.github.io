
---
title: Sass
published: 2025-10-23
description: ''
image: ''
tags: ['code', 'js']
category: 代码日常
draft: false
lang: zh-CN
---
    
### 基本命令

- `sass test.scss test.css`: 将sass转换为css
- `sass --watch test.scss:test.css`: 监听test.sass，当文件改变自动生成test.css

### 属性选择
当遇到类似于 `border-color` `border-width`这种带中线的属性时，可以通过嵌套来设置。
属性后面要加冒号。

```scss
font: {  
  family: Helvetica, sans-serif;  
  size: 18px;  
  weight: bold;  
}  
  
text: {  
  align: center;  
  transform: lowercase;  
  overflow: hidden;  
}
```