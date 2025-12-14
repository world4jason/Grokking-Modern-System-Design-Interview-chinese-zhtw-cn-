# Design of a Web Crawler (網絡爬蟲設計)

## Design (設計)
本課介紹了網絡爬蟲過程的設計和工作流程中涉及的構建塊和附加組件，以及它們的需求。

### Components (組件)
以下是我們設計所需的構建塊和組件的詳細資訊：

- **Scheduler (調度器)**: This is one of the key building blocks that schedules URLs for crawling. It’s composed of two units: a priority queue and a relational database.
- **調度器**：這是調度 URL 進行爬取的關鍵構建塊之一。它由兩個單元組成：優先級隊列和關係資料庫。

  1. **A priority queue (URL frontier) (優先級隊列 (URL 前沿))**: The queue hosts URLs that are made ready for crawling based on the two properties associated with each entry: priority and updates frequency.
  1. **優先級隊列 (URL 前沿)**：隊列託管根據與每個條目關聯的兩個屬性 (優先級和更新頻率) 準備進行爬取的 URL。

```
作為一項要求，我們需要根據內容為 URL 分配可變優先級。此屬性定義了 URL 在 URL 前沿中的優先級。
```

```
為了重新爬取，我們需要為每個 URL 定義重新爬取頻率。此屬性確保每個 URL 在 URL 前沿中有定義數量的放置。
```

  2. **Relational database (關係資料庫)**: It stores all the URLs along with the two associated parameters mentioned above. The database gets populated by new requests from the following two input streams:
  2. **關係資料庫**：它存儲所有 URL 以及上面提到的兩個關聯參數。資料庫由以下兩個輸入流的新請求填充：

  - 使用者添加的 URL，包括種子和運行時添加的 URL。
  - 爬蟲提取的 URL。

```
Question 1
我們可以估算優先級隊列的大小嗎？集中式和分佈式優先級隊列的優缺點是什麼。

Answer
假設：讓我們假設在任何時候，我們在 URL 前沿中大約有 100 萬個 URL 需要爬取。

Let’s calculate the size of the priority queue needed to store all of these URLs. (讓我們計算存儲所有這些 URL 所需的優先級隊列的大小。)
優先級隊列大小 = 100 萬個 URL × 2048 字節 = 2.048 GB

2.048 GB 對於隊列來說是一個合理的空間量，這表明我們可能不需要為 URL 前沿實施分佈式機制。

但是，集中式隊列的讀/寫帶寬有限，並且是單點故障。因此，為每個工作者設置一個子隊列將是最好的方法。

如果我們使用分佈式隊列，所有工作者如果願意，都可以從同一個隊列中獲取資料。但是擁有獨立隊列可以進一步優化爬取過程，特別是在高優先級和更頻繁爬取的情況下，例如那些需要比常規工作者更多的工作者來進行頻繁爬取的新聞網站。這種方法本質上將促進增加隊列大小的情況。

擁有單個隊列有利於冗餘鏈接的重複數據刪除，並且對整體爬蟲資源更好。我們將以另一種方式處理重新爬取優先級和頻率，如下一節所述，為此我們需要分發機制。
```

[優先級隊列](./pq1.jpg)
[優先級隊列](./pq2.jpg)

```
Question 2
我們如何在不同的工作者之間分配 URL 前沿，這將起到什麼作用？

Answer
正如之前定義的那樣，URL 前沿是調度器中使用的優先級隊列，它保存需要爬取的 URL。當我們談論它的分發時，我們的意思是獲取主機名 URL 的哈希值並將它們映射到特定的工作者。這樣，每個工作者都將擁有自己的子隊列。

這滿足了我們系統的以下兩個要求：

The workers don’t individually connect to more than one host web servers at a time. (工作者不會一次單獨連接到多個主機 Web 伺服器。)
The workers don’t overburden the host web servers with concurrent requests because we use FIFO sub-queues. (工作者不會因併發請求而使主機 Web 伺服器負擔過重，因為我們使用 FIFO 子隊列。)
```

