# Unique IDs with Causality (具有因果關係的唯一 ID)

## Causality (因果關係)
在上一課中，我們生成了唯一 ID 來區分各種事件。除了具有事件的唯一標識符外，我們還有興趣查找這些事件的順序。讓我們考慮一個例子，其中 Peter 和 John 是兩個 Twitter 使用者。John 發布了一條評論 (事件 A)，Peter 回覆了 John 的評論 (事件 B)。事件 B 依賴於事件 A，並且不能在它之前發生。這裡的事件不是並發的。

我們也可以有並發事件——即兩個彼此獨立發生的事件。例如，如果 Peter 和 John 評論兩條不同的推文，則它們之間沒有先後關係或因果關係。必須確定一個事件對另一個事件的依賴關係，但在並發事件的情況下則不然。

```
注意：上述場景也可以通過分配唯一 ID 並使用社交圖對事件的依賴關係進行編碼來處理。我們還可以使用單獨的時間資料結構和簡單的唯一 ID。但是，我們希望唯一 ID 具有雙重功能——提供唯一標識並有助於事件的因果關係。
[可視化](./nc)

某些應用程式需要事件具有唯一標識符並攜帶任何相關的因果關係資訊。這方面的一個例子是為鍵值存儲中鍵的並發寫入提供一個標識符，以實現最後寫入勝出策略。

我們可以使用邏輯時鐘或物理時鐘來推斷因果關係。某些系統有額外的要求，我們希望事件標識符的因果關係映射掛鐘時間。這方面的一個例子是符合歐洲 MiFID 法規的金融應用程式。MiFID 要求時鐘在 UTC 的 100 微秒內，以檢測大容量/高速市場交易期間的異常。

```
注意：邏輯時鐘或物理時鐘有許多微妙之處。我們可以參考下面標題為“分佈式系統中的時間”的文本來刷新我們的時間概念。
```
我們利用時間來決定生活中事件的順序。例如，如果 Sam 在早上 6 點洗澡，在早上 7:00 吃早餐，我們可以通過每個事件的時間戳確定 Sam 在早餐前洗澡。因此，可以使用時間戳來維持因果關係。

                                    Optional Revision: Time in a Distributed System (可選複習：分佈式系統中的時間)
Physical clocks (物理時鐘)
計算機中通常有兩種類型的物理時鐘：掛鐘 (time-of-day clock) 和單調計數器。

The time-of-day clock (掛鐘)
- 與單調計數器相比，這通常具有較低的分辨率。
- 網絡時間協議 (NTP) 可以向前或向後移動時鐘，因此它並不總是單調的。
- 它可能包含也可能不包含閏秒。

Monotonic counters (單調計數器)
- 單調計數器通常比掛鐘具有更高的分辨率。
- 單調計數器應用於兩個事件之間的持續時間，而不是時間。
- 這些在不同節點之間沒有意義。例如，即使在具有多個處理器的同一台伺服器上，每個處理器也可能有不同的計數器。在使用來自不同處理器的計數器時，應用程式需要小心。
- NTP 可能會在不違反單調性的情況下對其進行調整。
- NTP 最多只能將計數器的變化率加速或減慢 0.05%。

Reasons for clock drift (時鐘漂移的原因)
物理時鐘由於多種原因會隨著時間的推移而漂移：
- Temperature differences (溫差)
- The equipment’s age (設備老化)
- Manufacturing defects (製造缺陷)
- Virtualized clocks (虛擬化時鐘)
例如，如果每 30 秒同步一次，時鐘漂移為百萬分之 200 的伺服器意味著 6 毫秒的漂移，或者如果每 24 小時重新同步一次，則意味著 17 秒的漂移。

一項研究表明，在公共互聯網上，NTP 無法獲得優於 35 毫秒的時鐘精度，當網絡擁塞時，它可能會飆升至一秒。否則，NTP 使用多個時間伺服器並丟棄異常值。

The trade-off: complexity and cost versus clock accuracy (權衡：複雜性和成本與時鐘精度)
使用 GPS 和原子鐘、仔細的部署和監控，可以始終保持較小的時鐘漂移。但是，這樣的系統會增加成本，並增加系統的複雜性。

Logical clocks (邏輯時鐘)
- Lamport 時鐘為我們提供了先後關係。如果事件 A 發生在事件 B 之前，則 A 的時鐘值將小於 B 的時鐘值。有一個微妙的一點是，給定來自任何兩個伺服器的兩個事件的任何兩個時鐘值，我們無法比較它們來推斷先後關係，因為這兩個事件可能是並發的 (意味著沒有因果關係)。
- 我們可以使用向量時鐘使用時鐘值來推斷先後關係。為此，我們需要為向量中的每個參與實體提供一個計數器。
- 我們應該注意，先發生可能並不意味著兩個事件具有因果關係。可能是一個事件發生在另一個事件之前。通常，我們需要在先發生機制之上的應用程式級上下文來推斷真正的因果關係。
```


