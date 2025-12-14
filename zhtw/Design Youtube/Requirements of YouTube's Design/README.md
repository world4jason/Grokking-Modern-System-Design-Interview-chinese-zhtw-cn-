# Requirements of YouTube's Design (YouTube 的設計需求)
## Requirements (需求)
讓我們從設計類似 YouTube 的系統的需求開始。

### Functional requirements (功能需求)
我們要求我們的系統能夠執行以下功能：

1. **Stream videos (流式傳輸影片)**
2. **Upload videos (上傳影片)**
3. **Search videos according to titles (根據標題搜尋影片)**
4. **Like and dislike videos (喜歡和不喜歡影片)**
5. **Add comments to videos (為影片新增評論)**
6. **View thumbnails (查看縮略圖)**

[功能和非功能需求的表示](./youtube.jpg)

### Non-functional requirements (非功能需求)
我們的系統滿足以下要求也很重要：

- **High availability (高可用性)**：系統應具有高可用性。高可用性需要良好的正常運行時間百分比。通常，99% 及以上的正常運行時間被認為是好的。
- **Scalability (可擴展性)**：隨著使用者數量的增加，這些問題不應成為瓶頸：上傳內容的存儲、同時觀看所需的頻寬以及並發使用者請求的數量不應使我們的應用程式/Web 伺服器不堪重負。
- **Good performance (良好效能)**：流暢的流媒體體驗會帶來更好的整體效能。
- **Reliability (可靠性)**：上傳到系統的內容不應丟失或損壞。
對於 YouTube 的設計，我們不需要強一致性。考慮一個創作者上傳影片的例子。並非所有訂閱創作者頻道的用戶都應立即收到上傳內容的通知。

總而言之，功能需求是使用者將獲得的功能和功能性，而非功能需求是系統效能方面的期望。

根據需求，我們將估算所需的資源和系統設計。

## Resource estimation (資源估算)
估算需要確定我們在系統中需要的重要資源。

每分鐘都有數百分鐘的影片內容上傳到 YouTube。此外，大量使用者將同時流式傳輸內容，這意味著將需要以下資源：

- 需要儲存資源來儲存上傳和處理的內容。
- 通過並發處理可以處理大量請求。這意味著 Web/應用伺服器應該到位以服務這些使用者。
- 需要上傳和下載頻寬來為數百萬使用者提供服務。
為了將上述資源轉換為實際數字，我們假設以下內容：

- YouTube 使用者總數：15 億。
- 每日活躍使用者 (觀看或上傳影片的使用者)：5 億。
- 影片的平均長度：5 分鐘。
- 處理/編碼 (壓縮、格式更改等) 前平均 (5 分鐘長) 影片的大小：600 MB。
- 編碼後的平均影片大小 (使用不同的演算法針對不同的解析度，如 MPEG-4 和 VP9)：30 MB。

### Storage estimation (儲存估算)
為了找出 YouTube 的存儲需求，我們必須估算每分鐘上傳到 YouTube 的影片總數和每個影片的長度。讓我們考慮一分鐘內有 500 小時的內容上傳到 YouTube。由於每個 30 MB 的影片長 5 分鐘，因此我們需要 30/5 = 6 MB 來存儲 1 分鐘的影片。

讓我們通過假設以下內容將其放入公式中：

Total_{storage} : Total storage requirement. (總存儲需求)

Total_{upload/min} : Total content uploaded (in minutes) per minute. (每分鐘上傳的總內容 (以分鐘為單位))

- Example: 500 hours worth of video is uploaded in one minute. (例如：一分鐘內上傳了 500 小時的影片。)
Storage_{min} : Storage required for each minute of content (每分鐘內容所需的存儲空間)

然後，使用以下公式計算存儲空間：

Total_{storage} = Total_{upload/min} x Storage_{min}

下面是一個計算器，可以幫助我們估算所需的資源。我們將首先查看每分鐘存儲 500 小時上傳內容所需的存儲空間，其中每分鐘影片需要 6 MB 來存儲：

```
                            Storage Required for Storing Content per Minute on YouTube
                            
No. of video hours per minute (每分鐘影片小時數)                 	Minutes per hour (每小時分鐘數)	              MB per minute (每分鐘 MB)                	Storage per minute (GB) (每分鐘存儲 (GB))
500                                                 60                               6                           180

```

```
嘗試更改小時數和每分鐘 MB 的值，看看它們對存儲空間需求的影響。
```

上述數字對應於影片的壓縮版本。但是，出於我們將在接下來的課程中看到的原因，我們需要將影片轉碼為各種格式。因此，我們需要的存儲空間將超過上述估算值。

[YouTube 一年所需的總存儲空間](./storage.jpg)

