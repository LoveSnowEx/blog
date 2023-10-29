---
title: "[Laravel] 使用 SQLite 在本地進行測試"
slug: "use-sqlite-for-testing"
tags: [
    "laravel",
    "sqlite",
]
categories: [
    "Laravel筆記",
]
keywords: [
    "laravel",
    "sqlite",
    "testing",
    "測試",
]
description:
date: "2023-10-23T16:59:35+08:00"
lastmod: "2023-10-23T16:59:35+08:00"
image:
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## 前言

公司專案原本採用 MySQL 作為資料庫，但組長決定將測試轉換為本地 SQLite 執行，因此我記錄了在這過程中遭遇的問題和解決方法。

## 環境

- Laravel 5.6
- PHP 7.1
- PHPUnit 6.5

## 流程

1. 使用 [kitloong/laravel-migrations-generator](https://packagist.org/packages/kitloong/laravel-migrations-generator) 產生 migration 檔
2. 調整 migration 檔，使其成功在 SQLite 上執行
3. 建立 factory 和 seeder，產生測試資料
4. 在測試時，使用 SQLite 作為資料庫，執行 migration 和 seeder
5. 調整測試程式碼，使其能夠在 SQLite 上執行

## PHP/Laravel 問題集

### 安裝套件後套件沒有生效

原因：

- 套件被 cache 了
- composer 的 memory 不夠

解法：

```bash
php artisan clear-compiled
php artisan cache:clear
COMPOSER_MEMORY_LIMIT=-1 composer install
```

### 找不到 Seeder class

一樣是 cache 的問題

解法：

- 執行 `composer dump-autoload`

## Migration 問題集

名詞解釋：

- pk: primary key，主鍵
- ai: auto increment，自動遞增
- unique: 唯一

資料庫基本規則：

- 只能有一個 primary key
- 不能有相同名稱的 index
- pk 和 unique 都是一種 index
- 在 SQLite，ai 一定是 pk，但 pk 不一定是 ai
- 在 SQLite，對 table 新增欄位是無法指定位置的，只能新增在最後面

### 重複的 index 名稱

解法：

- 改名

> 建議命名方式：`idx_table_column[_column...]`

### 對相同欄位進行 index

解法：

- 只保留一個 index

### 對 id(pk) 進行 index

解法：

- 刪除 index，只保留 pk

### 同時有 id 和非 id 的 pk

因為 id 通常會設為 ai，而 ai 一定是 pk，所以這樣會導致有兩個 pk

解法：

- 刪除非 id 的 pk
- 將 id 設為非 ai

> 這個問題算是無解，從問題的根本上來說，有其它 pk 的 table 就不應該有 id 這個欄位，因為這麼做沒有意義

### SQLite 無法對現有的 table 指定位置新增欄位

解法：

- 建立一張暫存的 table 把資料複製過去，刪除原本的 table 並重新建立，再把資料複製回去，最後刪除暫存的 table

## 測試問題集

以下是在測試時遇到的問題和解法。

### PHPUnit 無法設定 env

因為 config 被 cache 了，所以無法讀取到 env 的設定

解法：

- 執行 `php artisan config:clear` 清除 cache

### `SQLSTATE[HY000] [2002]:Connection refused: from <database>.<table>`

SQLite 不支援 `from <database>.<table>` 的語法，只能用 `from <table>`

解法：

- 將 `from <database>.<table>` 改成 `from <table>`

> 這樣會走到預設的 database，所以要注意是否有調整好設定

### `SQLSTATE[23000]: Integrity constraint violation: 19 NOT NULL constraint failed`

insert 資料時，not null 的欄位沒有給值，或是 create 時沒有預設值

解法：

- 直接給值
- 設定預設值
- 使用 create 方法，自動填入預設值

### `InvalidArgumentException: Unexpected data found. Trailing data`

insert 資料時，欄位的值和欄位的型別不符

解法：

- 檢查欄位的值是否符合欄位的型別
- 檢查時間欄位的值是否使用了 `2006-01-02T15:04:05Z07:00` 的格式

### query 結果的欄位全都變成字串

Laravel 的 SQLite driver 預設會將所有欄位都轉成字串

解法：

- 在 model 加入 casts 屬性，將欄位轉成對應的型別

    ```php
    class Users extends Model
    {
        protected $casts = [
            'name' => 'string',
            'age' => 'integer',
        ];
    }
    ```

### `SQLSTATE[HY000]: General error: 1 unrecognized token: ":"`

SQLite 不支援 `:=` (assignment operator) 的語法，這個語法是 MySQL 的

解法：

- 無解，請使用其它做法替代

### `SQLSTATE[HY000]: General error: 1 RIGHT and FULL OUTER JOINs are not currently supported`

在 Laravel 中，SQLite 不支援 `RIGHT JOIN` 和 `FULL OUTER JOIN` 的語法

解法：

- `RIGHT JOIN`
  - 交換 table 的順序，改用 `LEFT JOIN`

      ```sql
      SELECT * FROM `table_a` RIGHT JOIN `table_b` ON `table_a`.`id` = `table_b`.`id`
      ```

      改成

      ```sql
      SELECT * FROM `table_b` LEFT JOIN `table_a` ON `table_a`.`id` = `table_b`.`id`
      ```

- `FULL OUTER JOIN`
  - 改用 `LEFT JOIN` 和 `UNION` 組合

      ```sql
      SELECT * FROM `table_a` FULL OUTER JOIN `table_b` ON `table_a`.`id` = `table_b`.`id`
      ```

      改成

      ```sql
      SELECT * FROM `table_a` LEFT JOIN `table_b` ON `table_a`.`id` = `table_b`.`id`
      UNION
      SELECT * FROM `table_b` LEFT JOIN `table_a` ON `table_a`.`id` = `table_b`.`id`
      ```