## Use UNIX time stamps (使用 UNIX 時間戳)
UNIX 時間戳精確到毫秒，可用於區分不同的事件。我們有一個 ID 生成伺服器，可以在一毫秒內生成一個 ID。任何生成唯一 ID 的請求都會路由到該伺服器，該伺服器返回時間戳，然後返回唯一 ID。在毫秒內生成 ID 的能力使我們能夠每秒生成一千個標識符。這意味著我們一天可以獲得 24(小時)* 60(分/小時) * 60(秒/分) * 1000(ID/秒) = 86400000 個 ID。
一天內 86400000 個 ID。這還不到每天十億。

```
注意：連接到以下終端以查看以毫秒為單位的 UNIX 時間戳。
```
我們的系統在生成 ID 方面運作良好，但它帶來了一個關鍵問題。ID 生成伺服器是單點故障 (SPOF)，我們需要處理它。為了應對 SPOF，我們可以添加更多伺服器。每個伺服器每一毫秒生成一個唯一 ID。為了使整個系統中的標識符唯一，我們將伺服器 ID 附加到 UNIX 時間戳。然後，我們添加一個負載平衡器以更有效地分配流量。使用 UNIX 時間戳生成唯一 ID 的設計如下：

[使用時間戳作為 ID](./timestamps.jpg)

### Pros (優點)
這種方法簡單、可擴展且易於實現。它還允許多個伺服器處理並發請求。

### Cons (缺點)
對於兩個並發事件，返回相同的時間戳，並且可以為它們分配相同的 ID。這樣，ID 就不再是唯一的了。

                        Requirements Fulfilled by Each Approach (每種方法滿足的需求)
                           Unique (唯一)          Scalable (可擴展)        Available (可用)           64-bit numeric ID (64 位數字 ID)            Causality maintained (維持因果關係)       
Using UUID                  ✖️                      ✔️                     ✔️                           ✖️                                          ✖️

Using a database            ✖️                      ✖️                     ✔️                           ✔️                                          ✖️

Using a range handler       ✔️                      ✔️                     ✔️                           ✔️                                          ✖️

