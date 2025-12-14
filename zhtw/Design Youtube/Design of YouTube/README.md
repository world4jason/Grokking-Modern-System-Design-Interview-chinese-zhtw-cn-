# Design of YouTube (YouTube 的設計)
## High-level design (高層設計)
高層設計顯示了我們將如何互連上一課中確定的各個組件。我們已經開始開發解決方案，以通過此設計支援功能和非功能需求。

[YouTube 的高層設計](./sys.jpg)

抽象設計的工作流程如下：

1. 使用者將影片上傳到伺服器。
2. 伺服器將元資料和隨附的使用者資料儲存到資料庫，同時將影片交給編碼器進行編碼 (參見上圖中的 2.1 和 2.2)。
3. 編碼器與轉碼器一起壓縮影片並將其轉換為多種解析度 (如 2160p、1440p、1080p 等)。影片存儲在 blob 儲存 (類似於 GFS 或 S3) 上。
4. 一些熱門影片可能會轉發到充當快取的 CDN。
5. 由於 CDN 靠近使用者，因此 CDN 允許使用者以低延遲流式傳輸影片。但是，CDN 並不是為最終使用者提供影片服務的唯一基礎設施，我們將在詳細設計中看到這一點。

```
Question
為什麼我們不直接將影片上傳到編碼器而不是伺服器？當前的策略不會引入額外的延遲嗎？

Answer
在編碼器和客戶端之間引入伺服器是個好主意有幾個原因：

- The client could be malicious and could abuse the encoder. (客戶端可能是惡意的，可能會濫用編碼器。)
- If the uploaded video is a duplicate, the server could filter it out. (如果上傳的影片是重覆的，伺服器可以將其過濾掉。)
- Encoders will be available on a private IP address within YouTube’s network and not available for public access. (編碼器將在 YouTube 網路內的私有 IP 地址上可用，不可公開訪問。)
```

## API design (API 設計)
讓我們根據我們提供的功能來了解 API 的設計。我們將設計 API 以將我們的功能集轉化為技術規範。在這種情況下，為了簡單和速度，可以使用 REST API。我們的 API 設計部分將幫助我們了解客戶端將如何從 YouTube 的後端應用程式請求服務。讓我們為以下每個功能開發 API：

- 上傳影片
- 流式傳輸影片
- 搜尋影片
- 查看縮略圖
- 喜歡或不喜歡影片
- 評論影片

[API 設計概述](./api.jpg)

### Upload video (上傳影片)
POST 方法可以將影片上傳到 /uploadVideo API：
```
uploadVideo(user_id, video_file, category_id, title, description, tags, default_language, privacy_settings)
讓我們看看這裡對以下參數的描述。
Parameter (參數)             Description (描述)

user_id                This is the user that is uploading the video. (這是正在上傳影片的使用者。)

video_file             This is the video file that the user wants to upload. (這是使用者想要上傳的影片文件。)

category_id            This refers to the category a video belongs to. Typical categories can be “Entertainment,” “Engineering,” “Science,” and so on. (這指的是影片所屬的類別。典型的類別可以是“娛樂”、“工程”、“科學”等。)

title                  This is the title of the video. (這是影片的標題。)

description            This is the description of the video. (這是影片的描述。)

tags                   This refers to the specific topics the content of the video covers. The tags can improve search results. (這指的是影片內容涵蓋的特定主題。標籤可以改善搜尋結果。)

default_language       This is the default language a page will show to the user when the video is streamed. (這是流式傳輸影片時頁面將向使用者顯示的預設語言。)

privacy_settings       This refers to the privacy of the video. Generally, videos can be a public asset or private to the uploader. (這指的是影片的隱私。通常，影片可以是公共資產或對上傳者私有。)
```

```
影片文件被分解成更小的數據包並按順序上傳到伺服器。如果失敗，YouTube 可以將資料存儲有限的時間，並在使用者重試時恢復上傳。要詳細了解該概念，請閱讀有關非同步 API 的更多資訊。
```

```
非同步 API：伺服器無法立即滿足請求的 API，因為資料太大或不易獲得。在這種情況下，客戶端會立即收到伺服器已接受請求的回應，而伺服器稍後會回應資料。這種機制節省了 CPU 和網路頻寬，並且不會讓客戶端等待。
```

