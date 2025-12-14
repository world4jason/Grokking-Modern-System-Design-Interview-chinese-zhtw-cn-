# Design of a Key-value Store (設計鍵值存儲)
## Requirements (需求)
讓我們列出設計鍵值存儲的需求，以克服傳統資料庫的問題。

### Functional requirements (功能需求)
功能需求如下：

- **Configurable service (可配置服務)**: Some applications might have a tendency to trade strong consistency for higher availability. We need to provide a configurable service so that different applications could use a range of consistency models. We need tight control over the trade-offs between availability, consistency, cost-effectiveness, and performance.
- **可配置服務**：某些應用程式可能傾向於以強一致性換取更高的可用性。我們需要提供可配置的服務，以便不同的應用程式可以使用一系列一致性模型。我們需要嚴格控制可用性、一致性、成本效益和性能之間的權衡。

- **Ability to always write (始終寫入的能力)**: The applications should always have the ability to write into the key-value storage. If the user wants strong consistency, this requirement might not always be fulfilled due to the implications of the CAP theorem.
- **始終寫入的能力**：應用程式應始終具有寫入鍵值存儲的能力。如果使用者想要強一致性，由於 CAP 定理的影響，這一要求可能並不總是得到滿足。

- **Hardware heterogeneity (硬件異構性)**: The system shouldn’t have distinguished nodes. Each node should be functionally able to do any task. Though servers can be heterogeneous, newer hardware might be more capable than older ones.
- **硬件異構性**：系統不應具有區分的節點。每個節點在功能上都應該能夠執行任何任務。雖然伺服器可以是異構的，但較新的硬件可能比較舊的硬件更有能力。

### Non-functional requirements (非功能需求)
非功能需求如下：

- **Scalable (可擴展)**: Key-value stores should run on tens of thousands of servers distributed across the globe. Incremental scalability is highly desirable. We should add or remove the servers as needed with minimal to no disruption to the service availability. Moreover, our system should be able to handle an enormous number of users of the key-value store.
- **可擴展**：鍵值存儲應該在全球分佈的數万台伺服器上運行。增量可擴展性是非常可取的。我們應該在對服務可用性的干擾最小或沒有干擾的情況下根據需要添加或刪除伺服器。此外，我們的系統應該能夠處理大量的鍵值存儲使用者。

- **Available (可用)**: We need to provide continuous service, so availability is very important. This property is configurable. So, if the user wants strong consistency, we’ll have less availability and vice versa.
- **可用**：我們需要提供持續的服務，所以可用性非常重要。此屬性是可配置的。因此，如果使用者想要強一致性，我們的可用性就會降低，反之亦然。

- **Fault tolerance (容錯)**: The key-value store should operate uninterrupted despite failures in servers or their components.
- **容錯**：儘管伺服器或其組件發生故障，鍵值存儲仍應不間斷地運行。

```
Question
為什麼我們需要在多個伺服器上運行鍵值存儲？

Answer
基於單節點的哈希表可能會因為以下一個或多個原因而不足：

無論我們的伺服器有多大，這台伺服器都無法滿足資料存儲和查詢需求。

這台大型伺服器的故障將導致所有人的服務停機。

因此，鍵值存儲應該使用許多伺服器來存儲和檢索資料。
```

## Assumptions (假設)
我們假設以下內容以使我們的設計保持簡單：

- 託管服務的資料中心是受信任的 (非敵對的)。
- 所有必要的身份驗證和授權都已完成。
- 使用者請求和響應通過 HTTPS 中繼。

## API design (API 設計)
鍵值存儲與普通哈希表一樣，提供兩個主要功能，即 get 和 put。

讓我們看看 API 設計。

**The get function (get 函數)**

獲取值的 API 調用應如下所示：
```
get(key)
```
我們根據參數鍵返回關聯值。當複製資料時，它定位與特定鍵關聯的對象副本，該副本對最終使用者隱藏。如果存儲配置了較弱的資料一致性模型，則由系統完成。例如，在最終一致性中，可能會針對一個鍵返回多個值。

```
Parameter         Description
                  這是我們想要獲取值的鍵。
```

**The put function (put 函數)**

將值放入系統的 API 調用應如下所示：
```
put(key, value)
```
它存儲與鍵關聯的值。系統自動確定應放置資料的位置。此外，系統通常保留有關存儲對象的元資料。此類元資料可以包括對象的版本。

```
Parameter      Description
               這是我們必須存儲值的鍵。

               這是要針對鍵存儲的對象。
```

```
Question
我們經常保留值的哈希 (有時是值 + 關聯鍵) 作為元資料以進行資料完整性檢查。這種哈希是在任何資料壓縮或加密之後進行，還是在之前進行？

Answer
正確答案可能取決於具體的應用程式。儘管如此，我們可以在任何壓縮或加密之前或之後使用哈希。但是我們需要在 put 和 get 操作中始終如一地這樣做。
```

### Data type (資料類型)
鍵通常是鍵值存儲中的主鍵，而值可以是任何任意二進制資料。

```
注意：Dynamo 在鍵上使用 MD5 哈希生成 128 位標識符。這些標識符幫助系統確定哪個伺服器節點將負責此特定鍵。
```

在下一課中，我們將學習如何設計我們的鍵值存儲。首先，我們將專注於向我們的系統添加可擴展性、複製和資料版本控制。然後，我們將確保功能需求並使我們的系統具有容錯能力。我們將首先滿足一些非功能需求，因為實現我們的功能需求取決於選擇的可擴展性方法。

```
注意：本章基於 Dynamo，這是鍵值存儲領域的一項有影響力的工作。

Dynamo：亞馬遜的高可用鍵值存儲 (https://assets.amazon.science/ac/1d/eb50c4064c538c8ac440ce6a1d91/dynamo-amazons-highly-available-key-value-store.pdf)
```
