# Evaluation of Uber’s Design (Uber 設計的評估)
## Fulfill non-functional requirements (滿足非功能需求)
讓我們評估一下我們的系統如何滿足非功能需求。

### Availability (可用性)
我們的系統具有高可用性。我們使用了 WebSocket 伺服器。如果使用者斷開連接，則通過負載平衡器與不同的伺服器重新創建會話。我們使用了具有主從複製模型的資料庫的多個副本。我們有 Cassandra 資料庫，它提供高可用性服務且無單點故障。我們使用了 CDN、快取和負載平衡器，這增加了我們系統的可用性。

### Scalability (可擴展性)
我們的系統具有高度可擴展性。我們使用了許多獨立的服務，這樣我們就可以根據需要獨立地橫向擴展這些服務。我們使用四叉樹進行搜尋，將地圖劃分為更小的段，從而縮短了搜索空間。我們使用了 CDN，它增加了處理更多使用者的能力。我們還使用了 NoSQL 資料庫 Cassandra，它是橫向可擴展的。此外，我們使用了負載平衡器，它通過在不同伺服器之間分發讀取工作負載來提高速度。

### Reliability (可靠性)
我們的系統非常可靠。即使乘客或司機的連接中斷，行程也可以繼續。這是通過將他們的手機用作本地存儲來實現的。使用多個 WebSocket 伺服器可確保順暢、近乎實時的運行。如果任何伺服器出現故障，使用者能夠與另一台伺服器重新連接。我們還使用了伺服器和資料庫的冗餘副本，以確保沒有單點故障。我們的服務是解耦和隔離的，這最終增加了可靠性。負載平衡器有助於將請求從任何故障伺服器轉移到健康伺服器。

### Consistency (一致性)
我們使用像 MySQL 這樣的資料庫來保持我們的資料在全球範圍內的一致性。此外，我們的系統進行同步複製以實現強一致性。由於行程的資料寫入者和查看者數量有限 (乘客、司機、一些內部服務)，使用傳統資料庫不會成為瓶頸。而且，在這種情況下，資料分片更容易。

### Fraud detection (欺詐檢測)
我們的系統能夠檢測與支付相關的任何欺詐活動。我們使用 RADAR 系統來檢測任何可疑活動。RADAR 識別欺詐企圖的開始並創建一條規則來防止它。

     Meeting Non-functional Requirements (滿足非功能需求)
Requirements (需求)                Techniques (技術)

Availability (可用性)               - Using server replicas (使用伺服器副本)
                               - Using database replicas with Cassandra database (使用帶有 Cassandra 資料庫的資料庫副本)
                               - Load balancers hide server failures from end users (負載平衡器向最終使用者隱藏伺服器故障)

Scalability (可擴展性)              - Horizontal sharding of the database (資料庫的水平分片)
                               - The Cassandra NoSQL database (Cassandra NoSQL 資料庫)

Reliability (可靠性)                - No single point of failure (無單點故障)
                               - Redundant components (冗餘組件)

Consistency (一致性)               - Strong consistency using synchronous replications (使用同步複製的強一致性)

Fraud detection (欺詐檢測)           - Using RADAR to recognize and prevent any fraud related to payments (使用 RADAR 識別和防止與支付相關的任何欺詐)
```

## Conclusion (結論)
本章教會了我們如何設計像 Uber 這樣的網約車服務。我們討論了它的功能和非功能需求。我們學習了如何使用四叉樹在地圖上有效地定位司機。我們還討論了如何使用路由演算法和機器學習有效地計算預計到達時間。此外，我們了解到，保護我們的服務免受欺詐活動的侵害對於企業的成功至關重要。
