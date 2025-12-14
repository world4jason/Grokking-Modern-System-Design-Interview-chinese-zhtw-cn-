# Requirements of a Web Crawler's Design (網絡爬蟲設計要求)

## Requirements (要求)
讓我們重點介紹網絡爬蟲的功能和非功能要求。

### Functional requirements (功能要求)
這些是使用者必須能夠執行的功能：

- **Crawling (爬取)**: The system should scour the WWW, spanning from a queue of seed URLs provided initially by the system administrator.
- **爬取**：系統應搜索 WWW，從系統管理員最初提供的種子 URL 隊列開始。

- **Storing (存儲)**: The system should be able to extract and store the content of a URL in a blob store. This makes that URL and its content processable by the search engines for indexing and ranking purposes.
- **存儲**：系統應能夠提取 URL 的內容並將其存儲在 blob 存儲中。這使得搜索引擎可以處理該 URL 及其內容以進行索引和排名。

- **Scheduling (調度)**: Since crawling is a process that’s repeated, the system should have regular scheduling to update its blob stores’ records.
- **調度**：由於爬取是一個重複的過程，系統應定期調度以更新其 blob 存儲的記錄。

```
我們從哪裡獲得這些種子 URL？

有兩種創建或收集種子 URL 的可能方法：

We can manually create them. (我們可以手動創建它們。)
We can scan the IP addresses for the presence of web servers. (我們可以掃描 IP 地址以查找 Web 伺服器的存在。)
These seed URLs must be of good quality. (這些種子 URL 必須質量上乘。)
```

```
為什麼高質量的種子 URL 很重要，如果種子質量不好會發生什麼？

當我們將 WWW 建模為 URL 將一個節點鏈接到另一個節點的圖形時，我們的目標是在爬取過程中盡可能多地發現它。

使用低質量的種子可能會將發現限制在 WWW 圖的一小部分。
```

```
我們如何選擇種子 URL 進行爬取？

有多種選擇種子 URL 的方法。其中一些是：

Location-based: We can have different seed URLs depending on the location of the crawler. (基於位置：根據爬蟲的位置，我們可以有不同的種子 URL。)
Category-based: Depending on the type of content we need to crawl, we can have various sets of seed URLs. (基於類別：根據我們需要爬取的內容類型，我們可以有各種種子 URL 集。)
Popularity-based: This is the most popular approach. It combines both the aforementioned approaches. It groups the seed URLs based on hot topics in a specific area. (基於流行度：這是最流行的方法。它結​​合了上述兩種方法。它根據特定區域的熱門話題對種子 URL 進行分組。)
```

### Non-functional requirements (非功能要求)
- **Scalability (可擴展性)**: The system should inherently be distributed and multithreaded, because it has to fetch hundreds of millions of web documents.
- **可擴展性**：系統本質上應該是分佈式和多線程的，因為它必須獲取數億個 Web 文檔。

- **Extensibility (可擴展性)**: Currently, our design supports HTTP(S) communication protocol and text files storage facilities. For augmented functionality, it should also be extensible for different network communication protocols, able to add multiple modules to process, and store various file formats.
- **可擴展性**：目前，我們的設計支持 HTTP(S) 通信協議和文本文件存儲設施。為了增強功能，它還應該可以擴展用於不同的網絡通信協議，能夠添加多個模塊來處理和存儲各種文件格式。

- **Consistency (一致性)**: Since our system involves multiple crawling workers, having data consistency among all of them is necessary.
- **一致性**：由於我們的系統涉及多個爬取工作者，因此在所有工作者之間保持資料一致性是必要的。

- **Performance (性能)**: The system should be smart enough to limit its crawling to a domain, either by time spent or by the count of the visited URLs of that domain. This process is called self-throttling. The URLs crawled per second and the throughput of the content crawled should be optimal.
- **性能**：系統應該足夠智能，可以將其爬取限制在某個域，可以通過花費的時間或訪問該域的 URL 數量來限制。此過程稱為自我節流。每秒爬取的 URL 和爬取的內容吞吐量應該是最佳的。

網站通常託管一個 robot.txt 文件，該文件將域指定的限制傳達給爬蟲。爬蟲應務必遵守這些限制。
```

- **Improved user interface—customized scheduling (改進的使用者介面——自定義調度)**: Besides the default recrawling, which is a functional requirement, the system should also support the functionality to perform non-routine customized crawling on the system administrator’s demands.
- **改進的使用者介面——自定義調度**：除了默認的重新爬取 (這是一個功能要求) 外，系統還應支持根據系統管理員的需求執行非常規自定義爬取的功能。

## Resource estimation (資源估算)
我們需要估算設計的各種資源需求。

**Assumptions (假設)**
這些是我們在估算資源需求時將使用的假設：

- 總共有 50 億個網頁。
- 每個網頁的文本內容為 2070 KB。
- 每個網頁的元資料為 500 字節。

一項研究表明，基於 892 個已處理網站，網頁內容的平均大小為 2070KB (2.07MB)。
```

