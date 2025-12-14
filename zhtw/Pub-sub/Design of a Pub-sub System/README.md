# Design of a Pub-sub System (Pub-sub 系統的設計)

## First design (第一種設計)
在上一課中，我們討論了生產者寫入主題，消費者訂閱主題以從該主題讀取消息。由於新消息添加到隊列的末尾，我們可以將分佈式消息隊列用於主題。

我們需要的組件如下所列：

- **Topic queue (主題隊列)**: Each topic will be a distributed messaging queue so we can store the messages sent to us from the producer. A producer will write their messages to that queue.
- **主題隊列**：每個主題都將是一個分佈式消息隊列，以便我們可以存儲生產者發送給我們的消息。生產者將其消息寫入該隊列。

- **Database (資料庫)**: We’ll use a relational database that will store the subscription details. For example, we need to store which consumer has subscribed to which topic so we can provide the consumers with their desired messages. We’ll use a relational database since our consumer-related data is structured and we want to ensure our data integrity.
- **資料庫**：我們將使用關係資料庫來存儲訂閱詳細資訊。例如，我們需要存儲哪個消費者訂閱了哪個主題，以便我們可以為消費者提供他們想要的消息。我們將使用關係資料庫，因為我們與消費者相關的資料是結構化的，我們要確保資料的完整性。

- **Message director (消息導向器)**: This service will read the message from the topic queue, fetch the consumers from the database, and send the message to the consumer queue.
- **消息導向器**：此服務將從主題隊列讀取消息，從資料庫中獲取消費者，並將消息發送到消費者隊列。

- **Consumer queue (消費者隊列)**: The message from the topic queue will be copied to the consumer’s queue so the consumer can read the message. For each consumer, we’ll define a separate distributed queue.
- **消費者隊列**：來自主題隊列的消息將被複製到消費者的隊列中，以便消費者可以讀取消息。對於每個消費者，我們將定義一個單獨的分佈式隊列。

- **Subscriber (訂閱者)**: When the consumer requests a subscription to a topic, this service will add an entry into the database.
- **訂閱者**：當消費者請求訂閱主題時，此服務將在資料庫中添加一個條目。

消費者將訂閱主題，系統將訂閱者的詳細資訊添加到資料庫中。生產者將寫入主題，消息導向器將從隊列中讀取消息，獲取應將消息添加到誰的詳細資訊，並將其發送給他們。消費者將從他們的隊列中消費消息。

```
注意：我們將對消息導向器和訂閱者使用故障轉移服務，以防止故障。
```

[使用分佈式消息隊列](./using.jpg)

使用分佈式消息隊列使我們的設計變得簡單。然而，所需的大量隊列是一個重大問題。如果我們有成千上萬個主題的數百萬訂閱者，那麼定義和維護數百萬個隊列是昂貴的。此外，我們將在所有訂閱者隊列中複製主題的同一條消息，這是不必要的重複並佔用空間。

```
Question 1
有沒有辦法避免為每個讀者維護單獨的隊列？

Answer
在消息隊列中，消息在讀者消費後消失。那麼，如果我們為每條消息添加一個計數器呢？計數器值隨著訂閱者消費消息而遞減。在計數器變為零之前，它不會刪除消息。現在，我們不需要為每個讀者保留單獨的隊列。
```

```
Question 2
前一種方法有什麼問題？

Answer
如果我們使用傳統的隊列 API，未讀消息可能會成為瓶頸。例如，如果 10 個讀者中有 9 個已經消費了隊列開頭的消息，那麼在第 10 個消費者也消費該消息之前，該消息不會被刪除，並且前 9 個消費者將無法前進。

我們需要更改存儲接口，以便消費者可以獨立地消費資料。我們的系統將需要保留足夠的元資料並跟踪每個消費者消費了什麼資訊，並在所有消費者都消費了該資訊時刪除消息。它類似於 Linux 文件硬鏈接中的引用計數機制。
```

## Second design (第二種設計)
讓我們考慮設計 pub-sub 系統的另一種方法。

### High-level design (高層設計)
在高層，pub-sub 系統將具有以下組件：

- **Broker (代理)**: This server will handle the messages. It will store the messages sent from the producer and allow the consumers to read them.
- **代理**：此伺服器將處理消息。它將存儲生產者發送的消息，並允許消費者讀取它們。

- **Cluster manager (集群管理器)**: We’ll have numerous broker servers to cater to our scalability needs. We need a cluster manager to supervise the broker’s health. It will notify us if a broker fails.
- **集群管理器**：我們將擁有眾多代理伺服器以滿足我們的擴展需求。我們需要一個集群管理器來監督代理的健康狀況。如果代理發生故障，它會通知我們。

