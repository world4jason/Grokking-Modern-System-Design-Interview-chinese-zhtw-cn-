# Memcached versus Redis (Memcached 與 Redis)
## Introduction (介紹)
本課程將討論分佈式快取的一些廣泛採用的現實世界實現。我們的重點是兩個著名的開源框架：Memcached 和 Redis。它們是高度可擴展、高性能和健壯的快取工具。這兩種技術都遵循客戶端-伺服器模型並實現亞毫秒級的延遲。讓我們討論它們中的每一個，然後比較它們的有用性。

Memcached 於 2003 年推出。它是一個鍵值存儲並發布式快取，旨在非常快速地存儲對象。Memcached 以鍵值對的形式存儲資料。鍵和值都是字符串。這意味著任何存儲的資料都必須進行序列化。因此，Memcached 不支持也無法操作不同的資料結構。

```
序列化是將資料轉換為可以傳輸或存儲在其他地方的格式的過程。稍後，應該可以重建或反序列化資料。
```

Memcached 有一個客戶端和伺服器組件，每個組件都是運行系統所必需的。系統設計方式是一半的邏輯包含在伺服器中，而另一半在客戶端中。但是，每個伺服器都遵循無共享架構。在這種架構中，伺服器彼此不知道，並且伺服器之間沒有同步、資料共享和通訊。

由於斷開連接的設計，Memcached 能夠使用高端系統每秒為數百萬個鍵提供幾乎確定性的查詢速度 (O(1))。因此，Memcached 提供高吞吐量和低延遲。
 
[典型 Memcached 集群的設計](./memcached.jpg)
 
從典型 Memcached 集群的設計中可以看出，Memcached 在水平方向上擴展得很好。客戶端進程通常與也與權威存儲 (後端資料庫) 交互的服務主機一起維護。

 
### Facebook and Memcached (Facebook 和 Memcached)
Facebook 中的資料訪問模式需要頻繁讀取和更新，因為視圖是動態呈現給使用者的，而不是提前生成的。因為 Memcached 很簡單，所以它是解決方案的簡單選擇，因為 Memcached 於 2003 年開始開發，而 Facebook 於 2004 年開發。事實上，在某些情況下，Facebook 和 Memcached 團隊一起尋找解決方案。

```
Redis 於 2009 年開發。因此，當時 Facebook 不可能使用 Redis。
Memcached 的一些簡單命令如下：

```
get <key_1> <key_2> <key_3> ...
set <key> <value> ...
delete <key>[<time>] ...
在 Facebook，Memcached 位於 MySQL 資料庫和 Web 層之間，使用分布在 800 多台伺服器上的大約 28 TB RAM (截至 2013 年，Facebook 和 Memcached - Tech Talk)。通過近似最近最少使用 (LRU) 逐出策略，Facebook 能夠實現 95% 的快取命中率。

下圖顯示了 Facebook 快取架構的高層設計。正如我們所看到的，在 Web 層發出的總共 5000 萬個請求中，只有 250 萬個請求到達持久層。

[Facebook 使用位於持久層和 Web 層之間的 Memcached 層](./fb.jpg)


Redis 是一個資料結構存儲，可用作快取、資料庫和消息代理。它以額外的複雜性為代價提供豐富的功能。它具有以下特點：

- **Data structure store (資料結構存儲)**：Redis 了解它存儲的不同資料結構。我們不必從中檢索資料結構，操作它們，然後將它們存儲回來。我們可以進行內部更改，這樣既省時又省力。
- **Database (資料庫)**：它可以將所有內存中的 blob 持久化到二級存儲上。
- **Message broker (消息代理)**：異步通訊是分佈式系統中的重要需求。Redis 可以將每秒數百萬條消息從系統中的一個組件轉換到另一個組件。

Redis 提供內置複製機制、自動故障轉移和不同級別的持久性。除此之外，Redis 理解 Memcached 協議，因此，使用 Memcached 的解決方案可以轉換為 Redis。Redis 一個特別好的方面是它將資料訪問與集群管理分開。它解耦了資料並控制平面。這導致可靠性和性能提高。最後，由於使用異步複製，Redis 不提供強一致性。

[使用冗餘輔助副本支持自動故障轉移的 Redis 結構](./redis_replicas.jpg)

### Redis cluster (Redis 集群)
Redis 具有提供高可用性的內置集群支持。這被稱為 Redis Sentinel。一個集群有一個或多個使用多線程代理查詢的 Redis 資料庫。Redis 集群執行自動分片，其中每個分片都有主節點和輔助節點。但是，資料庫或節點中的分片數量可配置以滿足應用程式的期望和需求。

每個 Redis 集群都由集群管理器維護，其工作是檢測故障並執行自動故障轉移。管理層由監控和配置軟體組件組成。

[Redis 集群架構](./redis_clusters.jpg)

### Pipelining in Redis (Redis 中的管道化)
由於 Redis 使用客戶端-伺服器模型，因此每個請求都會阻塞客戶端，直到伺服器收到結果。希望發送後續請求的 Redis 客戶端將不得不等待伺服器響應第一個請求。因此，整體延遲會更高。

Redis 使用管道化來加速該過程。管道化是將客戶端的多個請求組合在一起而不等待伺服器響應的過程。結果，它減少了多個請求的 RTT 跨度。

```
往返時間 (RTT) 是請求從客戶端傳輸到伺服器並返回的延遲。
```

[沒有管道化的 Redis 客戶端-伺服器通訊與有管道化的 Redis 客戶端-伺服器通訊](./redis_communication.jpg)