- **DNS resolver (DNS 解析器)**: The web crawler needs a DNS resolver to map hostnames to IP addresses for HTML content fetching. Since DNS lookup is a time-consuming process, a better approach is to create a customized DNS resolver and cache frequently-used IP addresses within their time-to-live because they’re bound to change after their time-to-live.
- **DNS 解析器**：網絡爬蟲需要 DNS 解析器將主機名映射到 IP 地址以進行 HTML 內容獲取。由於 DNS 查找是一個耗時的過程，因此更好的方法是創建自定義 DNS 解析器並在其生存時間內緩存常用 IP 地址，因為它們必然會在生存時間後發生變化。

- **HTML fetcher (HTML 獲取器)**: The HTML fetcher initiates communication with the server that’s hosting the URL(s). It downloads the file content based on the underlying communication protocol. We focus mainly on the HTTP protocol for textual content, but the HTML fetcher is easily extendable to other communication protocols, as is mentioned in the section on the non-functional requirements of the web crawler.
- **HTML 獲取器**：HTML 獲取器啟動與託管 URL 的伺服器的通信。它根據底層通信協議下載文件內容。我們主要關注文本內容的 HTTP 協議，但 HTML 獲取器很容易擴展到其他通信協議，正如網絡爬蟲非功能需求部分所述。

```
Question
爬蟲如何處理具有可變優先級的 URL？

Answer
爬蟲在每個階段都必須足夠警惕，以區分 URL 的各種優先級別。

讓我們逐個階段看看爬蟲設計如何處理此類情況：

1. 由於我們將 URL 前沿實現為調度器的優先級隊列，因此它會根據參數值自動處理放置。我們選擇了容錯和周期性參數作為 URL 的優先級指標。

這些參數的分配取決於網頁內容的性質。如果是新聞網頁，一天爬取多次是合適的，並且是我們的索引保持最新所必需的。任何偶爾更新的普通網頁可能具有標準訪問頻率，比如說兩週。

2. 同樣，在 HTML 獲取器級別，爬蟲根據 robots.txt 指南與主機伺服器通信，它可以在存儲階段將獲取 URL 的相關參數值傳回給調度器。

另一種解決方案不是將指標與 URL 關聯，而是為不同的優先級設置單獨的隊列。然後我們可以根據分配給它們的優先級從這些隊列中出列。這種方法只需要將 URL 放置在相應的隊列中，不需要腳本根據關聯的額外參數進行調度。

這完全取決於我們爬取應用程式的規模。
```

- **Service host (服務主機)**: This component acts as the brain of the crawler and is composed of worker instances. For simplicity, we will refer to this whole component or a single worker as a crawler. There are three main tasks that this service host/crawler performs:
- **服務主機**：此組件充當爬蟲的大腦，由工作者實例組成。為簡單起見，我們將整個組件或單個工作者稱為爬蟲。此服務主機/爬蟲執行三個主要任務：

  1. 它處理爬取操作的多工作者架構。根據可用性，每個工作者與 URL 前沿通信以使下一個可用的 URL 出列進行爬取。
  2. 每個工作者負責從 DNS 解析器獲取傳入 URL 的 DNS 解析。
  3. 每個工作者充當調度器和 HTML 獲取器之間的網關，將必要的 DNS 解析資訊發送到 HTML 獲取器以啟動通信。

[服務主機與其他組件的交互](./service_hosts.jpg)

