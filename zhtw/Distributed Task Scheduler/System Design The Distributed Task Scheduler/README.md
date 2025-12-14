# System Design: The Distributed Task Scheduler (系統設計：分佈式任務調度器)
## What is a task scheduler? (什麼是任務調度器？)
任務是一項需要資源 (CPU 時間、內存、存儲、網絡帶寬等) 持續一段指定時間的計算工作。例如，在 Facebook 或 Instagram 上上傳照片或視頻包含以下後台任務：

1. 以多種解析度對照片或視頻進行編碼。
2. 驗證照片或視頻以檢查內容貨幣化版權等。

```
內容貨幣化是一種利用內容的方式，以便服務可以在使用者消費內容時從中獲利。
```

上述所有任務的成功執行使照片或視頻可見。但是，照片和視頻上傳者不需要停止上述任務即可完成。

另一個例子是我們在 Facebook 上發布評論時。在評論傳遞給所有關注者之前，我們不會阻止評論發布者。該傳遞被委託給異步任務調度器以離線進行。

在一個系統中，許多任務爭奪有限的計算資源。通過智能地將資源分配給任務以滿足任務級和系統級目標來在任務和資源之間進行調解的系統稱為任務調度器。

## When to use a task schedular (何時使用任務調度器)
任務調度器是系統高效完成工作的關鍵組件。它允許我們使用有限的資源完成大量任務。它還有助於充分利用系統資源，為使用者提供不間斷的執行體驗等。以下是任務調度的一些用例：

**Single-OS-based node (基於單個操作系統的節點)**: It has many processes or tasks that contend for the node’s limited computational resources. So, we could use a local OS task scheduler that efficiently allocates resources to the tasks. It uses multi-feedback queues to pick some tasks and runs them on some processor.
**基於單個操作系統的節點**：它有許多進程或任務爭奪節點有限的計算資源。因此，我們可以使用本地操作系統任務調度器來有效地為任務分配資源。它使用多反饋隊列來挑選一些任務並在某個處理器上運行它們。

**Cloud computing services (雲計算服務)**: Where there are many distributed resources and various tasks from multiple tenants, there is a strong need for a task scheduler to utilize cloud computing resources efficiently and meet tenants’ demands. A local OS task scheduler isn’t sufficient for this purpose because the tasks are in the billions, the source of the tasks is not single, and the resources to manage are not in a single machine. We have to go for a distributed solution.
**雲計算服務**：如果有許多分佈式資源和來自多個租戶的各種任務，則強烈需要任務調度器來有效地利用雲計算資源並滿足租戶的需求。本地操作系統任務調度器不足以實現此目的，因為任務數以十億計，任務來源不是單一的，並且要管理的資源不在單台機器中。我們必須尋求分佈式解決方案。

**Large distributed systems (大型分佈式系統)**: In this system, many tasks run in the background against a single request by a user. Consider that there are millions to billions of users of a popular system like Facebook, WhatsApp, or Instagram. These systems require a task scheduler to handle billions of tasks. Facebook schedules its tasks against billions of parallel asynchronous requests by its users using Async.
**大型分佈式系統**：在這個系統中，許多任務在後台針對使用者的單個請求運行。考慮一下 Facebook、WhatsApp 或 Instagram 等流行系統有數百萬到數十億使用者。這些系統需要任務調度器來處理數十億個任務。Facebook 使用 Async 針對其使用者發出的數十億個並行異步請求來調度其任務。

```
不間斷的執行體驗：無需使用者干預即可一次又一次地啟動任務執行來運行週期性任務。

異步請求：異步請求是不在客戶端關鍵路徑上且通常具有一定延遲容忍度的請求。使用者請求這些任務，系統完成任務，而請求者無需等待工作完成。有關任務最終狀態的通知將在將來的某個時間傳達給請求者。
```

```
注意：Async 是 Facebook 自己的分佈式任務調度器，用於調度其所有任務。有些任務對時間更敏感，例如通知使用者活動直播已開始的任務。如果使用者在直播結束後才收到有關直播的通知，那就毫無意義了。有些任務可以延遲，例如向使用者提供好友建議的任務。Async 根據適當的優先級調度任務。
```

## Distributed task scheduling (分佈式任務調度)
及時決定任務並將資源分配給任務的過程稱為任務調度。操作系統級任務調度器和資料中心級任務調度器之間的視覺差異如下圖所示：

[操作系統級任務調度器與資料中心級任務調度器]

操作系統任務調度器在該節點的計算資源上調度節點的本地任務或進程。同時，資料中心的任務調度器調度來自使用資料中心資源的多個租戶的數十億個任務。

我們的目標是設計一個類似於資料中心級任務調度器的任務調度器，其中考慮了以下內容：

- 任務將來自許多不同的來源、租戶和子系統。
- 許多資源將分散在資料中心 (或者可能跨越多個資料中心)。

上述兩個需求使得任務調度問題充滿挑戰。我們將設計一個分佈式任務調度器，通過使其具有可擴展性、可靠性和容錯性來處理所有這些任務。

## How will we design a task scheduling system? (我們將如何設計任務調度系統？)
1. **Requirements (需求)**: We’ll identify the functional and non-functional requirements of a task scheduling system in this lesson.
1. **需求**：我們將在本課中確定任務調度系統的功能和非功能需求。
2. **Design (設計)**: This lesson will discuss the system design of our task scheduling system and explores the components of the system and database schema.
2. **設計**：本課將討論我們任務調度系統的系統設計，並探討系統的組件和資料庫模式。
3. **Design considerations (設計注意事項)**: In this lesson, we’ll highlight some design factors, such as task prioritization, resource optimization, and so on.
3. **設計注意事項**：在本課中，我們將重點介紹一些設計因素，例如任務優先級、資源優化等。
4. **Evaluation (評估)**: We’ll evaluate our design of task scheduler based on our requirements.
4. **評估**：我們將根據我們的需求評估我們的任務調度器設計。

讓我們從了解任務調度系統的需求開始。