管道化過程通過 RTT 和進行套接字級 I/O 的時間減少了延遲。此外，操作系統中通過系統調用進行的模式切換是一項昂貴的操作，通過管道化顯著減少。從客戶端管道化命令對伺服器如何處理這些請求沒有影響。

例如，客戶端管道化的兩個請求到達伺服器，伺服器無法處理第二個。伺服器為第一個提供結果並為第二個返回錯誤。客戶端獨立地將類似命令批處理在一起以實現最大吞吐量。

```
注意：如果客戶端和伺服器都在同一台機器上，管道化將延遲至少提高五倍。請求發送到環回地址 (127.0.0.1)。管道化的真正力量在請求發送到遠處機器的系統中凸顯出來。
```

## Memcached versus Redis (Memcached 與 Redis)
儘管 Memcached 和 Redis 都屬於 NoSQL 家族，但有一些微妙的方面使它們與眾不同：

- **Simplicity (簡單性)**：Memcached 很簡單，但它將管理集群的大部分工作留給了集群的開發人員。然而，這意味著使用 Memcached 進行更精細的控制。另一方面，Redis 自動化了大部分擴展和資料劃分任務。
- **Persistence (持久性)**：Redis 通過僅追加文件 (AOF) 和 Redis 資料庫 (RDB) 快照等屬性提供持久性。Memcached 中沒有持久性支持。但是可以通過使用第三方工具來滿足此限制。例如，libmemcached-tools
- **Data types (資料類型)**：Memcached 存儲對象，而 Redis 支持字符串、排序集、哈希映射、位圖和超日誌。但是，最大鍵或值大小是可配置的。
- **Memory usage (內存使用)**：這兩種工具都允許我們設置快取的最大內存大小。Memcached 使用板分配方法來減少碎片。但是，當我們更新現有條目的大小或存儲許多小對象時，可能會浪費內存。儘管如此，有一些配置解決方法可以解決這些問題。
- **Multithreading (多線程)**：Redis 作為單個進程使用一個核心運行，而 Memcached 可以使用多線程技術有效地使用多核系統。我們可以爭辯說，Redis 被設計為單線程進程，以降低多線程系統的複雜性。儘管如此，可以執行多個 Redis 進程以實現並發。同時，Redis 多年來通過調整其性能得到了改進。因此，Redis 可以有效地存儲小資料項。Memcached 可能是文件大小超過 100 K 的正確選擇。
- **Replication (複製)**：如前所述，Redis 通過少量命令自動化複製過程，而 Memcached 中的複製再次取決於第三方工具的使用。在架構上，Memcached 由於其簡單性而在水平方向上擴展良好。Redis 通過相當複雜的集群提供可擴展性。

下表總結了 Memcached 和 Redis 之間的一些主要區別和共同特徵：

```
                       Memcached 和 Redis 提供的功能
Feature (特徵)                         Memcached                           Redis

Low latency (低延遲)                   Yes                                 Yes

Persistence (持久性)                   Possible via third-party tools      Multiple options
                                     可以通過第三方工具                    多種選擇

Multilanguage support (多語言支持)      Yes                                 Yes

Data sharding (資料分片)               Possible via third-party tools      Built-in solution
                                     可以通過第三方工具                    內置解決方案

Ease of use (易用性)                   Yes                                 Yes

Multithreading support (多線程支持)     Yes                                 No

Support for data structure (資料結構支持) Objects                           Multiple data structures
                                       對象                                多種資料結構

Support for transaction (事務支持)      No                                  Yes

Eviction policy (逐出策略)             LRU                                 Multiple algorithms
                                                                         多種演算法

Lua scripting support (Lua 腳本支持)   No                                  Yes

Geospatial support (地理空間支持)       No                                  Yes
```

總而言之，Memcached 是較小、更簡單的讀取密集型系統的首選，而 Redis 適用於複雜且讀寫密集型的系統。

```
Question 1
問題 1
根據實現細節，這兩個框架 (Memcached 或 Redis) 中的哪一個與我們在上一課中設計的分佈式快取有驚人的相似之處？

Answer
答案
答案是 Memcached。原因如下：

- 客戶端軟體使用哈希演算法選擇要使用的快取伺服器。
- 伺服器軟體使用內部哈希表存儲每個鍵的值。
- 最近最少使用 (LRU) 用作逐出策略。
- 不同快取伺服器之間沒有通訊。
```

```
Question 2
問題 2
為什麼存在用於持久化 Memcached 資料的第三方工具？

Answer
答案
這是因為大量資料被讀取和寫入快取伺服器，並且無論出於何種原因，它們有時可能會崩潰。重新啟動後，在特定場景下從頭開始構建快取可能需要數小時，這最終會降低系統性能。因此，可以將快取資料持久化到磁盤以便在重啟時加載。
```

```
Question 3
問題 3
與僅存儲字符串相比，存儲不同資料結構的優勢是什麼？

Answer
答案
主要優點是 Redis 可以就地修改資料，而不會通過下載和上傳浪費網絡帶寬。它節省了網絡帶寬，但也通過避免資料的序列化和反序列化來節省時間和精力。
```

## Conclusion (結論)
如果沒有使用快取系統，就不可能想像高速大規模解決方案。在本章中，我們介紹了對快取系統的需求及其基本細節，並且我們編排了一個基本的分佈式快取系統。我們還熟悉了兩個最著名的快取框架的設計和功能。
