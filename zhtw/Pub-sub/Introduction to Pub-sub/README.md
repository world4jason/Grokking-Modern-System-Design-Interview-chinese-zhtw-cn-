# Introduction to Pub-sub (Pub-sub 簡介)
Pub-sub 消息傳遞提供異步通信。讓我們探討擁有 pub-sub 系統有益的用例。

## Use cases of pub-sub (Pub-sub 的用例)
下面列出了一些 pub-sub 的用例：

- **Improved performance (提高性能)**: The pub-sub system enables push-based distribution, alleviating the need for message recipients to check for new information and changes regularly. It encourages faster response times and lowers the delivery latency.
- **提高性能**：Pub-sub 系統支持基於推送的分發，減輕了消息接收者定期檢查新資訊和更改的需求。它鼓勵更快的響應時間並降低交付延遲。

- **Handling ingestion (處理攝取)**: The pub-sub helps in handling log ingestion. The user-interaction data can help us figure out useful analyses about the behavior of users. We can ingest a large amount of data to the pub-sub system, so much so that it can deliver the data to any analytical system to understand the behavior patterns of users. Moreover, we can also log the details of the event that’s happening while completing a request from the user. Large services like Meta use a pub-sub system called Scribe to know exactly who needs what data, and remove or archive processed or unwanted data. Doing this is necessary to manage an enormous amount of data.
- **處理攝取**：Pub-sub 有助於處理日誌攝取。使用者交互資料可以幫助我們找出關於使用者行為的有用分析。我們可以將大量資料攝取到 pub-sub 系統中，以至於它可以將資料傳遞給任何分析系統以了解使用者的行為模式。此外，我們還可以記錄在完成使用者請求時發生的事件的詳細資訊。像 Meta 這樣的大型服務使用名為 Scribe 的 pub-sub 系統來確切知道誰需要什麼資料，並刪除或歸檔已處理或不需要的資料。這樣做對於管理大量資料是必要的。

- **Real-time monitoring (實時監控)**: Raw or processed messages of an application or system can be provided to multiple applications to monitor a system in real time.
- **實時監控**：可以向多個應用程式提供應用程式或系統的原始或處理後的消息，以實時監控系統。

- **Replicating data (複製資料)**: The pub-sub system can be used to distribute changes. For example, in a leader-follower protocol, the leader sends the changes to its followers via a pub-sub system. It allows followers to update their data asynchronously. The distributed caches can also refresh themselves by receiving the modifications asynchronously. Along the same lines, applications like WhatsApp that allow multiple views of the same conversation—for example, on a mobile phone and a computer’s browser—can elegantly work using a pub-sub, where multiple views can act either as a publisher or a subscriber.
- **複製資料**：Pub-sub 系統可用於分發更改。例如，在領導者-跟隨者協議中，領導者通過 pub-sub 系統將更改發送給其跟隨者。它允許跟隨者異步更新他們的資料。分佈式緩存也可以通過異步接收修改來刷新自己。同樣，像 WhatsApp 這樣的應用程式允許對同一對話進行多個視圖 (例如，在手機和計算機瀏覽器上)，可以使用 pub-sub 優雅地工作，其中多個視圖可以充當發布者或訂閱者。

```
日誌攝取是指將日誌導入資料庫以供立即使用，例如監控。像 Facebook 這樣的大型應用程式會產生大量的資料和日誌。日誌的實時獲取和處理使我們能夠運行分析並在發生故障時採取快速行動。
```

[Pub-sub 系統的動機和用例](./real-time.jpg)

```
Question 1
Pub-sub 系統和隊列有什麼異同？

Answer
Pub-sub 系統和隊列很相似，因為它們將生產者生成的資訊傳遞給消費者。區別在於隊列中只有一個消費者消費一條消息，而 pub-sub 系統中同一條消息可以有多個消費者。
```

```
Question 2
在 pub-sub 系統中，生產者和消費者如何彼此解耦？

Answer
生產者不知道誰最終會閱讀他們的資訊。他們只是將其發送到系統，然後由消費者讀取。生產者不受緩慢的消費者、消費者數量或消費者故障的影響。我們可以獨立擴展它們。
```

## Requirements (需求)
我們旨在設計一個具有以下要求的 pub-sub 系統。

### Functional requirements (功能需求)
讓我們指定 pub-sub 系統的功能需求：

- **Create a topic (創建主題)**: The producer should be able to create a topic.
- **創建主題**：生產者應該能夠創建主題。

- **Write messages (寫消息)**: Producers should be able to write messages to the topic.
- **寫消息**：生產者應該能夠向主題寫消息。

