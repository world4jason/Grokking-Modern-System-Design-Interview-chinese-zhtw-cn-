# Design of a Monitoring System (監控系統的設計)

## Requirements (需求)
讓我們總結一下我們希望監控系統為我們做什麼：

- 監控伺服器上的關鍵本地進程是否崩潰。

- 監控伺服器上進程對 CPU/內存/磁盤/網絡頻寬的使用是否存在任何異常。

- 監控整體伺服器健康狀況，如 CPU、內存、磁盤、網絡頻寬、平均負載等。

- 監控伺服器上的硬件組件故障，例如內存故障、磁盤故障或變慢等。

- 監控伺服器連接伺服器外關鍵服務 (如網絡文件系統等) 的能力。

- 監控所有網絡交換機、負載平衡器和資料中心內的任何其他專用硬件。

- 監控伺服器、機架和資料中心級別的功耗。

- 監控伺服器、機架和資料中心上的任何電源事件。

- 監控外部客戶端的路由資訊和 DNS。

- 監控資料中心內部和資料中心之間的網絡鏈路和路徑延遲。

- 監控對等點的網絡狀態。

- 監控可能跨越多個資料中心的整體服務健康狀況——例如，CDN 及其性能。


我們希望自動監控能夠識別系統中的異常並通知警報管理器或在儀表板上顯示進度。雲服務提供商提供其服務的健康狀態：

- AWS: https://health.aws.amazon.com/health/status
- Azure: https://status.azure.com/en-us/status
- Google: https://status.cloud.google.com/

## Building block we will use (我們將使用的構建塊)
分佈式監控的設計將由以下構建塊組成：

Blob 存儲：我們將使用 blob 存儲來存儲有關指標的資訊。

## High-level design (高層設計)
監控服務的高層組件如下：

- **Storage (存儲)**: A time-series database stores metrics data, such as the current CPU use or the number of exceptions in an application.
- **存儲**：時間序列資料庫存儲指標資料，例如當前 CPU 使用率或應用程式中的異常數量。

- **Data collector service (資料收集器服務)**: This fetches the relevant data from each service and saves it in the storage.
- **資料收集器服務**：這從每個服務獲取相關資料並將其保存在存儲中。

- **Querying service (查詢服務)**: This is an API that can query on the time-series database and return the relevant information.
- **查詢服務**：這是一個 API，可以查詢時間序列資料庫並返回相關資訊。

[監控系統的高層設計](./system.jpg)

讓我們在下一課中深入研究上面提到的組件。
