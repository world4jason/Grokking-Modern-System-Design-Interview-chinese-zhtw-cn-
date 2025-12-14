# High-level Design of Twitter (Twitter 的高層設計)
## User-system interaction (使用者-系統互動)
讓我們從 Twitter 系統的高層設計開始。我們將首先簡要地強調和討論 Twitter 問題背景下的構建塊以及其他組件。稍後，我們將在本章中深入探討幾個組件。

[Twitter 組件](./components.jpg)

- **Users (使用者)** 發布推文，推文通過負載平衡器傳遞到伺服器。然後，系統將其儲存在持久儲存中。

- **DNS** 為最終使用者提供指定的 IP 地址，以開始與請求的服務進行通訊。

- **CDN** 位於使用者附近，以低延遲提供請求的資料。當使用者搜尋指定的術語或標籤時，系統首先在包含最常請求內容的 CDN 代理伺服器中進行搜尋。

- **Load balancer (負載平衡器)** 根據可用伺服器上的流量負載和使用者請求選擇可操作的應用程式伺服器。

- **Storage system (儲存系統)** 在上圖中代表各種類型的儲存 (基於 SQL 和基於 NoSQL)。我們將在本章後面討論重要的儲存系統。

- **Application servers (應用程式伺服器)** 提供各種服務，並具有業務邏輯來在不同組件之間進行編排，以滿足我們的功能需求。

我們在構建塊部分有關於 DNS、CDN、指定儲存系統 (資料庫、鍵值儲存、Blob 儲存) 和負載平衡器的詳細章節。我們將在接下來的課程中重點介紹特定於 Twitter 服務的更多細節。讓我們先了解服務 API。

## API design (API 設計)
本節將著重於設計關於我們提供的功能的各種 API。我們學習使用者如何通過 API 請求各種服務。我們將僅關注與我們的設計相關的 API 的重要參數。儘管前端伺服器可以調用另一個 API 或在從最終使用者接收的 API 中新增更多參數以滿足給定請求，但我們將在單個 API 中考慮為特定請求指定的所有相關參數。讓我們為以下每個功能開發 API：

- 發布推文
- 按讚或按不喜歡推文
- 回覆推文
- 搜尋推文
- 查看使用者或首頁時間線
- 關注或取消關注帳戶
- 轉發推文

### Post Tweet (發布推文)
POST 方法用於通過 /postTweet API 將推文從使用者發送到伺服器。
```
postTweet(user_id, access_type, tweet_type, content, tweet_length, media_field, list_of_followers, post_time, tweet_location, list_of_used_hashtags, list_of_tagged_people)
```

讓我們討論幾個參數：
Parameter (參數)             Description (描述)

user_id                     它指示發布推文的使用者的唯一 ID。

access_type                 它告訴我們推文是受保護的 (即僅對追蹤者可見) 還是公開的。

tweet_type                  它指示推文是基於文字、基於影片剪輯、基於圖像，還是由不同類型組成。

content                     它指定推文的實際內容 (文字)。

tweet_length                它表示推文中的文字長度。在影片的情況下，它告訴我們影片的持續時間和大小。

media_field                 它指定在每個推文中交付的媒體類型 (圖像、影片、GIF 等)。

list_of_followers           它提供發布推文的帳戶的當前追蹤者。此參數由前端伺服器在使用另一個 API 獲取此資訊後填充。

其餘參數不言自明。

```
注意：Twitter 使用 Snowflake 服務為推文生成唯一 ID。我們有一個詳細的章節 (定序器) 解釋此服務。
```

```
Question 1
一條推文最多可以有多少個標籤？

Answer
一條推文中的文字限制為 280 個字元。因此，使用者可以使用標籤，只要文字長度 (包括純文字、任何連結和標籤) 不超過 280 個字元的限制。
```

```
Question 2
為什麼在 /postTweet API 中需要 list_of_tagged_people？

Answer
系統必須通知在推文中標記的人。
```

### Like or dislike Tweet (按讚或按不喜歡推文)
當使用者按讚公開推文時，使用 /likeTweet API。
```
likeTweet(user_id, tweet_id, tweeted_user_id, user_location)
```
Parameter (參數)           Description (描述)

user_id                   它指示按讚推文的使用者的唯一 ID。

tweet_id                  它指示推文的唯一 ID。

tweeted_user_id           這是發布推文的使用者的唯一 ID。

user_location             它表示按讚推文的使用者的位置。
當使用者不喜歡別人的推文時，上面的參數也用於 /dislikeTweet API。