元資料包括網頁標題和顯示其目的的網頁描述。
```

### Storage estimation (存儲估算)
存儲 50 億個網頁的文本內容所需的集體存儲為：
每次爬取的總存儲量 = 50 億 × (2070 KB + 500 B) = 10.35 PB

[網絡爬蟲系統所需的總存儲空間](./storage.jpg)

### Traversal time (遍歷時間)
由於遍歷時間與存儲要求同樣重要，讓我們計算一次性爬取的大致時間。假設每個網頁的平均 HTTP 遍歷時間為 60 毫秒，遍歷所有 50 億頁的時間將是：
網頁遍歷是頁面大小的函數。較小的網頁花費少於 60 毫秒，較大的網頁花費更長；但是，數字 60 毫秒是平均遍歷時間。
```

總遍歷時間 = 50 億 × 60 毫秒 = 3 億秒 = 9.5 年

使用一個爬取實例遍歷整個互聯網大約需要 9.5 年，但我們希望在一天內實現我們的目標。我們可以通過設計我們的系統以支持多工作者架構並將任務分配給在不同伺服器上運行的多個工作者來實現這一點。

### Number of servers estimation for multi-worker architecture (多工作者架構的伺服器數量估算)
讓我們計算在一天內完成爬取所需的伺服器數量。假設每台伺服器只有一個工作者。

1 台伺服器完成任務所需的天數 = 9.5 年 × 365 天 ≈ 3468 天

一台伺服器需要 3,468 天才能完成任務。

我們需要多少台伺服器才能在一天內完成同樣的任務？

我們需要 3,468 台伺服器才能在一天內完成同樣的任務。

[網絡爬蟲系統所需的伺服器數量](./servers.jpg)

如果每台伺服器有 n 個線程，我們將用 3,468 除以 n。例如，如果一台伺服器能夠一次執行 10 個線程，那麼伺服器數量將減少到 3468/10 ≈ 347 台伺服器
 ```

### Bandwidth estimation (帶寬估算)
由於我們希望每天處理 10.35PB 的資料，所需的總帶寬將是：

10.35 PB / 86400 秒 ≈ 120 GB/秒 ≈ 960 Gb/秒

960Gb/sec 是所需的總帶寬。現在，假設任務平均分配給 3468 台伺服器，以便在一天內完成任務。因此，每台伺服器的帶寬將是：

(960Gb/秒) / 3468 台伺服器 ≈ 每台伺服器 277Mb/秒

[網絡爬蟲系統所需的總帶寬](./bandwidth.jpg)

讓我們試試初始假設，看看估算值在以下計算器中如何變化：

             Estimates Calculator for the Web Crawler (網絡爬蟲估算計算器)
Number of Webpages                                  5 Billion (50 億)
Text Content per Webpage                            2070 KB
Total Traversal Time on One Server                  9.5 Years (9.5 年)
Servers Required to Perform Traversal in One Day    3468 Servers (3468 台伺服器)
Bandwidth Estimate                                  958.33 Gb/sec
```

## Building blocks we will use (我們將使用的構建塊)
以下是我們將在設計中使用的主要構建塊列表：

[高級設計中的構建塊](./bb.jpg)

- **Scheduler (調度器)** is used to schedule crawling events on the URLs that are stored in its database.
- **調度器** 用於對存儲在其資料庫中的 URL 安排爬取事件。

- **DNS** 需要獲取網頁的 IP 地址解析。

- **Cache (緩存)** is utilized in storing fetched documents for quick access by all the processing modules.
- **緩存** 用於存儲獲取的文檔，以便所有處理模塊快速訪問。

- **Blob store (Blob 存儲)**’s main application is to store the crawled content.
- **Blob 存儲** 的主要應用是存儲爬取的內容。

- **HTML fetcher (HTML 獲取器)** establishes a network communication connection between the crawler and the web hosts.
- **HTML 獲取器** 在爬蟲和 Web 主機之間建立網絡通信連接。
- **Service host (服務主機)** manages the crawling operation among the workers.
- **服務主機** 管理工作者之間的爬取操作。
- **Extractor (提取器)** extracts the embedded URLs and the document from the web page.
- **提取器** 從網頁中提取嵌入的 URL 和文檔。
- **Duplicate eliminator (重複消除器)** performs dedup testing on the incoming URLs and the documents.
- **重複消除器** 對傳入的 URL 和文檔執行重複消除測試。

在下一課中，我們將重點介紹網絡爬蟲的高層和詳細設計。

[高級設計中的組件](./components.jpg)

除了這些基本的構建塊之外，我們的設計還包括一些額外的組件：
