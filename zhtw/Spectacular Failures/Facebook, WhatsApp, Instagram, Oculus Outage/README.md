# Facebook, WhatsApp, Instagram, Oculus Outage (Facebook, WhatsApp, Instagram, Oculus 中斷)

2021 年 10 月，Facebook 經歷了大約 6 個小時的全球中斷，影響了其其他關聯公司，包括 Messenger，WhatsApp，Mapillary，Instagram 和 Oculus。流行媒體顯著報導了這次失敗的影響。《紐約時報》報導了以下標題：“幾分鐘內消失，數小時無法使用：停電震動了 Facebook。”

據估計，這次停電使 Facebook 損失了約 1 億美元的收入，並且由於公司股票下跌損失了數十億美元。

讓我們看看導致這個全球性問題的事件順序。

## The sequence of events (事件順序)
以下事件順序導致 Facebook 及其隨附服務中斷：

A routine maintenance system was needed to find out the spare capacity on Facebook’s backbone network.
需要一個例行維護系統來找出 Facebook 骨幹網上的備用容量。
由於配置錯誤，維護系統斷開了骨幹網上所有資料中心之間的連接。早些時候，使用了自動配置審查工具來查找配置中的任何問題，但像這樣的工具並不完美。在這個具體案例中，審查工具遺漏了配置中存在的問題。

Facebook 的權威域名系統 (DNS) 有一條運行狀況檢查規則，如果它無法到達 Facebook 的內部資料中心，它將通過撤回路由停止回覆客戶端 DNS 查詢。

當託管 Facebook 權威 DNS 的網絡路由被撤回時，所有人類可讀名稱到 IP 的緩存映射很快在所有公共 DNS 解析器處超時。當客戶端解析 www.facebook.com 時，DNS 解析器首先轉到根 DNS 伺服器之一，根 DNS 伺服器提供 .com 的權威 DNS 伺服器列表。解析器連接到其中一個，然後它們提供 Facebook 權威 DNS 伺服器的 IP。但是，在路由撤回後，無法到達它們。

Then, no one was able to reach Facebook and its subsidiaries.
然後，沒有人能夠聯繫到 Facebook 及其子公司。
下面的幻燈片以圖形形式描述了事件。

[Facebook 的高級架構](./arch)

## Analysis (分析)
上述一系列事件的一些關鍵要點如下：

- **From common activity to catastrophe (從普通活動到災難)**: The withdrawal or addition of the network routes is a relatively common activity. The confluence of bugs (first a faulty configuration, and then a bug in an audit tool not able to detect such a problem) triggered a chain of events, which resulted in cascading failures. A cascading failure is when one failure can trigger another failure, ultimately bringing the whole system down.
- **從普通活動到災難**：網絡路由的撤回或添加是一種相對常見的活動。錯誤的彙集 (首先是錯誤的配置，然後是審計工具中的錯誤無法檢測到此類問題) 觸發了一系列事件，導致連鎖故障。連鎖故障是指一個故障可能觸發另一個故障，最終導致整個系統崩潰。

- **Reasons for slow restoration (恢復緩慢的原因)**: It might seem odd that it took six hours to restore the service. Wasn’t it easy to reannounce the withdrawn routes? At the scale that Facebook is operating on, rarely is anything done manually, and there are automated systems to perform changes. The internal tools probably relied on the DNS infrastructure, and when all data centers are offline from the backbone, it would have been virtually impossible to use those tools. Manual intervention would have been necessary. Manually bootstrapping a system of this scale isn’t easy. The usual physical and digital security mechanisms that were in place made it a slow process to intervene manually.
- **恢復緩慢的原因**：恢復服務花費了六個小時，這似乎很奇怪。重新宣布撤回的路線不是很容易嗎？在 Facebook 運營的規模上，很少有手動完成的事情，並且有自動化系統來執行更改。內部工具可能依賴於 DNS 基礎設施，當所有資料中心都從骨幹網斷開時，幾乎不可能使用這些工具。手動干預是必要的。手動引導這種規模的系統並不容易。現有的通常物理和數字安全機制使得手動干預的過程很慢。

- **Low probability events can occur (小概率事件可能發生)**: In retrospect, it might seem odd that authoritative DNS systems disconnect themselves if internal data centers aren’t accessible. This is another example where a very rare event, such as none of the data centers being accessible, happened, triggering another event.
- **小概率事件可能發生**：回想起來，如果內部資料中心無法訪問，權威 DNS 系統會自行斷開連接，這似乎很奇怪。這是另一個例子，發生了一個非常罕見的事件 (例如沒有資料中心可訪問)，從而觸發了另一個事件。

- **Pitfalls of automation (自動化的陷阱)**: Facebook has been an early advocate of automating network configuration changes, effectively saying that software can do a better job of running the network than humans, who are more prone to errors. However, software can have bugs, such as this one.
- **自動化的陷阱**：Facebook 一直是自動化網絡配置更改的早期倡導者，實際上是說軟體可以比人類更好地運行網絡，因為人類更容易出錯。但是，軟體可能會有錯誤，例如這一個。


## Lessons learned (經驗教訓)

- **Ready operations team (準備就緒的運營團隊)**: There can be a hidden single point of failure in complex systems. The best defense against such faults is to have the operations team ready for such an occurrence through regular training. Thinking clearly under high-stress situations becomes necessary to deal with such events.
- **準備就緒的運營團隊**：複雜系統中可能存在隱藏的單點故障。針對此類故障的最佳防禦措施是讓運營團隊通過定期培訓為此類事件做好準備。在這種事件中，需要在高壓情況下清晰思考。

