# Design Improvements of a Web Crawler (網絡爬蟲的設計改進)

## Introduction (簡介)
本課詳細介紹了增強我們網絡爬蟲設計的功能、性能和安全性所需的改進。我們將本課分為兩部分：

1. 功能和性能增強設計改進——可擴展性和多工作者架構。
2. 安全增強設計改進——爬蟲陷阱。

讓我們深入研究這些部分。

## Design improvements (設計改進)
我們目前的設計過於簡單，存在一些固有的缺點和挑戰。讓我們一一列舉，並在此過程中對我們的設計進行一些調整。

- **Shortcoming (缺點)**: Currently, our design supports the HTTP protocol and only extracts textual content. This leads to the question of how we can extend our crawler to facilitate multiple communication protocols and extract various types of files.
- **缺點**：目前，我們的設計支持 HTTP 協議，並且僅提取文本內容。這引出了一個問題，即我們如何擴展我們的爬蟲以支持多種通信協議並提取各種類型的文件。

- **Adjustment (調整)**: Since we have two separate components for serving communication handling and extracting, HTML Fetcher and Extractor, let’s discuss their modifications one by one.
- **調整**：由於我們有兩個獨立的組件分別用於服務通信處理和提取，即 HTML 獲取器和提取器，讓我們一一討論它們的修改。

1. **HTML Fetcher (HTML 獲取器)**: We have only discussed the HTTP module in this component so far because of the widely-used HTTP URLs scheme. We can easily extend our design to incorporate other communication protocols like File Transfer Protocol (FTP). The workflow will then have an intermediary step where the crawler invokes the concerned communication module based on the URL’s scheme. The subsequent steps will remain the same.
1. **HTML 獲取器**：到目前為止，由於廣泛使用的 HTTP URL 方案，我們僅討論了此組件中的 HTTP 模塊。我們可以輕鬆擴展我們的設計以包含其他通信協議，如文件傳輸協議 (FTP)。然後，工作流程將有一個中間步驟，爬蟲根據 URL 的方案調用相關的通信模塊。後續步驟將保持不變。

2. **Extractor (提取器)**: Currently, we only extract the textual content from the downloaded document placed in the Document Input Stream (DIS). This document contains other file types as well, for example, images and videos. If we wish to extract other content from the stored document, we need to add new modules with functionalities to process those media types. Since we use a blob store for content storage, storing the newly-extracted content comprising text, images, and videos won’t be a problem.
2. **提取器**：目前，我們僅從放置在文檔輸入流 (DIS) 中的下載文檔中提取文本內容。該文檔還包含其他文件類型，例如圖像和視頻。如果我們希望從存儲的文檔中提取其他內容，我們需要添加具有處理這些媒體類型功能的新模塊。由於我們使用 blob 存儲進行內容存儲，因此存儲包含文本、圖像和視頻的新提取內容將不成問題。

[HTML 獲取器和提取器的可擴展性](./extensibility.jpg)

- **Shortcoming (缺點)**: The current design doesn’t explain how the multi-worker concept integrates into this system.
- **缺點**：目前的設計沒有解釋多工作者概念如何集成到該系統中。

- **Adjustment (調整)**: Every worker needs a URL to work on from the priority queue. Different websites require different times for workers to finish crawling, so each worker will dequeue a new URL upon its availability.
- **調整**：每個工作者都需要從優先級隊列中獲取一個 URL 來處理。不同的網站需要不同的時間讓工作者完成爬取，因此每個工作者將在可用時將新 URL 出列。

有多種方法可以為我們的系統實現這種多工作者架構。其中一些如下：

1. **Domain level log (域級別日誌)**: The crawler assigns one whole domain to a worker. All the URLs branching out of the initial URL are the responsibility of the same crawler. We ensure this by caching the hash of the hostname against the worker ID for guaranteed future assignment to the same worker. This also helps us avoid any redundant crawling of that domain’s web pages by any worker.
1. **域級別日誌**：爬蟲將整個域分配給一個工作者。從初始 URL 分支出的所有 URL 均由同一個爬蟲負責。我們通過緩存主機名針對工作者 ID 的哈希值來確保這一點，以確保將來分配給同一工作者。這也有助於我們避免任何工作者對該域的網頁進行任何冗餘爬取。

