# Design of a Client-side Monitoring System (客戶端監控系統的設計)
服務無法看到未在其基礎架構上發生的錯誤。儘管如此，這些故障對客戶來說同樣令人沮喪，他們可能不得不問他們的朋友，“服務 X 對你來說也宕機了嗎？”或前往 Downdetector 等網站查看是否有人報告同樣的問題。他們可能會通過 Tweet 或其他一些溝通渠道報告問題。然而，所有這些情況的反饋循環都很慢。作為服務提供商，我們希望盡快發現此類問題以採取補救措施。讓我們設計這樣一個系統。

## Initial design (初始設計)
為了確保客戶端的請求到達伺服器，我們將充當客戶端並執行可達性和健康檢查。我們需要在全球範圍內建立各種有利位置。我們可以運行一項服務，我們稱之為探測器 (prober)，它定期向服務發送請求以檢查可用性。通過這種方式，我們可以從許多不同的地方監控我們服務的可達性。

[探測器檢查網站的可用性](./probe.jpg)

### Issues with probers (探測器的問題)
探測器可能會出現以下問題：

- **Incomplete coverage (覆蓋不完整)**: We might not have good coverage across all autonomous systems. There are 100,000 unique autonomous systems on the Internet as of March 2021. It’s not cost-effective or even possible to put those many probes across the globe. Country or ISP-specific regulations and the need for periodic maintenance are additional hurdles to implementing such a scheme.
- **覆蓋不完整**：我們可能無法很好地覆蓋所有自治系統。截至 2021 年 3 月，互聯網上有 100,000 個獨特的自治系統。在全球範圍內投放如此多的探測器並不划算，甚至是不可能的。國家或 ISP 特定法規以及定期維護的需求是實施此類計劃的額外障礙。

- **Lack of user imitation (缺乏使用者模仿)**: Such probes might not represent a typical user behavior to explain how a typical user will use the service.
- **缺乏使用者模仿**：此類探針可能不代表典型使用者行為來解釋典型使用者將如何使用該服務。

```
注意：初始設計基於主動探測。
```

## Improve the design (改進設計)
我們可以將探測器嵌入到實際應用程式中，而不是在有利位置使用探測器。我們將有以下兩個組件：

- **Agent (代理)**: This is a prober embedded in the client application that sends the appropriate service reports about any failures.
- **代理**：這是嵌入在客戶端應用程式中的探測器，負責發送有關任何故障的適當服務報告。

- **Collector (收集器)**: This is a report collector independent of the primary service. It’s made independent to avoid the situations where client agents want to report an error to the failed service. We summarize errors reports from collectors and look for spikes in the errors graph to see client-side issues.
- **收集器**：這是一個獨立於主要服務的報告收集器。使其獨立是為了避免客戶端代理想要向故障服務報告錯誤的情況。我們總結來自收集器的錯誤報告，並在錯誤圖中查找峰值以查看客戶端問題。

下圖顯示了當主要服務無法訪問時，代理如何訪問獨立收集器：

[評估代理的報告以查看錯誤峰值](./evaluate.jpg)

這些收集器是大數據處理系統的層次結構。我們可以將它們放置在客戶端網絡附近，隨著時間的推移，我們可以從所有這些本地站點收集這些統計資料。我們將使用在線流處理系統使此類系統接近實時。如果我們主要尋找摘要統計資料，我們的系統可以容忍丟失一些錯誤報告。一些報告將相對於整體使用者群體。我們可能會說 1% 的服務使用者是“一些”。如果我們不想丟失任何報告，我們將需要更加小心地設計系統，這將更加昂貴。

現在，我們將解決以下問題：

- 使用者可以激活和停用客戶端報告嗎？
- 在故障條件下，客戶端代理如何到達收集器？
- 我們將如何保護使用者隱私？

## Activate and deactivate reports (激活和停用報告)
我們將使用自定義 HTML 標頭將適當的資訊發送到收集器。雖然客戶端通過瀏覽器訪問服務，但特定瀏覽器應該了解此功能，以便在 HTTP 請求中適當地填寫標頭資訊。對於製作瀏覽器並提供服務的組織 (例如，基於 Chromium 的瀏覽器)，此類功能可以隨著時間的推移進行整合和標準化。

另一個解決方案是使用服務控制的客戶端應用程式，然後我們可以輕鬆地通過 HTTP 包含此類標頭。

如果客戶端已同意，則客戶端可以填寫請求標頭。然後，服務可以使用策略和收集端點的適當值進行回复。

[激活和停用報告](./activate_deactivate.jpg)

## Reach collectors under faulty conditions (在故障條件下到達收集器)
收集器需要位於與我們試圖監控的 Web 服務端點不同的故障域中。客戶端可以嘗試不同故障域中的各種收集器，直到一個工作。我們可以在以下示例中看到類似的模式。有時，我們將這種現象稱為在故障的爆炸半徑之外。

如果我們想查看 IP 的可達性，我們會將服務託管在不同的 IP 上。如果我們監控域的可用性，我們會將收集器託管在不同的域上。如果我們想檢測自治系統路由是否被劫持，我們將服務託管在不同的自治系統中。但是，對於最後一英里的錯誤，作為服務提供商，我們無能為力。我們可能會在客戶端積累此類事件，並在下次連接時報告它們。服務可以影響剩餘的組件故障。

                 Reaching Collectors Under Faulty Conditions (在故障條件下到達收集器)
1.2.3.4 無法到達                  不同的伺服器 IP

無法解析 example.com              不同的域

AS 1234 被劫持                    不同的 ASN

CDN 可用                          不同的/無 CDN

最後一英里問題                    服務沒有現成的備份
```


## Protect user privacy (保護使用者隱私)
使用客戶端軟件的人類使用者應完全控制，以準確知道收集了哪些資料並隨每個請求發送。使用者還應該能夠隨時重新激活該功能。如果我們使用我們的客戶端應用程式 (而不是瀏覽器應用程式)，我們在報告中可以包含哪些診斷資訊方面具有很大的靈活性。對於基於瀏覽器的客戶端，我們可以避免以下資訊：

- 我們可以避免包含 traceroute 跳數來查看客戶端到服務的路徑。使用者可能對其地理位置很敏感。這可能類似於收集位置資訊。
- 我們可以避免包含正在使用的 DNS 解析器。同樣，DNS 的詳細資訊可能會洩露有關位置的一些資訊。
- 我們可以避免包含往返時間 (RTT) 和數據包丟失資訊。

```
注意：作為指導原則，我們應盡量少收集資訊，並且只能將其用於使用者同意的特定目的。
```
理想情況下，對於基於 Web 的客戶端，我們只應收集任何請求成功時記錄在 Web 日誌中的資訊。

除非測試服務的標準功能並報告此類探測的結果，否則我們不應使用任何主動探測。因此，排除了 traceroute 和 RTT 或數據包丟失資訊。

由於加密，任何中介 (如 ISP 或中間盒) 都無法更改、添加或刪除錯誤報告機制。同樣，指定的收集器是此類資料可以去的唯一地方。

## Conclusion (結論)
在分佈式系統中，很難檢測和響應客戶端的錯誤。因此，有必要監控此類事件以提供良好的使用者體驗。

我們可以使用獨立代理處理錯誤，該代理將有關任何故障的服務報告發送到收集器。就基礎設施和部署而言，此類收集器應獨立於主要服務。