### Reply to Tweet (回覆推文)
當使用者回覆公開推文時，使用 /replyTweet API。
```
replyTweet(user_id, tweet_id, tweeted_user_id, reply_type, reply_length, ,list_of_followers)
```
Parameter (參數)             Description (描述)

list_of_followers           它指定回覆某人推文的使用者追蹤者列表。
reply_type 和 reply_length 參數分別與 tweet_type 和 tweet_length 相同。

### Search Tweet (搜尋推文)
當使用者在首頁時間線中搜尋任何關鍵字時，使用 GET 方法。以下是 /searchTweet API：
```
searchTweet(user_id, search_term, max_result, exclude, media_field, expansions, sort_order, next_token, user_location)
在這種情況下引入的一些新參數是：
Parameter (參數)             Description (描述)

search_term                 它是包含搜尋關鍵字或短語的字串。

max_result                  它是每個響應頁面返回的推文數量。預設情況下，每個響應的推文為 10。

exclude                     它指定從返回的推文中排除什麼，即回覆和轉發。返回推文的最大限制為 3200，但是當我們排除回覆時，最大限制減少到 800 條推文。

media_field                 它指定在每個返回的推文中交付的媒體 (圖像、影片、GIF)。

expansions                  它使我們能夠請求返回推文中的其他資料對象，例如任何提到的使用者、引用的推文、附加媒體、附加地點對象等。

sort_order                  它指定返回推文的順序。預設情況下，它將首先返回最近的推文。

next_token                  它用於獲取下一頁結果。例如，如果 max_result 設置為 100 條推文，並且結果集包含 200 條推文，則直接從響應中提取 next_token 的值以請求包含接下來的 100 條推文的下一頁。最後一個結果 (頁面) 將沒有 next_token。
#### Response (響應)
讓我們看一個 JSON 格式的示例響應。id 是發布推文的使用者的唯一 ID，text 是推文的內容。result_count 是返回推文的計數，我們在請求中的 max_result 中設置了該計數。在這裡，我們僅顯示預設欄位。

```json
{
  "data": [
    {
      "id": "7300333948034441183",
      "text": "This is a most matched Tweet"
    },
    {
      "id": "6498431343154916376",
      "text": "This is next to most matched Tweet"
    },
                   : 
                   :
                   :
    {
      "id": "6427456107642019844",
      "text": "This is the last matched Tweet in the current page of the result.
    }
  ],
  "meta": {
    "newest_id": "7300333948034441183",
    "oldest_id": "6427456107642019844",
    "result_count": 10
  }
}
```

```
注意：Twitter 執行各種類型的搜尋。以下是其中的兩種：

一種搜尋類型返回過去七天的結果，所有註冊使用者通常都使用該類型。

另一種類型返回所有曾經發布的推文上的所有匹配結果 (提醒服務不會刪除已發布的推文)。實際上，匹配項可以包含 Twitter 上的第一條推文。此搜尋通常用於學術研究。
```

### View home_timeline (查看首頁時間線)
當使用者通過 /viewHome_timeline API 查看其首頁時間線時，GET 方法是合適的。
```
viewHome_timeline(user_id, tweets_count, max_result, exclude, next_tokan, list_of_followers, user_location)
在 /viewUser_timeline API 中，我們將排除 list_of_followers 和 user_location 以獲取使用者時間線。

```
Question
決定在響應中返回哪些推廣廣告 (推文) 時，viewHome_timeline 方法中的哪個參數最相關？

Answer
發送指定推廣廣告的決定是根據 user_location 參數做出的。例如，使用者屬於紐約市，因此極有可能獲得與該地區相關或源自該地區的推廣廣告。
```

### Follow the account (關注帳戶)
當使用者在 Twitter 上關注某人的帳戶時，使用 /followAccount API。
```
followAccount(account_id, followed_account_id,)
```
Parameter (參數)                Description (描述)

account_id                     它指定在 Twitter 上關注該帳戶的使用者的唯一 ID。

followed_account_id            它指示使用者關注的帳戶的唯一 ID。
當使用者在 Twitter 上取消關注某人的帳戶時，/unfollowAccount API 將使用相同的參數。

### Retweet a Tweet (轉發推文)
當註冊使用者在 Twitter 上轉發 (重新發布) 某人的推文時，將調用以下 /retweet API：
```
retweet(user_id, tweet_id, retweet_user_id, list_of_followers)
當使用者撤消某人推文的轉發時，/undoRetweet API 將需要相同的參數。
