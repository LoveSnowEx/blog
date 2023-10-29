---
title: "1 - Web 基本概念-API"
slug: "web-essential-api"
tags: [
    "web",
]
categories: [
    "網站教學"
]
keywords: [
    "web",
    "archtecture",
    "frontend",
    "backend",
    "api",
    "網站",
    "架構",
    "前端“,
    ”後端",
]
description:
date: "2023-01-18T23:02:00+08:00"
lastmod: "2023-01-18T23:02:00+08:00"
image:
license: "CC BY-NC-ND"
hidden: false
comments: true
draft: false
---

## API

為了讓前後端可以溝通，必須透過訂定 API (Application Programming Interface)，使前後端有一致的共識進行溝通。

以下是 3 種常見的 API 架構：

| Archtecture | 難易度 | 耦合性 | 社群 | 訊息格式 |
| :-: | :-: | :-: | :-: | :-: |
| REST    | 一般 | 低 | 　大　 | XML, JSON, HTML, plain text |
| GraphQL | 較難 | 中 | 成長中 | JSON |
| RPC     | 一般 | 高 | 　大　 | JSON, XML, Protobuf, Thrift, FlatBuffers |

![不同 API 架構所使用的溝通方式](https://i.imgur.com/YH73dZF.png)

### 1. REST

REST (REpresentational State Transfer) 其實是一種設計風格而不是標準。REST 通常使用 HTTP、URI、XML、JSON 以及 HTML 這些現有的廣泛流行的協定和標準。而符合 REST 設計風格的 Web API 稱為 RESTful，是當今最主流的設計，目前大部分的網站都是使用此設計。

在 RESTful 架構中，瀏覽器使用 GET、POST、PUT 和 DELETE 四種請求方式分別對指定的 URL 資源進行增刪查改操作。

透過 URI 實現對資源的管理及訪問，RESTful 具有擴展性強、結構清晰的特點。

### 2. GraphQL

GraphQL 是一種資料查詢操作語言。相較於 REST 以及其他 Web Service架構，GraphQL 提供了一種更加高效、強大和靈活的開發 Web API 的方式

GraphQL 使用 JSON 進行 query 及 mutation 兩種操作，由客戶端根據所需定義資料結構，同時由伺服器端負責返回相同資料結構的對應資料的方式避免了伺服器端大量冗餘資料的返回，但與此同時也意味著這種方式不能有效利用起查詢結果的 Web 快取。

### 3. RPC

RPC 是一種電腦通訊協定。該協定允許執行於一台電腦的程式呼叫另一個位址空間（通常為一個開放網路的一台電腦）的子程式，而程式設計師就像呼叫本地程式一樣，無需額外地為這個互動作用撰寫程式（無需關注細節）。

> 筆者對此架構較不熟悉，只知道使用上真的是如同直接呼叫函式一般。
>
> 有興趣的人可以搜尋 [gRPC](https://zh.wikipedia.org/zh-tw/GRPC) 的相關資訊，是目前較新的 RPC 技術。  
> gRPC 是由 Google 開發的開源 RPC 框架，支援雙向串流，配合 HTTP/2 具有低延遲的特性，使用 Binary 傳輸資料，（反）序列化速度快，壓縮效率高。

### 4. 其他

除了上述三種 API 外，另外還有一種叫做 SOAP，使用 XML 格式，但由於學習難度較為困難且速度較慢，不推薦使用，本文不對其進行介紹。

> XML 格式有較多冗餘資料，在序列化及反序列化也較為費時，且建立時較為複雜，需仰賴語言是否相對應的處理工具。

## 結語

本章節帶各位認識了幾種 API，讓大家知道前後端都是透過什麼進行溝通的。

每個 API 各有各自的特性，在開發時選擇其中一個使用就可以了。
根據需求，不同的 API 其實是可以在一個專案同時使用的，也不一定要拘泥於使用單種 API。
另外 API 不限於前後端的溝通，有些資料庫也是透過 API 進行溝通的，API 只是一種構通方式，如何運用取決於程式設計師。

下一章將介紹**資料庫**，讓大家知道資料是如何儲存於伺服器上的。

## 參考資料

- [Comparing API Architectural Styles: SOAP vs REST vs GraphQL vs RPC](https://www.altexsoft.com/blog/soap-vs-rest-vs-graphql-vs-rpc)
- [APIs REST, GraphQL or gRPC – Who wins this game?](https://www.sensedia.com/post/apis-rest-graphql-or-grpc-who-wins-this-game)