- **Storage (存儲)**: We’ll use a relational database to store consumer details, such as subscription information and retention period.
- **存儲**：我們將使用關係資料庫來存儲消費者詳細資訊，例如訂閱資訊和保留期。

- **Consumer manager (消費者管理器)**: This is responsible for managing the consumers. For example, it will verify if the consumer is authorized to read a message from a certain topic or not.
- **消費者管理器**：這負責管理消費者。例如，它將驗證消費者是否有權從特定主題讀取消息。

除了這些組件之外，我們還有以下設計注意事項：

- **Acknowledgment (確認)**: An acknowledgment is used to notify the producer that the received message has been stored successfully. The system will wait for an acknowledgment from the consumer if it has successfully consumed the message.
- **確認**：確認用於通知生產者已成功存儲接收到的消息。如果消費者成功消費了消息，系統將等待消費者的確認。

- **Retention time (保留時間)**: The consumers can specify the retention period time of their messages. The default will be seven days, but it is configurable. Some applications like banking applications require the data to be stored for a few weeks as a business requirement, while an analytical application might not need the data after consumption.
- **保留時間**：消費者可以指定其消息的保留期。預設值為 7 天，但它是可配置的。一些應用程式 (如銀行應用程式) 需要將資料存儲數週作為業務需求，而分析應用程式可能在消費後不需要資料。

[Pub-sub 系統的高層設計](./high_level_design.jpg)

讓我們詳細了解每個組件的作用。

## Broker (代理)
代理伺服器是我們 pub-sub 系統的核心組件。它將處理讀寫請求。一個代理將有多個主題，其中每個主題可以有多個與之關聯的分區。我們使用分區將消息存儲在本地存儲中以實現持久性。因此，這提高了可用性。分區包含封裝在段中的消息。段有助於使用偏移地址識別消息的開始和結束。使用段，消費者通過從特定偏移地址讀取來從分區消費他們選擇的消息。下圖描繪了上面描述的概念。

[描述消息如何存儲在分區內的段中的插圖](./how_messages_are_stored.jpg)

眾所周知，主題是存儲在代理本地存儲中的持久消息序列。將資料添加到主題後，就無法對其進行修改。從主題讀取和寫入消息對於計算機來說是一項 I/O 任務，擴展此類任務具有挑戰性。這就是我們將主題拆分為多個分區的原因。屬於單個主題的資料可以存在於眾多分區中。例如，假設我們有主題 A，我們為其分配了三個分區。生產者將把他們的消息發送到相關主題。接收到的消息將根據循環算法發送到各個分區。我們將使用循環的變體：加權循環。以下幻燈片顯示了消息如何存儲在屬於單個主題的各個分區中。

[代理](./how_broker_works)

```
Question 1
嚴格排序確保消息按生成的順序存儲。我們如何確保消息的嚴格排序？

Answer
我們將為每個分區分配一個唯一的 ID，partition_ID。使用者可以在寫入系統時提供 partition_ID。這樣，消息將發送到指定的分區，並且排序將是嚴格的。我們寫入 pub-sub 系統的 API 調用如下所示：

write(topic_ID, partition_ID, message)

如果使用者未提供 partition_ID，我們將使用加權循環算法來決定將哪條消息發送到哪個分區。

讓 pub-sub 的客戶端能夠選擇分區似乎很奇怪。但是，這樣的設施可以成為客戶端獲取特定時間段資料的基礎——例如，獲取昨天的資料。為了簡單起見，我們將不在設計中包含基於時間的讀取。
```

```
Question 2
如果所有分區都在同一個代理上，會出現什麼問題？

Answer
如果代理發生故障或死亡，分區中的所有消息都將丟失。為了避免這種情況，我們需要確保分區分佈在不同的代理上。
```

```
Question 3
為什麼我們不能使用像 S3 這樣的 blob 存儲來保存消息，而不是代理的本地存儲？

Answer
像 S3 這樣的 Blob 存儲沒有針對寫入和讀取短尺寸資料進行優化。如果我們的資料是地理複製的，上述問題就會加劇。

因此，我們使用了具有基於追加寫入的伺服器本地持久存儲。傳統硬盤經過專門調整，可通過寫入連續軌道或扇區來提供良好的寫入性能。讀取吞吐量和延遲對於磁盤的連續區域也很有好處，因為它允許廣泛的資料緩存。
```

```
Question 4
如果我們使用循環算法將消息發送到分區，系統如何在需要讀取時知道在哪裡查找？

Answer
我們的系統需要持久保存適當的元資料。此元資料將保存段或消息的邏輯索引與伺服器標識或分區標識符之間的映射。

我們稍後將在課程中討論消費者管理器，它將保留所需的資訊。
```

[多個主題](./multiple_topics)

