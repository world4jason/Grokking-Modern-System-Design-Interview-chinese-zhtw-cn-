# Requirements of Yelp’s Design (Yelp 設計的需求)
## Requirements (需求)
讓我們確定我們系統的需求。

### Functional requirements (功能需求)
我們系統的功能需求如下：

- **User accounts (使用者帳戶)**：使用者將擁有帳戶，他們可以在其中執行不同的功能，如登錄、註銷、添加、刪除和更新地點訊息。

```
注意：可以有兩種類型的使用者：可以在平台上添加地點的企業主，以及可以搜索、查看和對地點進行評級的其他使用者。
```

- **Search (搜索)**：使用者應該能夠根據他們的 GPS 位置 (經度、緯度) 和/或地點名稱搜索附近的地點或感興趣的地點。

- **Feedback (反饋)**：使用者應該能夠添加關於地點的評論。評論可以包含圖像、文字和評級。

[功能需求](./fr.jpg)

### Non-functional requirements (非功能需求)
我們系統的非功能需求如下：

- **High availability (高可用性)**：系統應該對使用者高度可用。

- **Scalability (可擴展性)**：系統應該能夠根據請求的數量進行擴展和縮減。請求的數量可能因時間和天數而異。例如，午餐時間的搜索量通常多於午夜。同樣，在旅遊季節，我們的系統收到的請求將比一年中的其他月份多。

- **Consistency (一致性)**：系統對使用者來說應該是一致的。所有使用者都應該對有關地點、評論和圖像的資料有一致的視圖。

- **Performance (效能)**：搜尋時，系統應以最小的延遲響應建議。

## Resource estimation (資源估算)
讓我們假設我們有：

- 總共 1.78 億獨立使用者。
- 6000 萬日活躍使用者。
- 5 億個地點。

### Number of servers estimation (伺服器數量估算)
我們需要處理來自 6000 萬日活躍使用者的並發請求。正如我們在粗略估算課程中討論的那樣，我們假設 RPS 為 8,000。
```
日活躍使用者數/伺服器 RPS = 6000 萬 / 8000 = 7500 台伺服器
```

```
       估算伺服器數量

Number of Daily Active Users (in Millions)	  60 (日活躍使用者數 (以百萬計))
RPS of a Server                                  8000 (伺服器 RPS)
Number of Servers Required                       7500 (所需伺服器數量)
```

[Yelp 所需的伺服器數量](./servers.jpg)

### Storage estimation (儲存估算)
讓我們計算資料所需的儲存空間。讓我們做以下假設：

我們總共有 5 億個地點。

對於每個地點，我們需要 1,296 字節的儲存空間。

- 每個地點附有一張照片，所以我們有 5 億張照片。

- 對於每張照片，我們需要 280 字節的儲存空間。在這裡，我們考慮表中照片實體的行大小，其中包含指向 blob 存儲中實際照片的鏈接。

- 每天至少添加 100 萬條關於不同地點的評論。

- 對於每條評論，我們需要 537 字節的儲存空間。

- 我們總共有 1.78 億使用者。

- 對於每個使用者，我們需要 264 字節的儲存空間。
```
注意：每個地點、照片、評論和使用者使用的字節數基於我們將在下一課中討論的資料庫模式。
以下計算器計算我們所需的總儲存空間：

```
                 估算儲存需求
(訊息類型)	(大小)     (實體所需 (字節))	     (計數 (以百萬計))	(總大小 (以 GB 計))
Place	(地點)                               1296	                             500	                       648
Photo	(照片)                                280	                             500	                       140
Review	(評論)                                537	                             1                            0.54
User	(使用者)                                264	                             178                         46.99
Total Storage Required (所需總儲存空間)                                                                                   835.53            
                            
```

[Yelp 所需的總儲存量](./storage.jpg)


### Bandwidth estimation (頻寬估算)
為了估算 Yelp 的頻寬需求，我們將頻寬計算分為傳入和傳出流量。

對於傳入流量，讓我們假設如下：

- 平均每天增加五個地點。
- 對於每個地點，我們佔用 1,296 字節。
- 每個地點還附有一張 3 MB 的照片。這是我們保存在 blob 存儲中的照片大小。
- 每天增加 100 萬條關於不同地點的評論。
- 每條評論佔用 537 字節。
我們將每天的總訊息大小除以 86,400，將其轉換為每秒頻寬。
```
                        估算傳入頻寬需求
 Average Number of Places Added Daily (每日平均增加地點數)                             5
 Storage Needed for Each Place (Bytes) (每個地點所需儲存空間 (字節))                         1296
 Size of Photo (in MBs) (照片大小 (MB))                                           3
 Total Size of Place Information (Bytes) (地點訊息總大小 (字節))                       15006480
 Average Number of Reviews Added Daily (in Millions) (每日平均增加評論數 (百萬))               1
 Storage Needed for Each Review (Bytes) (每條評論所需儲存空間 (字節))                         537
 Total Size of Reviews (Bytes) (評論總大小 (字節))                                 537000000
 Total Incoming Bandwidth (KBps) (總傳入頻寬 (KBps))                               6.39
 Total Incoming Bandwidth (Kbps) (總傳入頻寬 (Kbps))                              51.12

```

對於傳出流量，讓我們假設如下：

- 一次搜索平均返回 20 個地點。
- 每個地點都附有一張平均大小為 3 MB 的照片。
- 每個返回的條目都包含地點和照片訊息。
考慮到有 6000 萬日活躍使用者，我們得出以下估算：
```
                估算傳出頻寬需求
 Average Number of Places Returned on Each Search Request (每次搜索請求返回的平均地點數)               20
 Size of Place (in Bytes) (地點大小 (字節))                                             1296
 Size of Photo (in MB) (照片大小 (MB))                                                   3
 Total Size of Place Information (Bytes) (地點訊息總大小 (字節))                         60025920
 Outgoing Bandwidth Required for a Single Request (Kbps) (單個請求所需的傳出頻寬 (Kbps))             0.69
 Outgoing Bandwidth Required for a Single Request (KBps) (單個請求所需的傳出頻寬 (KBps))             5.52
 Daily Active Users (in Millions) (日活躍使用者 (百萬))                                     60
 Total Outgoing Bandwidth Required (Kbps) (所需總傳出頻寬 (Kbps))                          331200000
 Total Outgoing Bandwidth Required (Gbps) (所需總傳出頻寬 (Gbps))                           331.2

假設上傳的內容未壓縮，我們總共需要大約 51 Kbps 的傳入流量和大約 331 Gbps 的傳出流量。

總頻寬需求 = 51 Kbps + 331 Gbps ≈ 331 Gbps。

[Yelp 所需的總頻寬](./bandwidth.jpg)

## Building blocks we will use (我們將使用的構建塊)
Yelp 的設計過程利用了課程最初幾章中已經討論過的許多構建塊。我們在設計 Yelp 時將考慮以下概念：

[Yelp 高層設計中的構建塊](./bb.jpg)

- **Caching (快取)**：我們將使用快取來存儲有關熱門地點的訊息。
- **Load balancer (負載平衡器)**：我們將使用負載平衡器來管理大量請求。
- **Blob storage (Blob 儲存)**：我們將圖像存儲在 Blob 儲存中。
- **Database (資料庫)**：我們將在資料庫中存儲有關地點和使用者的訊息。
我們還將依靠 Google Maps 來了解在特定半徑內搜索地點的功能。
