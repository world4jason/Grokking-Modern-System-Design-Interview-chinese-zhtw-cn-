# Initial Design of Quora (Quora 的初步設計)
## Initial design (初步設計)
Quora 的初步設計將由以下構建塊和組件組成：

- **Web and application servers (Web 和應用程式伺服器)**：典型的 Quora 頁面由各種服務生成。Web 和應用程式伺服器維護各種流程以生成網頁。 Web 伺服器具有管理器進程，應用程式伺服器具有用於處理各種請求的工作進程。管理器進程使用路由器庫在工作進程之間分配工作。路由器庫由管理器進程入隊任務，並由工作進程出隊。每個應用程式伺服器維護多個記憶體中隊列以處理不同的使用者請求。下圖提供了 Web 和應用程式伺服器的抽象視圖：

[Quora 的 Web 和應用程式伺服器](./servers.jpg)

- **Data stores (資料儲存)**：不同類型的資料需要儲存在不同的資料儲存中。我們可以使用像 MySQL 這樣的關聯式資料庫來儲存關鍵資料，如問題、答案、評論和贊成/反對，因為它提供了更高程度的一致性。像 HBase 這樣的 NoSQL 資料庫可用於儲存頁面瀏覽量、用於對答案進行排名的分數，以及從資料中提取的特徵以便以後用於推薦。因為重新計算特徵是一項昂貴的操作，所以 HBase 是高頻寬儲存和檢索資料的不錯選擇。我們需要高讀/寫吞吐量，因為大數據處理系統使用高並行性來有效地獲取所需的統計資料。此外，blob 儲存需要儲存問題和答案中發布的影片和圖像。

```
Quora 成立於 2009 年，而 HBase 是 Apache 於 2008 年開發的。因為它是開源的並且仿照 Google 的 BigTable，所以它適合儲存大量的小型資料。此外，它具有高讀/寫吞吐量。因此，Quora 在成立之初使用它是很自然的選擇。
```

- **Distributed cache (分散式快取)**：為了提高效能，使用了兩個分散式快取系統：Memcached 和 Redis。Memcached 主要用於儲存經常訪問的關鍵資料，否則這些資料存儲在 MySQL 中。另一方面，Redis 主要用於儲存答案的在線瀏覽計數器，因為它允許在儲存中遞增。因此，根據使用情況採用了兩種快取系統。除了這兩個，CDN 還提供經常訪問的影片和圖像。

- **Compute servers (計算伺服器)**：需要一組計算伺服器來促進基於一組屬性的推薦和排名等功能。這些功能可以在在線或離線模式下計算。計算伺服器使用機器學習 (ML) 技術提供有效的推薦。自然地，這些計算伺服器擁有大量的 RAM 和處理能力。

online mode: Recommendations are computed as soon as they are requested. (在線模式：推薦在請求時立即計算。)
offline mode: Recommendations are computed beforehand and served when requested. (離線模式：推薦預先計算並在請求時提供。)
```

當然，其他基本構建塊 (如負載平衡器、監控服務和速率限制器) 也將成為設計的一部分。高層設計如下：

[Quora 的高層設計](./hld.jpg)


## Workflow (工作流程)
Quora 的設計很複雜，因為我們有大量的功能和非功能需求。因此，我們將根據每個功能解釋工作流程：

- **Posting question, answers, comments (發布問題、答案、評論)**：Web 伺服器通過負載平衡器接收使用者請求並將其定向到應用程式伺服器。同時，Web 伺服器生成部分網頁，並讓應用程式伺服器中的工作進程完成其餘的頁面生成。問題和答案資料儲存在 MySQL 資料庫中，而任何影片和圖像都儲存在 blob 儲存中。類似的方法用於發布評論以及贊成或反對答案。通過對不同任務採用不同的隊列來執行任務優先順序。我們執行優先順序是因為某些任務需要立即關注 (例如，為使用者請求從資料庫中獲取資料)，而其他任務則不那麼緊急 (例如，發送每週電子郵件摘要)。工作進程將通過從這些隊列中獲取任務來執行任務。

```
Question 1
像 Quora 這樣的服務何時需要通知功能？

Answer
Quora 在以下情況下需要通知服務：

使用者發布了一個潛在受訪者訂閱的主題的問題。

使用者發布了另一個使用者提出的問題的答案。

使用者感興趣或撰寫的貼文收到了新評論或贊成/反對票等。
```

```
Question 2
Quora 如何通過上述機制向使用者發送通知？

