# Design of a CDN (CDN 的設計)

## CDN design (CDN 設計)
我們將分兩個階段解釋我們的 CDN 設計。在第一階段，我們將涵蓋組成 CDN 的組件。到此階段結束時，我們將了解為什麼需要特定組件。在第二階段，我們將通過解釋每個組件如何與其他組件互動來開發功能齊全的 CDN 來探索工作流程。讓我們深入研究一下。

### CDN components (CDN 組件)
組成 CDN 的組件如下：

- **Clients (客戶端)**：終端使用者使用各種客戶端 (如瀏覽器、智慧型手機和其他設備) 從 CDN 請求內容。
- **Routing system (路由系統)**：路由系統將客戶端引導至最近的 CDN 設施。為了有效地做到這一點，此組件從各種系統接收輸入，以了解內容放置的位置、對特定內容的請求數量、一組特定伺服器正在處理的負載以及各種內容的 URI (統一資源識別碼) 命名空間。在下一課中，我們將討論不同的路由機制，以將使用者轉發到最近的 CDN 設施。
- **Scrubber servers (清洗伺服器)**：清洗伺服器用於區分好流量和惡意流量，並防止 DDoS 等知名攻擊。清洗伺服器通常僅在檢測到攻擊時使用。在這種情況下，流量會被清洗或清理，然後路由到目標目的地。
- **Proxy servers (代理伺服器)**：代理或邊緣代理伺服器將 RAM 中的內容服務給使用者。代理伺服器將熱門資料儲存在 RAM 中，儘管它們也可以將冷資料儲存在 SSD 或硬碟中。這些伺服器還提供記帳資訊並從分發系統接收內容。
- **Distribution system (分發系統)**：分發系統負責將內容分發到不同 CDN 設施的所有邊緣代理伺服器。該系統使用網際網路和類似廣播的智慧方法在活動的邊緣代理伺服器之間分發內容。
- **Origin servers (源伺服器)**：CDN 基礎設施利用從源伺服器接收的資料為使用者提供服務。源伺服器將 CDN 中任何不可用的資料提供給客戶端。源伺服器將使用適當的儲存來保存內容和其他映射元數據。不過，我們不會在這裡討論源基礎設施的內部架構。
- **Management system (管理系統)**：從業務和管理方面來看，管理系統在 CDN 中很重要，其中資源使用和統計數據會被持續觀察。此組件測量重要指標，如延遲、停機時間、封包遺失、伺服器負載等。對於第三方 CDN，記帳資訊也可用於計費目的。

[CDN 組件]

### Workflow (工作流程)
抽象設計的工作流程如下：

1. 源伺服器將 CDN 中快取的所有物件的 URI 命名空間委派提供給請求路由系統。
2. 源伺服器將內容發布到負責跨活動邊緣代理伺服器分發資料的分發系統。
3. 分發系統在代理伺服器之間分發內容，並向請求路由系統提供回饋。此回饋有助於為請求客戶端優化最近代理伺服器的選擇。此回饋包含有關哪些內容快取在哪些代理伺服器上的資訊，以便將流量路由到相關的代理伺服器。
4. 客戶端向路由系統請求合適的代理伺服器。
5. 請求路由系統返回適當代理伺服器的 IP 地址。
6. 出於安全原因，客戶端請求路由通過清洗伺服器。
7. 清洗伺服器將良好的流量轉發到邊緣代理伺服器。
8. 邊緣代理伺服器服務客戶端請求，並定期將記帳資訊轉發給管理系統。管理系統更新源伺服器，並向路由系統發送有關統計數據和內容詳細資訊的回饋。但是，如果內容在代理伺服器中不可用，請求將路由到源伺服器。如果內容在邊緣代理伺服器中找不到，也可以有代理伺服器的層次結構。對於這種情況，請求會轉發到父代理伺服器。

## API Design (API 設計)
本節將討論 CDN 提供的功能的 API 設計。這將幫助我們了解 CDN 如何接收來自客戶端的請求、從源伺服器接收內容以及與網路中的其他組件通訊。讓我們為以下每個功能開發 API：

- 檢索內容 (Retrieve content)
- 傳遞內容 (Deliver content)
- 請求內容 (Request content)
- 搜尋內容 (Search content)
- 更新內容 (Update content)
- 刪除內容 (Delete content)

內容可以是任何東西，比如檔案、影片、音訊或其他 Web 物件。在這裡，我們將使用“內容”一詞來指代所有上述內容。為了清晰起見，我們不會在以下 API 中討論與隱私相關的參數——例如內容是公開還是私有、誰應該能夠存取此內容、是否應該加密等等。

### Retrieve (proxy server to origin server) (檢索：代理伺服器到源伺服器)
如果代理伺服器請求內容，GET 方法通過下面的 `/retrieveContent` API 檢索內容：

