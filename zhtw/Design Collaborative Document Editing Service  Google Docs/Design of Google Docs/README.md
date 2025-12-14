# Design of Google Docs (Google Docs 的設計)
## Design (設計)
我們將分兩步完成我們的設計。第一步，我們將解釋不同的組件和構建塊以及在我們設計中選擇它們的原因。第二步將描述我們如何通過描繪工作流程來滿足各種功能需求。

### Components (組件)
我們利用以下一組組件來完成我們的設計：

- **API gateway (API 閘道器)**：不同的客戶端請求將通過 API 閘道器被攔截。根據請求，可以將單個請求轉發到多個組件、拒絕請求或使用已經快取的響應立即回覆，所有這些都通過 API 閘道器進行。編輯請求、文件評論、通知、身分驗證和資料存儲請求都將通過 API 閘道器。
- **Application servers (應用程式伺服器)**：應用程式伺服器將執行通常需要計算能力的業務邏輯和任務。例如，某些文件可能會從一種檔案類型轉換為另一種檔案類型 (例如，從 PDF 轉換為 Word 文件) 或支援匯入和匯出等功能。它也是推薦引擎屬性收集的核心。
- **Data stores (資料儲存)**：將使用各種資料儲存來滿足我們的需求。我們將使用關聯式資料庫來保存使用者資訊和文件相關資訊，以施加權限限制。我們可以使用 NoSQL 來儲存使用者評論以便更快地存取。為了保存文件的編輯歷史記錄，我們可以使用時間序列資料庫 (time series database)。我們將使用 Blob 儲存來儲存文件中的影片和圖片。最後，我們可以使用像 Redis 這樣的分散式快取和 CDN 為終端使用者提供良好的效能。我們專門使用 Redis 來儲存不同的資料結構，包括使用者會話、預輸入服務的功能和經常存取的文件。CDN 儲存經常用到的文件和重物件，如圖片和影片。
- **Processing queue (處理隊列)**：由於文件編輯需要頻繁地向伺服器發送小規模資料 (通常是字元)，因此將這些資料排隊進行定期批次處理是個好主意。我們將字元、圖片、影片和評論新增到處理隊列中。使用 HTTP 調用發送每個微小的字元效率很低。因此，我們將使用 WebSockets 減少開銷並觀察不同使用者對文件的實時更改。
- **Other components (其他組件)**：其他組件包括維護使用者會話資訊的會話伺服器。我們將通過會話伺服器管理文件存取權限。本質上，還將有配置、監控、發布-訂閱和日誌記錄服務，這些服務將處理諸如監控和在伺服器故障情況下選舉領導者、排隊任務 (如使用者通知) 以及記錄除錯資訊等任務。

下圖提供了不同組件和構建塊如何協調以提供服務的描述。

[協同文件編輯服務的詳細設計](./design.jpg)

Question 1 (問題 1)
為什麼我們應該使用 WebSockets 而不是 HTTP 方法？為什麼 WebSockets 最適合這種通訊？

Answer (答案)
WebSockets offer us the following characteristics: (WebSockets 為我們提供以下特徵：)

- They have a long-lasting connection between clients and servers. (它們在客戶端和伺服器之間有持久的連接。)
- They enable full-duplex communication. That is, we can simultaneously communicate from client to server and vice versa. (它們支援全雙工通訊。也就是說，我們可以同時從客戶端到伺服器進行通訊，反之亦然。)
- There’s no overhead of HTTP request or response headers. (沒有 HTTP 請求或響應標頭的開銷。)
The lightweight nature of WebSockets reduces the latency and allows the server to push changes to clients as soon as they are available. (WebSockets 的輕量級特性減少了延遲，並允許伺服器在更改可用時立即將其推送到客戶端。)
```

Question 2 (問題 2)
上面的設計描述的是微服務架構而不是單體架構。為什麼這裡適合這樣做？

Answer (答案)
Microservices are preferred for the following reasons: (微服務是首選，原因如下：)

- Development is simpler and faster. (開發更簡單、更快速。)
- Each service within the architecture is isolated. That is, failure of one service doesn’t produce a cascading effect. (架構中的每個服務都是隔離的。也就是說，一個服務的故障不會產生級聯效應。)
- Different components may have different programming language requirements for various reasons. Microservices give the freedom of using different programming languages for different components. (不同的組件可能有不同的程式語言需求。微服務為不同組件提供了使用不同程式語言的自由。)
- Because of microservices’ modular nature, it’s easy to scale and update services individually. (由於微服務的模組化性質，很容易單獨擴展和更新服務。)
```

Question 3 (問題 3)
哪種排隊演算法最適合上述設計中的操作隊列？

Answer (答案)
具有嚴格順序的先進先出 (FIFO) 最適合操作隊列，以便按使用者請求的順序執行操作。
```

### Workflow (工作流程)
在接下來的步驟中，我們將解釋不同的請求在到達 API 閘道器後將如何被受理：

- **Collaborative editing and conflict resolution (協同編輯和衝突解決)**：每個請求都被轉發到操作隊列。這是解決同一文件不同協作者之間衝突的地方。如果沒有衝突，數據將通過會話伺服器批次處理並存儲在時間序列資料庫中。像影片和圖片這樣的資料被壓縮以優化儲存，而字元則立即處理。
- **History (歷史)**：借助時間序列資料庫，可以恢復文件的不同版本。可以使用比較版本並識別差異的 DIFF 操作來比較不同版本，以恢復同一文件的舊版本。
- **Asynchronous operations (非同步操作)**：通知、電子郵件、查看次數和評論是可以通過像 Kafka 這樣的發布-訂閱組件排隊的非同步操作。API 閘道器生成這些請求並將它們轉發到發布-訂閱模組。共享文件的使用者可以通過此過程生成通知。
- **Suggestions (建議)**：建議以預輸入服務的形式出現，為常用單詞和短語提供自動完成建議。預輸入服務還可以從文件中提取屬性和關鍵字，並向使用者提供建議。由於單詞數量可能很高，我們將為此目的使用 NoSQL 資料庫。此外，最常用的單詞和短語將存儲在像 Redis 這樣的快取系統中。
- **Import and export documents (匯入和匯出文件)**：應用程式伺服器執行許多重要任務，包括匯入和匯出文件。應用程式伺服器還將文件從一種格式轉換為另一種格式。例如，.doc 或 .docx 文件可以轉換為 .pdf，反之亦然。應用程式伺服器還負責預輸入服務的特徵提取。

注意：我們使用 WebSockets 加快了整體效能，並使我們能夠促進在同一文件上協作的使用者之間的聊天。如果我們將 WebSockets 與一種類似 Redis 的快取結合起來，就有可能開發出一種有效的聊天功能。
```

Question 1 (問題 1)
我們正在通過非同步方法實現查看計數器，這意味著文件的查看次數可能已過時。我們可以使用分片計數器或 Redis 計數器來獲得有效的結果嗎？

Answer (答案)
兩種解決方案 (分片計數器或 Redis 計數器) 都可以工作，儘管它們對於計算文件查看次數來說似乎有些過分。

為了提供近乎即時的查看計數，我們可以使用流式發布-訂閱系統，例如 Kafka，其中主題可以基於文件標識符。
```

Question 2 (問題 2)
上面的詳細設計沒有描述查看計數器資料保存在哪裡。什麼是查看計數器的合適儲存，以及需要進行哪些設計更改？

Answer (答案)
出於可擴展性目的，適合將查看計數器資料儲存在 NoSQL 中，因為 NoSQL 的讀/寫延遲通常較低。

為了完成設計，我們必須將查看計數器與 NoSQL 資料庫連接起來。
```
