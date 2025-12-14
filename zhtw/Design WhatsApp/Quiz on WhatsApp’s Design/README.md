# Quiz on WhatsApp’s Design (WhatsApp 設計的測驗)


```
1
WebSocket 管理器負責哪個功能？

A)
將資料路由給使用者

B)
維護使用者與其 WebSocket 處理程序之間的映射

C)
將使用者重新連接到 blob 儲存

D)
以上皆非
```

```
2
如果訊息發送給離線使用者會發生什麼？

A)
訊息被丟棄，並向發送者發送錯誤回應。

B)
訊息被轉發給另一個使用者。

C)
訊息暫時存儲在資料庫中，除非使用者上線。

D)
以上皆非
```

```
3
為什麼 WhatsApp 不能使用 Mnesia 資料庫來儲存媒體文件而不是使用 blob 儲存？

A)
Mnesia 提供有限的容量。

B)
Blob 存儲經過優化以存儲和檢索大文件。

C)
Mnesia 比 blob 儲存更昂貴。

D)
以上皆非
```

```
4
(填空。) 如果媒體文件被分享和訪問多次，它將從 blob 儲存存儲在 ________ 中。

A)
Kafka

B)
Mnesia 資料庫

C)
a CDN

D)
MySQL 資料庫
```

```
5
(填空。) WebSocket 伺服器使用快取來 ________。

A)
臨時存儲訊息

B)
永久存儲訊息

C)
存儲使用者及其對應的 WebSocket 處理程序的映射

D)
存儲每個使用者的元資料
```

Answers (答案)

1. (B)
2. (C)
3. (B)
4. (C)
5. (C)
