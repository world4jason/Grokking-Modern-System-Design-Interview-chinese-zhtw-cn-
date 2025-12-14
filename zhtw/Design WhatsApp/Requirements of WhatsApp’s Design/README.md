# Requirements of WhatsApp’s Design (WhatsApp 的設計需求)
## Requirements (需求)
我們對 WhatsApp messenger 的設計應滿足以下需求。

### Functional requirements (功能需求)
- **Conversation (對話)**：系統應支援使用者之間的一對一和群組對話。

- **Acknowledgment (確認)**：系統應支援訊息傳遞確認，例如已發送、已送達和已讀。

- **Sharing (分享)**：系統應支援分享媒體文件，例如圖像、影片和音訊。

- **Chat storage (聊天存儲)**：當使用者離線時，系統必須支援持久儲存聊天訊息，直到訊息成功傳遞。

- **Push notifications (推送通知)**：一旦離線使用者的狀態變為在線，系統應能夠通知他們有新訊息。

### Non-functional requirements (非功能需求)
- **Low latency (低延遲)**：使用者應能夠以低延遲接收訊息。

- **Consistency (一致性)**：訊息應按照發送順序傳遞。此外，使用者必須在其所有設備上看到相同的聊天記錄。

- **Availability (可用性)**：系統應該具有高可用性。但是，為了保持一致性，可用性可能會受到影響。

- **Security (安全性)**：系統必須通過端到端加密來保障安全。端到端加密確保只有通訊雙方才能看到訊息內容。中間的任何人，甚至 WhatsApp，都不應該有權訪問。

[WhatsApp 系統的非功能需求](./nfr.jpg)

## Resource estimation (資源估算)
WhatsApp 是全球使用最廣泛的訊息應用程式。據 WhatsApp 稱，它支援全球超過 20 億使用者，每天分享超過 1000 億條訊息。我們需要估算儲存容量、頻寬和伺服器數量，以支援如此龐大的使用者和訊息數量。

### Storage estimation (儲存估算)
由於每天通過 WhatsApp 分享超過 1000 億條訊息，讓我們根據這個數字估算儲存容量。假設每條訊息平均佔用 100 個位元組。此外，WhatsApp 伺服器僅將訊息保留 30 天。因此，如果使用者在這些天內沒有連接到伺服器，訊息將從伺服器中永久刪除。

100 billion/day ∗ 100 Bytes = 10 TB/day

對於 30 天，儲存容量將變為如下：

30 ∗ 10 TB/day = 300 TB/month

除了聊天訊息，我們還有媒體文件，每條訊息佔用超過 100 個位元組。此外，我們還必須儲存使用者的資訊和訊息的元資料——例如，時間戳、ID 等。在此過程中，我們還需要加密和解密以進行安全通訊。因此，我們還需要儲存加密密鑰和相關元資料。所以，準確地說，我們每月需要超過 300 TB，但為了簡單起見，讓我們堅持每月 300 TB 這個數字。

[WhatsApp 一個月所需的總儲存空間](./storage.jpg)

### Bandwidth estimation (頻寬估算)
根據儲存容量估算，我們的服務每天將獲得 10TB 的資料，這給了我們 926 Mb/s 的頻寬。

10 TB/86400sec ≈ 926 Mb/s

```
注意：為了使我們的設計簡單，我們忽略了媒體內容 (圖像、影片、文件等)。所以，數字 926 看起來可能很低。
我們還需要同等數量的傳出頻寬，因為發送者的相同訊息需要傳遞給接收者。

[WhatsApp 所需的總頻寬](./bandwidth.jpg)

                   High-level Estimates (高層估算)
Type (類型)                                      Estimates (估算)

Total messages per day (每天總訊息數)                   100 billion (1000 億)

Storage required per day (每天所需儲存)                  10 TB

Storage for 30 days (30 天的儲存)                       300 TB

Incoming data per second (每秒傳入資料)                  926 Mb/s

Outgoing data per second (每秒傳出資料)                  926 Mb/s

```

### Number of servers estimation (伺服器數量估算)
WhatsApp 在單個伺服器上處理大約 1000 萬個連接，這對於伺服器來說似乎相當高。但是，通過廣泛的效能工程是有可能的。我們需要了解系統的所有深入細節，例如伺服器的內核、網路庫、基礎設施配置等。
```
注意：我們經常可以通過對整個軟體堆棧進行仔細的效能工程來優化通用伺服器以執行特殊任務。
讓我們轉向伺服器數量的估算：

No. of servers (伺服器數量) = Total connections per day (每天總連接數) / No. of connections per server (每個伺服器的連接數) = 2 billion (20 億) / 10 million (1000 萬) = 200 servers (200 台伺服器)

因此，根據上述估算，我們需要 200 台聊天伺服器。


[WhatsApp 所需的聊天伺服器數量](./servers.jpg)

### Try it out (試一試)
讓我們分析每天的訊息數量如何影響儲存和頻寬需求。為此，我們可以更改下表中的值以計算估算值：

Number of users per day (in billions) (每天使用者數 (十億))	2
Number of messages per day (in billions) (每天訊息數 (十億))	100
Size of each message (in bytes) (每條訊息的大小 (位元組))	100
Number of connections a server can handle (in millions) (伺服器可以處理的連接數 (百萬))	10
Storage estimation per day (in TB) (每天儲存估算 (TB))	10
Incoming and Outgoing bandwidth (Mb/s) (傳入和傳出頻寬 (Mb/s))	926.4
Number of chat servers required (所需聊天伺服器數量) 200
```

## Building blocks we will use (我們將使用的構建塊)
WhatsApp 的設計利用了以下構建塊，這些構建塊也在最初的章節中討論過：

[設計 WhatsApp 所需的構建塊](./bb.jpg)

- **Databases (資料庫)** 需要儲存使用者和群組的元資料。
- **Blob storage (Blob 儲存)** 用於儲存訊息中分享的多媒體內容。
- **CDN** 用於有效地傳遞經常分享的多媒體內容。
- **Load balancer (負載平衡器)** 在可用伺服器池中分發傳入請求。
- **Caches (快取)** 需要保留各種服務使用的頻繁訪問的資料。
- **Messaging queue (訊息隊列)** 用於在使用者離線時將訊息臨時保留在資料庫上的隊列中。
在下一課中，我們將重點介紹 WhatsApp messenger 的高層設計。
