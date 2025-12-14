# Requirements of a Newsfeed System’s Design (動態消息系統的設計需求)
## Requirements (需求)
為了限制問題的範圍，我們將專注於以下功能和非功能需求：

### Functional requirements (功能需求)
- **Newsfeed generation (動態消息生成)**：系統將根據使用者關注的頁面、群組和追蹤者生成動態消息。一個使用者可能有許多朋友和追蹤者。因此，系統應該能夠從所有朋友和追蹤者那裡生成動態消息。這裡的挑戰是可能有大量的內容。我們的系統需要決定為使用者選擇哪些內容，並進一步對其進行排名以決定首先顯示哪些內容。
- **Newsfeed contents (動態消息內容)**：動態消息可能包含文字、圖片和影片。
- **Newsfeed display (動態消息顯示)**：系統應該根據某種排名機制將新的傳入貼文附加到所有活躍使用者的動態消息中。排名後，我們將排名較高的內容首先顯示給使用者。

### Non-functional requirements (非功能需求)
- **Scalability (可擴展性)**：我們提出的系統應該具有高度可擴展性，以支持任何平台 (如 Twitter、Facebook 和 Instagram) 上不斷增長的使用者數量。

- **Fault tolerance (容錯性)**：由於系統應該處理大量資料；因此，分區容錯性 (系統組件之間發生網路故障時的系統可用性) 是必要的。

- **Availability (可用性)**：服務必須具有高可用性以保持使用者與平台的互動。根據 PACELC 定理，系統可以在可用性和容錯性方面犧牲強一致性。

- **Low latency (低延遲)**：系統應實時提供動態消息。因此，最大延遲不應超過 2 秒。

```
PACELC 定理是 CAP 定理的擴展，它指出，在發生網路分區 (Partition) 的情況下，應該在可用性 (Availability) 或一致性 (Consistency) 之間進行選擇；否則 (Else)，在延遲 (Latency) 和一致性 (Consistency) 之間進行選擇。
```

## Resource estimation (資源估算)
讓我們假設設計動態消息系統的平台每天有 10 億使用者，其中平均有 5 億是每日活躍使用者。此外，每個使用者平均有 300 個朋友並關注 250 個頁面。基於假設的統計資料，讓我們看看流量、儲存和伺服器估算。

### Traffic estimation (流量估算)
讓我們假設每個每日活躍使用者每天打開應用程式 (或社交媒體頁面) 10 次。每天的總請求數將是：

500M * 10 = 50 億請求/天 ≈ 58K 請求/秒。

[動態消息系統的流量估算](./traffic.jpg)

### Storage estimation (儲存估算)
讓我們假設動態消息將離線生成並在請求時呈現。此外，我們將為每個使用者預先計算前 200 個貼文。讓我們計算使用者元資料、包含文字的貼文和媒體內容的儲存估算。

1. **Users’ metadata storage estimation (使用者元資料儲存估算)**：假設一個使用者的元資料所需的儲存空間為 50 KB。對於 10 億使用者，我們需要 1B * 50KB = 50 TB。
1B * 50KB = 50TB。

我們可以調整估算的數字並在以下計算器中計算我們所需數字的儲存量：
           Storage Estimation for the Users' Metadata (使用者元資料儲存估算)
Number of users (in billion) (使用者數量-十億)	                     1
Required storage for one users' metadata (in KBs) (一個使用者元資料所需儲存-KB)    50
Total storage required for all users (in TBs) (所有使用者所需總儲存-TB)        50
```

2. **Textual post’s storage estimation (文字貼文儲存估算)**：所有貼文都可能包含一些文字，我們假設平均為 50KB。5 億使用者的前 200 個貼文的儲存估算將是：

200 * 500M * 50KB = 5PB

3. **Media content storage estimate (媒體內容儲存估算)**：除了文字，貼文還可以包含媒體內容。因此，我們假設 1/5 的貼文有影片，4/5 包括圖片。假設平均圖片大小為 200KB，影片大小為 2MB。

一個使用者的 200 個貼文的儲存估算：(200 * 2MB * 1/5) + (200 * 200KB * 4/5) = 80MB + 32MB = 112MB

5 億使用者貼文所需的總儲存量：112MB * 500M = 56PB

所以我們至少需要 56PB 的 blob 儲存來儲存媒體內容。

[動態消息系統每天 5 億活躍使用者 (每個約 200 個貼文) 所需的儲存量](./storage.jpg)

                  Storage Estimation of Posts Containing Text and Media Content (包含文字和媒體內容的貼文儲存估算)
Number of active users (in million) (活躍使用者數-百萬)                               500
Maximum allowed text storage per post (in KBs) (每個貼文允許的最大文字儲存-KB)                     50
Number of precomputed posts per user (top N) (每個使用者預算出的貼文數-前 N 個)                      200
Storage required for textual posts (in PBs) (文字貼文所需儲存-PB)                         5
Total required media content storage for active users (in PBs) (活躍使用者所需總媒體內容儲存-PB)     56
```

### Number of servers estimation (伺服器數量估算)
考慮到上述流量和儲存估算，讓我們估算平穩運行所需的伺服器數量。回想一下，單個典型伺服器每秒可以服務 8000 個請求 (RPS)。由於我們的系統將有大約 5 億每日活躍使用者 (DAU)。因此，根據封底計算章節中的估算，我們需要的伺服器數量是：
DAU/ServerRPS = 500M/8000 = 62500 servers (62500 台伺服器)。

[動態消息系統所需的伺服器數量](./servers.jpg)

           Servers Estimation (伺服器估算)
Number of active users (in million) (活躍使用者數-百萬)        500
RPS of a server (伺服器 RPS)	                     8000
Number of servers required (所需伺服器數量)                62500
```

## Building blocks we will use (我們將使用的構建塊)
動態消息系統的設計利用以下構建塊：

[設計動態消息系統的構建塊](./bb.jpg)

- **Database(s) (資料庫)** 需要儲存來自不同實體的貼文和生成的個性化動態消息。它還用於儲存使用者的元資料及其與其他實體 (如朋友和追蹤者) 的關係。
- **Cache (快取)** 是一個重要的構建塊，用於保存頻繁訪問的資料，無論是貼文和動態消息還是使用者的元資料。
- **Blob storage (Blob 儲存)** 對於儲存媒體內容 (例如圖片和影片) 至關重要。
- **CDN** 有效地向終端使用者交付內容，減少後端伺服器的延遲和負擔。
- **Load balancers (負載平衡器)** 對於在可用伺服器池中分配數百萬傳入客戶端的動態消息請求是必要的。

在下一課中，我們將專注於動態消息系統的高層和詳細設計。