Using UNIX time stamps      ✖️                     weak (弱)               ✔️                           ✔️                                         weak (弱)
```
## Twitter Snowflake
讓我們嘗試有效地利用時間。我們可以使用目標 64 位中的一些位來存儲時間，其餘的用於存儲其他資訊。劃分概述如下：

[Twitter Snowflake 中位劃分的概述](./snowflakes.jpg)

位劃分的解釋如下：

• **Sign bit (符號位)**: A single bit is assigned as a sign bit, and its value will always be zero. It makes the overall number positive. Doing so helps to ensure that any programming environment using these identifiers interprets them as positive integers.
• **符號位**：分配單個位作為符號位，其值始終為零。它使整體數字為正。這樣做有助於確保任何使用這些標識符的編程環境將它們解釋為正整數。

• **Time stamp (時間戳)**: 41 bits are assigned for milliseconds. The Twitter Snowflake default epoch will be used. Its value is 1288834974657, which is equivalent to November 4, 2010, 01:42:54 UTC. We can initiate our own epoch when our system will be deployed, say January 1, 2022, at 12 midnight can be the start of our epoch from zero. The maximum time to deplete this range is shown below:
• **時間戳**：分配 41 位用於毫秒。將使用 Twitter Snowflake 默認紀元。其值為 1288834974657，相當於 UTC 時間 2010 年 11 月 4 日 01:42:54。我們可以在部署系統時啟動我們自己的紀元，例如 2022 年 1 月 1 日午夜 12 點可以是我們從零開始的紀元。耗盡此範圍的最長時間如下所示：

範圍耗盡時間 = 2^41 標識符 / 365(天/年) x 24 (小時/天) x 60 (分/小時) x 60 (秒/分) x 1000 (標識符/秒) = 大約 69 年

上述計算給了我們 69 年的時間，之後我們需要一種新的算法來生成 ID。正如我們之前所看到的，如果我們每秒可以生成 1,000 個標識符，我們將無法實現每天 10 億個標識符的目標。雖然現在，在 Snowflake 提案中，當我們利用工作者 ID 和機器本地序列號時，我們有足夠的標識符可用。

• **Worker number (工作者編號)**: The worker number is 10 bits. It gives us 2^{10} = 1,024 worker IDs. The server creating the unique ID for its events will attach its ID.
• **工作者編號**：工作者編號為 10 位。它給了我們 2^{10} = 1,024 個工作者 ID。為其事件創建唯一 ID 的伺服器將附加其 ID。

• **Sequence number (序列號)**: The sequence number is 12 bits. For every ID generated on the server, the sequence number is incremented by one. It gives us 2^{12} = 4,096 unique sequence numbers. We’ll reset it to zero when it reaches 4,096. This number adds a layer to avoid duplication.
• **序列號**：序列號為 12 位。對於伺服器上生成的每個 ID，序列號加一。它給了我們 2^{12} = 4,096 個唯一序列號。當它達到 4,096 時，我們會將其重置為零。這個數字增加了一層以避免重複。

以下幻燈片顯示了時間戳到 UTC 的轉換。
[snowflakes](./conversion)


### Pros (優點)
Twitter Snowflake 使用時間戳作為第一個組件。因此，它們是按時間排序的。ID 生成器也具有高可用性。

### Cons (缺點)
在死區期間生成的 ID 是一個問題。死區是未向伺服器發出生成 ID 的請求時。這些 ID 將被浪費，因為它們佔用了標識符空間。可能的唯一範圍將比預期更早耗盡，並在我們的全局使用者 ID 集中產生間隙。

```
Question
你能找出上面展示的設計的另一個缺點嗎？

Answer
物理時鐘不可靠。對於這類時鐘，誤差可能為每天 17 秒。如果我們在伺服器上使用這些測量時間，時間會漂移。

考慮到單個伺服器，我們不會受到時間漂移的影響，因為所有事務都落在單個伺服器上。但是在分佈式環境中，時鐘將不會保持同步。

由於測量準確時間的不可靠性，無論我們多久將這些時鐘與彼此或其他具有準確測量方法的時鐘同步一次，分佈式系統中涉及的各種時鐘之間總是存在偏差。
```

```
物理時鐘：用於測量和指示時間的設備，例如鍾擺、沙漏、石英鐘等。
```
該系統的另一個弱點是它對時間的依賴。NTP 會影響該系統的運行。如果其中一台伺服器上的時鐘向未來漂移兩秒，則其他伺服器將落後兩秒。NTP 時鐘識別出它並重新校準其時鐘。現在，所有伺服器都將對齊。但是，在該漂移過程中，可能已經為尚未發生的時間生成了 ID，現在我們將擁有一對具有相同時間戳的可能的非並發事件。最後，我們事件的因果關係將無法維持。

```
注意：網絡時間協議 (NTP) 是一種用於在分組交換、可變延遲資料網絡上的計算機系統之間進行時鐘同步的網絡協議。NTP 旨在將所有參與的計算機同步到協調世界時 (UTC) 的幾毫秒內。減輕了可變網絡延遲的影響。
```
擁有準確的時間仍然是一個問題。我們可以讀取具有微秒甚至納秒分辨率的機器掛鐘。即使有這種細粒度的測量，NTP 的風險仍然存在。由於我們不能依賴物理時鐘，讓我們使用邏輯時鐘。

Risks of NTP (NTP 的風險)
除非對資料包進行加密簽名以進行身份驗證，否則 NTP 伺服器很容易受到中間人攻擊。所涉及的計算開銷可能使其在繁忙的伺服器上不切實際，尤其是在拒絕服務攻擊期間。來自中間人攻擊的 NTP 消息欺騙可用於移動客戶端計算機上的時鐘，並允許基於繞過加密密鑰過期的幾種攻擊。受假 NTP 消息影響的一些服務包括 TLS、DNSSEC、各種緩存方案 (例如 DNS 緩存)、BGP、比特幣和許多持久登錄方案。
```


