# Evaluation of Quora’s Design (Quora 設計的評估)
## Fulfilling requirements (滿足需求)
我們已經使用了各種技術來滿足我們的功能需求。但是，我們需要確定是否滿足了非功能需求。我們將強調我們用於解決非功能需求的一些機制：

- **Scalability (可擴展性)**：我們的系統具有高度可擴展性，原因有幾個。更新的設計使用強大且同質的服務主機。Quora 使用強大的機器，因為服務主機使用記憶體中快取、某種程度的隊列，並維護管理器、工作者和路由庫。這些服務主機的水平擴展很方便，因為它們是同質的。

```
Quora 現在使用具有 32 核、共享 20 MB 快取和高網路吞吐量的 Amazon EC2 實例。
```

在資料庫方面，我們的設計垂直分片 MySQL 資料庫，這避免了由於 MySQL 伺服器過載而導致的可擴展性問題。為了減少複雜的連接查詢，預計將進行連接操作的表被放置在同一分片或分區中。

```
注意：如前所述，垂直分片可能不夠，因為每個分片都可以在水平方向上變大。對於大型 MySQL 表，寫入成為瓶頸。因此，我們的設計可能必須堅持水平分片，這是資料庫擴展中眾所周知的做法。
```

- **Consistency (一致性)**：由於 Quora 提供了各種功能，因此可以為不同類型的資料選擇不同的一致性方案。例如，某些像問題和答案這樣的關鍵資料應該同步儲存。在這種情況下，效能可能會受到影響，因為使用者不希望對他們的問題得到瞬時回覆。這意味著，根據使用者的問題和潛在受訪者的可用性，使用者可能會在五分鐘、一小時、一天或根本沒有回覆時收到回覆。

其他資料 (如查看計數) 可能不一定同步儲存，因為 Quora 服務的目標不是確保所有使用者在發布問題後立即看到相同的查看次數。對於這種情況，為了提高效能，傾向於最終一致性。

```
注意：一般來說，我們的設計配備了強大的技術來整體降低使用者感知的延遲。
```

- **Availability (可用性)**：提高可用性的一些主要點子包括不同組件之間的隔離、保持冗餘實例、使用 CDN、使用像 ZooKeeper 這樣的配置服務，以及使用負載平衡器對使用者隱藏故障。

```
解耦服務有助於在單個服務或主機發生故障期間進行維護、升級和恢復。如果在同一節點或主機上託管服務，則發生故障的節點會導致同時丟失多個服務。
```

然而，我們的設計缺乏任何災難恢復管理，我們將在下一節中探討這一點。

- **Performance (效能)**：這種設計具有很強的效能，因為我們為正確的功能使用了正確的技術。例如，我們出於不同的原因使用了幾個資料儲存。最重要的是，我們根據用例和訪問頻率使用了不同的分散式快取。此外，我們使用 Kafka 對類似任務進行排隊並將其分配給 cron 作業，如果通過 API 調用執行這些任務，否則將花費很長時間。

```
注意：Quora 聲稱使用其自定義隊列解決方案，每秒可以處理大約 15,000 個任務。
```

Facts (事實)
Quora 開發了一個名為 Pycache 的自定義記憶體中快取系統，可以在本地機器中儲存數十 GB 的快取。

Pycache API 包含簡單但有效的函式，如 pycache.get(key), pycache.set(key, value), pycache.delete(key), pycache.iteritems(), 和 pycache.update()。

set 和 delete 函式都有變體可以一次設置和刪除多個條目。

