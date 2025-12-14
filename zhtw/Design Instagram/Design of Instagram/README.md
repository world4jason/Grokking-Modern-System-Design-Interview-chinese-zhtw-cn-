# Design of Instagram (Instagram 的設計)
## High-level design (高層設計)
我們的系統應該允許我們在高層次上上傳、查看和搜索圖片和影片。要上傳圖片和影片，我們需要存儲它們，在獲取時，我們需要從存儲中檢索資料。此外，使用者還應該被允許相互關注。

[Instagram 的高層設計](./hld.jpg)

## API design (API 設計)
本節描述使用者呼叫用來在 Instagram 上執行不同任務 (上傳、按讚和查看照片/影片) 的 API。我們將為這些任務實現 REST API。讓我們為以下每個功能開發 API：

- 發布照片和影片
- 關注和取消關注使用者
- 對貼文按讚或不按讚
- 搜尋照片和影片
- 生成動態消息 (news feed)

以下所有調用都將有一個 userID，唯一指定執行操作的使用者。我們只討論調用中的新參數。

### Post photos or videos (發布照片或影片)
POST 方法用於通過 /postMedia API 從使用者向伺服器發布照片/影片。/postMedia API 如下：

```
postMedia(userID, media_type, list_of_hashtags, caption)
```

Parameter (參數)       Description (描述)

media_type            它指示貼文中媒體的類型 (照片或影片)。

list_of_hashtags      它代表貼文中的所有主題標籤 (上限 30 個主題標籤)。

caption               這是使用者貼文中的文字 (上限為 2,200 個字元)。
```

### Follow and unfollow users (關注和取消關注使用者)
當使用者在 Instagram 上關注其他使用者時，使用 /followUser API。/followUser API 如下：

```
followUser(userID, target_userID)
```

Parameter (參數)       Description (描述)

target_userID         它指示要關注的使用者。
當使用者在 Instagram 上取消關注某人時，/unfollowUser API 使用相同的參數。

### Like or dislike posts (對貼文按讚或不按讚)
當使用者喜歡 Instagram 上某人的貼文時，使用 /likePost API。

```
likePost(userID, target_userID, post_id)
```

Parameter (參數)        Description (描述)

target_userID          它指定貼文被按讚的使用者。

post_id                它指定貼文的唯一 ID。
當使用者不喜歡 Instagram 上某人的貼文時，/dislikePost API 使用相同的參數。


### Search photos or videos (搜尋照片或影片)
當使用者使用關鍵字或主題標籤搜尋任何照片或影片時，使用 GET 方法。/searchPhotos API 如下：

```
searchPhotos(userID, keyword)
```

Parameter (參數)    Description (描述)

keyword            它指示使用者在搜尋欄中輸入的字串 (使用者名稱、主題標籤和地點)。
```

注意：Instagram 在搜尋特定關鍵字時會顯示覆蓋率最高的貼文 (按讚和觀看次數較多的貼文)。例如，如果使用者使用“London, United Kingdom”進行基於位置的搜尋，Instagram 將按從最大到最小覆蓋率的順序顯示貼文。資料將在滾動時加載，而不是顯示所有貼文。
```

### Generate news feed (生成動態消息)
當使用者通過 /viewNewsfeed API 查看其動態消息時，使用 GET 方法。/viewNewsfeed API 如下：

```
viewNewsfeed(userID, generate_timeline)
```

Parameter (參數)             Description (描述)

generate_timeline           它指示使用者請求生成動態消息的時間。Instagram 顯示上次動態消息請求和當前動態消息請求之間使用者未看過的貼文。
```


## Storage schema (儲存模式)
現在讓我們定義我們的資料模型：

### Relational or non-relational database (關聯式或非關聯式資料庫)
為我們的 Instagram 系統選擇合適的資料庫至關重要，但哪個是正確的選擇——SQL 還是 NoSQL？我們的資料本質上是關聯式的，我們需要資料的順序 (貼文應按時間順序出現)，即使在發生故障的情況下也不會有資料丟失 (資料持久性)。此外，在我們的情況下，我們將從關聯式查詢中受益，例如根據使用者 ID 獲取關注者或圖片。因此，基於 SQL 的資料庫滿足這些需求。

因此，我們將選擇關聯式資料庫並將我們的相關資料儲存在該資料庫中。


### Define tables (定義資料表)
在基本層面上，我們需要以下資料表：

- **Users (使用者)**：這儲存所有與使用者相關的資料，如 ID、姓名、電子郵件、簡介、位置、帳戶建立日期、上次登入時間等。

- **Followers (關注者)**：這儲存使用者的關係。在 Instagram 中，我們有一個單向關係，例如，如果使用者 A 接受來自使用者 B 的關注請求，使用者 B 可以查看使用者 A 的貼文，但反之則無效。

- **Photos (照片)**：這儲存所有與照片相關的資訊，如 ID、位置、標題、建立時間等。我們還需要保留使用者 ID 以確定哪張照片屬於哪個使用者。使用者 ID 是來自 users 資料表的外鍵。

- **Videos (影片)**：這儲存所有與影片相關的資訊，如 ID、位置、標題、建立時間等。我們還需要保留使用者 ID 以確定哪個影片屬於哪個使用者。使用者 ID 是來自 users 資料表的外鍵。

Question (問題)
我們應該將照片和影片儲存在哪裡？

Answer (答案)
我們將照片和影片儲存在 blob 儲存 (如 S3) 中，並將照片或影片的路徑保存在資料表中，因為在分散式儲存中保存較大資料效率更高。
下圖可視化了資料模型：

[Instagram 的資料模型](./model.jpg)

### Data estimation (資料估算)

讓我們弄清楚每個資料表將儲存多少資料。下面計算器中的每行大小 (以位元組為單位) 顯示了每個資料表的每行資料。它還計算指定計數所需的儲存空間。例如，5 億使用者的儲存需求為 111000 MB，單個使用者的 250 個關注者為 2000 MB，6000 萬張照片為 23640 MB，3500 萬個影片為 13790 MB。

您可以更改計算器中的值以觀察所需儲存的變化。這使我們可以估計資料在我們的資料表中增長的速度。

Table Name (資料表名稱)      Per row size (in bytes) (每行大小-位元組)    Count in Millions (以百萬計數)    Storage Needed (in MBs) (所需儲存-MB)
Users	                     222                                          500	                              111000
Followers	                   8                                            250                                 2000
Photos                       394                                           60	                              23640
Videos	                     394                                           35                                 13790
```

注意：大多數現代服務同時使用 SQL 和 NoSQL 儲存。Instagram 官方使用 SQL (PostgreSQL) 和 No-SQL (Cassandra) 資料庫的組合。像時間線生成這樣的鬆散結構資料通常儲存在 No-SQL 中，而關聯式資料保存在基於 SQL 的儲存中。
在下一課中，我們將確定更多組件來調整我們的設計。
