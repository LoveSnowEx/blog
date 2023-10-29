---
title: "[Go 教學] 0. 第一支程式 \"Hello World!\""
slug: "hello-world"
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
    "hello world",
]
description:
date: "2023-01-23T13:15:04+08:00"
lastmod: "2023-01-23T13:15:04+08:00"
image:
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## 建立第一支程式

在專案根目錄建立一個名為 `hello.go` 的檔案。

這邊有幾個知識點補充：

1. 所謂的專案 (Project)，指的是一個產品或者服務，為了達成一個目標而建立的項目。在本教學中，我們的練習用的也是一個專案。
2. 專案根目錄 (Project Root Directory)，就是我們存放專案的路徑位置。
3. .go 是 Golang 程式使用的副檔名。

```tree
.
└── hello.go
```

使用文字編輯器，對 `hello.go` 進行編輯。

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

## 執行

使用 `go run <檔案>` 即可執行程式。

```shell
go run hello.go
```

輸出：

```text
Hello, World!
```

這樣就代表你成功啦！

## 補充說明

1. 所有的 go 檔都必須標示 package，如第 1 行的 `package main`。
2. `Println(...)` 是屬於 `fmt` (全名 format) 這個 package 底下的函式，需要先 import 才可使用 (第 3 行)。
3. 使用外部 package 的 函式必須以 `<package>.<function>` 的方式使用，如 fmt.Println()。
4. `func main()` 是 golang 的 main function 形式，也是程式的入口點 (entry point)。
5. `go run` 的目標必須是 main package，且有 main function。