下表概述了使用不同設計方法滿足的要求。

```
                            Unique (唯一)           Scalable (可擴展)       Available (可用)          64-bit numeric ID (64 位數字 ID)         Causality maintained (維持因果關係)
   
Using UUID                   ✖️                       ✔️                     ✔️                           ✖️                                         ✖️

Using a database             ✖️                       ✖️                     ✔️                           ✔️                                         ✖️

Using a range handler        ✔️                       ✔️                     ✔️                           ✔️                                         ✖️

Using UNIX time stamps       ✖️                      weak (弱)               ✔️                           ✔️                                        weak (弱)

Using Twitter Snowflake      ✔️                       ✔️                     ✔️                           ✔️                                        weak (弱)

## Using logical clocks (使用邏輯時鐘)
我們可以利用需要單調遞增事件標識符的邏輯時鐘 (Lamport 和向量時鐘)。

### Lamport clocks (Lamport 時鐘)
在 Lamport 時鐘中，每個節點都有其計數器。所有系統節點都配備了一個數字計數器，在首次激活時從零開始。在執行事件之前，數字計數器加一。從此事件發送到另一個節點的消息具有計數器值。當另一個節點收到消息時，它首先通過取其時鐘值的最大值來更新其邏輯時鐘。然後，它獲取消息中發送的值，然後執行消息。

Lamport 時鐘使用先發生關係提供事件的唯一部分排序。我們還可以通過標記唯一的節點/進程標識符來獲得事件的總排序，儘管這種排序不是唯一的，並且會隨著節點標識符的不同分配而改變。但是，我們應該注意，Lamport 時鐘不允許我們在全局級別推斷因果關係。這意味著我們不能簡單地比較任何伺服器上的兩個時鐘值來推斷先發生關係。向量時鐘克服了這個缺點。

### Vector clocks (向量時鐘)
向量時鐘維護因果歷史——即關於事件先發生關係的所有資訊。因此，我們必須選擇一種有效的資料結構來捕獲每個事件的因果歷史。

考慮下面顯示的設計。我們將通過連接相關信息來生成我們的 ID，就像 Twitter snowflake 一樣，劃分如下：

- **Sign bit (符號位)**: A single bit is assigned as a sign bit, and its value will always be zero.
- **符號位**：單個位被分配為符號位，其值始終為零。

- **Vector clock (向量時鐘)**: This is 53 bits and the counters of each node.
- **向量時鐘**：這是 53 位與每個節點的計數器。

- **Worker number (工作者編號)**: This is 10 bits. It gives us 2^{10} = 1,024 worker IDs.
- **工作者編號**：這是 10 位。它給了我們 2^{10} = 1,024 個工作者 ID。

以下幻燈片解釋了使用向量時鐘生成唯一 ID，其中節點 A、B 和 C 駐留在資料中心。
```
注意：在下面的幻燈片中，為了便於理解，我們沒有將資料轉換為位。我們將用於唯一 ID 的模式如下：

