---
title: "[心得] 部落格建立紀錄"
slug: "blog-build-record"
tags: [
    "hugo",
]
categories: [
    "心得感想",
]
keywords: [
    "hugo",
    "blog",
    "experience",
    "部落格",
    "建立",
    "心得",
]
description:
date: "2023-01-09T13:49:34+08:00"
lastmod: "2023-01-23T21:19:00+08:00"
image:
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## 前言

在這個部落格建立以前，我沒有做筆記的習慣，偶爾想記錄東西時才會寫到自己的 hackmd。  
另外也覺得放在 hackmd 上的東西不會被其他人看見，所以平時很懶得整理內容。

升上大學後，同學們之間會互相分享自己的學習紀錄、技術分享和比賽心得等文章，將這些文章記錄在自己的部落格上，反觀我自己寫在 hackmd 上的東西完全不會讓我想丟出來。  
同時近期為了寫履歷求職，我需要有一個管道介紹自己，基於這些原因，我想藉著這個機會建立一個屬於自己的部落格，除了介紹自己以外，也把自己的所見所聞給記錄下來，以及我也想將自己學到的技術寫成教學供人參考學習。

## 架設工具選擇

在選擇架設工具的時候，主要是以支援 markdown 為主，因為我 HackMD 用習慣了，對 markdown 的編輯比較熟悉，同時也方便我將文章從 HackMD 遷移，未來若需要再遷移也相對方便。

於是我用上述條件搜尋相關資料後有以下三個選擇：

- Jekyll
- Hexo
- Hugo

最後我選擇了 Hugo，原因是因為 Hugo 是目前看起來評價最好的，除了使用方式簡單外，生成網頁的速度也很快，方便我快速架站，另外它還是使用我最喜歡的 Go 語言編寫的！

## 主題

Hugo 預設是沒有主題的，需要自行安裝。  
而由於 Hugo 的主題比較少，所以我挑來挑去只覺得 [Stack](https://github.com/CaiJimmy/hugo-theme-stack) 這個主題比較對味。  
而在尋找 Stack 使用方式的過程中，我發現了由 [Mantyke](https://github.com/Mantyke) 所製作的 [Hugo-stack-theme-mod](https://github.com/Mantyke/Hugo-stack-theme-mod)。

## 架設

### 專案設定

將 repository clone 下來並進入專案根目錄

```shell
git clone git@github.com:Mantyke/Hugo-stack-theme-mod.git ./blog
cd blog
```

生成靜態網頁

```shell
hugo
```

這邊我使用[轉檔網站](https://www.convertsimple.com/convert-yaml-to-toml/)將 config.yaml 轉成 config.toml
並修改第一段如下：

```toml
DefaultContentLanguage = "en"
baseurl = "http://blog.lovesnowex.tk"
disqusShortname = "stack"
hasCJKLanguage = true
languageCode = "en-us"
paginate = 5
theme = "stack"
title = "LoveSnowEx's Blog"
```

### 伺服器設定

我使用 Caddy + Docker 做為伺服器，原因是因為設定方便，並且可以自動簽發 https。  

在專案根目錄建立 Caddyfile

```caddy
blog.lovesnowex.tk

root * /srv

# Templates give static sites some dynamic features
templates

# Compress responses according to Accept-Encoding headers
encode zstd gzip

# Make HTML file extension optional
try_files {path}.html {path}

# Serve everything else from the file system
file_server

log {
    format console
}

handle_errors {
    rewrite * /{http.error.status_code}.html
    file_server
}
```

以及 docker-compose.yaml

```yaml
version: "3.7"

services:
  caddy:
    image: caddy:2.6-alpine
    container_name: blog-caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - $PWD/Caddyfile:/etc/caddy/Caddyfile
      - $PWD/public:/srv
```

最後將 docker 跑起來就大功告成了

```shell
docker compose up --build -d
```

## 後記

現在我有了可以做筆記的地方，未來打算一週至少更新一篇文章，可能是介紹我的 repository，可能是寫教學文，也可能是生活上心得感想，希望是能持之以恆的更新啦哈哈。
