# System Design: The Distributed Search (系統設計：分佈式搜尋)
## Why do we need a search system? (為什麼我們需要搜尋系統？)
現在，我們幾乎在每個網站上都能看到搜尋欄。我們使用該搜尋欄從該網站上提供的大量內容中挑選出相關內容。搜尋欄使我們能夠快速找到我們正在尋找的內容。例如，Educative 網站上有大量的課程。如果我們沒有搜尋功能，使用者將不得不滾動瀏覽許多頁面並閱讀每個課程的名稱才能找到他們正在尋找的課程。

讓我們再舉一個例子。即使有數十億個視頻上傳並存儲在 YouTube 上。想像一下，如果 YouTube 不為我們提供搜尋欄。我們將如何在多年來在 YouTube 上發佈的數百萬個視頻中找到特定的視頻？瀏覽所有這些視頻並找到我們需要的視頻可能需要幾個月的時間。使用者發現僅通過滾動瀏覽就很難找到他們正在尋找的內容。

[搜尋欄](./searchBar.jpg)

搜尋引擎是一個更大的例子。互聯網上有數十億個網站。每個網站都有許多網頁，每個網頁上都有大量的內容。擁有如此多的內容，如果沒有搜尋引擎，互聯網實際上將毫無用處，使用者最終會迷失在無關資料的海洋中。從本質上講，搜尋引擎是海量可用資料的過濾器。它們讓使用者能夠快速獲取真正感興趣的資訊，而無需篩選太多不必要的網頁。

在每個搜尋欄背後，都有一個搜尋系統。

## What is a search system? (什麼是搜尋系統？)
搜尋系統是一個從使用者那裡獲取一些文本輸入 (搜尋查詢) 並在幾秒鐘或更短的時間內返回相關內容的系統。搜尋系統主要由三個組件組成，即：

- 爬蟲 (Crawler)，用於獲取內容並建立文檔。
- 索引器 (Indexer)，用於構建可搜尋的索引。
- 搜尋器 (Searcher)，通過在索引器建立的索引上運行搜尋查詢來響應搜尋查詢。

```
對於搜尋引擎，文檔由從網頁中提取的文本組成。在電影商店的網頁中，文檔可以是一個 JSON 對象，其中包含我們想要執行搜尋查詢的視頻的標題、描述和其他元資料。文檔可以是 JSON 或任何其他合適的格式。文檔存儲在分佈式存儲 (如 S3 或 HDFS) 上。
```

[搜尋系統的三個階段](./3phases.jpg)

注意：我們有一個單獨的章節專門解釋爬蟲組件。在本章中，我們將重點關注索引。

## How will we design a distributed search system? (我們將如何設計分佈式搜尋系統？)
我們將分佈式搜尋系統的設計分為五個課程：

1. **Requirements (需求)**: In this lesson, we list the functional and non-functional requirements of a distributed search system. We also estimate our system’s resources, such as servers, storage, and the bandwidth needed to serve a number of queries.
1. **需求**：在本課中，我們列出了分佈式搜尋系統的功能和非功能需求。我們還估算了系統的資源，例如伺服器、存儲和服務一定數量查詢所需的帶寬。

2. **Indexing (索引)**: This lesson provides us with background knowledge on the process of indexing with the help of an example. After discussing indexing, we also look into a centralized architecture of distributed search systems.
2. **索引**：本課通過一個示例為我們提供了有關索引過程的背景知識。在討論索引之後，我們還將研究分佈式搜尋系統的集中式架構。

3. **Initial design (初步設計)**: This lesson consists of the high-level design of our system, its API, and the details of the indexing and searching process.
3. **初步設計**：本課包括我們系統的高級設計、其 API 以及索引和搜尋過程的細節。

4. **Final design (最終設計)**: In this lesson, we evaluate our previous design and revamp it to make it more scalable.
4. **最終設計**：在本課中，我們評估了之前的設計並對其進行了修改，使其更具可擴展性。

5. **Evaluation (評估)**: This lesson explains how our designed distributed search system fulfills its requirements.
5. **評估**：本課解釋了我們設計的分佈式搜尋系統如何滿足其需求。

讓我們從了解設計分佈式搜尋系統的需求開始。
