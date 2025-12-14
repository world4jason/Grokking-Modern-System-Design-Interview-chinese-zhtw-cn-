# Put Back-of-the-envelope Numbers in Perspective (透視粗略估算數字)

## Why do we use back-of-the-envelope calculations? (為什麼我們使用粗略估算？)

分散式系統具有通過網路連接的計算節點 (compute nodes)。有各種各樣的可用計算節點，它們可以以許多不同的方式連接。粗略估算 (Back-of-the-envelope calculations) 幫助我們忽略系統的細枝末節 (至少在設計層面)，並專注於更重要的方面。

粗略估算的一些範例可能是：
伺服器可以支援的並發 TCP 連接數。

Web、資料庫或快取伺服器每秒可以處理的請求數 (RPS)。

服務的儲存需求。
```

為此類計算選擇不合理的數字可能會導致設計缺陷。由於我們在許多設計問題中需要良好的估算，我們將在本課中詳細討論所有相關概念。這些概念包括：
資料中心伺服器的類型。

不同組件的實際存取延遲。

伺服器可以處理的 RPS 估算。

頻寬、伺服器和儲存估算的範例。
```

## Types of data center servers (資料中心伺服器的類型)
資料中心沒有單一類型的伺服器。企業解決方案使用商用硬體來節省成本並開發可擴展的解決方案。在下文中，我們討論資料中心內常用於處理不同工作負載的伺服器類型。

### Web servers (Web 伺服器)
為了可擴展性，Web 伺服器與應用伺服器解耦。Web 伺服器是負載平衡器之後的第一個接觸點。資料中心有裝滿 Web 伺服器的機架，通常處理來自客戶端的 API 呼叫。根據提供的服務，Web 伺服器中的記憶體和儲存資源可以是中小型。然而，此類伺服器需要良好的計算資源。例如，Facebook 使用了具有 32 GB RAM 和 500 GB 儲存空間的 Web 伺服器。但為了其高階計算需求，它與 Intel 合作構建了自訂的 16 核心處理器。

注意：本課中引用的許多數字來自 Facebook 在 2011 年開源的資料中心設計。由於 2004 年左右摩爾定律引發的性能放緩，這些數字並未過時。
```

### Application servers (應用伺服器)
應用伺服器運行核心應用軟體和業務邏輯。Web 伺服器和應用伺服器之間的區別有些模糊。應用伺服器主要提供動態內容，而 Web 伺服器主要向客戶端 (主要是 Web 瀏覽器) 提供靜態內容。它們可能需要廣泛的計算和儲存資源。儲存資源可以是揮發性和非揮發性的。Facebook 使用了 RAM 高達 256 GB 的應用伺服器，以及兩種類型的儲存——傳統旋轉磁碟和快閃記憶體——容量高達 6.5 TB。

### Storage servers (儲存伺服器)
隨著網際網路使用者的爆炸性成長，巨型服務儲存的資料量成倍增加。此外，各種類型的資料現在正儲存在不同的儲存單元中。例如，YouTube 使用以下資料儲存：
Blob 儲存用於其編碼影片。

一個臨時處理佇列儲存，可以容納每天上傳到 YouTube 進行處理的數百小時影片內容。

名為 Bigtable 的專用儲存，用於儲存大量影片縮圖。

用於使用者和影片元數據 (評論、喜歡、使用者頻道等) 的關聯式資料庫管理系統 (RDBMS)。
```

其他資料儲存仍用於分析——例如，Hadoop 的 HDFS。儲存伺服器主要包括結構化 (例如 SQL) 和非結構化 (NoSQL) 資料管理系統。

回到 Facebook 的例子，我們知道他們使用了儲存容量高達 120 TB 的伺服器。隨著使用中的伺服器數量增加，Facebook 能夠容納 exabytes (EB) 的儲存。1 EB 是 10^{18} Bytes。按照慣例，我們以 10 為基底而不是以 2 為基底來衡量儲存和網路頻寬。然而，這些伺服器的 RAM 只有 32 GB。

注意：上述伺服器並不是資料中心中唯一的伺服器類型。組織還需要用於配置、監控、負載平衡、分析、會計、快取等服務的伺服器。
```

Facebook 開源的數字目前已過時。在下表中，我們描述了可用於當今資料中心的伺服器的功能：

          典型的伺服器規格 (Typical Server Specifications)
組件 (Component)               數量 (Count)

插槽數量 (Number of sockets)    2

處理器 (Processor)             Intel Xeon X2686

核心數量 (Number of cores)      36 cores (72 threads)

RAM                           256 GB

快取 (L3) (Cache (L3))         45 MB

儲存容量 (Storage capacity)     15 TB
```

上面的數字靈感來自 Amazon 裸機伺服器 (bare-metal server)，但也可能有更強大或較不強大的機器支援更高的 RAM (高達 8 TB)、磁碟儲存 (高達 24 個磁碟，每個高達 20 TB，約 2021 年) 和快取記憶體 (高達 120 MB)。

## Standard numbers to remember (要記住的標準數字)
服務的規劃和實施需要付出很多努力。但是，如果沒有對機器可以處理的工作負載類型有任何基本了解，這種規劃是不可能的。延遲在決定機器可以處理的工作負載量方面起著重要作用。下表描述了系統設計人員為了進行資源估算應該知道的一些重要數字。

                             重要延遲 (Important Latencies)
組件 (Component)                                              時間 (奈秒/nanoseconds)

L1 快取參考 (L1 cache reference)                               0.9 

L2 快取參考 (L2 cache reference)                               2.8 

