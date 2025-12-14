# Requirements of Quora's Design (Quora 的設計需求)
## Requirements (需求)
讓我們了解以下功能和非功能需求：

### Functional requirements (功能需求)
使用者應該能夠執行以下功能：

- **Questions and answers (問題與回答)**：使用者可以提問和回答問題。問題和答案可以包含圖片和影片。
- **Upvote/downvote and comment (贊成/反對和評論)**：使用者可以對答案進行贊成、反對和評論。
- **Search (搜尋)**：使用者應該有一個搜尋功能來尋找其他使用者已經在平台上提出的問題。
- **Recommendation system (推薦系統)**：使用者可以查看他們的動態消息，其中包括他們感興趣的主題。動態消息還可以包括需要回答的問題或讀者感興趣的答案。系統應利用推薦系統促進使用者發現。
- **Ranking answers (回答排名)**：我們根據回答的有用性對回答進行排名，從而增強使用者體驗。最有用的答案將排在最高位置並列在頂部。

[Quora 的功能和非功能需求](./quora.jpg)

### Non-functional requirements (非功能需求)
- **Scalability (可擴展性)**：隨著功能和使用者數量隨時間增長，系統應該具有良好的可擴展性。這意味著效能和可用性不應受到使用者數量增加的影響。
- **Consistency (一致性)**：設計應確保不同使用者對同一內容的看法是一致的。特別是，對於任何觀眾群體，像問題和答案這樣的關鍵內容應該是相同的。但是，Quora 的所有使用者不一定要立即看到新發布的問題、答案或評論。
- **Availability (可用性)**：系統應具有高可用性。這適用於伺服器收到大量併發請求的情況。
- **Performance (效能)**：系統應為使用者提供流暢的體驗，而沒有明顯的延遲。

## Resource estimation (資源估算)
在本節中，我們將估算 Quora 服務的資源需求。我們將做出假設以獲得實際且易於處理的估算。我們將估算為大量使用者提供服務所需的伺服器數量、儲存和頻寬。

**Assumptions (假設)**：將我們的估算建立在一些基本假設之上非常重要。因此，我們假設如下：

- 共有 10 億使用者，其中 3 億是每日活躍使用者。
- 假設 15% 的問題有圖片，5% 的問題嵌入了影片。一個問題不能同時包含兩者。
- 我們假設一張圖片估計為 250 KB，一個影片被認為是 5 MB。

### Number of servers estimation (伺服器數量估算)
讓我們估算我們設計的每秒請求數 (RPS)。如果平均有 3 億每日活躍使用者，每個使用者每天可以產生 20 個請求，那麼一天的總請求數將是：

300 x 10^6 x 20 = 6 x 10^9

因此，RPS = 6 x 10^9/86400 = 大約每秒 69500 個請求。

                    Estimating RPS (估算 RPS)

Daily active users (每日活躍使用者)	300	million
Requests per day per user (每個使用者每天的請求數)	20
Requests Per Second (RPS) (每秒請求數)	69444
```

我們已經在封底計算章節中確定，我們將使用以下公式來估算實際的伺服器數量：

(每日活躍使用者數 / 伺服器的 RPS = 300 x 10^6/8000 = 37500)

[Quora 所需的估計伺服器數量](./servers.jpg)

```
因此，為 3 億使用者 (平均每秒產生 69,500 個請求) 提供服務所需的伺服器總數將為 37,500 台。
```

### Storage estimation (儲存估算)
讓我們記住我們的假設，即 15% 的問題有圖片，5% 有影片。所以，我們將做出以下假設來估算我們設計的儲存需求：

- 3 億活躍使用者每人每天發布 1 個問題，每個問題平均有 2 個回覆，總共有 10 個贊成票和 5 條評論。
- 一個問題的文字內容所需的集體儲存等於 1 KB。

    Storage Requirements Estimation Calculator (儲存需求估算計算器)
             
Questions per user (每個使用者的問題數)                    	1	per day (每天)

Total questions per day (每天總問題數)	                300	millions (百萬)

Size of textual content per question (每個問題的文字內容大小)  	1	KB

Image size (圖片大小)                            	250	KB

Video size (影片大小)	                            5 	MB

Questions containing images (包含圖片的問題)	            15	percent (百分比)

Questions containing videos (包含影片的問題)           	5	percent (百分比)

Storage for textual content (文字內容儲存)	            0.3	TB

Storage for image content (圖片內容儲存)	              11.25	TB

Storage for video content (影片內容儲存)	              75	TB                           

```

