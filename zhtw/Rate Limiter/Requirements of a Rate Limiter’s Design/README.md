# Requirements of a Rate Limiter’s Design (速率限制器設計的需求)
## Requirements (需求)
我們本課的重點是設計具有以下功能和非功能需求的速率限制器。

### Functional requirements (功能需求)
- 限制客戶端在時間窗口內可以發送到 API 的請求數量。
- 使每個窗口的請求限制可配置。
- 確保當在單個伺服器或伺服器組合中超過定義的閾值時，客戶端會收到消息 (錯誤或通知)。

### Non-functional requirements (非功能需求)
- **Availability (可用性)**: Essentially, the rate limiter protects our system. Therefore, it should be highly available.
- **可用性**：從本質上講，速率限制器保護我們的系統。因此，它應該具有高可用性。
- **Low latency (低延遲)**: Because all API requests pass through the rate limiter, it should work with a minimum latency without affecting the user experience.
- **低延遲**：因為所有 API 請求都通過速率限制器，所以它應該以最小的延遲工作，而不影響使用者體驗。
- **Scalability (可擴展性)**: Our design should be highly scalable. It should be able to rate limit an increasing number of clients’ requests over time.
- **可擴展性**：我們的設計應該具有高度可擴展性。它應該能夠隨著時間的推移對越來越多的客戶端請求進行速率限制。

## Types of throttling (節流類型)
速率限制器可以執行三種類型的節流。

1. **Hard throttling (硬節流)**: This type of throttling puts a hard limit on the number of API requests. So, whenever a request exceeds the limit, it is discarded.
1. **硬節流**：這種類型的節流對 API 請求的數量設置了硬性限制。因此，每當請求超過限制時，它就會被丟棄。
2. **Soft throttling (軟節流)**: Under soft throttling, the number of requests can exceed the predefined limit by a certain percentage. For example, if our system has a predefined limit of 50 messages per minute with a 5% exceed in the limit, we can let the client send 525 requests per minute.
2. **軟節流**：在軟節流下，請求數量可以超過預定義限制一定百分比。例如，如果我們的系統預定義限制為每分鐘 500 條消息，允許超過 5%，我們可以讓客戶端每分鐘發送 525 個請求。
3. **Elastic or dynamic throttling (彈性或動態節流)**: In this throttling, the number of requests can cross the predefined limit if the system has excess resources available. However, there is no specific percentage defined for the upper limit. For example, if our system allows 500 requests per minute, it can let the user send more than 500 requests when free resources are available.
3. **彈性或動態節流**：在這種節流中，如果系統有可用的過剩資源，請求數量可以超過預定義的限制。但是，上限沒有定義具體的百分比。例如，如果我們的系統允許每分鐘 500 個請求，當有空閒資源可用時，它可以讓使用者發送超過 500 個請求。

```
Linux 操作系統提供了一種稱為 cgroups (控制組) 的內核功能，該功能限制、核算和隔離進程集合的資源——CPU 時間、系統內存、磁盤存儲、I/O 和網絡頻寬。通過使用 cgroups，系統管理員可以監控、拒絕訪問特定資源，並在運行的系統上動態地重新配置 cgroups。cgroups 功能提供以下好處，通過這些好處，系統管理員可以獲得對系統資源的細粒度控制：

- 資源限制：使用此功能，可以對組施加限制，使其不超過配置的內存限制和文件系統緩存。

- 優先級：通過此功能，可以優先考慮某些組使用更大份額的 CPU 週期或磁盤 I/O 吞吐量。

- 核算：此功能用於測量組的資源使用情況，也可用於計費目的。

- 控制：系統管理員可以通過此功能控制進程組、它們的檢查點和重啟。

cgroup 提供如此豐富的功能，可在系統 (單個伺服器) 級別使用，不僅可以限制單個使用者的資源使用，還可以限制使用者組或進程的資源使用。

我們可以通過設計一個服務來擴展上述概念，該服務從限速服務獲取輸入並在集群的本地節點上強制執行限制。
```

## Where to place the rate limiter (在哪裡放置速率限制器)
有三種不同的方式來放置速率限制器。

1. **On the client side (在客戶端)**: It is easy to place the rate limiter on the client side. However, this strategy is not safe because it can easily be tampered with by malicious activity. Moreover, the configuration on the client side is also difficult to apply in this approach.
1. **在客戶端**：很容易將速率限制器放在客戶端。然而，這種策略並不安全，因為它很容易被惡意活動篡改。此外，客戶端的配置在這種方法中也很難應用。

2. **On the server side (在伺服器端)**: As shown in the following figure, the rate limiter is placed on the server-side. In this approach, a server receives a request that is passed through the rate limiter that resides on the server.
2. **在伺服器端**：如下圖所示，速率限制器放置在伺服器端。在這種方法中，伺服器接收通過駐留在伺服器上的速率限制器的請求。

[放置在伺服器端的速率限制器](./rate_limiter_server.jpg)

3. **As middleware (作為中間件)**: In this strategy, the rate limiter acts as middleware, throttling requests to API servers as shown in the following figure.
3. **作為中間件**：在這種策略中，速率限制器充當中介軟體，限制對 API 伺服器的請求，如下圖所示。

[作為中間件的速率限制器](./rate_limiter_middleware.jpg)

放置速率限制器取決於許多因素，並且是一個主觀決定，基於組織的技術堆棧、工程資源、優先級、計劃、目標等。

```
注意：許多現代服務使用 API 向客戶端提供其功能。API 端點可以是限制傳入客戶端流量的一個很好的有利位置，因為所有流量都通過它們。
```