Quora 開發並開源了一個名為 Asynq 的工具，簡化了對 Memcached 的快取請求的批處理。因此，開發人員可以更快地編寫程式碼——並且減少了到快取服務的往返延遲。
```

Meeting Non-functional Requirements (滿足非功能需求)

Requirements (需求)           Techniques (技術)

Scalability (可擴展性)        Based on AWS, which supports automatic scaling. (基於 AWS，支持自動縮放。)

                              Uses the same servers to reduce complexity in horizontal scalability (使用相同的伺服器來降低水平可擴展性的複雜性)
                            
                              sharding of MySQL database. (MySQL 資料庫的分片。)
                            
                              Employs various data stores for different purposes. (為不同目的採用各種資料儲存。)
                            
                              Asynq can enable developers to code quickly by batching cache requests, separate compute, and feature extraction modules. Therefore, a generic feature extraction facility allows scalability of different recommendation systems. (Asynq 可以通過批處理快取請求、分離計算和特徵提取模組，使開發人員能夠快速編碼。因此，通用的特徵提取設施允許不同推薦系統的可擴展性。)

Consistency (一致性)          Uses MySQL and synchronous replication within a data center for critical data. (在資料中心內對關鍵資料使用 MySQL 和同步複製。)
                            
                              Offers eventual consistency for non-critical data like the view counter. (為非關鍵資料 (如查看計數器) 提供最終一致性。)
                            
Availability (可用性)         Use of different data stores prevents failure of multiple services at once by using database sharding and replicas. (通過使用資料庫分片和副本，使用不同的資料儲存可以防止多個服務同時發生故障。)

                              Uses CDN as a backup to serve static/dynamic data in case of failures. (在發生故障的情況下，使用 CDN 作為備份來提供靜態/動態資料。)

                              ZooKeeper enables service hosts to get updates about MySQL shards. (ZooKeeper 使服務主機獲得有關 MySQL 分片的更新。)

                              Load balancers hide server failures from end users. (負載平衡器對最終使用者隱藏伺服器故障。)

                              AWS supports an availability above 99 percent. (AWS 支持 99% 以上的可用性。)

                              Thrift isolates services and therefore failures. (Thrift 隔離服務，從而隔離故障。)

Performance (效能)            MyRocks has a much lower P99 latency. (MyRocks 具有低得多的 P99 延遲。)
                       
                              Uses the right programming language to deliver tasks quickly, such as C++ for the routing library. (使用正確的編程語言快速交付任務，例如用於路由庫的 C++。)

                              Uses `Multiget()`  to retrieve multiple entries from Memcached at once. (使用 `Multiget()` 一次從 Memcached 檢索多個條目。)

                              Eliminates network round trip time with Asynq. (使用 Asynq 消除網路往返時間。)

                              Kafka improves the performance of service hosts. (Kafka 提高了服務主機的效能。)

                              Sharding improves QPS of MySQL. (分片提高了 MySQL 的 QPS。)

                              Custom, in-memory caching system reduces the latency of frequently accessed data. (自定義的記憶體中快取系統減少了頻繁訪問資料的延遲。)
```

## Disaster recovery (災難恢復)
我們提議的和詳細的設計沒有迎合自然災害的情況。雖然我們滿足了其他非功能需求，但如果沒有災難恢復管理計劃，持久性、容錯性和可用性是不完整的。本節將探討一些提供抗災能力的機制。

處理災難的首要方法是頻繁備份。備份的頻率取決於資料的大小。每日備份適合我們的設計，因為我們可以毫不費力地備份單個資料儲存和分片。當然，備份將存儲在遠端目的地，因為自然災害可能會摧毀某個地點的整個設施。

```
自然災害：災難包括大規模洪水、地震、火災和多個主幹網際網路連接中斷，導致整個資料中心或多個資料中心癱瘓。
```

```
注意：在遠端位置進行定期備份是不夠的。及時快速恢復備份資料才算完成了災難恢復計劃。
```

以下是設計災難恢復計劃的重要問題：

- 哪些資料和系統被認為對於從災難中恢復至關重要？
- 從備份設施恢復有多快？
- 所有系統都可以通過備份恢復嗎？
- 我們如何處理在災難發生前無法複製的潛在資料丟失？

下圖顯示了災難恢復方案如何工作的簡單架構：

[災難恢復管理](./recovery.jpg)

該方法非常簡單。資料、應用程式伺服器和配置在同一區域的 Amazon S3 儲存服務中進行備份。S3 儲存之間的區域複製便於傳輸到另一個區域。稍后，可以從另一個區域的 S3 儲存恢復應用程式和資料庫伺服器。

恢復策略簡單有效，但也有幾個缺點：

- 由於我們每天進行備份，因此可能會丟失一些未備份的資料。但是，如果我們在跨區域進行同步複製，則可以緩解此問題。
- 恢復可能需要很長時間 (幾個小時)，並且大多數資料庫在恢復資料時不提供查詢服務。

```
注意：一般來說，Amazon 提供具有高可靠性和可用性的服務。例如，S3 服務報告一年內 99.999999999% 的持久性和 99.9% 的可用性。
```

## Conclusion (結論)
在整個設計中，我們了解了 Quora 如何隨著使用者數量的增加擴展其服務。設計的一個有趣方面包括 MySQL 資料庫的垂直分片。除此之外，Quora 設計討論了各種技術來滿足功能和非功能需求。然而，我們的範圍不包括使用自然語言處理 (NLP) 等技術來消除使用者問題中的拼寫錯誤或搜尋期間的預輸入服務。

```
Question
針對不同類型的資料使用不同的資料儲存對災難恢復有何好處？

Answer
當我們使用不同的資料儲存時，這意味著我們可以同時恢復各種形式的資料，而不是一大塊資料。由於我們恢復具有可變恢復時間的資料儲存，因此最終恢復時間等於所有資料儲存中的最大恢復時間。

然而，使用許多資料庫也意味著管理員需要了解檢查點或恢復它們的詳細資訊。
```
