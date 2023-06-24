---
title: "Go 教學 - 常數和 iota"
slug: "const-and-iota"
tags: [
    "Go",
]
categories: [
    "Go 教學",
]
description: 
date: "2023-02-15T17:36:09+08:00"
lastmod: "2023-02-15T17:36:09+08:00"
image: 
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

const 在 Go 中是常數的意思，無法被修改，在編譯時期值就定義完成。

## 1. 常數宣告 Constants Declaration

const 跟 var 的寫法類似：

```go
const PI float64 = 3.14
const MaxMonth int = 12
const Greeting string = "Hello"
```

省略型別：

```go
const PI = 3.14
const MaxMonth = 12
const Greeting = "Hello"
```

群組宣告：

```go
const (
    PI = 3.14
    MaxMonth = 12
    Greeting = "Hello"
)
```

> Go 的常數只可以宣告基本型別，也就是 bool, int, float32 和 string 等，  而 slice, map, pointer, struct 等都是不行的。
> 另外，const 不能使用 `:=` 進行宣告。

## 2. iota

宣告常數時可以搭配 `iota` 使用，`iota` 表示在同一個 const group 內的行索引數，(也就是小括號內的第幾行)，每遇到新的 `const` 字詞，itoa 就會歸零。

```go
type Month int

const (
    January Month = 1 + iota  // 1
    February                  // 2
    March                     // 3
    April                     // 4
    May                       // 5
    June                      // 6
    July                      // 7
    August                    // 8
    September                 // 9
    October                   // 10
    November                  // 11
    December                  // 12
)

type Weekday int

const (
    Sunday Weekday = iota  // 0
    Monday                 // 1
    Tuesday                // 2
    Wednesday              // 3
    Thursday               // 4
    Friday                 // 5
    Saturday               // 6
)

const a, b, c = iota, iota, iota  // 0, 0, 0

const d = iota // 0
const e = iota // 0
const f = iota // 0
```

進階用法，搭配 shift 運算：

```go
const (
    _  = iota             // 0
    KB = 1 << (10 * iota) // 2 ^ 10
    MB                    // 2 ^ 20
    GB                    // 2 ^ 30
)
```
