# Requirements of Google Docs’ Design (Google Docs 設計需求)
## Requirements (需求)
讓我們看看設計協同編輯服務的功能需求和非功能需求。

### Functional requirements (功能需求)
使用者將能夠使用我們的協同文件編輯服務執行的活動如下：

- **Document collaboration (文件協作)**：多個使用者應該能夠同時編輯一份文件。此外，大量使用者應該能夠查看一份文件。
- **Conflict resolution (衝突解決)**：系統應將一個使用者所做的編輯推送到所有其他協作者。如果使用者編輯文件的同一部分，系統還應解決他們之間的衝突。
- **Suggestions (建議)**：使用者應該獲得有關完成文件中常用單詞、短語和關鍵字的建議，以及有關修復語法錯誤的建議。
- **View count (查看次數)**：文件的編輯者應該能夠查看文件的查看次數。
- **History (歷史)**：使用者應該能夠查看文件的協作歷史記錄。

現實世界的文件編輯器還必須具有文件建立、刪除和管理使用者存取等功能。我們專注於上面列出的核心功能，但我們也在前面的課程中討論了其他功能的可能性。

[協同編輯服務的功能需求和非功能需求](./req.jpg)

### Non-functional Requirements (非功能需求)
- **Latency (延遲)**：不同的使用者可以連接起來協作處理同一份文件。對於從不同區域連接的使用者來說，保持低延遲具有挑戰性。
- **Consistency (一致性)**：系統應該能夠解決併發編輯文件的使用者之間的衝突，從而實現文件的一致視圖。同時，不同區域的使用者應該看到文件的更新狀態。保持一致性對於連接到相同和不同區域的使用者都很重要。
- **Availability (可用性)**：服務應始終可用，並顯示出針對故障的穩健性。
- **Scalability (可擴展性)**：大量使用者應該能夠同時使用該服務。他們可以查看同一份文件或建立新文件。

## Resource estimation (資源估算)
讓我們根據以下假設進行一些資源估算：

- 我們假設有 8000 萬每日活躍使用者 (DAU)。
- 同時編輯一份文件的最大使用者數為 20。
- 文字文件的大小為 100 KB。
- 30% 的文件包含圖片，而只有 2% 的文件包含影片。
- 文件中圖片所需的集體儲存為 800 KB，而每個影片為 3 MB。
- 使用者每天建立一份文件。

基於這些假設，我們將進行以下估算。

### Storage estimation (儲存估算)
考慮到每個使用者每天都能建立一份文件，每天總共建立 8000 萬份文件。下面，我們估算一天所需的儲存：

注意：我們可以調整下表中的值以查看儲存需求估算如何變化。
```

Estimation for Storage Requirements (儲存需求估算)

Number of documents created by each user (每個使用者建立的文件數)     1 per day (每天 1 份)
Number of active users (活躍使用者數)                              80 Million (8000 萬)
Number of documents in a day (一天中的文件數)                      80 Million (8000 萬)
Storage required for textual content per day (每天文字內容所需儲存)  8 TB
Storage required for images per day (每天圖片所需儲存)             19.2 TB
Storage required for video content per day (每天影片內容所需儲存)  4.8 TB
Total storage required per day (每天所需總儲存)                    32TB
```

- 一天的文件總數：80M×1document=80Mdocuments (每天 8000 萬份文件)
- 每個文字文件的儲存：80M×100KBs=8TBs
- 一天圖片所需的儲存：(80M×30)/100 ×800KBs=19.2 TBs (30% 的文件包含圖片。)
- 一天影片內容所需的儲存：(80M×2)/100 ×3MBs=4.8 TBs (2% 的文件包含影片。)

[線上協同文件編輯服務每天所需的儲存](./storage.jpg)

一天所需的總儲存如下：8 + 19.2 + 4.8 = 32 TBs per day (每天 32 TB)

注意：雖然我們的功能需求指出我們應該保留文件的歷史記錄，但為了簡潔起見，我們沒有包括歷史資料的儲存需求。
```

### Bandwidth estimation (頻寬估算)
Incoming traffic (傳入流量)：假設每天有 32 TB 的資料上傳到協同編輯服務的網路，傳入流量的網路需求如下：

32 TB/86400 ×8=3Gbps approximately (大約 3Gbps)

Outgoing traffic (傳出流量)：為了估算傳出流量頻寬，我們將假設每個使用者每天查看的文件數量。讓我們考慮一個典型使用者每天查看五份文件。然後，適用以下計算：

注意：我們可以調整下表中的值以查看計算如何變化。
```

