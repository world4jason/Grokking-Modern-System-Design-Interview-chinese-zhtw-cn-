# Requirements of a Distributed Task Scheduler's Design (分佈式任務調度器設計的需求)
## Requirements (需求)
讓我們從了解設計任務調度器的功能和非功能需求開始。

### Functional requirements (功能需求)
分佈式任務調度器的功能需求如下：

- **Submit tasks (提交任務)**: The system should allow the users to submit their tasks for execution.
- **提交任務**：系統應允許使用者提交任務以供執行。
- **Allocate resources (分配資源)**: The system should be able to allocate the required resources to each task.
- **分配資源**：系統應能夠為每個任務分配所需的資源。
- **Remove tasks (刪除任務)**: The system should allow the users to cancel the submitted tasks.
- **刪除任務**：系統應允許使用者取消已提交的任務。
- **Monitor task execution (監控任務執行)**: The task execution should be adequately monitored and rescheduled if the task fails to execute.
- **監控任務執行**：應充分監控任務執行情況，如果任務執行失敗，則應重新調度。
- **Efficient resource utilization (高效資源利用)**: The resources (CPU and memory) must be used efficiently in terms of time, cost, and fairness. Efficiency means that we do not waste resources. For example, if we allocate a heavy resource to a light task that can easily be executed on a cheap resource, it means that we have not efficiently utilized our resources. Fairness is all tenants’ ability to get the resources with equally likely probability in a certain cost class.
- **高效資源利用**：必須在時間、成本和公平性方面有效地利用資源 (CPU 和內存)。效率意味着我們不浪費資源。例如，如果我們將繁重的資源分配給可以輕鬆在廉價資源上執行的輕量級任務，則意味着我們沒有有效地利用資源。公平性是指所有租戶在特定成本類別中以相同可能性獲得資源的能力。
- **Release resources (釋放資源)**: After successfully executing a task, the system should take back the resources assigned to the task.
- **釋放資源**：成功執行任務後，系統應收回分配給該任務的資源。
- **Show task status (顯示任務狀態)**: The system should show the users the current status of the task.
- **顯示任務狀態**：系統應向使用者顯示任務的當前狀態。

[分佈式任務調度器的功能需求](./fr.jpg)

### Non-functional requirements (非功能需求)
分佈式任務調度器的非功能需求如下：

- **Availability (可用性)**: The system should be highly available to schedule and execute tasks.
- **可用性**：系統應具有高可用性以調度和執行任務。
- **Durability (持久性)**: The tasks received by the system should be durable and should not be lost.
- **持久性**：系統接收到的任務應持久保存，不應丟失。
- **Scalability (可擴展性)**: The system should be able to schedule and execute an ever-increasing number of tasks per day. 
- **可擴展性**：系統應能夠每天調度和執行不斷增加的任務數量。
- **Fault-tolerance (容錯性)**: The system must be fault-tolerant by providing services uninterrupted despite faults in one or more of its components.
- **容錯性**：系統必須具有容錯性，即使一個或多個組件出現故障，也能不間斷地提供服務。
- **Bounded waiting time (有限等待時間)**: This is how long a task needs to wait before starting execution. We must not execute tasks much later than expected. Users shouldn’t be kept on waiting for an infinite time. If the waiting time for users crosses a certain threshold, they should be notified.
- **有限等待時間**：這是任務在開始執行之前需要等待的時間。我們不得比預期晚很多執行任務。不應讓使用者無限期地等待。如果使用者的等待時間超過某個閾值，則應通知他們。

[分佈式任務調度器的非功能需求](./nfr.jpg)

到目前為止，在本課中，我們大致了解了任務調度器，並區分了集中式和分佈式任務調度器。最後，我們列出了分佈式任務調度器系統的需求。

### Building blocks we will use (我們將使用的構建模組)
我們將在任務調度系統的設計中利用以下構建模組：

[任務調度器的構建模組](./bb.jpg)

- **Rate limiter (速率限制器)** is required to limit the number of tasks so that our system is reliable.
- **速率限制器**需要限制任務數量，以便我們的系統可靠。
- **A sequencer (序列生成器)** is needed to uniquely identify tasks.
- **序列生成器**需要唯一標識任務。
- **Database(s) (資料庫)** are used to store task information.
- **資料庫**用於存儲任務資訊。
- **A distributed queue (分佈式隊列)** is required to arrange tasks in the order of execution.
- **分佈式隊列**需要按執行順序排列任務。
- **Monitoring (監控)** is essential to check the health of resources and to detect failed tasks to provide reliable service to the users.
- **監控**對於檢查資源的健康狀況和檢測失敗的任務以向使用者提供可靠的服務至關重要。

我們已經確定了任務調度器的需求。在下一課中，我們將根據這些需求設計我們的任務調度系統。
