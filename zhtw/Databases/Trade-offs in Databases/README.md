# Trade-offs in Databases (資料庫中的權衡)
## Which is the best database sharding approach? (哪種是最好的資料庫分片方法？)
垂直和水平分片都涉及向我們的計算基礎設施新增資源。我們的業務利益相關者必須決定哪種適合我們的組織。我們必須相應地擴展我們的資源，以便我們的組織和業務增長，防止停機並減少延遲。我們可以通過結合 CPU、物理記憶體需求、硬碟調整和網路頻寬的調整來擴展這些資源。

以下各節解釋了不分片與分片的優缺點。

### Advantages and disadvantages of a centralized database (集中式資料庫的優缺點)
#### Advantages (優點)
- 資料維護 (例如更新和備份集中式資料庫) 很簡單。

- 與分散式資料庫相比，集中式資料庫提供更強的一致性和 ACID 事務。

- 與分散式資料庫相比，集中式資料庫為終端程式設計師提供了更簡單的程式設計模型。

- 對於企業來說，將少量資料儲存在單個節點上更有效率。

#### Disadvantages (缺點)
- 當存取集中式資料庫的每秒查詢數接近單節點極限時，集中式資料庫可能會變慢，從而導致終端使用者的高延遲。

- 集中式資料庫具有單點故障。因此，它無法存取的概率要高得多。

### Advantages and disadvantages of a distributed database (分散式資料庫的優缺點)
#### Advantages (優點)
- 在分散式資料庫中存取資料既快速又容易，因為資料是從最近的資料庫分片或經常使用的分片中檢索的。

- 具有不同分佈透明度級別的資料可以儲存在不同的地方。
- 由查詢組成的密集事務可以劃分為多個優化的子查詢，這些子查詢可以並行處理。

#### Disadvantages (缺點)
- 有時，需要來自多個站點的資料，這比預期的要花費更多時間。

- 關係在不同節點之間垂直或水平分區。因此，諸如連接 (join) 之類的操作需要通過仔細獲取資料來重建完整的關係。這些操作可能會變得更加昂貴和複雜。

- 很難在分散式資料庫的各個站點之間保持資料的一致性，這需要額外的措施。

- 分散式資料庫中的更新和備份需要時間來同步資料。

### Query optimization and processing speed in a distributed database (分散式資料庫中的查詢優化和處理速度)
分散式資料庫中的事務取決於查詢類型、涉及的站點 (分片) 數量、通訊速度和其他因素，例如底層硬體和使用的資料庫類型。但是，作為一個例子，讓我們假設一個查詢存取駐留在不同站點的三個資料表：Store (商店)、Product (產品) 和 Sales (銷售)。

每個資料表中的屬性數量如下圖所示：

[由三個資料表組成的資料庫模式：Store, Product, Sales](./3tables.jpg)

假設兩個資料表在不同站點上的分佈如下：

- Store 表有 10,000 個元組儲存在站點 A。
- Product 表有 100,000 個元組儲存在站點 B。
- Sales 表有 1,000,000 個元組儲存在站點 A。

現在，假設我們需要處理以下查詢：

```sql
Select Store_key from (Store JOIN Sales JOIN Product)
where Region= 'East' AND Brand='Wolf';
```

上述查詢對 Store、Sales 和 Product 表執行連接操作，並從連接操作結果生成的表中檢索 Store_key 值。

接下來，假設每個儲存的元組長 200 位元。那等於 25 位元組。此外，某些中間結果的估計基數如下：

- Wolf 品牌的數量是 10。
- East 地區商店的數量是 100,000。

通訊假設如下：

Data rate (資料速率) = 50M bits per second (每秒 50M 位元)
Access delay (存取延遲) = 0.1 second (0.1 秒)

#### Parameters assumption (參數假設)
在使用不同方法處理查詢之前，讓我們定義一些參數：

a=Total access delay (總存取延遲)
b= Data rate (資料速率)
v= Total data volume (總資料量)

現在，讓我們根據以下公式計算總通訊時間 T：
T=a+v/b

讓我們嘗試以下可能的方法來執行查詢。

#### Possible approaches (可能的方法)

- 將 Product 表移動到站點 A 並在 A 處處理查詢。
T=100,000×200 / 50,000,000 = 0.5 秒 (seconds)
這裡，0.1 是站點 A 上資料表的存取延遲，100,000 是 Product 表中的元組數。每個元組的大小 (以位元為單位) 為 200，資料速率為 50,000,000。200 和 50,000,000 這兩個數字在以下所有計算中都是相同的。

- 將 Store 和 Sales 移動到站點 B 並在 B 處處理查詢：
T=0.2+ ((10,000+1,000,000)×200/50,000,000) = 4.24 秒 (seconds)
這裡，0.2 是 Store 和 Product 表的存取延遲。數字 10,000 和 1,000,000 分別是 Store 和 Product 表中的元組數。

- 將站點 B 的 Brand 限制為 Wolf (稱為投影 (projection)) 並將結果移動到站點 A：
T=0.1 + (10×200)/50,000,000 = 大約 0.1 秒 (seconds)
這裡，0.1 是 Product 表的存取延遲。Wolf 品牌的數量是 10，因此是元組的數量。

當我們比較這三種方法時，第三種方法為我們提供了最小的延遲 (0.1 秒)。這個例子表明，仔細的查詢優化在分散式資料庫中也很關鍵。

### Conclusion (結論)
考慮到查詢已優化，跨多個節點的資料分發 (垂直和水平分片) 旨在改進以下功能：

- Reliability (fault-tolerance) (可靠性 (容錯性))
- Performance (效能)
- Balanced storage capacity and dollar costs (平衡的儲存容量和美元成本)

集中式和分散式資料庫都有其優缺點。我們應該根據應用程式的需求進行選擇。