- **Simple system design (簡單的系統設計)**: As systems get bigger, they become more complex, and they have emergent behaviors. To understand the overall behavior of the system, it might not be sufficient to understand only the behavior of its components. Cascading failures can arise. This is one reason to keep the system design as simple as possible for the current needs and evolve the design slowly. Unfortunately, there’s no perfect solution to deal with this problem. We should accept the possibility of such failures, perform continuous monitoring, have the ability to solve issues when they arise, and learn from the failures to improve the system.
- **簡單的系統設計**：隨著系統變大，它們變得更加複雜，並且具有湧現行為。要了解系統的整體行為，僅了解其組件的行為可能是不夠的。可能會出現連鎖故障。這是保持系統設計盡可能簡單以滿足當前需求並緩慢演變設計的原因之一。不幸的是，沒有完美的解決方案來處理這個問題。我們應該接受此類故障的可能性，執行持續監控，有能力在問題出現時解決問題，並從故障中學習以改進系統。

- **Contingency plan (應急計劃)**: Some third-party services rely on Facebook for single sign-on. When the outage occurred, third-party services were up and running, but their clients were unable to use them because Facebook’s login facility was also unavailable. This is another example of assuming that some service will always remain available and of a hidden single point of failure.
- **應急計劃**：一些第三方服務依賴 Facebook 進行單點登錄。當發生中斷時，第三方服務已啟動並運行，但其客戶無法使用它們，因為 Facebook 的登錄功能也不可用。這是假設某些服務將始終保持可用以及隱藏的單點故障的另一個例子。

- **Hosting DNS to independent third-party providers (將 DNS 託管給獨立的第三方提供商)**: There are a few services that are so robustly designed and perfected over time that their clients start assuming that the service is and will always be 100% available. The DNS is one such service, and it has been very carefully crafted. Designers often assume that it will never fail. Hosting DNS to independent third-party providers might be a way to guard against such problems. DNS allows multiple authoritative servers, and an organization might have many at different places. Although, we should note that DNS at Facebook’s scale isn’t simple, is tightly connected to their backbone infrastructure, and changes frequently. Delegating such a piece to an independent third party is expensive, and it might reveal internal service details. So, there can be a trade-off between business and robustness needs.
- **將 DNS 託管給獨立的第三方提供商**：有一些服務設計得非常強大，並且隨著時間的推移不斷完善，以至於其客戶開始假設該服務是並且將永遠是 100% 可用的。DNS 就是這樣一種服務，並且已經過非常精心的製作。設計師通常認為它永遠不會失敗。將 DNS 託管給獨立的第三方提供商可能是預防此類問題的一種方法。DNS 允許多個權威伺服器，一個組織可能在不同地方有許多伺服器。雖然，我們應該注意，Facebook 規模的 DNS 並不簡單，與其骨幹基礎設施緊密相連，並且經常變化。將這樣一塊委託給獨立的第三方是昂貴的，並且可能會洩露內部服務細節。因此，在業務需求和穩健性需求之間可能存在權衡。

- **Trade-offs (權衡)**: There can be some surprising trade-offs. An example here is the need for data security and the need for rapid manual repair. Because so many physical and digital safeguards were in place, manual intervention was slow. This is a catch-22 situation—lowering security needs can cause immense trouble, and slow repair for such events can also make it hard for the companies. The hope is that the need for such repairs is a very rare event.
- **權衡**：可能會有一些令人驚訝的權衡。這裡的一個例子是資料安全需求與快速手動修復需求。由於採取了如此多的物理和數字保護措施，手動干預很慢。這是一個進退兩難的局面——降低安全需求可能會造成巨大麻煩，而此類事件的修復緩慢也會使公司處境艱難。希望這種維修的需求是一個非常罕見的事件。

- **Surge in load (負載激增)**: The failure of large players disrupts the entire Internet. Third-party public resolvers, such as Google and Cloudflare, saw a surge in the load due to unsuccessful DNS retries.
- **負載激增**：大型參與者的失敗擾亂了整個互聯網。由於 DNS 重試失敗，第三方公共解析器 (如 Google 和 Cloudflare) 的負載激增。

- **Resuming the service (恢復服務)**: Restarting a large service isn’t as simple as flipping a switch. When the load suddenly becomes almost zero, turning them up suddenly may lead to a many megawatt uptick in power usage. This might even cause issues for the electric grid. Complex systems usually have a steady state, and if they go out of that steady state, care must be taken to bring them back.
- **恢復服務**：重啟大型服務並不簡單，不像撥動開關那麼簡單。當負載突然幾乎為零時，突然調高它們可能會導致電力使用量增加許多兆瓦。這甚至可能導致電網問題。複雜系統通常具有穩態，如果它們脫離穩態，必須小心將它們帶回。

```
Question
我們可以做些什麼來防範 Facebook 經歷的那種故障？

Answer
一些可能的解決方案如下所示：

網絡驗證最近獲得了發展勢頭，並顯示出及早發現錯誤的希望。此類工具使用基礎設施的抽象模型。

可以有多個審計層。第二層可以使用模擬器來確保在配置更改後，關鍵網絡基礎設施仍可從多個全球有利位置獲得。

應盡一切努力縮小配置更改的範圍，以避免連鎖效應。

關鍵基礎設施可以以這樣一種方式進行編程：如果發生壞事，它可以恢復到上一個已知良好狀態。由於組件數量龐大，這說起來容易做起來難。

- 網絡驗證是使用網絡工具進行調查、故障排除、維護、操作和報告，以確保硬體、軟體和網絡配置將無錯誤運行。
```
