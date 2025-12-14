# Design of a Distributed Search (分佈式搜尋的設計)
## High-level design (高級設計)
在進行詳細討論之前，讓我們先勾勒出分佈式搜尋系統的整體設計。如下圖所示，該系統有兩個階段。離線階段涉及使用者無需做任何事情的資料爬取和索引。在線階段包括使用者根據搜尋查詢搜尋結果。

[分佈式搜尋系統的高級設計](./design.jpg)

- **The crawler (爬蟲)**: The crawler collects content from the intended resource. For example, if we build a search for a YouTube application, the crawler will crawl through all of the videos on YouTube and extract textual content for each video. The content could be the title of the video, its description, the channel name, or maybe even the video’s annotation to enable an intelligent search based not only on the title and description but also on the content of that video. The crawler formats the extracted content for each video in a JSON document and stores these JSON documents in a distributed storage.
- **爬蟲**：爬蟲從目標資源中收集內容。例如，如果我們為 YouTube 應用程式構建搜尋，爬蟲將爬取 YouTube 上的所有視頻並提取每個視頻的文本內容。內容可以是視頻的標題、描述、頻道名稱，甚至可以是視頻的註釋，以便不僅基於標題和描述可以進行智能搜尋，還可以基於該視頻的內容進行智能搜尋。爬蟲將每個視頻的提取內容格式化為 JSON 文檔，並將這些 JSON 文檔存儲在分佈式存儲中。

- **The indexer (索引器)**: The indexer fetches the documents from a distributed storage and indexes these documents using MapReduce, which runs on a distributed cluster of commodity machines. The indexer uses a distributed data processing system like MapReduce for parallel and distributed index construction. The constructed index table is stored in the distributed storage.
- **索引器**：索引器從分佈式存儲中獲取文檔，並使用 MapReduce (在商用機器的分佈式集群上運行) 對這些文檔進行索引。索引器使用 MapReduce 等分佈式資料處理系統進行並行和分佈式索引構建。構建的索引表存儲在分佈式存儲中。

- **The distributed storage (分佈式存儲)**: The distributed storage is used to store the documents and the index.
- **分佈式存儲**：分佈式存儲用於存儲文檔和索引。

- **The user (使用者)**: The user enters the search string that contains multiple words in the search bar.
- **使用者**：使用者在搜尋欄中輸入包含多個單詞的搜尋字符串。

- **The searcher (搜尋器)**: The searcher parses the search string, searches for the mappings from the index that are stored in the distributed storage, and returns the most matched results to the user. The searcher intelligently maps the incorrectly spelled words in the search string to the closest vocabulary words. It also looks for the documents that include all the words and ranks them.
- **搜尋器**：搜尋器解析搜尋字符串，從存儲在分佈式存儲中的索引中搜尋映射，並將最匹配的結果返回給使用者。搜尋器智能地將搜尋字符串中拼寫錯誤的單詞映射到最接近的詞彙單詞。它還會查找包含所有單詞的文檔並對其進行排名。

```
MapReduce：正如維基百科所述，“MapReduce 是一個編程模型和相關實現，用於在商用機器集群上使用並行、分佈式算法處理和生成大資料集。”
```

```
索引表由術語及其映射組成。
```

## API design (API 設計)
由於使用者僅以字符串形式發送請求，因此 API 設計非常簡單。

**Search (搜尋)**: The search function runs when a user queries the system to find some content.
**搜尋**：當使用者查詢系統以查找某些內容時，將運行搜尋功能。

```
search(query)
```

```
參數          描述

              这是使用者在搜尋欄中輸入的文本查詢，根據該查詢查找結果。
```


## Detailed discussion (詳細討論)
由於索引器是搜尋系統中的核心組件，我們在上一課中討論了索引技術以及與集中式索引相關的問題。在本課中，我們考慮索引和搜尋的分佈式解決方案。

### Distributed indexing and searching (分佈式索引和搜尋)
讓我們看看如何開分佈式索引和搜尋系統。我們知道索引系統的輸入是我們在爬取過程中創建的文檔。為了以分佈式方式開發索引，我們採用大量低成本機器 (節點) 並根據它們擁有的資源對文檔進行分區或劃分。所有節點都是連接的。一組節點稱為一個集群。