## Two models for implementing a rate limiter (實現速率限制器的兩種模型)
一個速率限制器可能不足以處理巨大的流量以支持數百萬使用者。因此，更好的選擇是使用多個速率限制器作為獨立節點的集群。由於將有許多速率限制器及其相應的計數器 (或其速率限制)，因此有兩種方法可以使用資料庫來存儲、檢索和更新計數器以及使用者資訊。

1. **A rate limiter with a centralized database (具有集中式資料庫的速率限制器)**: In this approach, rate limiters interact with a centralized database, preferably Redis or Cassandra. The advantage of this model is that the counters are stored in centralized databases. Therefore, a client can’t exceed the predefined limit. However, there are a few drawbacks to this approach. It causes an increase in latency if an enormous number of requests hit the centralized database. Another extensive problem is the potential for race conditions in highly concurrent requests (or associated lock contention).
1. **具有集中式資料庫的速率限制器**：在這種方法中，速率限制器與集中式資料庫 (最好是 Redis 或 Cassandra) 交互。這種模型的優點是計數器存儲在集中式資料庫中。因此，客戶端無法超過預定義的限制。但是，這種方法有一些缺點。如果有大量請求訪問集中式資料庫，它會導致延遲增加。另一個廣泛的問題是高並發請求中潛在的競爭條件 (或相關的鎖爭用)。

2. **A rate limiter with a distributed database (具有分佈式資料庫的速率限制器)**: Using an independent cluster of nodes is another approach where the rate-limiting state is in a distributed database. In this approach, each node has to track the rate limit. The problem with this approach is that a client could exceed a rate limit—at least momentarily, while the state is being collected from everyone—when sending requests to different nodes (rate-limiters). To enforce the limit, we must set up sticky sessions in the load balancer to send each consumer to exactly one node. However, this approach lacks fault tolerance and poses scaling problems when the nodes get overloaded.
2. **具有分佈式資料庫的速率限制器**：使用獨立的節點集群是另一種方法，其中速率限制狀態位於分佈式資料庫中。在這種方法中，每個節點都必須跟踪速率限制。這種方法的問題是，當向不同節點 (速率限制器) 發送請求時，客戶端可能會超過速率限制——至少在從每個人那裡收集狀態時是暫時的。為了強制執行限制，我們必須在負載平衡器中設置粘性會話，以將每個消費者發送到恰好一個節點。但是，這種方法缺乏容錯能力，並且在節點過載時會帶來擴展問題。

除了上述兩個概念之外，另一個問題是使用所有傳入請求共享的全局計數器還是每個使用者的單獨計數器。例如，令牌桶算法可以通過兩種方式實現。在第一種方法中，所有請求都可以共享單個桶中的令牌總數，而在第二種方法中，單獨的桶被分配給使用者。選擇使用共享還是單獨的計數器 (或桶) 取決於用例和速率限制規則。

```
Question 1
速率限制器可以用作負載平衡器嗎？

Answer
負載平衡器可防止過多的請求轉發到應用程式伺服器。它們要麼根據限制拒絕請求，要麼將請求發送到隊列以供稍後處理。但是，負載平衡器對傳入請求一視同仁，沒有偏見。例如，假設我們的 Web 服務公開了幾個操作。其中一些操作很快，而另一些則很慢。慢速操作的請求比快速操作需要更多的時間和處理能力。負載平衡器不知道此類操作的成本。因此，如果我們的目標是限制特定操作的請求數量，我們應該在應用程式伺服器而不是負載平衡器級別進行限制。
```

```
Question 2
假設一個場景，其中客戶端打算使用兩台虛擬機 (VM) 發送對特定服務的請求，其中一台使用 VPN 連接到不同區域。假設節流標識符基於使用者憑據工作。因此，兩個會話的使用者 ID 將相同。此外，假設來自不同 VM 的請求可能會訪問不同的資料中心。在這種情況下，節流如何工作以防止使用者超過速率限制？

Answer
為了限制傳入請求的速率，我們有兩種不同的選擇來放置速率限制器。

每個資料中心的速率限制器：限制來自使用者的傳入請求的一種方法是使用每個資料中心的速率限制。每個資料中心將有自己的速率限制器，限制傳入請求。在這種方法中，速率 (計數或速率限制) 相對較低。因此，單位時間內允許的請求數量有限。此外，這種方法提供了更低的延遲，因為請求通常被定向到地理位置最近的資料中心。通常，資料中心內的延遲小於一毫秒，並且在某些鏈路故障的情況下有多個冗余路徑可用。

跨資料中心的共享速率限制器：另一種方法是使用跨多個資料中心的共享速率限制器。這樣，從兩個 VM 接收到的請求將由單個速率限制器進行節流。在這種情況下允許的請求數量更高。但是，這種方法相對較慢，因為在將請求定向到任何最近的資料中心之前，它將通過共享速率限制器。跨地理分佈的資料中心的延遲通常很高且可變，並且可用的冗余路徑不多。
```

## Building blocks we will use (我們將使用的構建塊)
速率限制器的設計利用了我們在最初章節中討論的以下構建塊。

[速率限制器設計中的構建塊](./bb.jpg)

- **Databases (資料庫)** are used to store rules defined by a service provider and metadata of users using the service.
- **資料庫**用於存儲服務提供商定義的規則以及使用該服務的使用者的元資料。
- **Caches (緩存)** are used to cache the rules and users’ data for frequent access.
- **緩存**用於緩存規則和使用者資料以供頻繁訪問。
- **Queues (隊列)** are essential for holding the incoming requests that are allowed by the rate limiter.
- **隊列**對於保存速率限制器允許的傳入請求至關重要。

在下一課中，我們將專注於基於本課討論的需求的速率限制器的高層和詳細設計。
