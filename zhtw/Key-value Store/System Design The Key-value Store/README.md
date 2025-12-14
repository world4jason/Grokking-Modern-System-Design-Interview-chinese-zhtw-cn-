# System Design: The Key-value Store (系統設計：鍵值存儲)
## Introduction to key-value stores (鍵值存儲簡介)
鍵值存儲是分佈式哈希表 (DHT)。鍵由哈希函數生成，並且應該是唯一的。在鍵值存儲中，鍵綁定到特定值，並且不對值的結構做任何假設。值可以是 blob、圖像、伺服器名稱或使用者想要針對唯一鍵存儲的任何內容。

通常，最好將值的大小保持相對較小 (KB 到 MB)。我們可以將大資料放入 blob 存儲中，並將指向該資料的鏈接放入值字段中。鍵值存儲在許多情況下都很有用，例如在 Web 應用程式中存儲使用者會話和構建 NoSQL 資料庫。

在分佈式環境中擴展具有強一致性和高可用性的傳統資料庫具有挑戰性。許多現實世界的服務 (如 Amazon、Facebook、Instagram、Netflix 等) 使用主鍵訪問資料存儲，而不是傳統的在線事務處理 (OLTP) 資料庫。鍵值存儲的使用示例包括暢銷書列表、購物車、客戶偏好、會話管理、銷售排名和產品目錄。

[鍵值存儲](./kv.jpg)

```
注意：許多應用程式可能不需要傳統關係資料庫管理系統 (RDBMS) 提供的豐富編程模型。將 RDBMS 用於此類應用程式在成本和性能方面通常很昂貴。
```

## How will we design a key-value store? (我們將如何設計鍵值存儲？)
我們將鍵值系統設計分為以下四課：

- **Design a Key-value Store (設計鍵值存儲)**: We’ll define the requirements of a key-value store and design the API.
- **設計鍵值存儲**：我們將定義鍵值存儲的需求並設計 API。
- **Ensure Scalability and Replication (確保可擴展性和複製)**: We’ll learn to achieve scalability using consistent hashing and replicate the partitioned data.
- **確保可擴展性和複製**：我們將學習使用一致性哈希實現可擴展性並複製分區資料。
- **Versioning Data and Achieving Configurability (資料版本控制和實現可配置性)**: We’ll learn to resolve conflicts that occur due to changes made by more than one entity, and we’ll make our system more configurable for different use cases.
- **資料版本控制和實現可配置性**：我們將學習解決由於多個實體進行更改而發生的衝突，並使我們的系統針對不同的用例更具可配置性。
- **Enable Fault Tolerance and Failure Detection (啟用容錯和故障檢測)**: We’ll learn to make a key-value store fault tolerant and how to detect failures in the system.
- **啟用容錯和故障檢測**：我們將學習使鍵值存儲具有容錯能力，以及如何檢測系統中的故障。