L3 快取參考 (L3 cache reference)                               12.9 

主記憶體參考 (Main memory reference)                            100 

使用 Snzip 壓縮 1KB (Compress 1KB with Snzip)                  3,000 (3 微秒/microseconds)

從記憶體順序讀取 1 MB (Read 1 MB sequentially from memory)       9,000 (9 微秒/microseconds)

從 SSD 順序讀取 1 MB (Read 1 MB sequentially from SSD)           200,000 (200 微秒/microseconds)

同一資料中心內的往返 (Round trip within same datacenter)          500,000 (500 微秒/microseconds)

以 ~1GB/sec 速度從 SSD 順序讀取 1 MB                            1,000,000 (1 毫秒/milliseconds)

磁碟尋道 (Disk seek)                                           4,000,000 (4 毫秒/milliseconds)

從磁碟順序讀取 1 MB (Read 1 MB sequentially from disk)           2,000,000 (2 毫秒/milliseconds)

發送封包 SF->NYC (Send packet SF->NYC)                         71,000,000 (71 毫秒/milliseconds)
```

除了上面列出的延遲之外，還有以每秒查詢數 (QPS) 衡量的吞吐量數字，典型的單伺服器資料儲存可以處理這些數字。

            重要速率 (Important Rates)

MySQL 處理的 QPS (QPS handled by MySQL)           1000

鍵值存儲處理的 QPS (QPS handled by key-value store) 10,000

快取伺服器處理的 QPS (QPS handled by cache server)  100,000–1 M
```

上面的數字是近似值，並且根據許多原因 (如查詢類型 (點和範圍)、機器規格、資料庫設計、索引等) 而有很大差異。
單個項目的查找稱為點查詢 (Point query)。例如，SELECT * FROM STUDENTS WHERE NAME = "John"; 這是一個點查詢，因為搜尋的是單個值 "John"。

檢索邊界之間資訊/記錄的查詢。例如，SELECT * FROM STUDENTS WHERE age BETWEEN 15 AND 18;。
```

## Requests estimation (請求估算)

本節討論典型伺服器一秒鐘可以處理的請求數。在伺服器內，資源有限，根據客戶端請求的類型，不同的資源可能會成為瓶頸。讓我們了解兩種類型的請求。
CPU 密集型請求 (CPU-bound requests)：這些是限制因素為 CPU 的請求類型。

記憶體密集型請求 (Memory-bound requests)：這些是受限於機器記憶體量的請求類型。
```

讓我們估算每種類型請求的 RPS。但在此之前，我們需要假設以下內容：

我們的伺服器具有我們在上表中定義的典型伺服器的規格。
作業系統和其他輔助處理程序總共消耗了 16 GB 的 RAM。

每個工作權 (worker) 消耗 300 MB 的 RAM 儲存空間來完成請求。

為了簡單起見，我們假設 CPU 從 RAM 獲取資料。因此，快取系統確保所有所需的內容都可用於服務，而無需存取儲存層。

每個 CPU 密集型請求需要 200 毫秒，而記憶體密集型請求需要 50 毫秒才能完成。

讓我們對每種類型的請求進行計算。

**CPU 密集型 (CPU bound)**：用於計算 CPU 密集型請求的 RPS 的簡單公式是：

```
其中
RPS cpu：CPU 密集型 RPS

NUM cpu：CPU 執行緒數，也稱為硬體執行緒

TASK time：每個任務完成所需的時間
```

那麼，RPS cpu = 72 / 200ms = 360 RPS

上面計算的原理是，我們可以將一秒鐘想像成一個盒子，我們計算有多少個小盒子 (任務) 可以放入大盒子中——即一定數量的 CPU 在一秒鐘內可以完成的任務數。因此，更多的 CPU/執行緒將導致更高的 RPS。


**記憶體密集型請求 (Memory-bound requests)**：對於記憶體密集型請求，我們使用以下公式：
```
其中
RPS memory：記憶體密集型 RPS

RAM size：RAM 總大小

WORKER memory：記憶體中管理請求的工作權 (worker)
那麼，RPS memory = 240GB / 300MB x 1 / 50 ms = 16,000 RPS

繼續我們在 CPU 密集型處理程序解釋中的盒子類比，這裡我們先計算有多少個盒子 (伺服器可以託管多少個記憶體密集型處理程序)，然後我們可以在每個大盒子中放入多少個小盒子 (任務)。

服務同時接收 CPU 密集型和記憶體密集型請求。考慮一半請求是 CPU 密集型而另一半是記憶體密集型的情況，我們可以處理總共 360/2 + 16,000/2 = 8180，大約 8000 RPS。

上面的計算僅是用於建立對估算 RPS 涉及的基本因素的理解的近似值。實際上，還有許多其他因素在起作用。例如，如果資料在 RAM 中不可用，或者如果向資料庫伺服器發出請求 (這也將包括資料庫和網路延遲)，則需要延遲來進行磁碟尋道。此外，查詢的類型也很重要。當然，故障、程式碼中的錯誤、節點故障、停電、網路中斷等都是不可避免的因素。

在典型的一天中，各種類型的請求到達，一個僅從 RAM 提供靜態內容的強大伺服器可能處理多達 500k RPS。在頻譜的另一端，像影像處理這樣的計算密集型任務可能只允許最大 50 RPS。

注意：實際上，容量估算是一個難題，組織多年來一直在學習如何改進它。監控系統會關注我們基礎設施的所有部分，以便在伺服器過載時向我們發出早期警告。
```