- **Extractor (提取器)**: Once the HTML fetcher gets the web page, the next step is to extract two things from the webpage: URLs and the content. The extractor sends the extracted URLs directly and the content with the document input stream (DIS) to the duplicate eliminator. DIS is a cache that’s used to store the extracted document, so that other components can access and process it. Over here, we can use Redis as our cache choice because of its advanced data structure functionality.
- **提取器**：一旦 HTML 獲取器獲取了網頁，下一步就是從網頁中提取兩樣東西：URL 和內容。提取器將提取的 URL 直接發送給重複消除器，並將內容隨文檔輸入流 (DIS) 發送給重複消除器。DIS 是一個緩存，用於存儲提取的文檔，以便其他組件可以訪問和處理它。在這裡，我們可以使用 Redis 作為我們的緩存選擇，因為它具有高級資料結構功能。

一旦驗證資料存儲中不存在重複項，提取器就會將 URL 發送到包含 URL 前沿的任務調度器，並將內容存儲在 blob 存儲中以進行索引。

- **Duplicate eliminator (重複消除器)**: Since the web is all interconnected, the probability of two different URLs referring to the same web page or different URLs referring to various web pages having the same content is evident. The crawler needs a component to perform a dedup test to eliminate the risk of exploiting resources by storing and processing the same content twice. The duplicate eliminator calculates the checksum value of each extracted URL and compares it against the URLs checksum data store. If found, it discards the extracted URL. Otherwise, it adds a new entry to the database with the calculated checksum value.
- **重複消除器**：由於網絡是相互連接的，兩個不同的 URL 指向同一個網頁或不同的 URL 指向俱有相同內容的不同網頁的可能性是顯而易見的。爬蟲需要一個組件來執行重複消除測試，以消除因兩次存儲和處理相同內容而利用資源的風險。重複消除器計算每個提取的 URL 的校驗和值，並將其與 URL 校驗和資料存儲進行比較。如果找到，它將丟棄提取的 URL。否則，它將使用計算出的校驗和值向資料庫添加一個新條目。

[重複消除測試在行動](./dedup.jpg)

重複消除器對提取的內容重複相同的過程，並將新網頁的校驗和值添加到文檔校驗和資料存儲中以供將來匹配。

我們為重複消除器提出的設計可以針對這兩個問題變得健壯：

1. 通過使用 URL 重定向，新 URL 可以通過 URL 重複消除測試。但是，文檔重複消除的第二階段不允許 blob 存儲中的內容重複。
2. 通過僅更改文檔中的一個字節，修改後文檔的校驗和將與原始文檔不同。

- **Blob store (Blob 存儲)**: Since a web crawler is the backbone of a search engine, storing and indexing the fetched content and relevant metadata is immensely important. The design needs to have a distributed storage, such as a blob store, because we need to store large volumes of unstructured data.
- **Blob 存儲**：由於網絡爬蟲是搜索引擎的支柱，因此存儲和索引獲取的內容和相關元資料非常重要。設計需要具有分佈式存儲，例如 blob 存儲，因為我們需要存儲大量非結構化資料。

下圖顯示了整體網絡爬蟲設計的圖示：

[整體網絡爬蟲設計](./design.jpg)

### Workflow (工作流程)
1. **Assignment to a worker (分配給工作者)**: The crawler (service host) initiates the process by loading a URL from the URL frontier’s priority queue and assigns it to the available worker.
1. **分配給工作者**：爬蟲 (服務主機) 通過從 URL 前沿的優先級隊列加載 URL 並將其分配給可用的工作者來啟動該過程。

2. **DNS resolution (DNS 解析)**: The worker sends the incoming URL for DNS resolution. Before resolving the URL, the DNS resolver checks the cache and returns the requested IP address if it’s found. Otherwise, it determines the IP address, sends it back to the worker instance of the crawler, and stores the result in the cache.
2. **DNS 解析**：工作者發送傳入的 URL 進行 DNS 解析。在解析 URL 之前，DNS 解析器檢查緩存並在找到請求的 IP 地址時返回它。否則，它確定 IP 地址，將其發送回爬蟲的工作者實例，並將結果存儲在緩存中。

