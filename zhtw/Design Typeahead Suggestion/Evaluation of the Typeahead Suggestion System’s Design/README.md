# Evaluation of the Typeahead Suggestion System’s Design (預輸入建議系統設計的評估)

## Fulfill requirements (滿足需求)
擬議的預輸入建議系統的非功能需求是低延遲、容錯和可擴展性。

- **Low latency (低延遲)**：我們可以在各個級別最小化系統的延遲。我們可以通過以下選項最小化延遲：

  - 減少樹的深度，從而減少整體遍歷時間。
  - 離線更新 trie，這意味著更新操作所花費的時間不在客戶端的關鍵路徑上。
  - 使用地理分佈的應用程式和資料庫伺服器。這樣，服務在使用者附近提供，這也減少了任何通訊延遲並有助於減少延遲。
  - 在 NoSQL 資料庫集群之上使用 Redis 和 Cassandra 快取集群。
  - 適當地對 trie 進行分區，從而實現負載的適當分發並獲得更好的效能。

- **Fault tolerance (容錯性)**：由於提供了樹的複制和分區，系統以高彈性運行。如果一台伺服器發生故障，其他伺服器將隨時待命提供服務。

- **Scalability (可擴展性)**：由於我們提出的系統非常靈活，隨著負載的增加，可以新增或移除更多伺服器。例如，如果查詢數量增加，樹的分區或分片數量也會相應增加。

Approaches to Fulfill Non-functional Requirements (滿足非功能需求的方法)
Non-functional Requirements (非功能需求)    Approaches (方法)

Low latency (低延遲)                         - Reducing the depth of the tries makes the traversal faster (減少 trie 的深度使遍歷更快)
                                           - Updating the tries offline and not in real time) (離線更新 trie 而不是實時更新)
                                           - Partitioning of the tries (Trie 的分區)
                                           - Caching servers (快取伺服器)
                                           
Fault tolerance (容錯性)                     - Replicating the tries and the NoSQL databases (複製 trie 和 NoSQL 資料庫)

Scalability (可擴展性)                       - Adding or removing application servers based on the incoming traffic (根據傳入流量新增或移除應用程式伺服器)
                                           - Increasing the trie partitions (增加 trie 分區)

```


## Client-side optimisation (客戶端優化)
為了改善使用者體驗，我們可以實施以下客戶端優化：

- 如果使用者在一段時間內沒有按任何鍵——例如，任何大於 160 毫秒的延遲，這是兩次擊鍵之間的平均延遲，客戶端才應嘗試聯繫伺服器。這樣，我們也可以避免不必要的頻寬消耗。當使用者快速輸入時，此建議可能沒什麼用。
- 客戶端最初可以等待使用者輸入幾個字元。
- 客戶端可以保存最近建議歷史的本地副本。最近歷史在建議列表中的重用率相對較高。
- 最關鍵的因素之一是盡快與伺服器建立連接。一旦使用者訪問搜尋頁面，客戶端就可以與伺服器建立連接。因此，當使用者輸入第一個字元時，客戶端不會浪費時間建立連接。通常，通過 WebSocket 協議與伺服器建立連接。
- 為了提高效率，伺服器可以將其部分快取推送到 CDN 和 Internet 交換點 (IXP) 的其他邊緣快取，甚至客戶端的 Internet 服務提供商 (ISP) 內部。


## Personalization (個性化)
使用者根據他們以前的搜尋、位置、語言和其他因素收到預輸入建議。我們可以將每個使用者的個人歷史分別保存在伺服器上，並將其快取在客戶端。在將最終集合傳輸給使用者之前，伺服器可能會包含這些自定義短語。個性化搜尋應始終優先於其他類型的搜尋。


## Summary (總結)
在這個設計問題中，我們學習了如何將資源密集型處理推送到離線基礎設施並使用適當的資料結構，使我們能夠以低延遲為客戶提供服務。許多優化適用於特定用例。我們看到了 trie 資料結構的多種優化，用於壓縮資料存儲和快速服務。