這種方法最適合實現反向 URL 索引，這涉及以相反順序遍歷 Web 地址目錄。它確保了 URL 存儲過程的存儲效率以供以後使用，並防止了重複消除測試的大量重複字符串匹配過程。

2. **Range division (範圍劃分)**: The range of URLs given to each crawler; the crawler distributes a range of URLs from the priority queue among workers to avoid clashes. Like the previous approach, the crawler must hash the range associated with each worker.
2. **範圍劃分**：給定給每個爬蟲的 URL 範圍；爬蟲將優先級隊列中的 URL 範圍分配給工作者以避免衝突。與前一種方法一樣，爬蟲必須哈希與每個工作者關聯的範圍。

3. **Per URL crawling (逐個 URL 爬取)**: Queue all the URLs; a worker takes a URL and enqueues the subsequently found URLs into the priority queue. Those new URLs are readily available to crawl through for other workers. To avoid enqueuing multiple similar links that direct to the same web page, we calculate the checksum of the canonicalized URL.
3. **逐個 URL 爬取**：將所有 URL 排隊；工作者獲取一個 URL，並將隨後找到的 URL 入列到優先級隊列中。這些新 URL 隨時可供其他工作者爬取。為了避免入列指向同一網頁的多個相似鏈接，我們計算規範化 URL 的校驗和。

## Crawler traps (爬蟲陷阱)
爬蟲陷阱是一個 URL 或一組 URL，會導致無限的爬蟲資源耗盡。本節致力於爬蟲陷阱的分類、識別和預防。

### Classification (分類)
爬蟲陷阱可以有多種分類方案，但讓我們根據 URL 方案對它們進行分類。

大多數情況下，網絡爬蟲陷阱是網站結構不佳的結果，例如：

- **URLs with query parameters (帶有查詢參數的 URL)**: These query parameters can hold an immense amount of values, all while generating a large number of useless web pages for a single domain: HTTP://www.abc.com?query.
- **帶有查詢參數的 URL**：這些查詢參數可以包含大量的值，同時為單個域生成大量無用的網頁：HTTP://www.abc.com?query。

- **URLs with internal links (帶有內部鏈接的 URL)**: These links redirect in the same domain and can create an infinite cycle of redirection between the web pages of a single domain, making the crawler crawl the same content over and over again.
- **帶有內部鏈接的 URL**：這些鏈接在同一個域重定向，並可以在單個域的網頁之間創建無限的重定向循環，使爬蟲一遍又一遍地爬取相同的內容。

- **URLs with infinite calendar pages (帶有無限日曆頁面的 URL)**: These have never-ending combinations of web pages based on the varying date values, and can create a large number of pointless web pages for a single domain.
- **帶有無限日曆頁面的 URL**：這些 URL 具有基於不同日期值的永無止境的網頁組合，並且可以為單個域生成大量無意義的網頁。

- **URLs for the dynamic content generation (用於動態內容生成的 URL)**: These are query-based and can generate a massive number of web pages based on the dynamic content resulting from these queries. Such URLs might become a never-ending crawl on a single domain.
- **用於動態內容生成的 URL**：這些是基於查詢的，並且可以根據這些查詢產生的動態內容生成大量網頁。此類 URL 可能會成為單個域上永無止境的爬取。

- **URLs with repeated/cyclic directories (帶有重複/循環目錄的 URL)**: They form an indefinite loop of redirections. For example, HTTP://www.abc.com/first/second/first/second/....
- **帶有重複/循環目錄的 URL**：它們形成無限的重定向循環。例如，HTTP://www.abc.com/first/second/first/second/....

[網絡爬蟲陷阱的分類](./classification.jpg)

