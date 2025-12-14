# Design of a Distributed Task Scheduler (分佈式任務調度器的設計)
## Components (組件)
我們可以考慮多個級別的調度。我們可能被要求設計一個組織內部完成的調度，以便在他們自己的機器集群上運行任務。在那裡，他們必須找到充足的資源，並需要決定先運行哪個任務。

另一方面，我們也可能被要求設計雲提供商用於調度來自多個客戶端的任務的調度。雲提供商需要決定先運行哪個任務以及先處理哪個客戶端，以便在不同租戶之間提供適當的隔離。

所以，一般來說，我們系統的大組件有：

- **Clients (客戶端)**: They initiate the task execution.
- **客戶端**：它們啟動任務執行。
- **Resources (資源)**: The task is executed on these components.
- **資源**：任務在這些組件上執行。
- **Scheduler (調度器)**: A scheduler performs processes between clients and resources and decides which task should get resources first.
- **調度器**：調度器在客戶端和資源之間執行流程，並決定哪個任務應首先獲得資源。

[調度器將任務放入隊列以進行資源分配](./sys.jpg)

如上圖所示，有必要將傳入的任務放入隊列中。原因如下：

- 我們現在可能沒有足夠的可用資源。
- 存在任務依賴性，某些任務需要等待其他任務。
- 我們需要將客戶端與任務執行解耦，以便它們可以將工作移交給我們的系統。然後我們的系統將其排隊等待執行。

讓我們設計一個應該能夠調度任何任務的任務調度系統。通常，許多任務的壽命相對較短——從幾秒到幾分鐘。對於長時間運行的任務，我們可能需要在應用程式級別進行定期校驗和和恢復的能力，以便從可能的故障中恢復。

讓我們假設我們機隊中的某個單一伺服器可以滿足每個任務的計算需求。對於需要許多伺服器的任務，要麼應用程式需要將它們分解為更小的任務供我們的系統使用，要麼從集群管理器採用長期資源獲取。

## Design (設計)
當任務進行調度時，它應該包含以下資訊：

- **Resource requirements (資源需求)**: The requirements include how many CPU cores it needs, how much RAM is required to execute this task, how much disk space is required, what should the disk access rate be (input/output rate per second, or IOPS), and how many TCP ports the task needs for the execution, and so on. But, it is difficult for the clients to quantify these requirements. To remedy this situation, we have different tiers of resources like basic, regular, and premium. The client can specify the requirement in terms of these tiers.
- **資源需求**：需求包括它需要多少個 CPU 核心，執行此任務需要多少 RAM，需要多少磁碟空間，磁碟訪問速率 (每秒輸入/輸出速率，或 IOPS) 應該是多少，以及任務執行需要多少個 TCP 端口等等。但是，客戶端很難量化這些需求。為了糾正這種情況，我們有不同層級的資源，如基本、常規和高級。客戶端可以根據這些層級指定需求。

- **Dependency (依賴性)**: Broadly, tasks can be of two types: dependent and independent.
- **依賴性**：廣義上講，任務可以分為兩種類型：依賴和獨立。
  - **Dependent tasks (依賴任務)** require executing one or more additional tasks for their complete execution. These tasks must run in a sequence. For a dependent task, the client should provide a list of the tasks on which a given task is dependent.
  - **依賴任務**需要執行一個或多個額外任務才能完成執行。這些任務必須按順序運行。對於依賴任務，客戶端應提供給定任務所依賴的任務列表。
  - **Independent tasks (獨立任務)** don’t depend on the execution of any other task. Independent tasks can run in parallel. We should know whether a task is dependent or independent. The dependency information helps to execute both dependent tasks in order and independent tasks in parallel for efficient utilization of resources.
  - **獨立任務**不依賴於任何其他任務的執行。獨立任務可以並行運行。我們應該知道一個任務是依賴的還是獨立的。依賴資訊有助於按順序執行依賴任務並並行執行獨立任務，以有效利用資源。

任務調度器的設計如下圖所示：

[任務調度器的設計](./details.jpg)

- **Clients (客戶端)**: The clients of the cloud providers are individuals or organizations from small to large businesses who want to execute their tasks.
- **客戶端**：雲提供商的客戶端是希望執行其任務的個人或從小型到大型企業的組織。

