# Introduction to Distributed Logging (分佈式日誌記錄介紹)
## Logging in a distributed system (分佈式系統中的日誌記錄)
在當今世界，越來越多的設計轉向微服務架構，而不是單體架構。在微服務架構中，每個微服務的日誌積累在各自的機器中。如果我們想知道多個微服務處理的某個事件，就很難進入每個節點，找出流程並查看錯誤消息。但是，如果我們可以從頭到尾跟蹤任何特定流程的日誌，那就會很方便。

此外，微服務也不必只部署在一個節點上。它可以部署在數千個節點上。考慮以下示例，其中數百個微服務是相互依賴的，一個服務的故障可能導致其他服務的故障。如果我們沒有日誌，我們可能無法確定故障的根本原因。這強調了日誌記錄的必要性。

[日誌記錄](./logging)

## Restrain the log size (限制日誌大小)
日誌的數量隨著時間的推移而增加。一次可能需要記錄數百條並發消息。但問題是，它們都重要到需要記錄嗎？為了解決這個問題，日誌必須結構化。我們需要決定在應用程式或日誌記錄級別上記錄到系統中的內容。

### Use sampling (使用採樣)
我們將確定在此方法中我們應該記錄到系統中的哪些消息。考慮一種情況，我們有很多來自同一組事件的消息。例如，有人在帖子下評論，其中 X 對 Y 的帖子發表了評論，然後 Z 對 Y 的帖子發表了評論，依此類推。我們可以不記錄所有資訊，而是使用採樣器服務僅記錄較大塊中的一小部分消息。這樣，我們可以決定要記錄的最重要消息。

```
注意：對於像 Facebook 這樣每秒發生數十億事件的大型系統，記錄所有事件是不可行的。適當的採樣閾值和策略對於有選擇地挑選代表性資料集是必要的。
```

```
Question
問題
採樣方法在哪種情況下不起作用？

Answer
Let’s consider an application that processes a financial ATM transaction. It runs various services like fraud detection, expiration time checking, card validation, and many more. If we start to miss out logging of any service, we cannot identify an end-to-end flow that affects the debugging in case an error occurs. Using sampling, in this case, is not ideal and results in the loss of useful data.
答案
讓我們考慮一個處理金融 ATM 交易的應用程式。它運行各種服務，如欺詐檢測、過期時間檢查、卡驗證等。如果我們開始錯過任何服務的日誌記錄，我們無法識別在發生錯誤時影響調試的端到端流程。在這種情況下使用採樣並不理想，並且會導致有用資料的丟失。
我們還可以對消息類型進行分類並應用過濾器，識別重要消息並僅將其記錄到系統中。
```

### Use categorization (使用分類)
讓我們看看各種編程語言提供的日誌記錄支持。例如，有 log4j 和 Python 中的 logging。以下嚴重級別通常用於日誌記錄：

- DEBUG
- INFO
- WARNING
- ERROR
通常，生產日誌設置為打印嚴重程度為 WARNING 及以上的消息。但是對於更詳細的流程，嚴重級別也可以設置為 DEBUG 和 INFO 級別。

## Structure the logs (構建日誌)
應用程式可以自由選擇其日誌資料的結構。例如，應用程式可以自由地將日誌寫為二進制或文本資料，但在日誌上強制執行某種結構通常很有幫助。結構化日誌的第一個好處是日誌編寫者和讀者之間更好的互操作性。其次，結構可以使日誌處理系統的工作更容易。

```
注意：日誌的結構本身就是一個豐富的主題。我們建議感興趣的學習者參考 Ryan Braud 的博士論文，題為“基於查詢的分佈式系統調試”。
```

## Points to consider while logging (日誌記錄時要考慮的點)
我們在記錄時應該小心。日誌記錄資訊應僅包含相關資訊，而不應違反安全問題。對於安全資料，我們應該記錄加密資料。我們在記錄時應考慮以下幾點：

- 避免記錄個人身份資訊 (PII)，如姓名、地址、電子郵件等。
- 避免記錄敏感資訊，如信用卡號、密碼等。
- 避免過多的資訊。記錄所有資訊是不必要的。它只會佔用更多空間並影響性能。日誌記錄作為一項 I/O 密集型操作，有其性能損失。
- 日誌記錄機制應該是安全的，不易受到攻擊，因為日誌包含應用程式的流程，而不安全的日誌記錄機制容易受到黑客攻擊。

### Vulnerability in logging infrastructure (日誌記錄基礎設施中的漏洞)
Log4j (Java 的著名日誌框架) 最近發現了一個零日漏洞。自 2013 年以來，Log4j 就包含了隱藏漏洞 Log4Shell (CVE-2021-44228)。Apache 給了 Log4Shell 最高的可用分數，即 CVSS 嚴重性評級為 10。該漏洞利用執行簡單，影響數億台設備。安全專家確信，此漏洞可能允許在國際範圍內進行毀滅性的網絡攻擊，因為它可以使攻擊者運行惡意代碼並控制機器。