### Stream video (流式傳輸影片)
GET 方法最適合 /streamVideo API：
```
streamVideo(user_id, video_id, screen_resolution, user_bitrate, device_chipset)
在這種情況下引入的一些新事物是以下參數：
Parameter (參數)               Description (描述)

screen_resolution        The server can best optimize the video if the user's screen resolution is known. (如果知道使用者的螢幕解析度，伺服器可以最好地優化影片。)

user_bitrate             The transmission capacity of the user is required to understand which quality of video chunks should be transferred to the client or user. (需要使用者的傳輸容量來了解應將哪種質量的影片塊傳輸給客戶端或使用者。)

device_chipset           Many YouTube users watch content on handheld devices, which makes it important to know the handling capabilities of these devices to better serve the users. (許多 YouTube 使用者在手持設備上觀看內容，這使得了解這些設備的處理能力以更好地為使用者服務變得重要。)
```

```
伺服器將在其存儲中存儲同一影片的不同質量，並根據使用者的傳輸速率為使用者提供服務。
### Search videos (搜尋影片)
/searchVideo API 使用 GET 方法：
```
searchVideo(user_id, search_string, length, quality, upload_date)
```

Parameter (參數)            Description (描述)

search_string           This is ahe string used for searching videos by their title. (這是用於按標題搜尋影片的字串。)

length (optional)       This is used to filter videos based on their length in terms of time. (這用於根據時間長度過濾影片。)

quality (optional)      This is used to filter videos based on the resolution, like 2048p, 1440p, 1080p, and so on. (這用於根據解析度過濾影片，如 2048p、1440p、1080p 等。)

upload_date (optional)  This is used to filter videos based on their upload date to YouTube. (這用於根據上傳到 YouTube 的日期過濾影片。)
### View thumbnails (查看縮略圖)
我們可以使用 GET 方法訪問 /viewThumbnails API：
```
viewThumbnails(user_id, video_id)
```

Parameter (參數)   Description (描述)

video_id     This specifies the unique ID of the video associated with the thumbnails. (這指定了與縮略圖關聯的影片的唯一 ID。)
此 API 將按順序返回影片的縮略圖。


### Like and dislike a video (喜歡和不喜歡影片)
喜歡和不喜歡 API 使用 GET 方法。如下所示，這相當簡單。
```
likeDislike(user_id, video_id, like)
我們可以對喜歡和不喜歡功能使用相同的 API。根據傳遞給 like 欄位的參數，我們可以相應地更新資料庫——即 0 表示喜歡，1 表示不喜歡。

### Comment video (評論影片)
就像喜歡和不喜歡 API 一樣，我們只需要向 API 提供評論字串。此 API 也將使用 GET 方法。
```
commentVideo(user_id, video_id, comment_text)
```

Parameter (參數)       Description (描述)

comment_text     This refers to the text that is typed by the user on the particular video. (這指的是使用者在特定影片上鍵入的文本。)
## Storage schema (儲存模式)
API 設計中的上述每個功能都需要資料庫的支援——我們需要在儲存模式中儲存上述詳細資訊，以便為 API 網關提供服務。

[儲存模式](./db.jpg)

```
注意：為簡單起見，省略了許多關於可以映射到 YouTube 提供的服務的資料庫表的底層細節。例如，一個影片可以有不同的質量，而在“Video”表中沒有提到這一點。
```

## Detailed design (詳細設計)
現在，讓我們回到我們的高層設計，看看我們是否可以進一步探索設計的部分。特別是，以下領域需要更多討論：

- **Component integration (組件集成)**：我們將介紹伺服器和存儲組件之間的一些互連，以更好地了解系統將如何工作。
- **Thumbnails (縮略圖)**：使用者通過縮略圖查看影片的某些部分很重要。因此，我們將在詳細設計中添加縮略圖生成和存儲。
- **Database structure (資料庫結構)**：我們的估算表明我們需要巨大的存儲空間。我們還需要存儲各種類型的資料，例如影片、影片元資料和縮略圖，出於效能原因，每一種都需要專門的資料存儲。了解資料庫細節將使我們能夠設計一個滯後盡可能小的系統。

讓我們看看下面的圖表。我們將分兩個步驟解釋我們的設計，第一步著眼於新添加的組件是什麼，第二步考慮它們如何協調以構建 YouTube 系統。

### Detailed design components (詳細設計組件)
由於我們強調了流暢流媒體、伺服器級細節和縮略圖功能的要求，以下設計將滿足我們的期望。讓我們在這裡解釋每個添加組件的目的：

