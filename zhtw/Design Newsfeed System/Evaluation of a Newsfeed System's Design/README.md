# Evaluation of a Newsfeed System’s Design (評估動態消息系統的設計)
## Fulfill requirements (滿足需求)
我們對提議的動態消息系統設計的非功能需求是可擴展性、容錯性、可用性和低延遲。讓我們討論提議的系統如何滿足這些需求：

1. **Scalability (可擴展性)**：提議的系統具有可擴展性，可以處理不斷增長的使用者數量。所需的資源，包括負載平衡器、Web 伺服器和其他相關伺服器，按需新增/刪除。

2. **Fault tolerance (容錯性)**：由使用者元資料、貼文和動態消息組成的資料複製使系統具有容錯性。此外，冗餘資源始終存在，以處理伺服器或其組件的故障。

3. **Availability (可用性)**：系統通過提供冗餘伺服器並在伺服器上複製資料來實現高可用性。當使用者由於伺服器中的某些故障而斷開連接時，會通過負載平衡器與不同的伺服器重新建立連線。此外，資料 (使用者元資料、貼文和動態消息) 儲存在不同且冗餘的資料庫集群中，這提供了高可用性和持久性。

4. **Low latency (低延遲)**：我們可以通過以下方式最小化各個級別的系統延遲：
  - 地理位置分佈的伺服器及其關聯的快取。這樣，我們將服務帶到了使用者附近。
  - 為經常訪問的動態消息和媒體內容使用 CDN。

## Quiz on the newsfeed system’s design (動態消息系統設計測驗)
通過測驗測試您對動態消息系統設計相關概念的理解。

```
哪個組件負責儲存使用者、朋友和追蹤者之間的關係？

The users database (使用者資料庫)

The graph database (圖資料庫)

The posts database (貼文資料庫)

None of the above (以上均非)
```

```
完全構建和排名的動態消息儲存在哪裡？

The published feeds cache (已發布動態快取)

The newsfeed cache (動態消息快取)

The posts cache (貼文快取)

None of the above (以上均非)
```

```
Web 伺服器何時調用貼文服務？

When new posts are created (建立新貼文時)

When a request for a newsfeed is received (收到動態消息請求時)

When users are to be notified due to some failure events (當由於某些故障事件需要通知使用者時)

All of the above (以上均是)
```

```
以下哪個選項儲存在 blob 儲存中？

Comments on posts (貼文評論)

Count of likes and dislikes (按讚和不按讚的計數)

Media content (媒體內容)

All of the above (以上均是)
```

Answers (答案)
1. (B)

2. (B)

3. (A)

4. (C)

## Summary (總結)
在本章中，我們學習了大規模設計動態消息系統。我們的設計對海量使用者資料進行排名，以向使用者展示精心策劃的內容，從而獲得更好的使用者體驗和參與度。我們的動態消息設計足夠通用，可用於許多地方，例如 Twitter 動態、Facebook 貼文、YouTube 和 Instagram 推薦、新聞應用程式等。
