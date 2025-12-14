# Detailed Design of the Typeahead Suggestion System (預輸入建議系統的詳細設計)
## Detailed design (詳細設計)
讓我們瀏覽一下下圖中組件的流程和互動。我們的設計分為兩個主要部分：

- **A suggestion service (建議服務)**
- **An assembler (組裝器)**

[預輸入建議系統的詳細設計](./design.jpg)

### Suggestion service (建議服務)
當使用者在搜尋框中輸入查詢的同時，getSuggestions(prefix) API 調用會到達建議服務。前十個熱門查詢從分散式快取 Redis 返回。

### Assembler (組裝器)
在上一課中，我們討論了如何構建、分區和存儲 trie 在資料庫中。但是，trie 的創建和更新不應出現在使用者查詢的關鍵路徑中。我們不應該實時更新 trie，原因如下：

- 每秒可能有數百萬使用者輸入查詢。在這種傳入流量巨大的階段，對每個查詢實時更新 trie 會減慢我們的建議服務。
- 我們必須提供在創建或更新 trie 後可能不會頻繁更改的熱門建議。因此，頻繁更新 trie 就不那麼重要了。
鑑於上述原因，我們有一個單獨的服務稱為 assembler (組裝器)，負責在一定的可配置時間後創建和更新 trie。組裝器由以下不同的服務組成：

- **Collection service (收集服務)**：每當使用者輸入時，此服務都會收集包含短語、時間和其他元資料的日誌，並將其轉儲到稍後處理的資料庫中。由於這些資料量巨大，Hadoop Distributed File System (HDFS) 被認為是存儲這些原始資料的合適存儲系統。

               Raw Data Collected by the Collection Service (收集服務收集的原始資料)
Phrases (短語)     Date and Time (DD-MM-YYYY  HH:MM:SS) (日期和時間)

UNIVERSAL          23-03-2022 10:16:18

UNIVERSITY         23-03-2022 10:20:11

UNIQUE             23-03-2022 10:21:10

UNIQUE             23-03-2022 10:22:24

UNIVERSITY         23-03-2022 10:25:09
下表顯示了來自收集服務的原始資料示例。我們記錄時間，以便系統知道何時更新特定短語的頻率。

- **Aggregator (聚合器)**：收集服務收集的原始資料通常不是合併的形式。我們需要合併原始資料以進一步處理它並創建或更新 trie。聚合器從 HDFS 檢索資料並將其分發給不同的工作者。通常，MapReducer 負責在給定的時間間隔內聚合前綴的頻率，並定期在關聯的 Cassandra 資料庫中更新頻率。Cassandra 適合此目的，因為它可以以表格格式存儲大量資料。

下表顯示了特定時期內處理和合併的資料。此表由聚合器定期更新，並存儲在像 Cassandra 這樣的資料庫中的雜湊表中。為簡單起見，我們假設我們的資料不區分大小寫。

   Useful Information Extracted from the Raw Data (從原始資料中提取的有用資訊)
Phrases (短語)        Frequency (頻率)    Time Interval (時間間隔)

UNIVERSAL             1400                1st 15 minutes (前 15 分鐘)

UNIVERSITY            1340                1st 15 minutes (前 15 分鐘)

UNIQUE                1200                1st 15 minutes (前 15 分鐘)

```

- **Trie builder (Trie 構建器)**：此服務負責創建或更新 trie。它通過 ZooKeeper 將這些新的和更新的 trie 存儲在其 trie 資料庫中的相應分片上。Trie 存儲在檔案中的持久存儲中，以便在必要時我們可以輕鬆重建我們的 trie。MongoDB 等 NoSQL 文件資料庫適合存儲這些 trie。當機器重新啟動時，需要 trie 的這種存儲。

Trie 從 Cassandra 資料庫中的聚合資料更新。trie 的現有快照將使用所有新術語及其相應頻率進行更新。否則，將使用 Cassandra 資料庫中的資料創建一個新的 trie。

一旦創建或更新了 trie，系統就會將其提供給建議服務。

```
Question
我們是應該為每個使用者收集資料並構建一個 trie，還是應該在所有使用者之間共享它？

Answer
由於我們的目標是設計一個規模類似於 Google 搜尋的系統，數十億使用者將使用我們的服務。由於使用者數量巨大，為每個使用者維護一棵樹將變得複雜且耗時。如果有多個使用者輸入了一些常見搜尋，也可能會出現重複的樹，從而導致資源浪費。

因此，我們的設計假設一個在使用者之間共享的通用 trie，其中排名將基於 trie 中的單個短語和術語的頻率。
```