- **Load balancers (負載平衡器)**：為了在 Web 伺服器之間分配大量使用者請求，我們需要負載平衡器。
- **Web servers (Web 伺服器)**：Web 伺服器接收使用者請求並對其做出回應。這些可以被視為我們滿足使用者請求的 API 伺服器的介面。
- **Application server (應用伺服器)**：應用程式和業務邏輯駐留在應用伺服器中。它們準備 Web 伺服器處理最終使用者查詢所需的資料。
- **User and metadata storage (使用者和元資料儲存)**：由於我們有大量使用者和影片，因此保存影片元資料和與使用者相關內容所需的存儲必須存儲在不同的存儲集群中。這是因為為了可擴展性目的，應該解耦大量不太相關的資料。
- **Bigtable**：對於每個影片，我們需要多個縮略圖。Bigtable 是存儲縮略圖的不錯選擇，因為它具有高吞吐量和存儲鍵值資料的可擴展性。Bigtable 最適合存儲每個低於 10 MB 的大量資料項。因此，它是 YouTube 縮略圖的理想選擇。
- **Upload storage (上傳儲存)**：上傳儲存是可以存儲使用者上傳影片的臨時儲存。
- **Encoders (編碼器)**：每個上傳的影片都需要壓縮並轉碼成各種格式。縮略圖生成服務也從編碼器獲得。
- **CDN and colocation sites (CDN 和主機託管站點)**：CDN 和主機託管站點存儲靠近使用者的流行和中等流行內容，以便於訪問。主機託管中心用於由於商業原因無法投資資料中心設施的地方。
```
Bigtable: Chang, Fay, et al. “Bigtable: A distributed storage system for structured data.” ACM Transactions on Computer Systems (TOCS) 26.2 (2008): 1-26.
```


[YouTube 組件的詳細設計](./c.jpg)

### Design flow and technology usage (設計流程和技術使用)
既然我們了解了每個組件的目的，讓我們在以下步驟中討論不同組件中使用的流程和技術：

1. 使用者可以通過連接到 Web 伺服器上傳影片。Web 伺服器可以運行 Apache 或 Lighttpd。Lighttpd 更受歡迎，因為它速度快，可以提供靜態頁面和影片。
2. 來自 Web 伺服器的請求被傳遞到應用伺服器，應用伺服器可以聯繫各種資料存儲以讀取或寫入使用者、影片或影片元資料。有單獨的 Web 和應用伺服器，因為我們希望將客戶端服務與應用程式和業務邏輯解耦。可以在此層上使用不同的編程語言來有效地執行不同的任務。例如，C 編程語言可用於加密。此外，這為我們提供了一個額外的快取層，其中請求最多的對象存儲在應用伺服器上，而請求最頻繁的頁面將存儲在 Web 伺服器上。
3. 使用多個存儲單元。讓我們瀏覽一下其中的每一個：

    I. 上傳儲存用於在臨時編碼之前存儲使用者上傳的影片。
  
    II. 使用者帳戶資料存儲在單獨的資料庫中，而影片元資料單獨存儲。這個想法是將訪問頻率較高和較低的存儲集群彼此分開，以獲得最佳訪問時間。如果並發讀寫數量有限，我們可以使用 MySQL。但是，隨著使用者數量——也就是並發讀寫數量——的增加，我們可以轉向 NoSQL 类型的資料管理系統。
 
   III. 由於 Bigtable 基於 Google 文件系統 (GFS)，它旨在以低檢索延遲存儲大量小文件。它是存儲縮略圖的合理選擇。
 
4. 編碼器生成縮略圖，並在元資料資料庫中存儲與影片相關的其他元資料。它還將分別向 CDN 和主機託管伺服器提供流行和中等流行的內容。
5. 使用者最終可以從任何可用的站點流式傳輸影片。

```
主機託管伺服器放置在配備存儲和網路資源的主機託管/運營商站點中。這些資源可以根據需要租給客戶。
```

## YouTube search (YouTube 搜尋)
由於 YouTube 是訪問量最大的網站之一，大量使用者將使用搜尋功能。儘管我們已經介紹了分佈式搜尋的構建塊，但我們將提供有關 YouTube 系統內搜尋如何工作的基本概述。

上傳到 YouTube 的每個新影片都將進行資料提取處理。我們可以使用 JSON 文件來存儲提取的資料，其中包括以下內容：

- 影片標題。
- 頻道名稱。
- 影片描述。
- 影片內容，可能從成績單中提取。
- 影片長度。
- 類別。
每個 JSON 文件都可以稱為文檔。接下來，關鍵字將從文檔中提取並存儲在鍵值存儲中。鍵值存儲中的鍵將保存使用者搜尋的所有關鍵字，而鍵值存儲中的值將包含每個鍵的出現次數、頻率以及在不同文檔中出現的位置。當使用者搜尋關鍵字時，將返回具有最相關關鍵字的影片。

[YouTube 搜尋如何工作的抽象](./search.jpg)

上述方法過於簡單，關鍵字的相關性並不是影響 YouTube 搜尋的唯一因素。實際上，許多其他因素也很重要。處理引擎將通過過濾和排名影片來改善搜尋結果。它將利用其他因素，如觀看次數、影片觀看時間和上下文，以及使用者的歷史記錄，來改善搜尋結果。
