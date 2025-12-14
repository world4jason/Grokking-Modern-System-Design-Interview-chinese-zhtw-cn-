# AWS Kinesis Outage Affecting Many Organizations (影響許多組織的 AWS Kinesis 中斷)

Amazon Kinesis 允許我們聚合、處理和分析實時流資料，以獲得及時的見解並對其提供的資訊做出快速反應。它持續每秒從數十萬個來源捕獲 GB 級的資料。Kinesis 服務的前端處理身分驗證、限制，並通過資料庫分片將工作負載分發到其後端“主力”集群。2020 年 11 月 25 日，Amazon Kinesis 服務在 US-East-1 (北弗吉尼亞州) 地區中斷，影響了數千個其他第三方服務。這次故障嚴重到導致大部分互聯網服務癱瘓。

## Sequence of events (事件順序)
- 據亞馬遜稱，該事件是由 AWS 前端伺服器機群增加少量容量引發的，時間定於太平洋標準時間凌晨 2:44 至凌晨 3:47。

- 新容量的增加導致機群中的所有伺服器都超過了操作系統配置允許的最大線程數。

- 由於超過了線程限制，緩存構建無法完成，前端伺服器最終得到了無用的分片映射，導致它們無法將請求路由到後端集群。

- 其他主要亞馬遜服務也停止工作，包括 Amazon Cognito 和 CloudWatch。

  - Amazon Cognito 使用 Kinesis Data Streams 來收集和分析 API 使用模式。由於 Kinesis Data Streams 長期存在的問題，緩衝代碼 (Cognito 服務所需) 中的一個隱藏錯誤導致 Cognito Web 伺服器開始阻塞積壓的 Kinesis Data Stream 緩衝區。結果，Cognito 客戶目睹了 Cognito 使用者池和身份池的 API 故障和延遲增加，使得外部使用者無法進行身份驗證或接收臨時 AWS 憑證。

  - CloudWatch 使用 Kinesis Data Streams 處理指標和日誌資料。CloudWatch 中的 PutMetricData 和 PutLogEvents API 遇到較高的錯誤率和延遲，並且警報設置為 INSUFFICIENT DATA。由於較高的錯誤率和延遲，絕大多數指標無法處理。當 CloudWatch 遇到這些更大的問題時，內部和外部客戶端無法將所有指標資料持久保存到 CloudWatch 服務。

[事件順序](./seq.jpg)

- 由於 CloudWatch 指標的問題，兩個服務也受到了影響。首先，依賴 CloudWatch 測量的自動擴展策略遭受延遲。其次，Lambda 受到了影響。目前，發布指標資料到 CloudWatch 是 Lambda 函數調用的一部分。如果 CloudWatch 不可用，Lambda 指標代理旨在將指標資料在本地緩衝一段時間。指標資料緩衝變得如此之大，以至於在用於 Lambda 函數調用的底層服務主機上產生了內存擁塞，導致更高的錯誤率。

- CloudWatch Events 和 EventBridge 受到 API 故障增加和事件處理延遲的困擾。Elastic Container Service (ECS) 和 Elastic Kubernetes Service (EKS) 使用 EventBridge 來驅動用於管理客戶端集群和作業的內部流程。這對新集群配置、現有集群擴展和任務取消配置產生了影響。

- 除服務問題外，亞馬遜在向客戶傳達服務狀態方面也經歷了延遲。亞馬遜使用兩個儀表板與客戶進行通信，即服務運行狀況儀表板和個人運行狀況儀表板。服務運行狀況儀表板通知所有客戶事件，例如當前事件。由於依賴於受此事件影響的 Cognito，它已關閉。

- 除了亞馬遜服務的中斷外，這還對數千個第三方在線服務、應用程式和網站產生了連鎖反應。這包括 Adobe Spark、Acorns、Coinbase、華盛頓郵報和數百個此類服務。

## Analysis (分析)
- **Complexity of enhancing scalability (增強可擴展性的複雜性)**: We’ve been stressing the need for horizontal scalability in all of our design problems. This outage event shows that in practice, adding more capacity to a serving cluster so that we don’t deny any client requests can be challenging and can have unintended side effects.
- **增強可擴展性的複雜性**：我們一直在強調所有設計問題中橫向可擴展性的必要性。此次中斷事件表明，在實踐中，向服務集群添加更多容量以免拒絕任何客戶端請求可能具有挑戰性，並且可能會產生意想不到的副作用。

- **The need for a trained team (需要訓練有素的團隊)**: Training the production team to deal with such unforeseen situations is a challenging task, but it’s worth it and can result in expedited recovery. In addition, randomly restarting the front-end servers and suspecting that the memory pressure causes the problem alludes to the challenges of reaching the root causes under the stress of time.
- **需要訓練有素的團隊**：培訓生產團隊處理此類不可預見的情況是一項艱鉅的任務，但這是值得的，並且可以加快恢復速度。此外，隨機重啟前端伺服器並懷疑內存壓力導致問題，暗示了在時間壓力下尋找根本原因的挑戰。

- **Reading from authoritative servers during bootstrap (引導期間從權威伺服器讀取)**: During the bootstrap process, it’s a good idea to take data from the authoritative metadata store rather than from the front-end servers to reduce the impact of such failures.
- **引導期間從權威伺服器讀取**：在引導過程中，最好從權威元資料存儲而不是從前端伺服器獲取資料，以減少此類故障的影響。

