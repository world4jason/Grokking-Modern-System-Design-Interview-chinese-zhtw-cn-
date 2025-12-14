# Evaluation of Google Docs’ Design (Google Docs 設計的評估)

我們已經解釋了設計以及它如何滿足協同文件編輯服務的功能需求。本課將重點關注我們的設計如何滿足非功能需求。特別是，我們將關注一致性、延遲、可擴展性和可用性。

## Consistency (一致性)
我們已經研究了如何通過兩種技術：操作轉換 (OT) 和無衝突複製資料類型 (CRDT) 來實現文件中衝突解決的強一致性。此外，時間序列資料庫使我們能夠保留事件的順序。一旦 OT 或 CRDT 解決了任何衝突，最終結果將儲存在資料庫中。這有助於我們在單個操作方面實現一致性。

我們還有興趣使不同伺服器上的文件狀態在資料中心內保持一致。為了同時在同一個資料中心內複製文件的更新狀態，我們可以使用像 Gossip 協議 (流言協議) 這樣的點對點協議。這種策略不僅會提高一致性，還會提高可用性。

也稱為流行病協議 (epidemic protocol)，用於在網路中的所有節點之間傳播資料以實現一致性。
```

Question (問題)
為什麼我們應該在協同文件編輯服務中使用強一致性而不是最終一致性來解決衝突？

Answer (答案)
從 Amazon 的 Dynamo 系統中，我們了解到，如果我們使用最終一致性來解決衝突，我們可能會有多個版本的文檔最終協調，無論是自動還是手動。在自動協調的情況下，文件可能會突然更新，這違背了協作的目的。第二種情況，手動解決，是我們想要避免的繁瑣勞動。

因此，我們使用強一致性來解決衝突，邏輯上集中的伺服器向所有客戶端提供最終的事件順序。我們使用複製的操作隊列，這樣即使我們的排序服務模組宕機，它也可以輕鬆地在新的伺服器上重新啟動並從中斷的地方繼續。在重新生成故障組件時，客戶端可能會遇到短暫的服務不可用。
```

## Latency (延遲)
延遲可能感覺像是一個挑戰，特別是當兩個使用者彼此距離很遠或距離伺服器很遠時。然而，使用者在他們端維護文件的副本，同時資料通過 WebSockets 傳播到終端伺服器。因此，使用者感知的延遲將很低。除此之外，使用者大多傾向於在文件中寫入少量的文字資料。因此，同一設施內的不同伺服器之間以及不同資料中心或區域之間的資料傳播將以低延遲進行。此外，像影片和圖片這樣的檔案可以存儲在 CDN 中以便快速提供服務，因為這些內容不會頻繁更改。

實際上，線上文件的讀者和作者數量有限。特別是對於讀者來說，延遲不成問題，因為文件只加載一次。因此，大多數讀者可以從同一個資料中心獲得服務。對於作者，應選擇最佳區域作為同一文件協作者之間的集中位置。然而，對於流行文件，非同步複製將是實現大量使用者獲得良好效能和低延遲的有效方法。一般來說，當複製是非同步時，實現強一致性會成為一個挑戰。

## Availability (可用性)
我們的設計通過使用副本並使用監控服務監控主伺服器和副本伺服器來確保可用性。操作隊列和資料儲存等關鍵組件內部管理其複製。

由於我們使用 WebSockets，我們的 WebSocket 伺服器可以將使用者連接到會話維護伺服器，該伺服器將確定使用者是否正在積極查看或協作處理文件。因此，保持多個 WebSocket 伺服器將增加設計的可用性。最後，我們採用快取服務和 CDN 來提高發生故障時的可用性。

但是，目前我們還沒有制定災難恢復管理方案。

                    Fulfilling Non-functional Requirements (滿足非功能需求)
Requirements (需求)        Techniques (技術)                 
Consistency (一致性)       - Gossip protocol to replicate operations of a document within the same data center (Gossip 協議在同一資料中心內複製文件的操作)
                           - Concurrency techniques like OT and CRDTs (OT 和 CRDT 等併發技術)
                           - Usage of time series database for maintaining the order of operations (使用時間序列資料庫來維護操作順序)
                           - Replication between data centers (資料中心之間的複製)

Latency (延遲)             - Employing WebSockets (採用 WebSockets)
                           - Asynchronous replication of data (資料的非同步複製)
                           - Choosing optimal location for document creation and serving (選擇建立和提供文件的最佳位置)
                           - Using CDNs for serving videos and images (使用 CDN 提供影片和圖片)
                           - Using Redis to store different data structures including CRDTs (使用 Redis 儲存包括 CRDT 在內的不同資料結構)
                           - Appropriate NoSQL databases for the required functionality (針對所需功能使用適當的 NoSQL 資料庫)

Availability (可用性)      - Replication of components to avoid SPOFs (組件複製以避免單點故障)
                           - Using multiple WebSocket servers for users that may occasionally disconnect (為偶爾斷開連接的使用者使用多個 WebSocket 伺服器)
                           - Component isolation improves availability (組件隔離提高可用性)
                           - Implementing disaster recovery protocols like backup, replication to different zones, and global server load balancing (實施災難恢復協議，如備份、複製到不同區域和全域伺服器負載平衡)
                           - Using monitoring and configuration services (使用監控和配置服務)
                           
Scalability (可擴展性)     - Different data stores for different purposes enable scalability (不同用途的不同資料儲存實現可擴展性)
                           - Horizontal sharding of RDBMS (RDBMS 的水平分片)
                           - CDNs capable of handling a large number of requests for big files (能夠處理大量大檔案請求的 CDN)
```

## Scalability (可擴展性)
由於我們使用了微服務架構，如果操作隊列上的請求數量超過其容量，我們可以輕鬆地單獨擴展每個組件。我們可以使用多個操作隊列。在這種情況下，每個操作隊列將負責單個文件。我們可以將不同使用者請求的與單個文件關聯的操作轉發到特定隊列。生成的隊列數量將等於活動文件的數量。因此，我們能夠實現水平可擴展性。

## Conclusion (結論)
在本章中，我們設計了一個線上協同文件編輯服務。在我們的設計中，我們提供了諸如協同編輯、保留版本歷史記錄以恢復到舊版本、向使用者提供常用術語和短語的建議以及文件查看次數等功能。我們還評估了在同一文件上協作的使用者之間新增聊天功能的可能性。該設計的一個獨特方面是不同使用者併發編輯操作之間的衝突解決。我們通過 OT 和 CRDT 解決了併發問題。