[vector-clock][worker-id]
```

[向量時鐘](./vector)

我們的向量時鐘方法有效。但是，為了完全捕獲因果關係，向量時鐘的大小必須至少為 n 個節點。因此，當參與節點的總數巨大時，向量時鐘需要大量的存儲空間。如今的一些系統 (例如 Web 應用程式) 將每個瀏覽器都視為系統的客戶端。這類資訊顯著增加了 ID 長度，使其難以處理、存儲、使用和擴展。

```
                            Unique (唯一)           Scalable (可擴展)       Available (可用)          64-bit numeric ID (64 位數字 ID)         Causality maintained (維持因果關係)
   
Using UUID                   ✖️                       ✔️                     ✔️                           ✖️                                         ✖️

Using a database             ✖️                       ✖️                     ✔️                           ✔️                                         ✖️

Using a range handler        ✔️                       ✔️                     ✔️                           ✔️                                         ✖️

Using UNIX time stamps       ✖️                      weak (弱)               ✔️                           ✔️                                        weak (弱)

Using Twitter Snowflake      ✔️                       ✔️                     ✔️                           ✔️                                        weak (弱)

Using vector clocks          ✔️                      weak (弱)               ✔️                        can exceed (可能超過)                         ✔️

```

```
Question
全局時鐘能幫我們解決問題嗎？

Answer
由於我們沒有全局時鐘，即使每個節點都可以為發生的事件分配唯一的時間戳，這些時間戳也將來自以不同速率運行的時鐘。這將使比較它們變得更加困難，並且它們將不是唯一的。

但是，如果我們有一個全局時鐘，它可以根據請求給我們時間並且總是準確的，那麼我們可以維持事件的因果關係以及唯一 ID。這樣的時鐘非常有價值，但在分佈式系統中處理時間很棘手。
```
## TrueTime API
Spanner 中的 Google TrueTime API 是一個有趣的選擇。它報告一個時間間隔，而不是特定的時間戳。當詢問當前時間時，我們會得到兩個值：最早的和最晚的。這些是最早可能和最晚可能的時間戳。

根據其不確定性計算，時鐘知道實際當前時間在該區間內的某個位置。該區間的寬度除其他因素外，還取決於本地石英鐘上次與更精確的時鐘源同步以來經過了多長時間。

Google 在每個資料中心部署 GPS 接收器或原子鐘，時鐘同步在 7 毫秒左右。這使得 Spanner 可以將時鐘不確定性保持在最低限度。區間的不確定性表示為 epsilon。

以下幻燈片解釋了 TrueTime 的時間主伺服器如何與多個資料中心的 GPS 和原子鐘配合使用。

[TrueTime](./Google)

以下幻燈片解釋了當客戶端請求提供 TrueTime 時如何計算時間。

[TrueTime](./TrueTime)

Spanner 保證兩個置信區間不重疊 (即 A_{earliest} < A_{latest} < B_{earliest} < B_{latest)，那麼 B 肯定發生在 A 之後。

我們使用 TrueTime 區間生成我們的唯一 ID。假設最早的區間是 T_{E}，最晚的是 T_{L}，不確定性是 ε。我們使用以毫秒為單位的 T_{E} 作為唯一 ID 中的時間戳。

- **Time stamp (時間戳)**: The time stamp is 41 bits. We use T_{E} as a time stamp.
- **時間戳**：時間戳為 41 位。我們使用 T_{E} 作為時間戳。

- **Uncertainty (不確定性)**: The uncertainty is four bits. Since the maximum uncertainty is claimed to be 6–10 ms, we’ll use four bits for storing it.
- **不確定性**：不確定性是四位。由於最大不確定性聲稱為 6–10 毫秒，我們將使用四位來存儲它。

- **Worker number (工作者編號)**: This is 10 bits. It gives us 2^{10} = 1,024 worker IDs.
- **工作者編號**：這是 10 位。它給了我們 2^{10} = 1,024 個工作者 ID。

- **Sequence number (序列號)**: This is eight bits. For every ID generated on the server, the sequence number is incremented by one. It gives us 2^{8} = 256 combinations. We’ll reset it to zero when it reaches 256.
- **序列號**：這是八位。對於伺服器上生成的每個 ID，序列號都會加一。它給了我們 2^{8} = 256 種組合。當它達到 256 時，我們會將其重置為零。

[節點 B 使用 TrueTime 為其事件生成唯一 ID]

### Pros (優點)
TrueTime 滿足所有要求。我們能夠生成全局唯一的 64 位標識符。維持了事件的因果關係。該方法具有可擴展性和高可用性。

### Cons (缺點)
如果兩個區間重疊，那麼我們不確定 A 和 B 發生的順序。它們可能是並發事件，但不能給予 100% 的保證。此外，Spanner 很昂貴，因為它確保了高資料庫一致性。由於其複雜的基礎設施需求和監控，類 Spanner 系統的成本也很高。

更新後的表格提供了生成唯一 ID 的不同系統設計之間的比較。

```
                            Unique (唯一)           Scalable (可擴展)       Available (可用)          64-bit numeric ID (64 位數字 ID)         Causality maintained (維持因果關係)
   
