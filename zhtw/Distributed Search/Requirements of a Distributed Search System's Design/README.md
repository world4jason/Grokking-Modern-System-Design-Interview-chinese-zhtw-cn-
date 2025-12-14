# Requirements of a Distributed Search System's Design (分佈式搜尋系統設計的需求)

## Requirements (需求)
讓我們了解分佈式搜尋系統的功能和非功能需求。

### Functional requirements (功能需求)
以下是分佈式搜尋系統的功能需求：

- **Search (搜尋)**: Users should get relevant content based on their search queries.
- **搜尋**：使用者應根據其搜尋查詢獲得相關內容。

[分佈式搜尋系統的功能需求](./fr.jpg)

### Non-functional requirements (非功能需求)
以下是分佈式搜尋系統的非功能需求：

- **Availability (可用性)**: The system should be highly available to the users.
- **可用性**：系統應對使用者高度可用。
- **Scalability (可擴展性)**: The system should have the ability to scale with the increasing amount of data. In other words, it should be able to index a large amount of data.
- **可擴展性**：系統應有能力隨著資料量的增加而擴展。換句話說，它應該能夠索引大量資料。
- **Fast search on big data (大數據上的快速搜尋)**: The user should get the results quickly, no matter how much content they are searching.
- **大數據上的快速搜尋**：無論使用者搜尋多少內容，都應快速獲得結果。
- **Reduced cost (降低成本)**: The overall cost of building a search system should be less.
- **降低成本**：構建搜尋系統的總成本應該較低。

[分佈式搜尋系統的非功能需求](./nfr.jpg)

## Resource estimation (資源估算)
讓我們估算分佈式搜尋系統所需的伺服器、存儲和帶寬總數。我們將使用 YouTube 搜尋的示例來計算這些數字。

### Number of servers estimation (伺服器數量估算)
為了估算伺服器數量，我們需要知道每天有多少每日活躍使用者 (Daily Active Users, DAU) 使用 YouTube 上的搜尋功能，以及我們的單個伺服器每秒可以處理多少請求。我們假設以下數字：

- 使用搜尋功能的每日活躍使用者數量為 300 萬。
- 單個伺服器可以處理的請求數為 1,000。

所需的伺服器數量使用以下公式計算：

活躍使用者數量 / 每個伺服器處理的查詢數 = 3K 伺服器

如果有 300 萬使用者同時搜尋，則同時生成 300 萬個搜尋請求。單個伺服器一次處理 1,000 個請求。將 300 萬除以 1,000 得到 3,000 台伺服器。

[YouTube 搜尋服務所需的伺服器數量](./number_of_servers.jpg)

### Storage estimation (存儲估算)
每個視頻的元資料存儲在單獨的 JSON 文檔中。每個文檔都由視頻 ID 唯一標識。此元資料包含視頻的標題、描述、頻道名稱和腳本。我們假設以下數字來估算索引一個視頻所需的存儲空間：

- 單個 JSON 文檔的大小為 200 KB。
- 從單個 JSON 文檔中提取的唯一術語或鍵的數量為 1,000。
- 將一個術語添加到索引表所需的存儲空間量為 100 位元組。

以下公式用於計算索引一個視頻所需的存儲空間：

總存儲/視頻 = 存儲/文檔 + (術語/文檔 × 存儲/術語)

                              Total Storage Required to Index One Video on YouTube (YouTube 上索引一個視頻所需的總存儲空間)
每個 JSON 文檔的存儲 (KB)            每個文檔的術語數                     每個術語的存儲 (Bytes)                  每個視頻的總存儲 (KB)
200                                      1000                                100                                     300
```                             

在上表中，我們計算了索引一個視頻所需的存儲空間。我們已經看到每個視頻所需的總存儲空間為 300 KB。假設 YouTube 上平均每天上傳的視頻數量為 6,000 個，讓我們計算一下索引每天上傳的視頻所需的總存儲空間。以下公式用於計算索引一天內上傳到 YouTube 的視頻所需的存儲空間：

總存儲/天 = 視頻數/天 × 總存儲/視頻

                                 Total Storage Required to Index Videos per Day on YouTube (YouTube 上每天索引視頻所需的總存儲空間)
每天的視頻數                 每個視頻的總存儲 (KB)                        每天的總存儲 (GB)
6000                                   300                                      1.8
在 YouTube 上索引每天上傳的 6,000 個視頻所需的總存儲空間為 1.8 GB。這個存儲需求只是對 YouTube 的估算。如果我們為多個租戶提供分佈式搜尋系統作為服務，存儲需求將會增加。

[總結 YouTube 每天上傳視頻的分佈式搜尋系統存儲需求](./daily_storage.jpg)



### Bandwidth estimation (帶寬估算)
每次搜尋請求都在使用者和伺服器之間傳輸資料。我們估算伺服器上的傳入流量和從伺服器輸出的流量所需的帶寬。這是計算所需帶寬的公式：

總帶寬 = 總請求_秒 × 總查詢_大小

**Incoming traffic (傳入流量)**

為了估算傳入流量帶寬，我們假設以下數字：

- 每天的搜尋請求數為 1.5 億。
- 搜尋查詢大小為 100 位元組。

我們可以使用上面給出的公式來計算傳入流量所需的帶寬。

                     Bandwidth Required for Incoming Search Queries per Second (每秒傳入搜尋查詢所需的帶寬)
每秒請求數                                查詢大小 (Bytes)                   帶寬 (Mb/s)
1736.11	                                      100	                         1.39

```

**Outgoing traffic (傳出流量)**

傳出流量是伺服器根據搜尋請求返回給使用者的響應。我們假設針對搜尋查詢的建議視頻數量為 80，一個建議的大小為 50 位元組。建議由視頻 ID 的有序列表組成。

為了估算傳出流量帶寬，我們假設以下數字：

- 每天的搜尋請求數為 1.5 億。
- 響應大小為 4,000 位元組。

我們可以使用相同的公式來計算傳出流量所需的帶寬。

               Bandwidth Required for Outgoing Traffic per Second (每秒傳出流量所需的帶寬)
每秒請求數                        查詢大小 (Bytes)                   帶寬 (Mb/s)
1736.11	                              4000	                              55.56
```               

[總結視頻搜尋的帶寬需求](./bandwidth.jpg)

注意：帶寬需求相對較低，因為我們假設是文本結果。許多搜尋服務可以返回小縮略圖和其他媒體來增強搜尋頁面。每頁面帶寬需求有意降低，以便服務可以向客戶端提供近乎實時的結果。
```


## Building blocks we will use (我們將使用的構建模組)
我們的設計中需要分佈式存儲。因此，我們可以使用 blob 存儲 (之前討論過的構建模組) 來存儲要索引的資料和索引本身。我們將使用通用術語，即“分佈式存儲”，而不是特定術語“blob 存儲”。

總之，我們解釋了搜尋系統的需求是什麼。我們進行了資源估算。最後，我們提到了我們將在分佈式搜尋系統設計中使用的構建模組。

[分佈式存儲：Blob 存儲](./bb.jpg)
