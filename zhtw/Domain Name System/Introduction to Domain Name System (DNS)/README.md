# Introduction to Domain Name System (DNS) (域名系統 (DNS) 簡介)

## The origins of DNS (DNS 的起源)
讓我們以手機為例，每個使用者都關聯一個唯一的號碼。為了給朋友打電話，我們最初可以嘗試記住一些電話號碼。但是，隨著聯繫人數量的增加，我們必須使用電話簿來記錄所有聯繫人。這樣，每當我們需要打電話時，我們就會查閱電話簿並撥打我們需要的號碼。

同樣，計算機由 IP 地址唯一標識——例如，104.18.2.119 是一個 IP 地址。我們使用 IP 地址訪問託管在機器上的網站。由於人類無法輕易記住 IP 地址來訪問域名 (例如 educative.io)，我們需要一個類似電話簿的存儲庫，它可以維護域名到 IP 地址的所有映射。在本章中，我們將看到 DNS 如何充當互聯網的電話簿。

[DNS 的起源](./dns_brief.jpg)

## What is DNS? (什麼是 DNS？)
域名系統 (DNS) 是互聯網的命名服務，它將人類友好的域名映射到機器可讀的 IP 地址。DNS 服務對使用者是透明的。當使用者在瀏覽器中輸入域名時，瀏覽器必須通過詢問 DNS 基礎設施將域名轉換為 IP 地址。一旦獲得所需的 IP 地址，使用者的請求就會轉發到目標 Web 伺服器。

下面的幻燈片顯示了 DNS 工作的高級流程：
[什麼是 DNS](./dns_flow)

整個操作執行得非常快。因此，最終使用者體驗到的延遲最小。我們還將在下一課中看到瀏覽器如何保存一些常用的映射以供以後使用。

## Important details (重要細節)
讓我們重點介紹一些關於 DNS 的重要細節，其中一些我們將在下一課中介紹：

**Name servers (名稱伺服器)**: It’s important to understand that the DNS isn’t a single server. It’s a complete infrastructure with numerous servers. DNS servers that respond to users’ queries are called name servers.
**名稱伺服器**：重要的是要理解 DNS 不是單個伺服器。這是一個擁有眾多伺服器的完整基礎設施。響應使用者查詢的 DNS 伺服器稱為名稱伺服器。

**Resource records (資源記錄)**: The DNS database stores domain name to IP address mappings in the form of resource records (RR). The RR is the smallest unit of information that users request from the name servers. There are different types of RRs. The table below describes common RRs. The three important pieces of information are type, name, and value. The name and value change depending upon the type of the RR.
**資源記錄**：DNS 資料庫以資源記錄 (RR) 的形式存儲域名到 IP 地址的映射。RR 是使用者向名稱伺服器請求的最小資訊單位。有不同類型的 RR。下表描述了常見的 RR。三個重要的資訊是類型、名稱和值。名稱和值根據 RR 的類型而變化。
[資源記錄](./common_types_of_rr.jpg)

**Caching (緩存)**: DNS uses caching at different layers to reduce request latency for the user. Caching plays an important role in reducing the burden on DNS infrastructure because it has to cater to the queries of the entire Internet.
**緩存**：DNS 在不同層使用緩存來減少使用者的請求延遲。緩存對於減輕 DNS 基礎設施的負擔起著重要作用，因為它必須滿足整個互聯網的查詢。

**Hierarchy (層次結構)**: DNS name servers are in a hierarchical form. The hierarchical structure allows DNS to be highly scalable because of its increasing size and query load. In the next lesson, we’ll look at how a tree-like structure is used to manage the entire DNS database.
**層次結構**：DNS 名稱伺服器呈層次結構形式。層次結構允許 DNS 高度可擴展，因為它的規模和查詢負載不斷增加。在下一課中，我們將研究如何使用類樹結構來管理整個 DNS 資料庫。
