# Requirements of Google Maps' Design (Google Maps 設計需求)
## Requirements (需求)
在開始需求之前，讓我們澄清一下，我們將通過選擇一些關鍵功能來設計一個像 Google Maps 這樣的系統，因為實際的 Google Maps 功能豐富且複雜。

讓我們列出設計中系統的功能和非功能需求。

### Functional requirements (功能需求)
我們系統的功能需求如下：

- **Identify the current location (識別當前位置)**：使用者應該能夠在世界地圖上大致確定他們的當前位置 (小數值的緯度和經度)。
- **Recommend the fastest route (推薦最快路線)**：給定起點和終點 (文字地名)，系統應根據交通類型推薦按距離和時間計算的最佳路線。
- **Give directions (給予指示)**：一旦使用者選擇了路線，系統應以文字格式列出指示，列表中每個項目引導使用者轉彎或向特定方向繼續行駛以到達目的地。

[Google Maps 功能需求](./fq.jpg)

### Non-functional requirements (非功能需求)
我們系統的非功能需求如下：

- **Availability (可用性)**：系統應具有高可用性。
- **Scalability (可擴展性)**：它應該是可擴展的，因為個人和其他企業應用程式 (如 Uber 和 Lyft) 都使用 Google Maps 來查找合適的路線。
- **Less response time (更少的響應時間)**：給定起點和終點，計算 ETA 和路線的時間不應超過兩三秒。
- **Accuracy (準確性)**：我們預測的 ETA 不應與實際行駛時間偏差太大。

[Google Maps 的非功能需求](./nfq.jpg)

注意：我們不會詳細討論如何獲取道路和佈局資料。政府機構提供地圖，在某些地方，Google 自己駕駛地圖車輛尋找道路及其交叉點。道路網路使用圖資料結構建模，其中交叉點是頂點，交叉點之間的道路是加權邊。

## Challenges (挑戰)
在設計像 Google Maps 這樣的系統時，我們需要關注的一些挑戰如下：

- **Scalability (可擴展性)**：鑑於一個擁有數十億節點和邊、跨越 194 個國家的圖，每秒為不同路線的數百萬次查詢提供服務需要強大的可擴展性措施。給定起點和終點的緯度和經度，一個簡單的方法是應用像 Dijkstra 這樣的演算法來找到起點和終點之間的最短路徑。然而，這種方法對於每秒發送數百萬次查詢的數十億使用者來說無法很好地擴展。這是因為在擁有數十億節點的圖上每秒運行一百萬次任何路徑搜尋演算法在時間和成本方面都是低效的，最終導致糟糕的使用者體驗。因此，我們的解決方案需要尋找替代技術以實現良好的擴展。

[跨越](./spanning.jpg)

- **ETA computation (ETA 計算)**：在道路空曠的理想情況下，使用我們要乘坐的車輛的距離和速度計算 ETA 很簡單。然而，我們不能忽視道路上的交通量和路況等因素，這些因素直接影響 ETA。例如，正在施工的道路、碰撞和高峰時段都可能減緩交通速度。量化上述因素以設計我們的系統並非易事。因此，我們將根據交通負載對上述因素進行分類以完成我們的設計。

[影響 ETA 計算的因素](./factors.jpg)

## Resource estimation (資源估算)
讓我們估算系統所需的伺服器總數、儲存和頻寬。

### Number of servers estimation (伺服器數量估算)
為了估算伺服器數量，我們需要知道有多少每日活躍使用者正在使用 Google Maps，以及單個 Google Maps 伺服器每秒可以處理多少請求。我們假設以下數字：

- 使用 Google Maps 的每日活躍使用者：3200 萬 (約 10 億月活躍使用者)。
- 單個伺服器每秒可以處理的請求數：8,000。
所需的伺服器數量已使用以下公式計算：

Number of active users/requests handled per server (活躍使用者數/每個伺服器處理的請求數) = 4K servers (4000 台伺服器)

[Google Maps 所需的伺服器數量](./servers.jpg)