The following are the default calculations (以下是預設計算):

- Total questions (總問題數): 300M×1=300×10^6 questions per day (每天 300×10^6 個問題).
- Storage required for textual content of all questions in one day (一天內所有問題文字內容所需儲存): 300×M×1×KB=300GB
- Storage required for images for one day (一天內圖片所需儲存): (300 x 10^6 x 15)/100 x 250 x 10^3 B = 11.25TB
- Storage required for video content for one day (一天內影片內容所需儲存): (300 x 10^6 x 15)/100 x 5 x 10^6 B= 75TB
Total storage required for one day (一天所需的總儲存量) = 0.3TB + 11.25TB + 75TB ~= 86.55TB per day

[總結 Quora 的儲存需求](./storage.jpg)

Quora 的每日儲存需求似乎很高。但對於擁有 3 億 DAU 的服務，每年的需求為 86.55TB * 365 = 31.6PB 是可行的。實際需求將會更高，因為我們忽略了許多事項所需的儲存。例如，非活躍 (10 億中) 使用者的資料將需要儲存。

### Bandwidth estimation (頻寬估算)
頻寬估算需要計算通過網路的傳入和傳出資料。

- **Incoming traffic (傳入流量)**：每天所需的傳入流量頻寬等於 86.55TB /86400 * 8 = 8Gbps
- **Outgoing traffic (傳出流量)**：我們假設 3 億活躍使用者每天查看 20 個問題，因此總頻寬需求可以在下面的計算器中找到：

    Bandwidth Requirements Estimation Calculator (頻寬需求估算計算器)
Total storage required per day (每天所需總儲存)	          86.55	TB

Incoming traffic bandwidth (傳入流量頻寬)	              8	Gbps

Questions viewed per user (每個使用者查看的問題數)	                20	per day (每天)

Total questions viewed (查看的總問題數)	                  69444	per second (每秒)

Bandwidth for text of all questions (所有問題文字的頻寬)	      0.56	Gbps

Bandwidth for 15% of image content (15% 圖片內容的頻寬)	      20.83	Gbps

Bandwidth for 5% of video content (5% 影片內容的頻寬)	        138.89	Gbps

Outgoing traffic bandwidth (傳出流量頻寬)	              160.3	Gbps                     
```

300M * 20 questions = 6 billion questions are viewed per day (每天查看 60 億個問題).
- Questions viewed per second (每秒查看的問題數): 6 x 10^9/86400 ~= 69.4 x 10^3 questions are viewed per second.
- Bandwidth for the textual content of all questions and their answers (所有問題及其答案的文字內容頻寬): 69.4 x 10^3 x 1 x 10^3 x 8 bits = 0.56 Gbps
- Bandwidth of the 15% of content which contain images per second (每秒包含圖片的 15% 內容的頻寬): 69.4 x 10^3 x 15/100 x 250 x 10^3 x 8 bits = 20.82Gbps
- Bandwidth for the 5% of content that contains video per second (每秒包含影片的 5% 內容的頻寬): 69.4 x 10^3 x 5/100 x 5 x 10^6 x 8 bits = 138.9Gbps
- Total outgoing traffic bandwidth (總傳出流量頻寬): 0.56Gbps + 20.82Gbps + 138.9Gbps = 160.3Gbps
We use rounding at each step in this explanation. The answers in the calculator above are slightly different due to rounding. (我們在本解釋的每一步都使用四捨五入。上面計算器中的答案由於四捨五入略有不同。)
```

[總結 Quora 的頻寬需求](./bandwidth.jpg)

Quora 的總頻寬需求等於：

Incoming + outgoing traffic bandwidth = 8Gbps + 160.3Gbps = 168.3Gbps (傳入 + 傳出流量頻寬 = 8Gbps + 160.3Gbps = 168.3Gbps)
```


## Building blocks we will use (我們將使用的構建塊)
我們將使用以下構建塊進行 Quora 的初步設計：

[我們設計所需的構建塊]

- **Load balancers (負載平衡器)** 將用於在服務主機之間分配流量負載。
- **Databases (資料庫)** 對於儲存各種資料至關重要，例如使用者問題和答案、評論以及贊成和反對。此外，使用者資料將儲存在資料庫中。我們可以使用不同類型的資料庫來儲存不同的資料。
- **Distributed caching system (分散式快取系統)** 將用於儲存頻繁訪問的資料。我們還可以使用快取來儲存不同問題的觀看計數器。
- **Blob store (Blob 儲存)** 將保存圖片和影片檔案。