Using UUID                   ✖️                       ✔️                     ✔️                           ✖️                                         ✖️

Using a database             ✖️                       ✖️                     ✔️                           ✔️                                         ✖️

Using a range handler        ✔️                       ✔️                     ✔️                           ✔️                                         ✖️

Using UNIX time stamps       ✖️                      weak (弱)               ✔️                           ✔️                                        weak (弱)

Using Twitter Snowflake      ✔️                       ✔️                     ✔️                           ✔️                                        weak (弱)

Using vector clocks          ✔️                      weak (弱)               ✔️                        can exceed (可能超過)                         ✔️

Using TrueTime               ✔️                       ✔️                     ✔️                           ✔️                                         ✔️

## Summary (總結)
我們希望避免重複的標識符。考慮一下如果生成重複的付款或採購訂單會發生什麼。

UUID 提供關於密鑰非衝突的概率保證。確定性地獲得非衝突保證可能需要在不同的分佈式實體或存儲之間達成共識，並從複製存儲中讀取。

隨著密鑰長度變大，這通常會導致資料庫中的元組更新變慢。因此，標識符應該足夠大，但又不能太大。

通常，希望沒有人能夠猜到下一個 ID。否則，可能會發生不良資料洩漏，組織的競爭對手可能會通過簡單地查看訂單 ID 來了解一天處理了多少訂單。在標識符的位中添加一些隨機數使其難以猜測，儘管這會帶來性能成本。

如果我們不想將 ID 與時間相關聯，我們可以使用簡單的計數器來生成唯一 ID。獲取時間戳比簡單的計數器慢。
如果我們不想將 ID 與時間相關聯，我們可以使用簡單的計數器來生成唯一 ID。獲取時間戳比簡單的計數器慢，儘管這需要我們持久存儲生成的 ID。計數器需要存儲在資料庫中。存儲有其自身的問題。其中包括多個並發寫入對資料庫來說變得不堪重負，以及資料庫成為單點故障。

對於某些分佈式資料庫，例如 Spanner，生成單調遞增或遞減的 ID 可能有害。Google 報告如下：“事實上，在 Spanner 中使用單調遞增 (或遞減) 值作為行鍵並不遵循最佳實踐，因為它會在資料庫中產生熱點，從而導致性能下降。”

注意：全局排序事件是一個昂貴的過程。在集中式資料庫中快速簡單的功能 (基於自動遞增的 ID)，在分佈式對應物中由於一些基本限制 (例如在遠程實體之間難以達成共識) 而變得緩慢而復雜。

例如，地理分佈式資料庫 Spanner 報告稱，“如果對單個單元格 (單行中的一列) 的讀取-更新事務具有 10 毫秒 (ms) 的延遲，那麼發出序列值的最大理論頻率是每秒 100 次。此最大值適用於整個資料庫，無論客戶端應用程式實例的數量或資料庫中的節點數量如何。這是因為單個節點始終管理單行。”如果以對全局排序和無間隙標識符的要求妥協，我們將能夠在更短的時間內獲得許多標識符，即更好的性能。
