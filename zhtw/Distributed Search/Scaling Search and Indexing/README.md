# Scaling Search and Indexing (擴展搜尋和索引)
雖然上一課提出的設計看起來很合理，但仍然存在一些嚴重的缺點。我們將在下面討論這些缺點：

1. **Colocated indexing and searching (共置索引和搜尋)**: We’ve created a system that colocates indexing and searching on the same node. Although it seems like efficient usage of resources, it has its downsides as well. Searching and indexing are both resource-intensive operations. Both operations impact the performance of each other. Also, this colocated design doesn’t scale efficiently with varying indexing and search operations over time. Colocating both these operations on the same machine can lead to an imbalance, and it results in scalability issues.
1. **共置索引和搜尋**：我們創建了一個將索引和搜尋共置在同一節點上的系統。雖然這看起來像是資源的有效利用，但它也有缺點。搜尋和索引都是資源密集型操作。這兩種操作都會影響彼此的性能。此外，這種共置設計無法隨著時間推移而有效地擴展以適應變化的索引和搜尋操作。將這兩種操作共置在同一台機器上會導致不平衡，並導致可擴展性問題。

2. **Index recomputation (索引重新計算)**: We assume that each replica will compute the index individually, which leads to inefficient usage of resources. Furthermore, index computation is a resource-intensive task with possibly hundreds of stages of pipelined operations. Thus, recomputing the same index over different replicas requires powerful machines. Instead, the logical approach is to compute the index once and replicate it across availability zones.
2. **索引重新計算**：我們假設每個副本將單獨計算索引，這會導致資源的低效利用。此外，索引計算是一項資源密集型任務，可能涉及數百個階段的流水線操作。因此，在不同的副本上重新計算相同的索引需要強大的機器。相反，邏輯方法是計算一次索引，並在可用區之間複製它。

由於這些關鍵原因，我們將尋求分佈式索引和搜尋的替代方法。

## Solution (解決方案)
我們只在主節點上計算倒排索引，而不是在每個副本上重新計算索引。接下來，我們將倒排索引 (二進制 blob/文件) 傳達給副本。這種方法的主要好處是它避免了在副本上使用重複數量的 CPU 和內存進行索引。

```
Question
上述提議的解決方案有什麼缺點？

Answer
由於倒排索引將被傳輸到副本，因此在複製倒排索引文件時會引入傳輸延遲，因為索引文件的大小可能非常大。

當主節點接收到新的索引操作時，倒排索引文件會發生變化。當一定數量的索引操作達到定義的閾值後，每個副本都需要獲取文件的最新版本。
```

## Separate the indexing and search (分離索引和搜尋)
隨著網絡和虛擬化技術的出現，雲計算已成為一項成功的技術。在這項技術中，我們可以訪問大量的帶寬 (高達 100 Gbps) 和可擴展的分佈式存儲。這些進步允許在索引和搜尋之間進行強大的分離，而不會產生索引延遲的負面後果。由於這種隔離，索引不會影響搜尋的可擴展性，反之亦然。此外，我們可以只複製索引文件，而不是在副本節點上重新計算索引 (這會浪費資源)。

我們將使用這些技術重新設計我們的分佈式索引和搜尋系統。該搜尋系統設計涉及三個組件：

1. **Indexer (索引器)**: It consists of a group of nodes to compute the index.
1. **索引器**：它由一組用於計算索引的節點組成。
2. **Distributed storage (分佈式存儲)**: This system is used to store partitions and the computed index.
2. **分佈式存儲**：該系統用於存儲分區和計算出的索引。
3. **Searcher (搜尋器)**: It consists of a number of nodes to perform searching.
3. **搜尋器**：它由許多用於執行搜尋的節點組成。

下圖描繪了索引器和搜尋器節點之間倒排索引的生成和傳輸：

[索引節點生成的索引存儲在分佈式存儲中，參與搜尋的節點從分佈式存儲中讀取索引以生成使用者查詢的結果](./query.jpg)

在上圖中，每個索引和搜尋操作都顯示了一個節點。但是，實際上，在索引階段會有 N 個節點，每個分區 (文檔集) 一個節點，用於生成倒排索引。倒排索引以二進制文件的形式存儲在節點的本地存儲中。快取這些 blob 文件將提高性能。這些二進制文件也被推送到分佈式存儲。如果發生硬體故障，則會添加新的搜尋器或索引器機器，並從分佈式存儲中檢索資料的副本。

上傳完成後，搜尋器節點下載索引文件。根據使用者搜尋模式，搜尋節點將維護常見查詢的快取並從 RAM 提供資料。使用者搜尋查詢將擴展到所有搜尋器節點，這些節點將根據各自的索引生成響應。前端伺服器中的合併節點將合併所有搜尋結果並將其呈現給使用者。

索引過程一有新文檔就對其進行索引。同時，搜尋器節點獲取更新的索引以提供改進的搜尋結果。

## Indexing explained (索引解釋)
到目前為止，我們已經解釋了使用低成本節點開發高度可擴展和高性能的設計。但是，我們不知道索引節點的內部結構。在本節中，我們將學習如何使用 MapReduce 分佈式模型和並行處理框架執行索引。

MapReduce 框架是在集群管理器和一組分類為 Mapper 和 Reducer 的工作節點的幫助下實現的。正如其名稱所示，MapReduce 由兩個階段組成：

1. The Map phase (Map 階段)
2. The Reduction phase (Reduction 階段)

此外，MapReduce 的輸入是許多分區或文檔集，而其輸出是聚合的倒排索引。

讓我們了解上述組件的用途：

- **Cluster manager (集群管理器)**: The manager initiates the process by assigning a set of partitions to Mappers. Once the Mappers are done, the cluster manager assigns the output of Mappers to Reducers.
- **集群管理器**：管理器通過將一組分區分配給 Mapper 來啟動該過程。Mapper 完成後，集群管理器將 Mapper 的輸出分配給 Reducer。
- **Mappers**：此組件從集群管理器分配給它的分區中提取和過濾術語。這些機器並行輸出倒排索引，作為 Reducer 的輸入。
- **Reducers**：Reducer 組合各種術語的映射以生成匯總索引。

集群管理器確保集群中的所有工作節點得到有效利用。MapReduce 旨在在部分故障下工作。如果一個節點發生故障，它會在另一個節點上重新安排工作。

請注意，只要 Mappers 正在工作，Reducers 就無法啟動。這意味著集群管理器可以使用同一個節點作為 Mapper 和 Reducer。

下面的幻燈片描繪了如何使用 MapReduce 生成倒排索引的簡化設置：

[索引](./indexing)

為了簡單起見，我們在上圖中僅顯示了每個術語的兩個指標：術語出現的文檔列表以及每個文檔中術語出現的頻率列表 (有關詳細資訊，請參閱索引)。

```
注意：上面的 MapReduce 設置是實際情況的簡化版本。需要 MapReduce 框架的複雜管道來管理現實世界搜尋引擎的複雜性。但是，基本原理與我們在此處介紹的相同。
```

## Summary (總結)
在本課中，我們通過使用專用節點進行索引和搜尋，解決了可擴展性 (由於共置索引和搜尋) 和資源浪費 (由於索引重新計算) 這兩個關鍵問題。這兩種操作都依賴於分佈式存儲。此外，我們對 MapReduce 框架進行了簡化描述，以並行化索引過程。
