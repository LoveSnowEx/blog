---
title: "[Go 教學] 1. 變數與零值"
slug: "variable-and-zero-value"
tags: [
    "golang",
]
categories: [
    "Go教學",
]
keywords: [
    "go",
    "golang",
    "tutorial",
    "variable",
    "zero Value",
    "變數",
    "零值",
]
description:
date: "2023-02-02T17:32:53+08:00"
lastmod: "2023-02-04T03:59:59+08:00"
image:
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## 變數宣告 Variable Declaration

使用 var 關鍵字進行宣告，型別需寫在變數名稱後方。

```go
var a int
var b bool
var str string
```

你也可以這麼寫：

```go
var (
    a int
    b bool
    str string
)
```

多變數宣告：

```go
var a, b, c int
```

## 初始化變數 Variable Declaration With Initialization

變數可以在編譯時期內被賦值，使用等號 `=` 進行初始化宣告。

```go
var a int = 3
var b bool = false
var str string = "Hello World!"
```

Go 的編譯器可以根據值來自動推導型別，故型別可省略。

```go
var a = 3
var b = false
var str = "Hello World!"
```

多變數 + 自動型別推導宣告：

```go
var a, b, str = 3, false, "Hello World!"
```

## 短變數宣告 Short Variable Declaration (最常用)

冒號等於 `:=` 可以進行變數宣告，並且自動推導型別。  
要注意的一點是，短變數宣告只可於函數內使用，因為函數外的每個語法都必須以關鍵字做為開頭。

```go
a := 3
b := false
str := "Hello World!"
```

多變數 + 短變數宣告：

```go
a, b, str := 3, false, "Hello World!"
```

> 原則上若無特別需求，盡量不要進行不同型別的多變數宣告，這會使程式的可讀性變差。

## 零值 Zero Value

變數宣告時若無指定初始值，則會被賦予該型別的零值。

以下是所有基本型別的零值：

| Description | Type | Value |
| ---------------------- | - | - |
| Integer numbers        | int, int8,  int16, int32, int64, uint, uint8, uint16, uint32, uint64, uintptr, byte(alias for uint8), rune(alias for int32) | 0 |
| Floating point numbers | float32, float64 | 0.0 |
| Complex numbers        | complex64, complex128 | 0+0i |
| Booleans               | bool | false |
| Strings                | string | "" |
| Others                 | interface, slice, map, pointer, func, chan | nil |

之後我會再額外對型別進行深度的講解，目前只需要知道未被手動初始化的變數必為零值即可。  
也因為這個特性，讓 Go 的使用上方便了許多。
