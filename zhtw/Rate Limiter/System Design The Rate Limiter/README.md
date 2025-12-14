# System Design: The Rate Limiter (系統設計：速率限制器)

## What is a rate limiter? (什麼是速率限制器？)
顧名思義，速率限制器限制了服務滿足的請求數量。它限制超過預定義限制的請求。例如，如果客戶端使用的特定服務 API 配置為每分鐘允許 500 個請求，那麼如果客戶端發出的請求數量超過該限制，該服務將阻止該客戶端進一步的傳入請求。

## Why do we need a rate limiter? (為什麼我們需要速率限制器？)
速率限制器通常用作服務的防禦層，以避免服務被過度使用，無論是有意還是無意。它還可以保護服務免受針對應用層的濫用行為的影響，例如拒絕服務 (DOS) 攻擊和暴力破解密碼嘗試。

下面，我們列出了可以使用速率限制器使服務更可靠的場景。

- **Preventing resource starvation (防止資源匱乏)**: Some denial of service incidents are caused by errors in software or configurations in the system, which causes resource starvation. Such attacks are referred to as friendly-fire denial of service. One of the common use cases of rate limiters is to avoid resource starvation caused by such denial of service attacks, whether intentional or unintentional.
- **防止資源匱乏**：一些拒絕服務事件是由系統中的軟件或配置錯誤引起的，這會導致資源匱乏。此類攻擊被稱為友軍拒絕服務。速率限制器的常見用例之一是避免由此類拒絕服務攻擊 (無論是有意還是無意) 引起的資源匱乏。

- **Managing policies and quotas (管理策略和配額)**: There is also a need for rate limiters to provide a fair and reasonable use of resources’ capacity when they are shared among many users. The policy refers to applying limits on the time duration or quantity allocated (quota).
- **管理策略和配額**：當許多使用者共享資源時，也需要速率限制器來提供公平合理的使用資源容量。該策略是指對分配的時間持續時間或數量 (配額) 施加限制。

- **Controlling data flow (控制資料流)**: Rate limiters could also be used in systems where there is a need to process a large amount of data. Rate limiters control the flow of data to distribute the work evenly among different machines, avoiding the burden on a single machine.
- **控制資料流**：速率限制器也可用於需要處理大量資料的系統中。速率限制器控制資料流，以將工作均勻分配給不同的機器，避免單台機器的負擔。

- **Avoiding excess costs (避免超額成本)**: Rate limiting can also be used to control the cost of operations. For example, organizations can use rate limiting to prevent experiments from running out of control and avoid large bills. Some cloud service providers also use this concept by providing freemium services to certain limits, which can be increased on request by charging from users.
- **避免超額成本**：速率限制也可用於控制運營成本。例如，組織可以使用速率限制來防止實驗失控並避免大額賬單。一些雲服務提供商也使用此概念，通過提供達到一定限制的免費增值服務，可以根據要求通過向使用者收費來增加限制。

[通過速率限制器限制對 API 伺服器的請求數量](./sys.jpg)

## How will we design a rate limiter? (我們將如何設計速率限制器？)
在接下來的課程中，我們將學習以下內容：

1. **Requirements (需求)**: This is where we discuss the functional and non-functional requirements of the rate limiter. We also describe the types of throttling and locations where a rate limiter can be placed to perform its functions efficiently.
1. **需求**：我們將在這裡討論速率限制器的功能和非功能需求。我們還描述了節流的類型以及可以放置速率限制器以有效執行其功能的位置。

2. **High-level design (高層設計)**: In this section, we look at the high-level design to provide an overview of a rate limiter.
2. **高層設計**：在本節中，我們將查看高層設計以提供速率限制器的概述。

3. **Detailed design (詳細設計)**: In this section, we discuss the detailed design of a rate limiter and explain various building blocks involved in the detailed design.
3. **詳細設計**：在本節中，我們將討論速率限制器的詳細設計，並解釋詳細設計中涉及的各種構建塊。

4. **Rate limiter algorithms (速率限制器算法)**: In this lesson, we explain different algorithms that play a vital role in the operations of a rate limiter.
4. **速率限制器算法**：在本課中，我們將解釋在速率限制器操作中起重要作用的不同算法。

5. **Quiz (測驗)**: To assess your understanding of rate limiters, we’ve provided a quiz at the end of this chapter.
5. **測驗**：為了評估您對速率限制器的理解，我們在本章末尾提供了一個測驗。

在下一課中，讓我們從了解速率限制器的需求和設計開始。