```
為了執行分佈式索引，集群中的機器通常具有運行 Linux 的雙處理器 x86 處理器，每台機器具有 2–4 GB 的內存。並不要求所有機器的規格都相同，儘管它們應該有些可比性。MapReduce 框架足夠智能，可以為更強大的機器分配更多工作。
```

我們使用大量小節點進行索引以實現成本效益。此過程要求我們在這些節點之間劃分或拆分輸入資料 (文檔)。但是，需要解決一個關鍵問題：我們如何執行此分區？

在分佈式索引中用於資料分區的兩種最常用技術如下：

- **Document partitioning (文檔分區)**: In document partitioning, all the documents collected by the web crawler are partitioned into subsets of documents. Each node then performs indexing on a subset of documents that are assigned to it.
- **文檔分區**：在文檔分區中，網絡爬蟲收集的所有文檔都被劃分為文檔子集。然後，每個節點對分配給它的文檔子集執行索引。

- **Term partitioning (術語分區)**: The dictionary of all terms is partitioned into subsets, with each subset residing at a single node. For example, a subset of documents is processed and indexed by a node containing the term “search.”
- **術語分區**：所有術語的字典被劃分為子集，每個子集駐留在單個節點上。例如，包含術語“search”的節點處理和索引文檔子集。

[分佈式搜尋中的資料分區類型](./types.jpg)

在術語分區中，搜尋查詢被發送到與查詢術語相對應的節點。這提供了更多的並發性，因為具有不同查詢術語的搜尋查詢流將由不同的節點提供服務。然而，術語分區在實踐中被證明是一項艱巨的任務。多詞查詢需要在節點組之間發送長映射列表進行合併，這可能比增加並發性帶來的好處更昂貴。

```
映射列表：術語出現的文檔列表以及術語在每個文檔中出現的頻率。
```

在文檔分區中，每個查詢都分佈在所有節點上，並且這些節點的結果在顯示給使用者之前進行合併。這種分區方法需要較少的節點間通訊。在我們的設計中，我們使用文檔分區。

接下來是文檔分區，讓我們研究一下索引構建和查詢的分佈式設計，如下圖所示。我們使用一個由許多低成本節點和一個集群管理器組成的集群。集群管理器使用 MapReduce 編程模型在每個分區上並行化索引的計算。MapReduce 可以處理單個大型伺服器難以處理的大得多的資料集。

[在商用機器集群的多個節點上並行進行分佈式索引和搜尋](./search_and_index.jpg)

上述系統的工作原理如下：

**Indexing (索引)**

- 我們已經有了爬蟲收集的文檔集。
- 集群管理器將輸入文檔集拆分為 N 個分區，在上圖中 N 等於 3。每個分區的大小由集群管理器根據資料大小、計算量、內存限制和集群中的節點數量決定。由於各種原因，所有節點可能不可用。集群管理器通過週期性的心跳監控每個節點的健康狀況。為了將文檔分配給 N 個分區之一，可以使用哈希函數。
- 進行分區後，集群管理器在集群中的 N 個節點上同時為所有 N 個分區運行索引算法。每個索引過程都會生成一個微小的倒排索引，該索引存儲在節點的本地存儲中。通過這種方式，我們生成 N 個微小的倒排索引，而不是一個大的倒排索引。

**Searching (搜尋)**

- 在搜尋階段，當使用者查詢進來時，我們在存儲在節點本地存儲上的每個微小倒排索引上運行並行搜尋，生成 N 個查詢。
- 每個倒排微小索引的搜尋結果是針對查詢術語的映射列表 (我們假設是單個單詞/術語的使用者查詢)。合併器聚合這些映射列表。
- 聚合映射列表後，合併器根據每個文檔中術語的頻率對聚合映射列表中的文檔列表進行排序。
- 排序後的文檔列表作搜尋結果返回給使用者。文檔以排序 (升序) 順序顯示給使用者。