大多數情況下，爬蟲陷阱是由於網站結構不佳而無意造成的。有趣的是，爬蟲陷阱也可以被故意放置，以動態生成無窮無盡的網頁系列，其純粹目的是耗盡爬蟲的帶寬。這些陷阱嚴重影響爬蟲的吞吐量並妨礙其性能。
```
吞吐量：在一段時間內爬取的有用內容
```

這些爬蟲陷阱可能對爬蟲有害，但它們也會嚴重影響網站的 SEO 排名。

### Identification (識別)
對爬蟲陷阱進行分類以正確識別它們並相應地考慮設計調整至關重要。

識別爬蟲陷阱的過程主要有兩層：

1. **Analyzing the URL scheme (分析 URL 方案)**: The URLs with a poor URL structure, for example, those with cyclic directories: HTTP://www.abc.com/first/second/first/second/... will create crawler traps. Hence, filtering such URLs beforehand will rescue our crawler resources.
1. **分析 URL 方案**：URL 結構較差的 URL (例如，具有循環目錄的 URL：HTTP://www.abc.com/first/second/first/second/...) 將創建爬蟲陷阱。因此，預先過濾此類 URL 將挽救我們的爬蟲資源。

2. **Analyzing the total number of web pages against a domain (分析針對某個域的網頁總數)**: An impossibly large number of web pages against a domain in the URL frontier is a strong indicator of a crawler trap. So, limiting the crawl at such a domain will be of utmost importance.
2. **分析針對某個域的網頁總數**：URL 前沿中針對某個域的網頁數量大得不可思議是爬蟲陷阱的有力指標。因此，限制對該域的爬取將至關重要。

[網絡爬蟲陷阱的識別過程](./id.jpg)

### Solution (解決方案)
我們的設計缺乏關於負責任的爬取機制的詳細資訊，無法避免上述識別出的爬蟲陷阱。

爬取是一項資源和時間消耗型任務，有效避免爬蟲陷阱以實現及時和有用的爬取至關重要。一旦爬蟲開始與伺服器通信以下載網頁內容，它就需要考慮多個因素，主要是在 HTML 獲取器級別。讓我們逐一探討這些因素。

1. 爬蟲必須實現應用層邏輯來對抗爬蟲陷阱。此邏輯可能基於觀察到的網頁數量超過指定閾值。

爬蟲必須足夠智能，能夠在有限的時間或網頁訪問後限制其在特定域的爬取。爬蟲應該巧妙地退出網頁並將該 URL 存儲為未來遍歷的禁區，以確保性能有效性。

2. 在啟動與 Web 伺服器的通信時，爬蟲需要獲取一個名為 robots.txt 的文件。此文件包含網站管理員列出的爬蟲注意事項。遵守此文檔是爬取過程不可或缺的一部分。它還允許爬蟲訪問某些優先進行爬取的域指定網頁，而不限制其訪問特定網頁。

本文檔的另一個基本組成部分是針對爬蟲的回訪頻率說明。熱門網站可能需要頻繁回訪，而內容很少更新的網站則相反。網站與網絡爬蟲通信的這種標準稱為機器人排除協議。該協議可防止爬蟲在不可爬取的網頁上不必要地花費爬取資源。

[機器人排除協議](./robots.txt.jpg)

```
robots.txt 文件並不能保護爬蟲免受惡意或故意的爬蟲陷阱的侵害。其他解釋的機制必須處理這些陷阱。
```

3. 由於每個域都有有限的傳入和傳出帶寬分配，因此爬蟲需要足夠禮貌以限制其在特定域的爬取。與其為每個域設置靜態爬取速度，不如根據域的首字節時間 (TTFB) 值調整爬取速度。TTFB 值越高，伺服器越慢。因此，過快地爬取該域可能會導致更多超時請求和不完整的爬取。

設計中的這些修改將確保爬蟲能夠避免爬蟲陷阱，從而優化資源的使用。