- **Identification of faults in initial stages (在初始階段識別故障)**: There’s a need for automated processes to identify the causes of failure within the initial stages of its occurrence.
- **在初始階段識別故障**：需要自動化流程來在故障發生初期識別故障原因。

- **Proper testing mechanisms can reduce the severity of the fault (適當的測試機制可以降低故障的嚴重性)**: The new capacity that’s causing the servers in the fleet to exceed the maximum number of threads is a potential bug that should have been fixed earlier than before restarting the servers. This reflects the inability to properly test the system. There should be a kind of simulator to test all cases before deploying additional capacity into the fleet.
- **適當的測試機制可以降低故障的嚴重性**：導致機群中的伺服器超過最大線程數的新容量是一個潛在的錯誤，應該在重新啟動伺服器之前修復。這反映了無法正確測試系統。在向機群部署額外容量之前，應該有一種模擬器來測試所有情況。

- **Finding potential bugs before planned events (在計劃事件之前發現潛在錯誤)**: The issue with Kinesis Data Streams triggered a latent bug in the buffering code that caused the Cognito web servers to begin to block on the backlogged Kinesis Data Stream buffers. The potential bugs should be identified and fixed before critical planned events, such as adding capacity or software and hardware maintenance.
- **在計劃事件之前發現潛在錯誤**：Kinesis Data Streams 的問題觸發了緩衝代碼中的潛在錯誤，導致 Cognito Web 伺服器開始阻塞積壓的 Kinesis Data Stream 緩衝區。應在關鍵計劃事件 (例如添加容量或軟體和硬體維護) 之前識別並修復潛在錯誤。

- **Adopting automated processes for resource allocation (採用自動化流程進行資源分配)**: The Cognito team sought to reduce the effect of the Kinesis faults in the early phases of the event by providing extra capacity and therefore boosting their ability to buffer calls to Kinesis. Instead of manually assigning capacity, there should be an automated system to increase or reallocate capacity in such events.
- **採用自動化流程進行資源分配**：Cognito 團隊試圖通過提供額外容量並因此提高其緩衝 Kinesis 調用的能力來減少事件早期階段 Kinesis 故障的影響。不應手動分配容量，而應有一個自動化系統在此類事件中增加或重新分配容量。

## Lessons learned (經驗教訓)
- **Testing (測試)**: Proper testing and identifying the potential bugs are essential. In this case, the number of threads exceeding a maximum limit defined by the operating system seems to be a possible bug.
- **測試**：正確的測試和識別潛在的錯誤至關重要。在這種情況下，線程數超過操作系統定義的最大限制似乎是一個可能的錯誤。

- **Ready operations team (準備就緒的運營團隊)**: A bug bringing an overall system to a halt is the single point of failure, which is possible in a complex system. The production team should be trained and ready for such events.
- **準備就緒的運營團隊**：導致整個系統停止的錯誤是單點故障，這在復雜系統中是可能的。生產團隊應接受培訓並為此類事件做好準備。

- **Reducing number of servers (減少伺服器數量)**: To get significant headroom in the thread count used as the total threads, there’s a need to move to more powerful CPU and memory servers. This will reduce the total number of servers and the threads required by each server to communicate across the fleet. Having fewer servers means that each server maintains fewer threads. Amazon is adding fine-grained alarming for thread consumption in the service.
- **減少伺服器數量**：為了在用作總線程的線程數中獲得顯著的餘量，需要轉移到更強大的 CPU 和內存伺服器。這將減少伺服器總數以及每個伺服器在機群中通信所需的線程。擁有更少的伺服器意味著每個伺服器維護更少的線程。亞馬遜正在為服務中的線程消耗添加細粒度警報。

- **Front-end fleet changes (前端機群變更)**: Several changes are required to radically improve the cold start time for the front-end fleet. Moreover, the front-end server cache needs to be moved to a dedicated fleet.
- **前端機群變更**：需要進行一些更改才能從根本上改善前端機群的冷啟動時間。此外，前端伺服器緩存需要移動到專用機群。

- **Avoiding recurrent failures (避免經常性故障)**: To avoid recurrent failures in the future, extensive AWS services, like CloudWatch and others, must be moved to a separate, partitioned front-end fleet.
- **避免經常性故障**：為了避免將來發生經常性故障，必須將廣泛的 AWS 服務 (如 CloudWatch 等) 移動到單獨的分區前端機群。

```
Question
亞馬遜應該採取哪些措施來防止他們在 Kinesis Data Streams 中面臨的那種故障？

Answer
可能的解決方案

將區域劃分為獨立的故障域將減少事件的爆炸半徑，並使生產團隊能夠快速從問題中恢復。

他們應該有一個像 Facebook 的資源配額系統這樣的系統，以便在計劃和非計劃事件時預留容量。

跨多個雲或 AWS 區域構建應用程式將使受影響的客戶更容易快速恢復。

需要極度解耦服務以消除交叉依賴問題。

複雜系統中的故障是不可避免的。但是，一些重要的服務 (如狀態儀表板) 應該託管在不同的伺服器上，無論是在服務內部還是在某些第三方的基礎設施中。
```
