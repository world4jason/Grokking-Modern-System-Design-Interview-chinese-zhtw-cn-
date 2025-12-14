# Introduction to Distributed Monitoring (分佈式監控介紹)
## Need for monitoring (監控的必要性)
讓我們回顧一下單個服務的故障如何影響相關系統的順利執行。為了避免級聯故障，監控可以通過早期預警或引導我們找到故障的根本原因來發揮至關重要的作用。

讓我們考慮這樣一個場景：使用者將視頻 intro-to-system-design 上傳到 YouTube。伺服器 A 中的 UI 服務獲取視頻資訊並將資料提供給伺服器 B 中的服務 2。服務 2 在資料庫中進行條目並將視頻存儲在 blob 存儲中。伺服器 C 中的另一個服務 3 管理資料庫 X 和 Y 的複製和同步。

在這種情況下，服務 3 由於某種錯誤而失敗，服務 2 在資料庫 X 中進行條目。資料庫 X 崩潰，獲取視頻的請求被路由到資料庫 Y。使用者想要播放視頻 intro-to-system-design，但它會給出“視頻未找到...”的錯誤。

[監控](./monitoring)

上面的例子相對簡單。實際上，由於我們在全球擁有許多資料中心，並且每個資料中心都有數百萬台伺服器，因此會遇到複雜的問題。由於人工管理員與伺服器的比例不斷下降，手動查找問題通常是不可行的。擁有監控系統可以降低運營成本並鼓勵以自動化方式檢測故障。

### Downtime cost (停機成本)
有容錯系統設計可以向最終使用者隱藏大多數故障，但在故障演變成更大的問題之前捕獲故障至關重要。服務中的意外中斷可能會造成高昂的代價。例如，在 2021 年 10 月，Meta 的應用程式宕機了近九個小時，導致每小時損失約 1300 萬美元。這些損失強調了中斷的潛在影響。

IT 基礎設施廣泛分佈在全球各地。下一段下面的插圖概述了 2021 年左右全球主要雲提供商的分佈式資料中心。資料中心通過專用或公共網絡連接。監控地理上分離的資料中心中的伺服器至關重要。

據亞馬遜稱，2021 年 12 月 7 日，“太平洋標準時間上午 7:30，一項自動擴展託管在主 AWS 網絡中的一項 AWS 服務容量的活動觸發了內部網絡中大量客戶端的意外行為。這導致連接活動激增，壓垮了內部網絡和主 AWS 網絡之間的網絡設備，導致這些網絡之間的通訊延遲。這些延遲增加了在這些網絡之間通訊的服務的延遲和錯誤，導致更多的連接嘗試和重試。這導致連接兩個網絡的設備持續擁塞和性能問題。”據估計，亞馬遜的中斷成本為每分鐘 66,240 美元。

[AWS、Azure 和 Google 全球分佈式資料中心概覽](./overview.jpg)

### Types of monitoring (監控類型)
讓我們考慮一個例子來了解我們想要監控的錯誤類型。在 Educative，每當學習者連接到可執行環境時，都會分配一個容器。考慮伺服器 A 中的服務 1，它負責在學習者連接時分配容器。伺服器 B 上的另一個服務 2 獲取此資訊並通知負責 UI 的服務。在伺服器 C 中運行的 UI 服務會為學習者更新 UI。假設服務 2 由於某種錯誤而失敗，學習者會看到“無法連接...”的錯誤。

Educative 開發人員如何發現學習者正面臨這個錯誤？

[類型](./types)

現在，如果學習者發出請求但從未到達 Educative 的伺服器怎麼辦。Educative 如何知道學習者正面臨問題？

通過上述示例，我們可以將監控重點分為兩類錯誤：

- **Service-side errors (伺服器端錯誤)**: These are errors that are usually visible to monitoring services as they occur on servers. Such errors are reported as error 5xx in HTTP response codes.
- **Service-side errors (伺服器端錯誤)**：這些錯誤通常對監控服務可見，因為它們發生在伺服器上。此類錯誤在 HTTP 響應代碼中報告為錯誤 5xx。

- **Client-side errors (客戶端錯誤)**: These are errors whose root cause is on the client-side. Such errors are reported as error 4xx in HTTP response codes. Some client-side errors are invisible to the service when client requests fail to reach the service.
- **Client-side errors (客戶端錯誤)**：這些是根本原因在客戶端的錯誤。此類錯誤在 HTTP 響應代碼中報告為錯誤 4xx。當客戶端請求無法到達服務時，某些客戶端錯誤對服務不可見。

我們將在接下來的章節“監控伺服器端錯誤”和“監控客戶端錯誤”中探討如何設計監控服務來處理這兩種情況。我們希望我們的監控系統能夠分析我們的全球分佈式服務。它可以更好地了解系統的組件及檢測和響應故障的敏捷性。