- **Rate limiter (速率限制器)**: The resources available for a client depend on the cost they pay. It is important to limit the number of tasks for the reliability of our service. For instance, X number of tasks per hour are allowed to enter the system. Others will get a message like “Limit exceeded” instead of accepting the task and responding late. A rate limiter limits the number of tasks the client schedules based on its subscription. If the limit is exceeded, it returns an error message to the client that the rate limit has been exceeded.
- **速率限制器**：客戶端可用的資源取決於他們支付的費用。限制任務數量對於我們服務的可靠性非常重要。例如，每小時允許 X 個任務進入系統。其他的將收到類似“超出限制”的消息，而不是接受任務並延遲響應。速率限制器根據其訂閱限制客戶端調度的任務數量。如果超出限制，它會向客戶端返回錯誤消息，指出已超出速率限制。

- **Task submitter (任務提交器)**: The task submitter admits the task if it successfully passes through the rate limiter. There isn’t a single task submitter. Instead, we have a cluster of nodes that admit the increasing number of tasks.
- **任務提交器**：如果任務成功通過速率限制器，任務提交器將接納該任務。沒有單一的任務提交器。相反，我們有一個節點集群來接納不斷增加的任務。

- **Unique ID generator (唯一 ID 生成器)**: It assigns unique IDs to the newly admitted tasks.
- **唯一 ID 生成器**：它為新接納的任務分配唯一 ID。

- **Database (資料庫)**: All of the tasks taken by the task submitter are stored in a distributed database. For each task, we have some attributes, and all of the attributes except one are stored in the relational database.
- **資料庫**：任務提交器獲取的所有任務都存儲在分佈式資料庫中。對於每個任務，我們都有一些屬性，除了這一個之外的所有屬性都存儲在關係資料庫中。
  - **Relational database (RDB) (關係資料庫)**: A relational database stores task IDs, user IDs, required resources, execution caps, the total number of attempts made by the client, delay tolerance, and so on, as shown in the following table. We can find the details on the RDB in Databases > Types of Databases.
  - **關係資料庫**：關係資料庫存儲任務 ID、使用者 ID、所需資源、執行上限、客戶端嘗試的總次數、延遲容忍度等，如下表所示。我們可以在資料庫 > 資料庫類型中找到有關 RDB 的詳細資訊。
  - **Graph database (GDB) (圖資料庫)**: This is a non-relational database that uses the graph data structure to store data. We use it to build and store a directed acyclic graph (DAG) of dependent tasks, topologically sorted by the task submitter, so that we can schedule tasks according to that DAG. We can find more details of the graph DB Databases > Types of Databases.
  - **圖資料庫**：這是一個非關係資料庫，使用圖資料結構存儲資料。我們使用它來構建和存儲依賴任務的有向無環圖 (DAG)，由任務提交器進行拓撲排序，以便我們可以根據該 DAG 調度任務。我們可以在資料庫 > 資料庫類型中找到有關圖形資料庫的更多詳細資訊。

Database Schema (資料庫模式)
列名                      資料類型              描述

                                              唯一標識每個任務。

                                              這是任務所有者的 ID。

                                              這可以是我們一次、每天、每週、每月或每年。

                                              這是任務執行失敗時的最大重試次數。

                                              客戶端必須指定提供的資源類別的類型，例如基本、常規或高級。指定的資源類別以字符串形式保存在 RDB 中。

                                              這是任務執行允許的最大時間。(此時間從資源分配給任務時開始。)

                                              這可以是等待中、進行中、已完成或已失敗。

                                              這表示在開始任務之前我們可以承受多少延遲。

                                              需要執行的腳本的路徑。腳本是放置在文件系統中的文件。該文件應該可以訪問以便可以執行，就像我們在 Google Colaboratory 中掛載 Google Drive 然後在那裡執行我們的代碼文件一樣。
