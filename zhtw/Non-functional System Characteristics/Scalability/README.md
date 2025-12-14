# Scalability (可擴展性)

## What is scalability? (什麼是可擴展性？)
可擴展性是系統在不影響性能的情況下處理不斷增加的工作負載的能力。例如，搜索引擎必須適應不斷增加的使用者數量以及它索引的資料量。

工作負載可以是不同類型的，包括以下內容：

• **Request workload (請求工作負載)**: This is the number of requests served by the system.
• **請求工作負載**：這是系統服務的請求數量。
• **Data/storage workload (資料/存儲工作負載)**: This is the amount of data stored by the system.
• **資料/存儲工作負載**：這是系統存儲的資料量。

## Dimensions (維度)
以下是可擴展性的不同維度：

• **Size scalability (規模可擴展性)**: A system is scalable in size if we can simply add additional users and resources to it.
• **規模可擴展性**：如果我們可以簡單地向系統添加額外的使用者和資源，則該系統在規模上是可擴展的。
• **Administrative scalability (管理可擴展性)**: This is the capacity for a growing number of organizations or users to share a single distributed system with ease.
• **管理可擴展性**：這是越來越多的組織或使用者輕鬆共享單個分佈式系統的能力。
• **Geographical scalability (地理可擴展性)**: This relates to how easily the program can cater to other regions while maintaining acceptable performance constraints. In other words, the system can readily service a broad geographical region, as well as a smaller one.
• **地理可擴展性**：這涉及程序在保持可接受的性能限制的同時，以多容易的方式滿足其他地區的需求。換句話說，系統可以隨時為廣闊的地理區域以及較小的區域提供服務。

## Different approaches of scalability (可擴展性的不同方法)

### Vertical scalability—scaling up (垂直可擴展性——向上擴展)
垂直擴展，也稱為“向上擴展”，是指通過向現有設備提供額外功能 (例如，額外的 CPU 或 RAM) 進行擴展。垂直擴展允許我們擴展現有的硬件或軟件容量，但我們只能將其擴展到伺服器的限制。垂直擴展的美元成本通常很高，因為我們可能需要奇特的組件來進行擴展。

### Horizontal scalability—scaling out (水平可擴展性——向外擴展)
水平擴展，也稱為“向外擴展”，是指增加網絡中的機器數量。我們為此目的使用商用節點，因為它們具有誘人的美元成本效益。這裡的問題是，我們需要構建一個系統，使許多節點可以像單個巨大的伺服器一樣協同工作。