```
URL 前沿的實現——從 FIFO 優先級隊列中出列 URL 並將所有提取的 URL 入列回隊列而不是一個接一個地爬取它們——確保我們以廣度優先搜索 (BFS) 而不是深度優先搜索 (DFS) 的方式爬取網絡。
```

```
Question
我們可以使用 DFS 代替 BFS 嗎？

Answer
當我們想要利用網站的持久連接來遍歷該特定域上的所有網頁時，我們可以使用 DFS。這可以節省時間，因為它可以幫助我們避免在會話過期的情況下重複重新連接到同一個網站。
```

3. **Communication initiation by the HTML fetcher (HTML 獲取器啟動通信)**: The worker forwards the URL and the associated IP address to the HTML fetcher, which initiates the communication between the crawler and the host server.
3. **HTML 獲取器啟動通信**：工作者將 URL 和關聯的 IP 地址轉發給 HTML 獲取器，HTML 獲取器啟動爬蟲和主機伺服器之間的通信。

4. **Content extraction (內容提取)**: Once the worker establishes the communication, it extracts the URLs and the HTML document from the web page and places the document in a cache for other components to process it.
4. **內容提取**：一旦工作者建立了通信，它就會從網頁中提取 URL 和 HTML 文檔，並將文檔放入緩存中以供其他組件處理。

5. **Dedup testing (重複消除測試)**: The worker sends the extracted URLs and the document for dedup testing to the duplicate eliminator. The duplicate eliminator calculates and compares the checksum of both the URL and the document with the checksum values that have already been stored.
5. **重複消除測試**：工作者將提取的 URL 和文檔發送到重複消除器進行重複消除測試。重複消除器計算並比較 URL 和文檔的校驗和與已存儲的校驗和值。

如果有匹配項，重複消除器將丟棄傳入的請求。如果沒有匹配項，它將新計算的校驗和值放入相應的資料存儲中，並允許提取器存儲內容。

6. **Content storing (內容存儲)**: The extractor sends the newly-discovered URLs to the scheduler, which stores them in the database and sets the values for the priority and recrawl frequency variables.
6. **內容存儲**：提取器將新發現的 URL 發送到調度器，調度器將它們存儲在資料庫中並設置優先級和重新爬取頻率變量的值。

提取器還將新發現文檔的所需部分 (目前在 DIS 中) 寫入資料庫。

7. **Recrawling (重新爬取)**: Once a cycle is complete, the crawler goes back to the first point and repeats the same process until the URL frontier queue is empty.
7. **重新爬取**：一旦一個週期完成，爬蟲就會回到第一點並重複相同的過程，直到 URL 前沿隊列為空。

```
注意：由於每個服務的多個實例和微服務架構，我們的設計可以使用客戶端負載平衡 (請參閱 Twitter 的客戶端負載平衡器)。
```

以下幻燈片詳細介紹了網絡爬蟲工作流程：

[工作流程](./workflow)

存儲在調度器資料庫中的 URL 分配了優先級和周期性。將新 URL 入列到 URL 前沿取決於這兩個因素。

```
Question
爬蟲需要多久重新爬取一次？

Answer
由於爬取不是一次性活動，爬蟲需要以適當的頻率規劃其默認回訪。讓我們將此回訪頻率定義為兩週。爬蟲每兩週重新訪問所有標準優先級 URL。一旦爬蟲訪問了一個 URL，它會將同一個 URL 重新附加到 URL 前沿，默認下次訪問時間等於兩週。

默認回訪時間取決於應用程式而不是系統，因此我們更改此變量以滿足我們的需求。這是一個將優先級比較器分配給 URL 的引人注目的案例。我們的系統可以使用對先前內容更改的預測分析來預測網站上的內容更改。這樣我們就可以自信地為每個 URL 分配優先級和回訪時間。例如，我們的分析可能會顯示新聞網站的頻繁更改，我們的爬蟲可能會建議每五分鐘後以高優先級重新訪問它們。
```

在下一課中，我們將探討我們設計中的一些缺點及其潛在的解決方法。
