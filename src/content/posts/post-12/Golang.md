
---
title: Golang
published: 2025-10-23
description: Golang语言据说和C速度差不多
image: ''
tags: ['code', 'Golang']
category: 代码日常
draft: false
lang: zh-CN
---
    #### for循环

`index`为索引，`value`为值

```go
func main() {
   sequence := "hello world"
   for index, value := range sequence {
      fmt.Println(index, value)
   }
}
```

#### 数组与切片

> 定义数组的长度时必须为常量表达式，即length必须为常量

>对于切片而言，`make`函数接收三个参数：类型，长度，容量。
```go
// 数组
length := 5 // 这是一个变量
var nums [length]int  // 无法通过编译

// 切片
var nums []int // 值
nums := []int{1, 2, 3} // 值
nums := make([]int, 0, 0) // 值
nums := new([]int) // 指针

```

#### 映射表

> map是无序的

```go
mp := map[int]string{
   0: "a",
   1: "a",
   2: "a",
   3: "a",
   4: "a",
}

mp := map[string]int{
   "a": 0,
   "b": 22,
   "c": 33,
}

```

#### 指针

> `new`函数只有一个参数那就是类型，并返回一个对应类型的指针，函数会为该指针分配内存，并且指针指向对应类型的零值

```go
   numPtr := new(int)  // 分配一个int类型的内存空间
   fmt.Println(numPtr) // 输出内存地址
   
   fmt.Println(*new(string))
   fmt.Println(*new(int)) // 0
   fmt.Println(*new([5]int)) // [0,0,0,0,0]
   fmt.Println(*new([]float64)) // []
```

#### 结构体

> **结构体**是一种聚合数据类型，用于将多个不同类型的字段组合成一个单一的复合类型。

```go
// 定义结构体
type Programmer struct {
	Name     string
	Age      int
	Job      string
	Language []string
}
// 使用结构体
programmer := Programmer{
   Name:     "jack",
   Age:      19,
   Job:      "coder",
   Language: []string{"Go", "C++"},
}
```