# Requirements of Uber’s Design (Uber 的設計需求)
## Requirements (需求)
讓我們從設計像 Uber 這樣的系統的需求開始。

### Functional requirements (功能需求)
我們系統的功能需求如下：

- **Update driver location (更新司機位置)**：司機是一個移動實體，因此司機的位置應定期自動更新。

- **Find nearby drivers (尋找附近的司機)**：系統應查找並向乘客顯示附近可用的司機。

- **Request a ride (請求乘車)**：乘客應該能夠請求乘車，之後最近的司機應該收到關於乘客請求的通知。

- **Manage payments (管理支付)**：在行程開始時，系統必須啟動支付流程並管理支付。

- **Show driver estimated time of arrival (ETA) (顯示司機預計到達時間)**：乘客應該能夠看到司機的預計到達時間。

- **Confirm pickup (確認接載)**：司機應該能夠確認他們已經接載了乘客。

- **Show trip updates (顯示行程更新)**：一旦司機和乘客接受行程，他們應該能夠持續看到行程更新，例如 ETA 和當前位置，直到行程結束。

- **End the trip (結束行程)**：司機在到達目的地後將行程標記為完成，然後他們就可以進行下一次行程。

```
Question
如果兩名司機距離乘客的距離相同怎麼辦？我們將如何選擇向其發送請求的司機？

Answer
這個決定將取決於多個因素，例如距離、車輛類型、司機的等級等。儘管如此，如果兩名司機相同，我們可以隨機選擇一名並向該司機發送請求。如果一名司機在幾秒鐘內不接受乘車，我們會撤回該司機的乘車優惠並將其呈現給新司機。
```

### Non-functional requirements (非功能需求)
我們系統的非功能需求如下：

- **Availability (可用性)**：系統應該具有高可用性。即使是一小部分時間的停機也可能導致行程失敗、司機無法找到乘客或乘客無法聯繫司機。

- **Scalability (可擴展性)**：系統應該具有可擴展性，以便隨著時間的推移處理不斷增加的司機和乘客數量。

- **Reliability (可靠性)**：系統應提供快速且無錯誤的服務。乘車請求和位置更新應順利進行。

- **Consistency (一致性)**：系統必須具有強一致性。某個區域的司機和乘客應該對系統有一致的視圖。

- **Fraud detection (欺詐檢測)**：系統應具有檢測與支付相關的任何欺詐活動的能力。

## Resource estimation (資源估算)
現在，讓我們估算我們設計的資源。假設它有大約 5 億乘客和約 500 萬司機。我們將假設以下數字進行估算：

- 我們有 2000 萬每日活躍乘客和 300 萬每日活躍司機。
- 我們有 2000 萬次每日行程。
- 所有活躍司機每四秒發送一次當前位置的通知。

### Storage estimation (儲存估算)
讓我們估算我們系統所需的儲存：

### Rider’s metadata (乘客元資料)
假設我們需要大約 1,000 個位元組來儲存每個乘客的資訊，包括 ID、姓名、電子郵件等，當乘客在我們的應用程式中註冊時。要儲存 5 億乘客，我們需要 500 GB 的存儲空間：

500 x 10^6 x 1000 = 500 GB

此外，如果我們每天有大約 500,000 名新乘客註冊，我們將需要額外的 500 MB 來儲存他們。

### Driver’s metadata (司機元資料)
假設我們需要大約 1,000 個位元組來儲存每個司機的資訊，包括 ID、姓名、電子郵件、車輛類型等，當司機在我們的應用程式中註冊時。要儲存 500 萬司機，我們需要 5 GB 的存儲空間：

5 x 10^6 x 1000 = 5 GB

此外，如果我們每天有大約 100,000 名新司機註冊，我們將需要大約 100 MB 來儲存他們。

### Driver location metadata (司機位置元資料)
假設我們需要大約 36 個位元組來儲存司機的位置更新。如果我們有 500 萬司機，我們僅需大約 180 MB 的儲存空間來儲存司機的位置。

### Trip metadata (行程元資料)
假設我們需要大約 100 個位元組來儲存單個行程資訊，包括行程 ID、乘客 ID、司機 ID 等。如果我們每天有 2000 萬次行程，我們需要大約 2 GB 的儲存空間來儲存行程資料。

讓我們計算 Uber 一天所需的總儲存空間：

                             Storage Capacity Estimation (儲存容量估算)