```
注意：我們設計了一個搜尋系統，利用分佈式系統並並行化索引和搜尋過程。這幫助我們通過在較小的文檔分區上工作來處理大型資料集。應該注意的是，搜尋和索引都在同一個節點上執行。我們將這個想法稱為共置 (colocation)。
```

提議的設計是有效的，我們可以在各個資料中心在全球範圍內複製它以方便所有使用者。因此，我們可以獲得以下優勢：

- 我們的設計不會受到單點故障 (SPOF) 的影響。
- 所有使用者的延遲將保持很小。
- 單個資料中心的維護和升級將成為可能。
- 我們系統的可擴展性 (每秒服務更多使用者) 將得到改善。

### Replication (複製)
我們為生成分配分區倒排索引的索引節點製作副本。我們可以通過帶有副本的多組節點回答查詢。總體概念很簡單。我們繼續使用與以前相同的架構，但不是只有一組節點，而是有 R 組節點來回答使用者查詢。R 是副本數。副本數可以根據請求數擴展或收縮，每組節點都具有回答每個查詢所需的所有分區。

每組節點託管在不同的可用區上，以便在資料中心發生故障時獲得更好的系統性能和可用性。

```
注意：負載均衡器組件對於將查詢分散到不同節點組並在出現任何錯誤時重試是必要的。
```

```
可用區 (AZs) 是資料中心區域內的隔離位置，公共雲服務在此啟動和運行。區域是公共雲服務提供商的資料中心所在的地理位置。
```

### Replication factor and replica distribution (複製因子和副本分佈)
通常，複製因子為 3 就足夠了。複製因子為 3 意味著三個節點託管相同的分區並生成索引。三個節點之一成為主節點，而其他兩個是副本。這些節點中的每一個都按相同的順序生成索引以收斂到相同的狀態。

為了說明，讓我們將資料 (文檔集) 分為四個分區。由於複製因子為 3，因此一個分區將由三個節點託管。我們假設有兩個可用區 (AZ1 和 AZ2)。在每個可用區中，我們有兩個節點。每個節點僅作為一個分區的主節點 (例如，AZ1 中的節點 1 是分區 P1 的主節點)。分區的三個副本 (粉色、藍色和紫色) 在兩個 AZ 實例之間共享，以便兩個副本位於一個區域中，第三個副本位於另一個區域中。三種顏色代表每個分區的三個副本。例如，以下內容適用於分區 P4：

- 由粉色表示的第一個副本放置在 AZ2 的節點 2 中
- 由藍色表示的第二個副本放置在 AZ2 的節點 1 中
- 由紫色表示的第三個副本放置在 AZ1 的節點 2 中

下圖中的每一組都由所有四個分區 (P1、P2、P3、P4) 的一個副本組成

[副本分佈：每個節點包含一個主分區和兩個副本](./replica_distribution.jpg)

在上圖中，P_1 的主副本由深紫色表示，P_2 的主副本由深藍色表示，P_3 和 P_4 的主副本由深粉色表示。

既然我們已經完成了複製，讓我們看看如何在這些副本中執行索引和搜尋。

**Indexing with replicas (使用副本進行索引)**

從上圖中，我們假設每個分區都被轉發到每個副本進行索引計算。讓我們看一個例子，我們想要索引分區 P_1。這意味著相同的分區將轉發到兩個可用區中的所有三個副本。因此，每個節點將同時計算索引並達到相同的狀態。

這種策略的優點是，如果主節點發生故障，索引操作不會受到影響。

**Searching with replicas (使用副本進行搜尋)**

我們有每個分區索引的三個副本。負載均衡器選擇每個分區的三個副本之一來執行查詢。增加副本數量提高了系統的可擴展性和可用性。現在，系統可以在相同的時間內處理三倍的查詢。

## Summary (總結)

在本課中，我們學習了如何使用以下策略處理大量資料和大量查詢：

- 並行索引和搜尋，這兩個過程都共置在同一個節點上。
- 複製每個分區，這意味著我們也複製索引和搜尋過程。

我們成功設計了一個系統，該系統可以隨著共置在同一節點上的讀取 (搜尋) 和寫入 (索引) 操作進行擴展。但是，這種擴展方法帶來了一些缺點。我們將在下一課中研究這些缺點及其解決方案。
