# Requirements of Instagram’s Design (Instagram 的設計需求)
## Requirements (需求)
為了使這種設計簡單化，我們將專注於 Instagram 的一些重要功能。讓我們列出我們系統的需求：

### Functional requirements (功能需求)
- **Post photos and videos (發布照片和影片)**：使用者可以在 Instagram 上發布照片和影片。

- **Follow and unfollow users (關注和取消關注使用者)**：使用者可以在 Instagram 上關注和取消關注其他使用者。

- **Like or dislike posts (對貼文按讚或不按讚)**：使用者可以對他們關注的帳戶的貼文按讚或不按讚。

- **Search photos and videos (搜尋照片和影片)**：使用者可以根據標題和位置搜尋照片和影片。

- **Generate news feed (生成動態消息)**：使用者可以查看由他們關注的所有使用者的照片和影片 (按時間順序) 組成的動態消息。使用者還可以在其動態消息中查看建議和推廣的照片。

### Non-functional requirements (非功能需求)
- **Scalability (可擴展性)**：系統應該是可擴展的，以便在計算資源和儲存方面處理數百萬使用者。

- **Latency (延遲)**：生成動態消息的延遲應該很低。

- **Availability (可用性)**：系統應具有高可用性。

- **Durability (持久性)**：任何上傳的內容 (照片和影片) 都不應丟失。

- **Consistency (一致性)**：我們可以在一致性上稍微妥協。如果內容 (照片或影片) 需要時間才能顯示在位於遠處的關注者的動態消息中，這是可以接受的。

- **Reliability (可靠性)**：系統必須能夠容忍硬體和軟體故障。

## Resource estimation (資源估算)
我們的系統是讀取密集的，因為服務使用者花費大量時間瀏覽他人的動態消息，而不是建立和發布新內容。我們的重點將是設計一個可以準時獲取照片和影片的系統。對使用者可以上傳的照片或影片的數量沒有限制，這意味著高效的儲存管理應該是該系統設計的主要考慮因素。Instagram 在全球支持約 10 億使用者，他們每天在 Instagram 上分享 9500 萬張照片和影片。我們將根據需求計算資源並設計我們的系統。

讓我們假設以下情況：

- 我們有 10 億使用者，其中 5 億是每日活躍使用者。

- 假設每天在 Instagram 上分享 6000 萬張照片和 3500 萬個影片。

- 我們可以認為每張照片的最大大小為 3 MB，每段上傳到 Instagram 的影片的最大大小為 150 MB。

- 平均而言，每個使用者每天向我們的服務發送 20 個請求 (任何類型)。

### Storage estimation (儲存估算)
我們需要估算儲存容量、頻寬和伺服器數量，以支持如此龐大的使用者和內容數量。

每天的儲存量將是：

60 million photos/day * 3 MB = 180 TeraBytes / day (6000 萬張照片/天 * 3 MB = 180 TB / 天)

35 million videos/day * 150 MB = 5250 TB / day (3500 萬個影片/天 * 150 MB = 5250 TB / 天)

Total content size (總內容大小) = 180 + 5250 = 5430 TB

一年所需的總空間：

5430 TB/day * 365 (days a year) = 1981950 TB = 1981.95 PetaBytes (5430 TB/天 * 365 (天/年) = 1981950 TB = 1981.95 PB)

[Instagram 儲存系統一年所需的總儲存量](./storage.jpg)

除了照片和影片，我們忽略了評論、狀態分享資料等。此外，我們還必須儲存使用者的資訊和貼文元資料，例如 userID、照片等。所以，準確地說，我們需要的儲存量超過 5430 TB/天，但為了保持我們的設計簡單，讓我們堅持 5430 TB/天。

### Bandwidth estimation (頻寬估算)
根據我們儲存容量的估算，我們的服務每天將獲得 5430 TB 的資料，這將給我們：

5430 TB/(24 * 60* 60) = 5430 TB/86400 sec ~= 62.84 GB/s ~= 502.8 Gbps

由於每張傳入的照片和影片都需要到達使用者的關注者，假設讀者與作者的比例為 100:1。因此，我們需要的頻寬是傳入頻寬的 100 倍。讓我們假設以下情況：

Incoming bandwidth (傳入頻寬) ~= 502.8 Gbps

Required outgoing bandwidth (所需傳出頻寬) ~= 100 * 502.8 Gbps ~= 50.28 Tbps

[所需的總頻寬](./bandwidth.jpg)

### Number of servers estimation (伺服器數量估算)
我們需要處理來自 5 億每日活躍使用者的併發請求。讓我們假設一個典型的 Instagram 伺服器每秒處理 100 個請求：

Requests by each user per day (每個使用者每天的請求數) = 20
Queries handled by a server per second (伺服器每秒處理的查詢數) = 100

Queries handled by a server per day (伺服器每天處理的查詢數) = 100 * 60 * 60 * 24 = 8640000

(活躍使用者數 * 每個使用者每天的請求數 / 伺服器每天處理的查詢數 = 約 1157 台伺服器)

此計算顯示我們需要 1157 台伺服器來處理 Instagram 系統中的查詢。

[Instagram 系統所需的伺服器數量](./servers.jpg)


## Try it yourself (自己試試)
讓我們分析每天的照片數量如何影響儲存和頻寬需求。為此，嘗試更改下表中的值以計算估算值。我們在以下計算中設定了每個使用者每天 20 個請求：

Requirements (需求) 	                            Calculations (計算)
Number of photos per day (in millions) (每天照片數-百萬)             60
Size of each photo (in MB) (每張照片大小-MB)                           3
Number of videos per day (in millions) (每天影片數-百萬)             35
Size of each video (in MB) (每個影片大小-MB)                          150
Number of daily active users (in millions) (每日活躍使用者數-百萬)          500
Number of requests a server can handle per day (伺服器每天可處理的請求數)	8640000
Storage estimation per day (in TB) (每天儲存估算-TB)                 5430
Incoming bandwidth (Gb/s) (傳入頻寬-Gb/s)                          502.8
Number of servers needed (所需伺服器數量)                           1157
```

## Building blocks we will use (我們將使用的構建塊)
在下一課中，我們將專注於 Instagram 的高層設計。該設計將利用許多在初始章節中也討論過的構建塊。我們將在設計中使用以下構建塊：

[Instagram 設計中使用的構建塊](./bb.jpg)

- **Load balancer (負載平衡器)** 在各個層面上確保請求在可用伺服器之間平滑分配。
- **Database (資料庫)** 用於儲存使用者和帳戶元資料及其之間的關係。
- **Blob storage (Blob 儲存)** 需要儲存各種類型的內容，如照片、影片等。
- **Task scheduler (任務調度程式)** 調度資料庫上的事件，例如刪除生存時間超過限制的條目。
- **Cache (快取)** 儲存最頻繁的內容相關請求。
- **CDN** 用於向終端使用者有效交付內容，從而減少終端伺服器的延遲和負擔。

在下一課中，我們將討論 Instagram 系統的高層設計。