```
retrieveContent(proxyserver_id, content_type, content_version, description)
```

讓我們看看參數的細節：
#### Details of Parameters (參數細節)
參數 (Parameter)              描述 (Description) 

proxyserver_id               這是請求代理伺服器的唯一 ID。

content_type                 此資料結構將包含有關請求內容的資訊。具體來說，它將包含類別 (音訊、影片、文件、腳本等)、請求它的客戶端類型以及請求的品質 (如果有的話)。

content_version              這表示內容的版本號。對於 /retrieveContent API，content_version 將包含駐留在代理伺服器中的內容的當前版本。如果代理伺服器沒有可用的先前版本，content_version 將為 NULL。

description                  這指定了內容細節——例如，如果 content_type 是影片，則為影片的副檔名、解析度細節等。
```

上述 API 以 JSON 檔案給出回應，其中包含文字、內容類型、內容中圖片或影片的連結等。

```Javascript
"Object_links": [
                 {
                 "name": "videos"
                 "link": https://app_server.com/api/assets/videos/
                 },
                 {
                 "name": "illustrations"
                 "link": https://app_server.com/api/assets/illustrations/
                 },
                ]
              
                將在代理伺服器下載各種物件的 JSON 檔案。
```

### Deliver (origin server to proxy servers) (傳遞：源伺服器到代理伺服器)
源伺服器使用此 API 通過分發系統將指定內容 (更新版本) 傳遞給代理伺服器。我們稱之為 `/deliverContent` API：

```
deliverContent(origin_id, server_list, content_type, content_version, description)
```

#### Details of Parameters (參數細節)
參數 (Parameter)         描述 (Description)

origin_id               這唯一識別每個源伺服器。

server_list             這識別分發系統將內容推送到其中的伺服器列表。

content_version         這表示源伺服器上內容的更新版本。接收內容的代理伺服器將丟棄以前的版本。
其餘參數已在上面解釋過。

### Request (clients to proxy servers) (請求：客戶端到代理伺服器)
使用者使用此 API 從代理伺服器請求內容。我們稱之為 `/requestContent` API：

```
requestContent(user_id, content_type, description)
```

#### Details of Parameter (參數細節)
參數 (Parameter)         描述 (Description)

user_id                 這是請求內容的使用者的唯一 ID。
指定的代理伺服器回應上述 API，將特定內容返回給請求的使用者。

```Javascript
"Object_links": [
                 {
                 "name": "components"
                 "link": https://cdn.app_server.com/api/components/
                 },
                 {
                 "name": "css"
                 "link": https://cdn.app_server.com/api/css/
                 },
                 {
                 "name": "illustrations"
                 "link": https://cdn.app_server.com/api/assets/illustrations/
                 },
                 {
                 "name": "videos"
                 "link": https://cdn.app_server.com/api/assets/videos/
                 },
                 {
                 "name": "icons"
                 "link": https://cdn.app_server.com/api/icons/
                 },
                 {
                 "name": "fonts"
                 "link": https://cdn.app_server.com/api/fonts/
                 },
              各種物件將在使用者端下載的 JSON 檔案。
```

### Search (proxy server to peer proxy servers) (搜尋：代理伺服器到對等代理伺服器)
雖然內容首先在代理伺服器本地搜尋，但代理伺服器也可以通過 `/searchContent` API 在同一 PoP 中的對等代理伺服器中探測請求的內容。這可能會將查詢泛洪 (flood) 到 PoP 中的所有代理伺服器。或者，我們可以使用 PoP 中的資料儲存來查詢內容，儘管代理伺服器需要維護哪些內容可用於哪個代理伺服器。

`/searchContent` API 如下所示：

```
searchContent(proxyserver_id, content_type, description)
```

### Update (proxy server to peer proxy servers) (更新：代理伺服器到對等代理伺服器)
代理伺服器使用 `/updateContent` API 更新 PoP 中對等代理伺服器中的指定內容。當在 CDN 上運行指定的隔離腳本以提供圖像調整大小、影片解析度轉換、安全性和更多服務時，它會這樣做。這類型的腳本稱為無伺服器腳本 (serverless scripting)。

`/updateContent` API 如下所示：
```
updateContent(proxyserver_id, content_type, description)
#### Details of Parameter (參數細節)
參數 (Parameter)         描述 (Description)

proxyserver_id          這唯一識別 PoP 中要更新內容的代理伺服器。
```

其餘參數已在上面解釋過。

注意：這裡不討論刪除 API。在我們的快取章節中，我們詳細討論了不同的驅逐機制。這些機制也適用於 CDN 內容驅逐。然而，可能會出現需要刪除 API 的情況。我們將在下一課討論一些內容一致性機制，比如內容在快取中停留多長時間。
在接下來的課程中，我們將深入探討 CDN 的特性。