```
Question
假設 YouTube 以五種不同的質量存儲影片，一分鐘影片的平均大小為 6 MB，那麼每分鐘的估計存儲需求是多少？

Answer
由於我們每分鐘需要 6 MB，並且我們需要以五種不同的格式存儲文件，因此總體需求為 5×6MB=30 MBs。

接下來，我們將計算以下值：

每分鐘上傳到 Youtube 的小時數 x 每小時的分鐘數 x 每分鐘的存儲需求 = 500 小時/分鐘 x 60 分鐘/小時 x 30 MB/分鐘 = 900 GB/分鐘
```

### Bandwidth estimation (頻寬估算)
將執行大量資料傳輸以流式傳輸和上傳影片到 YouTube。這就是為什麼我們也需要計算頻寬估算的原因。假設上傳：觀看比例為 1:300——也就是說，對於每個上傳的影片，我們每秒有 300 次影片觀看。我們還必須記住，當影片上傳時，它不是壓縮格式，而觀看的影片可以是不同的質量。讓我們估算上傳影片所需的頻寬。

我們假設：

Total_{bandwidth}: Total bandwidth required. (所需的總頻寬。)

Total_{content}: Total content (in minutes) uploaded per minute. (每分鐘上傳的總內容 (以分鐘為單位)。)

Size_{minute}: Transmission required (in MBs) for each minute of content. (每分鐘內容所需的傳輸 (以 MB為單位)。)

然後，使用以下公式進行下面的計算：

Total bandwidth (總頻寬) = Total content_transferred (傳輸的總內容) × Size minute (每分鐘大小)

                    The Bandwidth Required for Uploading Videos to YouTube (上傳影片到 YouTube 所需的頻寬)
No. of video hours per minute (每分鐘影片小時數)             	Minutes per hour (每小時分鐘數)                	MB per minute (每分鐘 MB)	Bandwidth required (Gbps) (所需頻寬 (Gbps))
500	                                               60	                           50	
```

```
我們如上所示以每秒位元數 (bps) 計算頻寬。因此，上述數字 (200 Gbps) 的詳細計算如下。

500 小時/分鐘 × 60 分鐘/小時 × 50 MB/分鐘 x 8 位元/60 秒 = 200 Gbps
```

```
Question
如果滿足上傳需求需要 200 Gbps 的頻寬，那麼流式傳輸影片需要多少頻寬？假設每分鐘影片平均需要 10 MB 的頻寬。

提示：提供了上傳：觀看比例。

Hide Answer
對於每個上傳的影片，有 300 個影片被觀看。因此，等式變為：

每分鐘觀看總小時數 × 每分鐘大小 (MB) × 觀看比例 = 500 小時/分鐘 x 60 分鐘/小時 x 10 MB/分鐘 x 300 = 90 TB/分鐘 x 8 位元 = 720 Tb/分鐘 = 12 Tbps
 [YouTube 所需的總頻寬](./bandwidth.jpg)

### Number of servers estimation (伺服器數量估算)
我們需要處理來自 5 億每日活躍使用者的並發請求。讓我們假設典型的 YouTube 伺服器每秒處理 8,000 個請求。

Number of Active Users/Queries handled per server (活躍使用者數量/每台伺服器處理的查詢) = 62,500 servers (62,500 台伺服器)

[YouTube 所需的伺服器數量](./servers.jpg)

```
注意：在現實世界的場景中，YouTube 的設計需要存儲縮略圖、使用者資料、影片元資料、使用者頻道資訊等。由於與影片文件相比，這些資料集的存儲需求並不顯著，因此為簡單起見我們忽略它。
```

## Building blocks we will use (我們將使用的構建塊)
既然我們已經完成了資源估算，讓我們確定將成為我們 YouTube 系統設計不可或缺一部分的構建塊。關鍵構建塊如下：

[高層設計中的構建塊](./bb.jpg)

- **Databases (資料庫)** 需要存儲影片、縮略圖、評論和使用者相關資訊的元資料。
- **Blob storage (Blob 儲存)** 對於存儲平台上的所有影片很重要。
- **CDN** 用於有效地向最終使用者傳遞內容，減少延遲和終端伺服器的負擔。
- **Load balancers (負載平衡器)** 是在可用伺服器池中分發數百萬傳入客戶端請求的必要條件。
除了我們的構建塊之外，我們預計在我們的高層設計中會使用以下組件：

- **Servers (伺服器)** 是運行應用邏輯和滿足使用者請求的基本要求。

- **Encoders and transcoders (編碼器和轉碼器)** 壓縮影片並將其轉換為不同的格式和質量，以根據屏幕解析度和頻寬支援不同數量的設備。

[YouTube 高層設計中的組件](./c.jpg)