- **Subscription (訂閱)**: Consumers should be able to subscribe to the topic to receive messages.
- **訂閱**：消費者應該能夠訂閱主題以接收消息。

- **Read messages (讀消息)**: The consumer should be able to read messages from the topic.
- **讀消息**：消費者應該能夠從主題讀取消息。

- **Specify retention time (指定保留時間)**: The consumers should be able to specify the retention time after which the message should be deleted from the system.
- **指定保留時間**：消費者應該能夠指定保留時間，在此之後消息應從系統中刪除。

```
主題是相關事件或消息的集合。生產者可以創建主題並將消息寫入主題。消費者可以訂閱主題並從該主題讀取。
```

- **Delete messages (刪除消息)**: A message should be deleted from the topic or system after a certain retention period as defined by the user of the system.
- **刪除消息**：根據系統使用者定義的保留期，應從主題或系統中刪除消息。

### Non-functional requirements (非功能需求)
在設計 pub-sub 系統時，我們考慮以下非功能需求：

- **Scalable (可擴展)**: The system should scale with an increasing number of topics and increasing writing (by producers) and reading (by consumers) load.
- **可擴展**：系統應隨著主題數量的增加以及寫入 (由生產者) 和讀取 (由消費者) 負載的增加而擴展。

- **Available (可用)**: The system should be highly available, so that producers can add their data and consumers can read data from it anytime.
- **可用**：系統應具有高可用性，以便生產者可以隨時添加資料，消費者可以隨時從中讀取資料。

- **Durability (持久性)**: The system should be durable. Messages accepted from producers must not be lost and should be delivered to the intended subscribers.
- **持久性**：系統應該是持久的。從生產者接收的消息不得丟失，並應傳遞給預期的訂閱者。

- **Fault tolerance (容錯)**: Our system should be able to operate in the event of failures.
- **容錯**：我們的系統應該能夠在發生故障時運行。

- **Concurrent (並發)**: The system should handle concurrency issues where reading and writing are performed simultaneously.
- **並發**：系統應處理同時執行讀寫操作的並發問題。

## API Design (API 設計)
我們將排除下面函數中的一些參數，例如生產者或消費者的標識符。讓我們假設此資訊可從底層連接上下文中獲得。此問題的 API 設計如下：

**Create a topic (創建主題)**

創建主題的 API 調用應如下所示：
```
create(topic_ID, topic_name)
```

如果成功創建主題，此函數返回確認，如果失敗則返回錯誤。

```
Parameter         Description                             
topic_ID          唯一標識主題。

topic_name        它包含主題的名稱。
```

**Read a message (讀取消息)**

從系統讀取資料的 API 調用應如下所示：
```
read(topic_ID)
```
使用 topic_ID 找到主題，該調用將返回包含消息的對象給調用者。

```
Parameter   Description
topic_ID    將從中讀取消息的主題 ID。
```

**Subscribe to a topic (訂閱主題)**

從系統訂閱主題的 API 調用應如下所示：
```
subscribe(topic_ID)
```
該函數將消費者添加為具有 topic_ID 的主題的訂閱者。
```
Parameter   Description
topic_ID    消費者將訂閱的主題 ID。
```


**Unsubscribe from a topic (取消訂閱主題)**

從系統取消訂閱主題的 API 調用應如下所示：
```
unsubscribe(topic_ID)
```
該函數將消費者從具有 topic_ID 的主題中移除訂閱。
```
Parameter    Description
topic_ID     消費者將取消訂閱的主題 ID。
```

**Delete a topic (刪除主題)**

從系統刪除主題的 API 調用應如下所示：
```
delete_topic(topic_ID)
```
該函數根據 topic_ID 刪除主題。
```
Parameter   Description
topic_ID    要刪除的主題 ID。
```

## Building blocks we will use (我們將使用的構建塊)

Pub-sub 的設計利用了前面章節中討論的許多構建塊。我們將考慮以下關於構建塊的課程。

[我們將使用的構建塊](./bb.jpg)

- **Database (資料庫)**: We’ll use databases to store information like subscription details.
- **資料庫**：我們將使用資料庫來存儲訂閱詳細資訊等資訊。
- **Distributed messaging queue (分佈式消息隊列)**: We’ll use use a messaging queue to store messages sent by the producer.
- **分佈式消息隊列**：我們將使用消息隊列來存儲生產者發送的消息。
- **Key-value (鍵值)**: We’ll use a key-value store to hold information about consumers.
- **鍵值**：我們將使用鍵值存儲來保存有關消費者的資訊。

在下一課中，我們將專注於設計 pub-sub 系統。