```

```
注意：如果我們使用地理複製資料存儲，我們可以在不同的資料中心運行任務調度系統的多個實例，以實現更大的規模和更高的資源利用率。
```

- **Batching and prioritization (批處理和優先級排序)**: After we store the tasks in the RDB, the tasks are grouped into batches. Prioritization is based on the attributes of the tasks, such as delay tolerance or the tasks with short execution cap, and so on. The top K priority tasks are pushed into the distributed queue, where K limits the number of elements we can push into the queue. The value of K depends on many factors, such as currently available resources, the client or task priority, and subscription level.
- **批處理和優先級排序**：我們將任務存儲在 RDB 中後，將任務分組為批次。優先級基於任務的屬性，例如延遲容忍度或執行上限短的任務等。前 K 個優先級任務被推入分佈式隊列，其中 K 限制了我們可以推入隊列的元素數量。K 值取決於許多因素，例如當前可用資源、客戶端或任務優先級以及訂閱級別。

- **Distributed queue (分佈式隊列)**: It consists of a queue and a queue manager. The queue manager adds, updates, or deletes tasks in the queue. It keeps track of the types of queues we use. It is also responsible for keeping the task in the queue until it executes successfully. In case a task execution fails, that task is made visible in the queue again. The queue manager knows which queue to run during the peak time and which queue to run during the off-peak time.
- **分佈式隊列**：它由一個隊列和一個隊列管理器組成。隊列管理器在隊列中添加、更新或刪除任務。它跟蹤我們使用的隊列類型。它還負責將任務保留在隊列中，直到執行成功。如果任務執行失敗，該任務將在隊列中再次可見。隊列管理器知道在高峰時間運行哪個隊列以及在非高峰時間運行哪個隊列。

- **Queue manager (隊列管理器)**: The queue manager deletes a task from the queue if it executes successfully. It also makes the task visible if its previous execution failed. It retries for the allowed number of attempts for a task in case of a failed execution.
- **隊列管理器**：如果任務執行成功，隊列管理器將從隊列中刪除該任務。如果任務之前的執行失敗，它也會使該任務可見。如果執行失敗，它會重試允許的任務嘗試次數。

- **Resource manager (資源管理器)**: The resource manager knows which of the resources are free. It pulls the tasks from the distributed queue and assigns them resources. The resource manager keeps track of the execution of each task and sends back their statuses to the queue manager. If a task goes beyond its promised or required resource use, that task will be terminated, and the status is sent back to the task submitter, which will notify the client about the termination of the task through an error message.
- **資源管理器**：資源管理器知道哪些資源是空閒的。它從分佈式隊列中提取任務並為其分配資源。資源管理器跟蹤每個任務的執行情況並將其狀態發回給隊列管理器。如果任務超出了其承諾或所需的資源使用量，該任務將被終止，並且狀態將發回給任務提交器，任務提交器將通過錯誤消息通知客戶端有關任務終止的資訊。

- **Monitoring service (監控服務)**: It is responsible for checking the health of the resource manager and the resources. If some resource fails, it alerts the administrators to repair the resource or add new resources if required. If resources are not being used, it alerts the administrators to remove them or power them off.
- **監控服務**：它負責檢查資源管理器和資源的健康狀況。如果某些資源出現故障，它會提醒管理員在需要時修復資源或添加新資源。如果未使用的資源，它會提醒管理員將其刪除或關閉電源。

```
Question
為什麼我們將任務存儲在資料庫中？為什麼我們不直接將任務推送到隊列？

Answer
隊列不會永久保存資料。我們為使用的隊列服務付費。因此，我們只將那些將很快準備執行的任務推送到隊列中。成功執行的任務需要從隊列中刪除。此外，還有不同的調度類型。任務可以一次、每天、每週、每月或每年調度。因此，我們必須將任務保存在存儲中的某個位置。
```

## Task submitter (任務提交器)
正如我們上面所看到的，我們在分佈式任務調度器設計中使用的每個組件都是分佈式的，因此具有可擴展性和可用性。但是，任務提交器可能是單點故障。因此，為了解決這個問題，我們使用節點集群。每個節點必須接納任務，將任務發送到唯一 ID 生成器以進行 ID 分配，然後將任務與任務 ID 一起存儲在分佈式資料庫中。

有一個集群管理器，每個節點都向其發送心跳，表明該節點工作正常。每個節點都會向集群管理器更新已接納的任務。集群管理器維護任務列表以及接納該任務的節點 ID。如果節點未能執行任務，集群管理器會將該任務移交給集群中的另一個節點。集群管理器本身也是複製的。

上面，我們設計了一個任務調度系統。我們將在下一課中討論任務調度器的設計注意事項。