Number of documents viewed by users (使用者查看的文件數)           5 per day (每天 5 份)
Number of active users (活躍使用者數)                            80 Million (8000 萬)
Number of documents viewed in a day (一天查看的文件數)           400 Million (4 億)
Number of documents viewed in a second (每秒查看的文件數)        4630 per second (每秒 4630 份)
Bandwidth required for textual content per second (每秒文字內容所需頻寬)      3.704 Gigabits per second(Gbps)
Bandwidth for image-based content per second (每秒基於圖片的內容頻寬)         8.89 Gbps
Bandwidth for video content per second (每秒影片內容頻寬)                     2.22 Gbps
Total outgoing bandwidth required (所需總傳出頻寬)                            14.81 Gbps
```

- 所有使用者每天查看的總文件數：80M×5=400M (每天查看 4 億份文件)
- 每秒查看的文件數：400M/86400 =4.6×10^3 (每秒查看 4.6×10^3 份文件)
- 計算文字內容所需的頻寬：4.6×10^3 ×100KB×8=3.7Gbps
- 計算基於圖片的內容所需的頻寬：(4.6×10^3 ×30)/100 ×800KB×8=8.8Gbps
- 計算影片內容所需的頻寬：(4.6×10^3×2)/100 ×3MB×8=2.2Gbps
- 總傳出頻寬：3.7 + 8.8 + 2.2 = 14.7 Gbps

[總結頻寬需求](./bandwidth.jpg)

注意：所需的總頻寬等於傳入和傳出流量的總和。=3+14.7≈18Gbps (大約 18Gbps)。
```

### Number of servers estimation (伺服器數量估算)
讓我們假設一個使用者每天能夠生成 100 個請求。記住每日活躍使用者的數量，每秒請求數 (RPS) 將如下所示：

100×80M= 8000M/86400=92.6 thousands/sec (每秒 9.26 萬)

                   Number of RPS (RPS 數量)
Requests by a user (使用者請求數)          100 per day (每天 100)
Number of DAU (DAU 數量)                   80 Million (8000 萬)
RPS (每秒請求數)                           92.6 Thousands per second (每秒 9.26 萬)              
```               

我們在粗略估算課程中討論過，RPS 不足以計算伺服器數量。我們將使用以下近似值來計算伺服器數量。

為了估算滿足 8000 萬使用者請求所需的伺服器數量，我們只需將使用者數量除以伺服器可以處理的請求數量。在“粗略估算”課程中，我們討論了我們的參考伺服器每秒可以處理 8,000 個請求。所以，我們看到以下內容：

Number of daily active users/Queries handled per second (每日活躍使用者數/每秒處理的查詢數) = 80M/8000 =10,000

非正式地，上面的等式假設一台伺服器可以處理 8,000 個使用者。

[所需伺服器數量](./servers.jpg)

## Building blocks we will use (我們將使用的構建塊)
我們將在設計協同文件編輯服務時使用以下構建塊。

[設計中需要整合的構建塊]

- **Load balancers (負載平衡器)** 將是使用者的第一個接觸點。
- **Databases (資料庫)** 將需要儲存若干東西，包括文字內容、文件歷史記錄、使用者資料等。為此，我們可能需要不同類型的資料庫。
- **Pub-sub systems (發布-訂閱系統)** 可以完成無法立即執行的任務。我們將在設計中非同步完成許多任務。因此，我們將使用發布-訂閱系統。
- **Caching (快取)** 將幫助我們提高設計的效能。
- **Blob storage (Blob 儲存)** 將儲存大檔案，如圖片和影片。
- **Queueing system (隊列系統)** 將對不同使用者請求的編輯操作進行排隊。因為許多編輯請求無法同時執行，我們必須將它們暫時放入隊列中。
- **CDN** 可以儲存文件中經常存取的媒體。我們還可以將經常請求的唯讀文件放入 CDN 中。
