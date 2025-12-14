# AWS Wide Spread Outage (AWS 廣泛中斷)

## Introduction (簡介)
太平洋標準時間 2021 年 12 月 7 日星期二上午 7:35 左右，多項 Amazon 服務和其他依賴 AWS 的服務因持續超過八小時的中斷事件而中斷。該事件影響了從家庭消費產品到眾多商業服務的方方面面。

這場持續數小時的中斷成為大眾媒體的頭條新聞，例如《金融時報》的這一條：“從憤怒的 Adele 粉絲到損壞的掃地機器人：AWS 中斷在美國引發連鎖反應。”此次中斷影響了全球數百萬使用者，包括使用 AWS 在線商店的個人和其他嚴重依賴 AWS 提供服務的企業。

AWS 造成的中斷凸顯了去中心化互聯網的必要性，即服務不依賴於少數幾家巨頭公司。根據 Gartner 的資料，80% 的雲市場僅由五家公司處理。亞馬遜擁有 41% 的雲計算市場份額，是最大的一家。

像上面這樣的中斷提醒我們著名的 Lamport 妙語：“分佈式系統中，一台你甚至不知道存在的計算機發生故障，可能會導致你自己的計算機無法使用。”

## Sequence of events (事件順序)
- 擴展主 AWS 網絡附近的一項 AWS 服務容量的自動操作引發了內部網絡中大量客戶的異常行為。

- 結果，連接活動顯著增加，淹沒了連接內部網絡和主 AWS 網絡的網絡設備。

- 這些網絡之間的通信被延遲。這些延遲增加了這些網絡之間交互的服務的延遲和故障，導致重試和 ping 請求增加。

- 結果，連接這兩個網絡的設備經歷了持續的過載和性能困難。

- 這種過載立即影響了 AWS 內部運營團隊的實時監控資料的可用性，阻礙了他們識別和糾正擁塞原因的能力。

- 運營商依靠日誌來弄清楚發生了什麼，最初觀察到內部 DNS 故障增加。

以下幻燈片顯示了導致中斷的一系列事件。

[事件順序](./arch)

## Analysis (分析)
- **Hampered AWS services (受阻的 AWS 服務)**: The networking difficulties affected a variety of AWS services, impacting customers that utilized these service capabilities. Since the primary AWS network remained unaffected, certain client applications that don’t depend on these capabilities suffered relatively minor consequences as a result of this occurrence. AWS users, such as Amazon RDS, EMR, and Workspaces, were unable to generate new resources due to the inability of the system to launch new EC2 instances.
- **受阻的 AWS 服務**：網絡困難影響了各種 AWS 服務，影響了使用這些服務功能的客戶。由於主 AWS 網絡未受影響，因此不依賴於這些功能的某些客戶端應用程式受此事件的影響相對較小。由於系統無法啟動新的 EC2 實例，Amazon RDS、EMR 和 Workspaces 等 AWS 使用者無法生成新資源。

- **Impaired control plane (受損的控制平面)**: Apart from the AWS services, the AWS control planes that are used for establishing and managing AWS resources were also impacted. These control planes take advantage of internal network-hosted services. For example, EC2 instances weren’t affected by this event, but EC2 APIs suffered from increased latency and error rates.
- **受損的控制平面**：除了 AWS 服務外，用於建立和管理 AWS 資源的 AWS 控制平面也受到影響。這些控制平面利用內部網絡託管的服務。例如，EC2 實例不受此事件的影響，但 EC2 API 的延遲和錯誤率增加。

- **Slow restoration (恢復緩慢)**: Since DNS is the basis for all communication across the web, operators focused on moving the internal DNS traffic away from congested areas of the network in order to improve availability. However, since monitoring services were unavailable, operators had to identify and disable major sources of traffic manually. This further improved the availability of services.
- **恢復緩慢**：由於 DNS 是網絡上所有通信的基礎，因此運營商專注於將內部 DNS 流量從網絡擁塞區域移開，以提高可用性。但是，由於監控服務不可用，運營商必須手動識別並禁用主要流量源。這進一步提高了服務的可用性。

- **Elastic Load Balancers (ELB) (彈性負載平衡器)**: Current Elastic Load Balancers were unaffected by the incident. However, the rising API error rates and latencies for the ELB APIs resulted in longer provisioning times for new load balancers.
- **彈性負載平衡器 (ELB)**：目前的彈性負載平衡器未受事件影響。但是，ELB API 的 API 錯誤率和延遲上升導致新負載平衡器的配置時間變長。

## Lessons learned (經驗教訓)
- **Independent communication system (獨立的通信系統)**: While the intention of having an internal network that’s separate from the main network is the right idea, they weren’t truly independent. A sequence of events highlighted their dependency. Finding such dependencies is crucial to truly benefit from independent networks for internal service use and external client use.
- **獨立的通信系統**：雖然擁有與主網絡分開的內部網絡的意圖是正確的，但它們並不是真正獨立的。一系列事件凸顯了它們的依賴性。發現這種依賴關係對於真正從用於內部服務和外部客戶端使用的獨立網絡中受益至關重要。

- **Contingency plan (應急計劃)**: Although AWS takes measures to prepare its infrastructure for sudden surges in customer requests or power usage, the organization still found itself in a difficult situation due to the unusual severity of the failure. Investing in greater risk-based contingency planning benefits organizations during times of crisis.
- **應急計劃**：儘管 AWS 採取措施準備其基礎設施以應對客戶請求或電力使用的突然激增，但由於故障的異常嚴重性，該組織仍發現自己處於困境。投資於基於風險的更大應急計劃有利於組織度過危機時期。

- **Ready operations team (準備就緒的運營團隊)**: A bug bringing an overall system to a halt is a single point of failure, which is possible in a complex system. The production team should be trained and ready for such events.
- **準備就緒的運營團隊**：導致整個系統停止的錯誤是單點故障，這在復雜系統中是可能的。生產團隊應接受培訓並為此類事件做好準備。

- **Multiple cloud computing providers (多個雲計算提供商)**: Organizations can replicate their operations among many cloud computing providers so that no single failure knocks them out of action. However, this is easier said than done. An alternative approach is to employ different regions of the same provider for various purposes.
- **多個雲計算提供商**：組織可以在許多雲計算提供商之間複製其運營，以便沒有任何單一故障會使其停止運作。然而，說起來容易做起來難。另一種方法是為了各種目的使用同一提供商的不同區域。

- **Testing (測試)**: Carrying out proper testing and identifying the potential bugs are both essential. In this case, overwhelming the network devices resulted in communication delays between these networks.
- **測試**：進行適當的測試和識別潛在的錯誤至關重要。在這種情況下，淹沒網絡設備導致這些網絡之間的通信延遲。

```
Question
我們可以做些什麼來防範亞馬遜經歷的一系列故障？

Answer
我們建議採取以下解決方案：

每一層的端到端透明度提供了正確運行站點和服務所需的資訊。

跨多個雲或 AWS 區域構建應用程式將使受影響的客戶更容易快速恢復。

需要極度解耦服務以消除交叉依賴問題。
```