Number of drivers (millions) (司機數量 (百萬))                                        5
Storage required to store a driver's location (Bytes) (儲存司機位置所需的儲存 (位元組))      36
Total storage required to store drivers’ locations (MB per day) (儲存司機位置所需的總儲存 (MB/天)) 180
Number of trips (millions) (行程數量 (百萬))                                         20
Storage required to store a trip (Bytes) (儲存行程所需的儲存 (位元組))                     100
Total storage required to store trips (GB per day) (儲存行程所需的總儲存 (GB/天))         2
Storage required for new riders daily (MB per day) (新乘客每天所需的儲存 (MB/天))         500
Storage required for new drivers daily (MB per day) (新司機每天所需的儲存 (MB/天))        100
Total storage (GB per day) (總儲存 (GB/天))                                           2.78
```

[Uber 一年所需的總儲存空間](./storage.jpg)

```
注意：我們可以調整表中的值以查看估算如何變化。
```

### Bandwidth estimation (頻寬估算)
我們將僅考慮司機的位置更新和行程資料進行頻寬計算，因為其他因素不需要大量頻寬。我們每天有 2000 萬次行程，這意味著我們每秒大約有 232 次行程。

20000000/86400 ≈ 232 trips per second (每秒行程數)

現在，每次行程大約需要 100 個位元組。因此，它每秒大約需要 23 KB 的頻寬。

232 x 100 = 23 KB x 8 = 185 kbps

如前所述，司機的位置每四秒更新一次。如果我們收到司機的 ID (3 個位元組) 和位置 (16 個位元組)，我們的應用程式將佔用以下頻寬：

3M active drivers x (3+16)B = 57MB x 8 = 456/4 = 114 Mbps (114 兆位/秒)

這些頻寬要求適中，因為我們沒有包括地圖和真實 Uber 服務中存在的其他組件的頻寬需求。

                       Bandwidth Requirements (頻寬需求)
Trips per second (每秒行程數)                                     232
Bandwidth for each trip (Bytes) (每次行程的頻寬 (位元組))             100
Total bandwidth for trips (kilo bits per second) (行程總頻寬 (Kbps)) 185.6
Active drivers (millions) (活躍司機 (百萬))                         3
Bandwidth for each driver (Bytes) (每個司機的頻寬 (位元組))             19
Bandwidth for drivers (Megabits per second) (司機頻寬 (Mbps))       114
Total bandwidth (Megabits per second) (總頻寬 (Mbps))               114.19
```

[Uber 所需的總頻寬](./bandwidth.jpg)

```
注意：我們可以調整表中的值以查看需求如何變化。
我們忽略了從 Uber 服務到使用者的頻寬，因為它非常小。從 Uber 服務向使用者發送地圖資料將需要更多頻寬，我們在 Google Maps 章節中也討論過這一點。

### Number of servers estimation (伺服器數量估算)
我們需要處理來自 2000 萬每日活躍使用者的併發請求。我們將使用以下公式來估算務實的伺服器數量。我們在封底計算一章中建立了此公式：

Number of daily active users/RPS of a server (每日活躍使用者數/伺服器的 RPS) = 20 x 10^6/8000 = 2500

[Uber 服務所需的伺服器數量](./servers.jpg)

                 Estimating the Number of Servers (估算伺服器數量)
Daily active users (millions) (每日活躍使用者 (百萬))   20
RPS of a server (伺服器的 RPS)                      8000
Number of servers required (所需的伺服器數量)         2500
```

```
注意：我們可以調整表中的值以查看估算如何變化。
```


## Building blocks we will use (我們將使用的構建塊)
Uber 的設計利用了以下構建塊：

[Uber 高層設計中的構建塊](./bb.jpg)

- **Databases (資料庫)** 儲存乘客、司機和行程的元資料。
- **Cache (快取)** 儲存最常請求的資料以進行快速響應。
- **CDNs** 用於有效地向最終使用者提供內容，減少延遲和終端伺服器的負擔。
- **Load balancer (負載平衡器)** 在相應的服務之間分發讀/寫請求。
- **Riders’ and drivers’ devices (乘客和司機的設備)** 應具有足夠的頻寬和 GPS 設備，以便使用地圖進行順暢導航。

```
注意：本章提供的資訊靈感來自 Uber 的工程部落格。
```