Hide Answer
由於任務被新增到不同的優先順序隊列中，因此可以維護一個通知隊列。當較高優先順序的任務入隊到高優先順序隊列時，通知任務將被新增到中等或較低優先順序的隊列中。下圖可視化了描述的概念：
```

[通知]

- **Answer ranking system (答案排名系統)**：問題的答案可以根據日期排序。儘管根據日期 (使用時間戳) 開發排名系統很方便，但使用者更喜歡在頂部看到最適當的答案。因此，Quora 使用 ML 對答案進行排名。隨著時間的推移，提取不同的特徵並為每種類型的問題儲存在 HBase 中。這些特徵被轉發到 ML 引擎以將最有用的答案排在頂部。我們不能僅將贊成票數作為排名答案的唯一指標，因為許多答案可能是笑話——這樣的答案也會獲得很多贊成票。最好離線實施排名系統，因為好的答案會隨著時間的推移獲得贊成票和瀏覽量。此外，離線模式對基礎設施造成的負擔較小。離線實施排名系統以及對特殊 ML 硬體的需求使其適合使用一些公共雲彈性服務。

- **Recommendation system (推薦系統)**：推薦系統負責多項功能。例如，我們可能需要開發使用者動態消息、查找相關問題和廣告、向潛在受訪者推薦問題，甚至突出顯示重複內容和違反服務使用條款的內容。與答案排名系統不同，推薦系統必須提供在線和離線服務。該系統從應用程式伺服器接收請求並將選定的特徵轉發到 ML 引擎。

- **Search feature (搜尋功能)**：隨著時間的推移，隨著問題和答案被輸入到 Quora 系統中，可以在 HBase 中建立索引。使用者搜尋查詢與索引匹配，並向使用者建議相關內容。經常訪問的索引可以從快取中提供，以實現低延遲。索引可以從問題、答案、主題標籤和使用者名稱構建。搜尋索引的標記化對於重新排序的單詞也會返回相同的結果 (有關詳細資訊，請參閱分散式搜尋中的擴展搜尋和索引章節)。

Fun facts (有趣的事實)
- Quora is mostly built on Amazon Web Services (AWS). (Quora 主要建立在 Amazon Web Services (AWS) 之上。)
- Initially, Quora used Amazon’s EC2 instances as their application servers used eight cores with an 8 MB cache. (最初，Quora 使用 Amazon 的 EC2 實例作為其應用程式伺服器，使用八核和 8 MB 快取。)
- Quora used the search server called Sphinx. Later, because of its slow performance, Quora custom-built its search engine using Thrift and Python Unicode libraries only. (Quora 使用名為 Sphinx 的搜尋伺服器。後來，由於其效能緩慢，Quora 僅使用 Thrift 和 Python Unicode 庫定製構建了其搜尋引擎。)
- The full-text search feature was launched by Quora in 2013. Before that, it was possible to get results for individual words only. (Quora 於 2013 年推出了全文搜尋功能。在此之前，只能獲得單個單詞的結果。)
```

## API design (API 設計)
我們將在本節中為 Quora 設計 API 調用。我們將僅為以下功能定義 API：

- 發布問題
- 發布答案
- 贊成或反對問題或答案
- 評論答案
- 搜尋

```
注意：我們不考慮推薦系統或排名的 API，因為它們不是由使用者作為顯式請求發出的。相反，Web 伺服器與其他組件協調以確保服務。
```

### Post a question (發布問題)
HTTP 的 POST 方法用於調用 /postQuestion API：
```
postQuestion(user_id, question, description, topic_label, video, image)
```

讓我們了解 API 調用的每個參數：
Parameter (參數)     Description (描述)

user_id             這是發布問題的使用者的唯一標識。

question            這是使用者提出的問題的文字。

description         這是問題的描述。這是一個可選欄位。

topic_label         這表示與使用者問題相關的領域類別。

video               這是嵌入在使用者問題中的影片檔案。

image               這是使用者問題一部分作為的圖像。
```

如果問題中沒有嵌入圖像或影片，則 video 和 image 參數可以為 NULL。否則，它將作為問題的一部分上傳。


### Post an answer (發布答案)
對於發布答案，POST 方法是 /postAnswer API 的合適選擇：
```
postAnswer(user_id, question_id, answer_text, video, image)
```

Parameter (參數)       Description (描述)

question_id           這指的是發布答案所針對的問題。

answer_text           這是響應者發布的文字答案。
其餘參數不言自明。

### Upvote an answer (贊成回答)
/upvote API 如下：
```
upvote(user_id, question_id, answer_id)
```
Parameter (參數)     Description (描述)

user_id             這表示贊成答案的使用者。

answer_id           這表示針對特定問題 (由 question_id 標識) 贊成的答案的標識。
```

```
注意：downvote API 與 upvote API 相同，因為兩者都是類似的功能。
```

### Comment on an answer (評論回答)
/comment API 具有以下結構：
```
comment(user_id, answer_id, comment_text)
```
Parameter (參數)         Description (描述)

user_id                 它表示評論答案的使用者。

comment_text            它表示使用者針對由 answer_id 標識的答案發布的文字。
### Search (搜尋)
/search API 具有以下詳細資訊：
```
search(user_id, search_text)
```

Parameter (參數)       Description (描述)

user_id               這是執行搜尋查詢的 user_id。在這種情況下它是可選的，因為未註冊的使用者也可以搜尋問題。

search_text           這是使用者輸入的搜尋查詢。
```

```
我們使用定序器來生成 API 調用中提到的不同 ID。
```

```
Question
為什麼在 Web 和應用程式伺服器之間有一個自定義路由層而不是負載平衡層？

Hide Answer
主要原因是效能。通用負載平衡層的應用程式可理解性很低，延遲也較高。

如上所述，管理器進程生成網頁的骨架，並將大部分工作留給工作進程。在這個意義上，一個管理器進程可以生成多個工作任務。自定義路由器庫是管理器和工作者之間的隊列，任何工作進程都可以使用管理器進程生成的任務。

這種設計提高了整體效能並支援可擴展性。

路由器庫：Quora 稱之為“Fastrouter”——它是一個自定義的 C++ 構建的隊列伺服器。
```
