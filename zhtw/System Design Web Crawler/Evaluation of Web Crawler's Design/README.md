# Evaluation of Web Crawler's Design (網絡爬蟲設計評估)

## Reviewing design requirements (審查設計要求)
讓我們評估一下我們的設計如何滿足擬議系統的非功能性需求。

### Scalability (可擴展性)
我們的設計表明，水平擴展我們的系統至關重要。因此，擬議的設計採用了以下設計選擇來滿足可擴展性要求：

- 該系統具有可擴展性，可以處理不斷增加的 URL。它包括所有必需的資源，包括調度器、網絡爬蟲工作者、HTML 獲取器、提取器和 blob 存儲，這些資源可以按需添加/刪除。

- 在分佈式 URL 前沿的情況下，系統利用一致性哈希在各種爬取工作者之間分配主機名，其中每個工作者都在伺服器上運行。這樣，添加或刪除爬蟲伺服器就不成問題了。

### Extensibility and modularity (可擴展性和模塊化)
到目前為止，我們的設計只關注一種特定類型的通信協議：HTTP。但根據我們的非功能性需求，我們系統的設計應該促進其他網絡通信協議 (如 FTP) 的包含。

為了實現這種可擴展性，我們只需在 HTML 獲取器中為新需要的通信協議添加額外的模塊。然後，各自的模塊將負責與主機伺服器進行和維護所需的通信。

沿著同樣的思路，我們希望我們的設計也能擴展其對其他 MIME類型的功能。針對不同 MIME 方案的模塊化方法促進了這一要求。工作者將調用相關的 MIME 處理模塊從存儲在 DIS 中的文檔中提取內容。

```
MIME：多用途互聯網郵件擴展的一種，或 MIME 類型，是一種互聯網標準，根據互聯網文件的性質和格式描述其內容。
[我們的網絡爬蟲滿足的要求](./r.jpg)

### Consistency (一致性)
我們的系統由多個爬取工作者組成。爬取內容之間的資料一致性至關重要。因此，為了避免資料不一致和爬取重複，我們的系統計算 URL 和文檔的校驗和，並將它們分別與 URL 和文檔校驗和資料存儲中現有的 URL 和文檔校驗和進行比較。

除了重複消除之外，為了通過容錯條件確保資料一致性，所有伺服器都可以定期將其狀態檢查點保存到備份服務 (例如 Amazon S3 或離線磁碟)。

### Performance (性能)
我們的網絡爬蟲的性能取決於以下因素：

- **URLs crawled per second (每秒爬取的 URL)**: We can improve this factor by adding new workers to the system.
- **每秒爬取的 URL**：我們可以通過向系統添加新的工作者來改善這個因素。

- **Utilizing blob storage for content storing (利用 blob 存儲進行內容存儲)**: This ensures higher throughput for the massive amount of unstructured data. It also indicates a fast retrieval of the stored content, because a single blob can support up to 500 requests per second.
- **利用 blob 存儲進行內容存儲**：這確保了大量非結構化資料的更高吞吐量。它還表明可以快速檢索存儲的內容，因為單個 blob 每秒最多可以支持 500 個請求。

- **Efficient implementation of the robots.txt file guideline (robots.txt 文件指南的有效實施)**: We can implement this performance factor by having an application-layer logic of setting the highest precedence of robots.txt guidelines while crawling.
- **robots.txt 文件指南的有效實施**：我們可以通過在爬取時設置 robots.txt 指南最高優先級的應用層邏輯來實現此性能因素。

- **Self-throttling (自我節流)**: We can have various application-level checks to ensure that our web crawler doesn’t hamper the performance of the website host servers by exhausting their resources.
- **自我節流**：我們可以進行各種應用級別的檢查，以確保我們的網絡爬蟲不會通過耗盡其資源來阻礙網站主機伺服器的性能。

                 Fulfilling Non-functional requirements (滿足非功能性需求)
Requirement (需求)                       Techniques (技術)

Scalability (可擴展性)                       - Addition/removal of different servers based on the increase/decrease in load (根據負載的增加/減少添加/刪除不同的伺服器)
                                  - Consistent hashing to manage server's addition and removal (一致性哈希來管理伺服器的添加和刪除)
                                  - Regular backup of the servers in Amazon S3 backup service to achieve fault tolerance (定期在 Amazon S3 備份服務中備份伺服器以實現容錯)

Extensibility and Modularity (可擴展性和模塊化)     - Addition of a newer communication protocol module in the HTML fetcher (在 HTML 獲取器中添加較新的通信協議模塊)
                                  - Addition of new MIME schemes while processing the downloaded document in DIS (在處理 DIS 中的下載文檔時添加新的 MIME 方案)

Consistency (一致性)                       - Calculation and comparison of checksums of URLs and Documents in the respective datastores (計算和比較相應資料存儲中 URL 和文檔的校驗和)

Performance (性能)                       - Increasing the number of workers performing the crawl (增加執行爬取的工作者數量)
                                  - Blob stores for storing the content (用於存儲內容的 Blob 存儲)
                                  - High priority to robots.txt file guidelines while crawling (爬取時高度重視 robots.txt 文件指南)
                                  - Self-throttle at a domain while crawling (爬取時在域中自我節流)

Scheduling (調度)                        - Pre-defined default recrawl frequency, or (預定義的默認重新爬取頻率，或)
                                  - Separate queues and their associated frequencies for various priority URLs (各種優先級 URL 的單獨隊列及其相關頻率)
```

### Scheduling (調度)
如前所述，我們可能需要以各種頻率重新爬取 URL。這些頻率由 URL 的應用決定。我們可以通過兩種不同的方式確定重新爬取的頻率：

1. 我們可以為每個 URL 分配一個默認或特定的重新爬取頻率。此分配取決於定義優先級的 URL 的應用。默認頻率分配給標準優先級 URL，較高的重新爬取頻率分配給較高優先級的 URL。

根據每個 URL 關聯的重新爬取頻率，我們可以決定將 URL 從調度器的資料庫入列到優先級隊列中。優先級定義了 URL 在隊列中的位置。

2. 第二種方法是為各種優先級 URL 設置單獨的隊列，首先使用高優先級隊列中的 URL，然後移動到較低優先級的 URL。

## Conclusion (結論)
網絡爬蟲系統需要使用微服務架構的多工作者設計。除了實現基本的爬取功能外，我們的設計還提供了與我們設計相關的潛在缺點和挑戰的見解，並通過適當的設計修改進一步糾正了這些缺點和挑戰。我們設計的顯著特徵如下：

1. Identification and design modification for crawler traps (爬蟲陷阱的識別和設計修改)
2. Extensibility of HTML fetching and content extraction modules (HTML 獲取和內容提取模塊的可擴展性)