我們討論了消息將存儲在段中。我們將使用偏移量標識每個段。由於這些是不可變記錄，讀者是獨立的，他們可以使用必要的 API 函數從此文件的任何位置讀取消息。以下幻燈片顯示了段級別的詳細資訊。

[新條目將添加到文件末尾](./1.jpg)

[消費者可以從文件的任何位置讀取。生產者添加到文件末尾](./2.jpg)

代理解決了我們第一個設計存在的問題。我們通過對主題進行分區避免了大量隊列。我們使用分區引入了並行性，避免了在消費消息時出現瓶頸。

## Cluster manager (集群管理器)
我們的集群中將有多個代理。集群管理器將執行以下任務：

- **Broker and topics registry (代理和主題註冊表)**: This stores the list of topics for each broker.
- **代理和主題註冊表**：這存儲每個代理的主題列表。

- **Manage replication (管理複製)**: The cluster manager manages replication by using the leader-follower approach. One of the brokers is the leader. If it fails, the manager decides who the next leader is. In case the follower fails, it adds a new broker and makes sure to turn it into an updated follower. It updates the metadata accordingly. We’ll keep three replicas of each partition on different brokers.
- **管理複製**：集群管理器使用領導者-跟隨者方法管理複製。其中一個代理是領導者。如果它失敗，管理器決定下一個領導者是誰。如果跟隨者失敗，它會添加一個新代理並確保將其變成更新的跟隨者。它相應地更新元資料。我們將在不同的代理上保留每個分區的三個副本。

[分區級別的複製](./replication.jpg)

## Consumer manager (消費者管理器)
消費者管理器將管理消費者。它有以下職責：

- **Verify the consumer (驗證消費者)**: The manager will fetch the data from the database and verify if the consumer is allowed to read a certain message. For example, if the consumer has subscribed to Topic A (but not to Topic B), then it should not be allowed to read from Topic B. The consumer manager verifies the consumer’s request.
- **驗證消費者**：管理器將從資料庫獲取資料並驗證消費者是否被允許讀取特定消息。例如，如果消費者訂閱了主題 A (但未訂閱主題 B)，則不允許從主題 B 讀取。消費者管理器驗證消費者的請求。

- **Retention time management (保留時間管理)**: The manager will also verify if the consumer is allowed to read the specific message or not. If, according to its retention time, the message should be inaccessible to the consumer, then it will not allow the consumer to read the message.
- **保留時間管理**：管理器還將驗證消費者是否被允許讀取特定消息。如果根據其保留時間，消息對消費者來說是不可訪問的，那麼它將不允許消費者讀取消息。

- **Message receiving options management (消息接收選項管理)**: There are two methods for consumers to get data. The first is that our system pushes the data to its consumers. This method may result in overloading the consumers with continuous messages. Another approach is for consumers to request the system to read data from a specific topic. The drawback is that a few consumers might want to know about a message as soon as it is published, but we do not support this function.
- **消息接收選項管理**：消費者有兩種獲取資料的方法。第一種是我們的系統將資料推送到其消費者。這種方法可能會導致消費者因連續消息而過載。另一種方法是消費者請求系統從特定主題讀取資料。缺點是少數消費者可能想在消息發布後立即了解消息，但我們不支持此功能。

因此，我們將支持這兩種技術。每個消費者都會通知代理它希望自動推送資料還是需要自己讀取資料。我們可以避免消費者過載，同時也為消費者提供自由。我們將把此資訊與其他消費者詳細資訊一起保存在關係資料庫中。

- **Allow multiple reads (允許多次讀取)**: The consumer manager stores the offset information of each consumer. We’ll use a key-value to store offset information against each consumer. It allows fast fetching and increases the availability of the consumers. If Consumer 1 has read from offset 0 and has sent the acknowledgment, we’ll store it. So, when the consumer wants to read again, we can provide the next offset to the reader for reading the message.
- **允許多次讀取**：消費者管理器存儲每個消費者的偏移量資訊。我們將使用鍵值對來存儲每個消費者的偏移量資訊。它允許快速獲取並提高消費者的可用性。如果消費者 1 已從偏移量 0 讀取並發送了確認，我們將存儲它。因此，當消費者想要再次讀取時，我們可以為讀者提供下一個偏移量來讀取消息。

## Finalized design (最終設計)
我們 pub-sub 系統的最終設計如下所示。

[最終設計](./final.jpg)

## Conclusion (結論)
我們看到了兩種 pub-sub 設計，一種使用隊列，另一種使用我們針對寫入和讀取小尺寸資料進行優化的自定義存儲。

Pub-sub 有許多用例。由於生產者和消費者之間的解耦，系統可以動態擴展，並且故障得到很好的遏制。此外，由於對資料消耗進行了適當的核算，pub-sub 是產生大量資料的大規模系統的首選系統。我們可以精確地確定哪些資料是需要的，哪些是不需要的。