### Storage estimation (儲存估算)
Google Maps 本質上是一個具有一次性儲存需求的系統。許多國家的道路資料已經新增，截至 2022 年已超過 20 PB。由於道路網路的變化很小，Google Maps 的每日儲存需求將可以忽略不計。此外，與完整網路資料相比，道路網路的短期變化是少量的資料。因此，我們的儲存需求不會迅速變化。

### Bandwidth estimation (頻寬估算)
作為標準做法，我們必須估算系統傳入和傳出流量所需的頻寬。Google Maps 的大部分頻寬需求來自使用者發送的請求。因此，我們設計了以下公式來計算頻寬：

Total bandwidth (總頻寬) = Total requests_second (每秒總請求數) × Total query_size (總查詢大小)

Total_{requests\_second} 代表每秒的請求數，而 Total_{query\_size} 代表每個請求的大小。

**Incoming Traffic (傳入流量)**
為了估算傳入查詢流量頻寬，我們假設以下數字：

- 單個使用者每天的最大請求數：50。
- 請求大小 (起點和終點)：200 Bytes。
使用上述假設，我們可以使用以下公式估算 Google Maps 上每秒的總請求數：

Total requests_second (每秒總請求數) = Daily active users (每日活躍使用者) × Requests per_user (每個使用者請求數) / (24 x 60 x 60) = 18,518 requests per second (每秒 18,518 個請求)。

我們可以通過在上述頻寬公式中插入上面計算的每秒請求數和每個請求的大小來計算 Google Maps 所需的傳入查詢流量頻寬。

                  Bandwidth Required for Incoming Query Traffic (傳入查詢流量所需頻寬)
No. of requests per second (每秒請求數)     Request size (Bytes) (請求大小)        	Bandwidth (Mb/s) (頻寬)
18518	                                    200                                     29.63
```

**Outgoing traffic (傳出流量)**

傳出應用程式流量將包括伺服器在使用者發出導航請求時為使用者生成的響應。響應由視覺和文字內容組成，通常包括地圖上顯示的路線、預計時間、距離以及關於路線中每個步驟的更多詳細資訊。我們假設以下數字來估算傳出流量頻寬：

- 每秒總請求數 (如上計算)：18,518。
- 響應大小：2 MB + 5 KB = 2005 KB。
  - 平均視覺資料大小：2 MB。
  - 平均文字資料大小：5 KB。
我們可以使用相同的公式計算傳出流量所需的頻寬。

          Bandwidth Required for the Outgoing Application Traffic (傳出應用程式流量所需頻寬)
No. of requests per second (每秒請求數)     Response size (KB) (響應大小) 	        Bandwidth (Gb/s) (頻寬)
18518	                                    2005                                    297.03
```

Total bandwidth (總頻寬) = Total requests_second (每秒總請求數) × Total query_size (總查詢大小) = 18518 × 2005KB × 8 bits = 297.03Gbps
```

[總結 Google Maps 的頻寬需求](./bandwidth.jpg)

## Building blocks we will use (我們將使用的構建塊)
現在我們已經完成了所需資源的估算，讓我們確定將成為 Google Maps 系統設計不可或缺部分的構建塊。下面，我們有關鍵的構建塊：

[高層和詳細設計中使用的構建塊](./bb.jpg)

- **Load balancers (負載平衡器)** 對於在不同伺服器和服務之間分配使用者請求是必要的。
- **Databases (資料庫)** 需要以圖的形式儲存資料以及元資料資訊。
- **Distributed search (分散式搜尋)** 需要在地圖上搜尋不同的地方。
- **Pub-sub system (發布-訂閱系統)** 需要在導航期間生成和響應重要事件並通知相應的服務。
- **Key-value store (鍵值儲存)** 也用於儲存一些元資料資訊。

除了上面提到的構建塊外，設計我們的地圖系統還需要其他組件。這些組件將在設計課程中討論。我們現在準備探索 Google Maps 的系統和 API 設計。
