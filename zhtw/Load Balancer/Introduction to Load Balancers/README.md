# Introduction to Load Balancers (負載平衡器簡介)

## What is load balancing? (什麼是負載平衡？)
典型的資料中心每秒可能會有數百萬個請求到達。為了服務這些請求，數千 (或十萬) 台伺服器協同工作以分擔傳入請求的負載。

```
注意：在這裡，我們必須考慮如何在所有可用伺服器之間分配傳入請求。
```

負載平衡器 (LB) 是該問題的答案。負載平衡器的工作是在可用伺服器池中公平地分配所有客戶端的請求。負載平衡器執行此工作以避免伺服器過載或崩潰。

負載平衡層是資料中心內防火牆後的第一個接觸點。如果服務每秒處理幾百甚至幾千個請求，則可能不需要負載平衡器。但是，隨著客戶端請求的增加，負載平衡器提供以下功能：

- **Scalability (可擴展性)**: By adding servers, the capacity of the application/service can be increased seamlessly. Load balancers make such upscaling or downscaling transparent to the end users.
- **可擴展性**：通過添加伺服器，可以無縫增加應用程式/服務的容量。負載平衡器使這種擴大規模或縮小規模對最終使用者透明。
- **Availability (可用性)**: Even if some servers go down or suffer a fault, the system still remains available. One of the jobs of the load balancers is to hide faults and failures of servers.
- **可用性**：即使某些伺服器宕機或發生故障，系統仍然保持可用。負載平衡器的工作之一是隱藏伺服器的故障和失敗。
- **Performance (性能)**: Load balancers can forward requests to servers with a lesser load so the user can get a quicker response time. This not only improves performance but also improves resource utilization.
- **性能**：負載平衡器可以將請求轉發到負載較小的伺服器，以便使用者可以獲得更快的響應時間。這不僅提高了性能，還提高了資源利用率。

這是負載平衡器工作原理的抽象描述：

[負載平衡器的簡化工作原理](./lbsymplified.jpg)

## Placing load balancers (放置負載平衡器)
通常，LB 位於客戶端和伺服器之間。請求通過負載平衡層到達伺服器並返回客戶端。但是，這並不是使用負載平衡器的唯一點。

讓我們考慮三個著名的伺服器組。即 Web、應用程式和資料庫伺服器。為了在可用伺服器之間分配流量負載，可以在這三個服務的伺服器實例之間使用負載平衡器，方式如下：

```
在應用程式的最終使用者和 Web 伺服器/應用程式網關之間放置 LB。
在 Web 伺服器和運行業務/應用程式邏輯的應用程式伺服器之間放置 LB。
在應用程式伺服器和資料庫伺服器之間放置 LB。
[三層架構中負載平衡器的可能用法](./lb_usage.jpg)


實際上，在系統設計中，負載平衡器可能用於具有多個實例的任何兩個服務之間。

## Services offered by load balancers (負載平衡器提供的服務)
LB 不僅使服務具有可擴展性、可用性和高性能，而且還提供一些關鍵服務，如下所示：

- **Health checking (健康檢查)**: LBs use the heartbeat protocol to monitor the health and, therefore, reliability of end-servers. Another advantage of health checking is the improved user experience.
- **健康檢查**：LB 使用心跳協議來監控終端伺服器的健康狀況，從而監控其可靠性。健康檢查的另一個優點是改善了使用者體驗。

- **TLS termination (TLS 終止)**: LBs reduce the burden on end-servers by handling TLS termination with the client.
- **TLS 終止**：LB 通過處理與客戶端的 TLS 終止來減輕終端伺服器的負擔。

- **Predictive analytics (預測分析)**: LBs can predict traffic patterns through analytics performed over traffic passing through them or using statistics of traffic obtained over time.
- **預測分析**：LB 可以通過對通過它們的流量進行分析或使用隨時間獲取的流量統計資料來預測流量模式。

- **Reduced human intervention (減少人工干預)**: Because of LB automation, reduced system administration efforts are required in handling failures.
- **減少人工干預**：由於 LB 自動化，處理故障所需的系統管理工作量減少了。

- **Service discovery (服務發現)**: An advantage of LBs is that the clients’ requests are forwarded to appropriate hosting servers by inquiring about the service registry.
- **服務發現**：LB 的一個優點是可以通過查詢服務註冊表將客戶端的請求轉發到適當的託管伺服器。

- **Security (安全性)**: LBs may also improve security by mitigating attacks like denial-of-service (DoS) at different layers of the OSI model (layers 3, 4, and 7).
- **安全性**：LB 還可以通過在 OSI 模型的不同層 (第 3、4 和 7 層) 緩解拒絕服務 (DoS) 等攻擊來提高安全性。


總體而言，負載平衡器為系統的整體設計提供了靈活性、可靠性、冗餘和效率。


```
心跳協議是一種識別分佈式系統中故障的方法。使用此協議，集群中的每個節點都會定期向監控服務報告其健康狀況。
```

```
TLS 終止 (也稱為 TLS/SSL 卸載) 是通過資料的加密/解密在客戶端和伺服器之間建立安全通信通道。
```

```
服務註冊表是 (微) 服務以及每個服務可用的實例的存儲庫。
```

```
DoS 是一種攻擊，客戶端通過大量流量淹沒伺服器以耗盡伺服器的資源 (處理和/或內存)，從而無法處理合法使用者的請求。
```

```
開放系統互連 (OSI) 模型是一個概念框架，將連接任何兩台機器的問題分為七個不同的層。
```

```
靈活性：即時透明地添加或刪除機器。
可靠性：可以通過健康監控刪除有缺陷的主機，這使得系統可靠。
冗餘：通往同一目的地的多條路徑或故障伺服器的負載被重新路由到故障轉移機器。
效率：從服務提供商的角度來看，將負載均勻分配到所有機器上以有效地使用它們。
```

```
Question
如果負載平衡器發生故障怎麼辦？它們不是單點故障 (SPOF) 嗎？

Answer
負載平衡器通常成對部署，作為災難恢復的一種手段。如果一個負載平衡器發生故障，並且沒有可以故障轉移的對象，則整個服務將中斷。通常，為了保持高可用性，企業使用負載平衡器集群，利用心跳通信隨時檢查負載平衡器的健康狀況。如果主 LB 發生故障，備份 LB 可以接管。但是，如果整個集群發生故障，在緊急情況下也可以執行手動重新路由。
```

在接下來的課程中，我們將看到如何在複雜的應用程式中使用負載平衡器，以及哪種類型的負載平衡器適合哪種用例。
