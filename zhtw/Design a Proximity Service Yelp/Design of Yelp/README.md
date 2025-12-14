# Design of Yelp (Yelp 的設計)
我們在上一課中確定了需求並計算了我們的 Yelp 系統的估算值。在本節課中，我們將討論API設計，瀏覽存儲模式，然後深入探討系統的構建塊和附加組件的細節。

## API design (API 設計)
讓我們討論 Yelp 的 API 設計。

### Search (搜尋)
我們需要實現搜尋功能。基於“咖啡館”等類別進行搜尋的 API 調用將是：
```
search(category, user_location, radius)
```
Parameter (參數)          Description (描述)

category (類別)           This is the type of search the user makes—for example, a search for restaurants, cinemas, cafes, and so on. (這是使用者進行的搜尋類型——例如，搜尋餐館、電影院、咖啡館等。)

user_location (使用者位置)      This contains the location of the user who’s searching with Yelp. (這包含使用 Yelp 搜尋的使用者的位置。)

radius (半徑)             This is the specified radius where the user is trying to find the required category. (這是使用者試圖查找所需類別的指定半徑。)
此過程返回一個 JSON 對象，其中包含指定類別中也落在指定半徑內的所有可能項目的列表。每個條目都有地名、地址、類別、評級和縮略圖。

基於“Burger Hut”等地名進行搜尋的 API 調用將是：
```
search(name_of_place, user_location, radius)
```

Parameter (參數)          Description (描述)

name_of_place (地名)       This contains the name of the place that the user wants to search for. (這包含使用者想要搜尋的地點的名稱。)
此過程返回一個包含指定地點訊息的 JSON 對象。

### Add a place (添加地點)
添加地點的 API 調用如下：
```
add_place(name_of_place, description_of_place, category, latitude, longitude, photo}
```

Parameter (參數)                 Description (描述)

name_of_place (地名)             This contains the name of the place, for example, "Burger Hut". (這包含地點的名稱，例如，“Burger Hut”。)

description_of_place (地點描述)      This contains a description of the place. For example, "Burger Hut sells the yummiest burgers". (這包含地點的描述。例如，“Burger Hut 賣最美味的漢堡”。)

category (類別)                  This specifies the category of the place—for example, "cafe". (這指定地點的類別——例如，“咖啡館”。)

latitude (緯度)                  This tells us the latitude of the place. (這告訴我們地點的緯度。)

longitude (經度)                 This tells us the longitude of the place. (這告訴我們地點的經度。)

photo (照片)                     This contains photos of the place. There can be a single or multiple photos. (這包含地點的照片。可以有一張或多張照片。)
此過程返回一個響應，說明已添加地點，如果添加地點失敗，則返回適當的錯誤。

### Add a review (添加評論)
添加評論的 API 調用如下：
```
add_review(place_ID, user_ID, review_description, rating)
```

Parameter (參數)             Description (描述)

place_ID (地點 ID)              This contains the ID of the place whose review is added. (這包含添加評論的地點的 ID。)

user_ID (使用者 ID)               This contains the ID of the user who adds the review. (這包含添加評論的使用者的 ID。)

review_description (評論描述)    This contains the review of the place—for example, "the food and ambiance were superb". (這包含地點的評論——例如，“食物和氛圍都很棒”。)

rating (評級)                This contains the rating of the place—for example, 4 out of 5. (這包含地點的評級——例如，5 分中的 4 分。)
此過程返回一個響應，說明已添加評論，如果添加評論失敗，則返回適當的錯誤。

## Storage schema (儲存模式)
讓我們定義我們系統的儲存模式。我們可能需要的一些表是“Place”、“Photos”、“Reviews”和“Users”。

讓我們定義“Place”表的列：

- **Place_ID**：我們使用序列生成器為地點生成一個 8 字節 (64 位) 的唯一 ID。
```
注意：我們使用唯一 ID 生成器生成 ID。
- **Name_of_Place**：這是一個包含地名的字符串。我們使用 256 字節。

- **Description_of_Place**：這保存地點的描述。我們使用 1,000 字節。

- **Category**：這指定地點的類型，如餐館、電影院、書店等 (8 字節)。

- **Latitude**：這存儲位置的緯度 (8 字節)。

- **Longitude**：這存儲位置的經度 (8 字節)。

- **Photos**：這包含另一個表“Photos”的外鍵 (8 字節)，該表包含與特定地點相關的所有照片。

- **Rating**：這存儲地點的評級。它顯示一個地點在五顆星中獲得了多少顆星。評級是根據使用者對其的評論計算得出的。

上面提到的列是表中最重要的列。我們可以添加更多列，如“菜單”、“地址”、“營業時間”等。因此，考慮到基本列，我們表的一行的大小將是：
```
Size = 8 + 256 + 1000 + 8 + 8 + 8 + 8 = 1296 bytes 
現在讓我們定義“Photos”表：

- **Photo_ID**：我們使用序列生成器為照片生成一個唯一 ID (8 字節或 64 位)。

- **Place_ID**：我們使用“Place”表中的外鍵 (8 字節) 來識別哪張照片屬於哪個地點。

