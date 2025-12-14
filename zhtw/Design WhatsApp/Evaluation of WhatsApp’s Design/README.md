# Evaluation of WhatsApp’s Design (WhatsApp 設計的評估)

## Fulfill the requirements (滿足需求)
我們對擬議的 WhatsApp 設計的非功能需求是低延遲、一致性、可用性和安全性。讓我們討論一下我們先前如何在系統中實現這些需求：

- **Low latency (低延遲)**：我們可以通過多種級別最小化系統的延遲：

  - 我們可以通過地理分佈的 WebSocket 伺服器及其關聯的快取來做到這一點。

  - 我們可以在 MySQL 資料庫集群之上使用 Redis 快取集群。

  - 我們可以使用 CDN 來分享文件和媒體內容。

- **Consistency (一致性)**：在具有嚴格順序的 FIFO 訊息隊列的幫助下，系統還提供了訊息的高度一致性。但是，訊息的排序需要定序器為每個訊息提供具有適當因果推斷機制的 ID。對於離線使用者，Mnesia 資料庫將訊息儲存在隊列中。訊息會在使用者上線後按順序發送。

- **Availability (可用性)**：如果我們有足夠的 WebSocket 伺服器並在多個伺服器上複製資料，則系統可以具有高可用性。當使用者由於 WebSocket 伺服器中的某些故障而斷開連接時，會通過負載平衡器與另一台伺服器重新創建會話。此外，訊息遵循主從複製模型儲存在 Mnesia 集群上，這提供了高可用性和持久性。

- **Security (安全性)**：系統還提供了端到端加密機制，以保護使用者之間的聊天。


```
Question
如果發生網路分區，系統應該在一致性和可用性之間做出什麼妥協？

Answer
根據 CAP 定理，在發生網路分區的情況下，系統將提供一致性或可用性。在我們的 WhatsApp messenger 系統中，訊息的正確排序至關重要。否則，使用者之間傳達的訊息上下文可能會發生重大變化。因此，如果發生網路分區，我們系統的可用性可能會受到影響。
```

          Approaches to Achieve the Non-functional Requirements (實現非功能需求的方法)
Non-functional Requirements (非功能需求)               Approaches (方法)

Minimizing latency (最小化延遲)                        - Geographically distributed cache management systems and servers (地理分佈的快取管理系統和伺服器)
                                          - CDN

Consistency (一致性)                               - Provide unique IDs to messages using Sequencer or other mechanisms (使用定序器或其他機制為訊息提供唯一 ID)
                                          - Use FIFO messaging queue with strict ordering (使用具有嚴格順序的 FIFO 訊息隊列)

Availability (可用性)                              - Provide multiple WebSocket servers and managers to establish connections between users (提供多個 WebSocket 伺服器和管理器以建立使用者之間的連接)
                                          - Replication of messages and data associated with users and groups on different servers (在不同伺服器上複製與使用者和群組關聯的訊息和資料)
                                          - Follow disaster recovery protocols (遵循災難恢復協議)

Security (安全性)                                  Via end-to-end encryption (通過端到端加密)         
```

## Trade-offs (權衡)
我們已經看到，我們提出的 WhatsApp 系統滿足了功能和非功能需求。但是，在擬議的 WhatsApp 設計中存在兩個主要權衡：

- 一致性和可用性之間存在權衡。
- 延遲和安全性之間存在權衡。

### The trade-off between consistency and availability (一致性和可用性之間的權衡)
根據 CAP 定理，在發生網路故障或分區的情況下，系統可以提供一致性或可用性。因此，在我們的 WhatsApp 設計案例中，我們必須選擇一致性或可用性。在 WhatsApp 中，使用者發送或接收訊息的順序至關重要。因此，我們應該優先考慮一致性而不是可用性。


### The trade-off between latency and security (延遲和安全性之間的權衡)
低延遲是系統設計中為使用者提供實時體驗的重要因素。然而，另一方面，如果不加密，通過 WhatsApp 分享訊息或資料可能是不安全的。缺乏適當的安全機制使資料容易受到未經授權的訪問。因此，我們可以接受一種權衡，即優先考慮訊息的安全傳輸而不是低延遲。

我們可能想知道權衡在哪裡。通常，通訊涉及多媒體。在發送端設備上近乎實時地加密它們並在接收端解密可能會給設備帶來負擔，從而導致延遲。該過程如下圖所示：

[端到端加密會導致處理延遲，從而影響延遲]


## Summary (總結)
在本章中，我們設計了一個 WhatsApp messenger。首先，我們確定了功能和非功能需求以及對設計至關重要的資源估算。其次，我們重點介紹了 WhatsApp 系統的高層和詳細設計，其中我們描述了負責不同服務的各種組件。最後，我們評估了非功能需求並強調了設計中的一些權衡。

這個設計問題強調了我們可以針對特定用例優化通用計算資源。WhatsApp 優化了其軟體堆棧，以處理商品伺服器上的大量連接。