- **Photo_path**：我們將照片存儲在 blob 存儲中，並將照片的路徑 (256 字節) 保存在此列中。
```
Size = 8 + 8 + 8 + 256 = 280 bytes 
我們需要另一個名為“Reviews”的表來存儲地點的評論、評級和照片。

- **Review_ID**：我們使用序列生成器為評論生成一個 8 字節 (64 位) 的唯一 ID。

- **Place_ID**：來自“Place”表的外鍵 (8 字節)，用於確定評級屬於哪個地點。

- **User_ID**：來自“Users”表的外鍵 (8 字節)，用於識別哪條評論屬於哪個使用者。

- **Review_description**：這保存評論的描述。我們使用 512 字節。

- **Rating**：這存儲一個地點在五顆星中獲得了多少顆星 (1 字節)。
```
Size = 8 + 8 + 8 + 512 + 1 = 537 bytes 
我們使用“Users”表來存儲使用者訊息。

- **User_ID**：我們使用序列生成器為使用者生成一個唯一 ID (8 字節)。

- **User_name**：這是一個包含使用者名稱的字符串。我們使用 256 字節。
```
Size = 8 + 256 = 264 bytes 
```

```
注意：以下模式中的 INT 包含我們使用唯一 ID 生成器生成的 8 字節 ID。
[儲存模式](./schema.jpg)

## Design (設計)
現在我們將討論 Yelp 設計中使用的各個構建塊和組件，以及它們如何協同工作以完成各種功能需求。

### Components (組件)
這些是我們系統的組件：

- **Segments producer (片段生產者)**：此組件負責與第三方世界地圖資料服務 (例如 Google Maps) 進行通訊。它獲取該資料並將世界劃分為稱為片段的較小區域。片段生產者有助於我們縮小要搜尋的地點數量。

- **QuadTree servers (四元樹伺服器)**：這些是一組伺服器，其樹包含片段中的地點。四元樹伺服器根據給定的半徑和使用者提供的位置查找地點列表，並將該列表返回給使用者。此組件主要有助於搜尋功能。

- **Aggregators (聚合器)**：四元樹積累所有地點並將其發送給聚合器。然後，聚合器聚合結果並將搜尋結果返回給使用者。

- **Read servers (讀取伺服器)**：我們使用一組讀取伺服器來處理所有讀取請求。由於我們有更多的讀取請求，因此將這些請求與寫入請求分開是有效的。每個讀取伺服器將搜尋請求定向到四元樹的伺服器，並將結果返回給使用者。

- **Write server (寫入伺服器)**：我們使用一組寫入伺服器來處理所有寫入請求。每個寫入伺服器處理使用者的寫入請求並相應地更新存儲。寫入請求的示例包括添加地點、撰寫評論、對地點進行評級等。

- **Storage (儲存)**：我們將使用兩種類型的存儲來滿足我們的多樣化需求。

  - **SQL database (SQL 資料庫)**：我們的系統將有不同的表，如“Users”、“Place”、“Reviews”、“Photos”等，如下所述。這些表中的資料本質上是關係型和結構化的。我們需要執行查詢，如使用者訪問過的地點、他們添加的評論或查看特定地點的所有評論。在基於 SQL 的資料庫中執行此類查詢很容易。我們還希望所有使用者對由於具有一致的資料視圖，基於 SQL 的資料庫更適合此類用例。我們將使用可靠且可擴展的資料庫，正如在資料庫構建塊中所討論的那樣。

  - **Key-value stores (鍵值存儲)**：我們需要有效地獲取片段中的地點。為此，我們將地點列表針對片段 ID 存儲在鍵值存儲中，以最大限度地減少搜尋時間。我們還通過針對唯一 ID 存儲四元樹資料，將四元樹訊息保存在鍵值存儲中。

- **Load balancer (負載平衡器)**：負載平衡器將使用者的傳入請求均勻地分發到所有伺服器。

[Yelp 設計](./design.jpg)

### Workflow (工作流程)
使用者發出搜尋請求。我們在考慮使用者位置 (緯度、經度) 的同時，查找給定半徑內的所有相關地點。

我們在下面根據所需功能解釋我們系統的詳細工作流程：

**Searching a place (搜尋地點)**：負載平衡器在收到讀取請求後將其路由到讀取伺服器。讀取伺服器將它們定向到四元樹伺服器，以查找落在給定半徑內的所有地點。然後，四元樹伺服器將結果發送到聚合器以對其進行細化並將其發送給使用者。

**Adding a place or feedback (添加地點或反饋)**：負載平衡器在收到寫入請求後將其路由到寫入伺服器。根據提供的內容 (即地點訊息或評論)，寫入伺服器在關係資料庫中添加條目並將所有相關圖像放入 blob 存儲中。

**Making segments (製作片段)**：片段生產者將從第三方地圖服務獲取的世界地圖拆分為更小的片段。每個片段內的地點都存儲在鍵值存儲中。儘管這是一次性工作，但對於較新的片段和地點，此過程會定期重複。由於添加新地點的概率很低，我們每個月更新一次片段。

我們已經討論了 Yelp 的設計、其 API 設計以及相關的儲存模式。在下一課中，我們將討論設計考慮因素。
