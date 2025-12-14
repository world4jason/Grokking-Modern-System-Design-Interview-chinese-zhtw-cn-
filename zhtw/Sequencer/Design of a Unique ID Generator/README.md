# Design of a Unique ID Generator (唯一 ID 生成器的設計)
在上一課中，我們看到許多用例都需要唯一標識符，例如識別對象 (例如推文、上傳的視頻等) 以及跟踪複雜服務網絡中的執行流。現在，我們將正式確定唯一標識符的要求，並討論三種逐步改進的設計以滿足我們的要求。

## Requirements for unique identifiers (唯一標識符的要求)
我們系統的要求如下：

- **Uniqueness (唯一性)**: We need to assign unique identifiers to different events for identification purposes.
- **唯一性**：我們需要為不同的事件分配唯一的標識符以進行識別。

- **Scalability (可擴展性)**: The ID generation system should generate at least a billion unique IDs per day.
- **可擴展性**：ID 生成系統每天應至少生成 10 億個唯一 ID。

- **Availability (可用性)**: Since multiple events happen even at the level of nanoseconds, our system should generate IDs for all the events that occur.
- **可用性**：由於即使在納秒級別也會發生多個事件，因此我們的系統應該為發生的所有事件生成 ID。

- **64-bit numeric ID (64 位數字 ID)**: We restrict the length to 64 bits because this bit size is enough for many years in the future. Let’s calculate the number of years after which our ID range will wrap around.
- **64 位數字 ID**：我們將長度限制為 64 位，因為這個位大小在未來的許多年裡已經足夠了。讓我們計算一下 ID 範圍將在多少年後用完。

Total numbers available = 2^{64} = 1.8446744 x 10^{19}
Estimated number of events per day = 1 billion = 10^{9}
Number of years to deplete identifier range = 2^64 / (365 x 10^9) = 50,539,024.8595 years (耗盡標識符範圍所需的年數 = 50,539,024.8595 年)

考慮到這些計算，64 位對於唯一 ID 長度應該足夠了。

讓我們深入探討上述問題可能的解決方案。

## First solution: UUID (第一種解決方案：UUID)
A straw man solution for our design uses UUIDs (universally unique IDs). This is a 128-bit number and it looks like 123e4567e89b12d3a456426614174000
我們設計的一個稻草人解決方案使用 UUID (通用唯一 ID)。這是一個 128 位數字，十六進制看起來像 123e4567e89b12d3a456426614174000。它給了我們大約 10^{38} 個數字。UUID 有不同的版本。我們選擇版本 4，它生成一個偽隨機數。

```
稻草人解決方案：通常是一個簡單的解決方案，其主要目的是促進對更好解決方案的討論。
```

每個伺服器都可以生成自己的 ID 並將 ID 分配給其各自的事件。UUID 不需要協調，因為它獨立於伺服器。使用 UUID 很容易進行擴展和縮減，並且該系統也具有高可用性。此外，它的碰撞概率很低。該方法的設計如下：

[使用 UUID 方法生成唯一 ID](./uuid.jpg)

### Cons (缺點)
使用 128 位數字作為主鍵會使主鍵索引變慢，從而導致插入變慢。一種解決方法可能是將 ID 解釋為十六進制字串而不是數字。但是，非數字標識符可能不適用於許多用例。該 ID 不是 64 位大小。此外，還有重複的可能性。雖然這種機會很小，但我們不能聲稱 UUID 是確定性唯一的。此外，隨著時間的推移提供給客戶端的 UUID 可能不會單調增加。下表總結了我們使用 UUID 滿足的要求：

               Requirements Filled with UUID (使用 UUID 滿足的需求)
                  Unique (唯一)      Scalable (可擴展)     Available (可用)      64-bit numeric ID (64 位數字 ID)

Using UUID        ✖️                 ✔️                   ✔️                    ✖️

```

## Second solution: using a database (第二種解決方案：使用資料庫)
讓我們嘗試模仿資料庫的自動遞增功能。考慮一個提供當前 ID 然後將值加一的中央資料庫。我們可以使用當前 ID 作為事件的唯一標識符。

[使用中央資料庫生成唯一 ID](./db.jpg)

```
Point to Ponder

Question
使用中央資料庫的潛在問題是什麼？

Answer
這個設計有一個相當大的問題：單點故障。依賴一個資料庫會嚴重影響系統。如果中央資料庫宕機，整個系統將停止工作。
```

為了解決單點故障的問題，我們修改了傳統的加一自動遞增功能。讓我們依賴一個值 m，而不是加一，其中 m 等於我們擁有的資料庫伺服器數量。每個伺服器生成一個 ID，下一個 ID 將 m 添加到前一個值。這種方法是可擴展的，並防止 ID 重複。下圖直觀地展示瞭如何使用資料庫生成唯一 ID：

[使用 m 值生成 ID](./1.jpg)

[使用 m 值生成 ID](./2.jpg)

### Pros (優點)
這種方法是可擴展的。我們可以添加更多伺服器，m 的值將相應更新。

### Cons (缺點)
雖然這種方法在一定程度上是可擴展的，但很難擴展到多個資料中心。添加和刪除伺服器的任務可能會導致重複的 ID。例如，假設 m=3，伺服器 A 生成唯一 ID 1、4 和 7。伺服器 B 生成 ID 2、5 和 8，而伺服器 C 生成 ID 3、6 和 9。伺服器 B 由於某種故障而停機。現在，值 m 更新為 2。伺服器 A 生成 9 作為其下一個唯一 ID，但此 ID 已經由伺服器 C 生成。因此，ID 不再唯一。

下表突出顯示了我們解決方案的局限性。唯一 ID 生成系統不應成為單點故障 (SPOF)。它應該是可擴展的和可用的。

                          Requirements Filled by UUID versus Using a Database (UUID 與使用資料庫滿足的需求對比)
                       Unique (唯一)     Scalable (可擴展)        Available (可用)         64-bit numeric ID (64 位數字 ID)
Using UUID               ✖️               ✔️                      ✔️                       ✖️
Using a database         ✖️               ✖️                      ✔️                       ✔️

```

## Third solution: using a range handler (第三種解決方案：使用範圍處理程序)
讓我們嘗試克服先前方法中發現的問題。我們可以在中央伺服器中使用範圍。假設我們有一到二十億的多個範圍，例如 1 到 1,000,000；1,000,001 到 2,000,000；等等。在這種情況下，中央微服務可以根據請求向伺服器提供範圍。

任何伺服器在第一次需要範圍或範圍用完時都可以申請範圍。假設伺服器有一個範圍，現在它將範圍的開始保存在局部變量中。每當發出 ID 請求時，它都會將局部變量值提供給請求者並將該值加一。

假設伺服器 1 申請了 300,001 到 400,000 的數字範圍。申請此範圍後，使用者 ID 300,001 分配給第一個請求。然後伺服器將 300,002 返回給下一個使用者，在範圍內遞增其當前位置。這將持續到伺服器釋放使用者 ID 400,000。然後應用程式伺服器向中央伺服器查詢下一個可用範圍並重複此過程。

這解決了使用者 ID 重複的問題。每個應用程式伺服器都可以並發響應請求。我們可以在一組伺服器上添加負載平衡器以減輕請求負載。

我們使用一個稱為範圍處理程序的微服務，它記錄所有已佔用和可用的範圍。每個範圍的狀態可以確定範圍是否可用。狀態 (即哪個伺服器分配了什麼範圍) 可以保存在複製存儲中。

此微服務可能成為單點故障，但在這種情況下，故障轉移伺服器充當救星。當主伺服器宕機時，故障轉移伺服器分發範圍。我們可以從複製存儲的最新檢查點恢復可用和不可用範圍的狀態。

[範圍處理程序微服務的設計](./range_handler.jpg)

### Pros (優點)
該系統是可擴展的、可用的，並且產生沒有重複的使用者 ID。此外，我們可以將此範圍維持在 64 位 (數字) 內。

### Cons (缺點)
當伺服器死亡時，我們會丟失很大一部分範圍，並且只有在伺服器再次上線後才能提供新範圍。我們可以通過向伺服器分配較短的範圍來克服這個缺點，儘管範圍應該足夠大以在一段時間內提供標識符。

下表總結了這種方法為我們實現的目標：

                     Requirements Filled by These Three Options (這三個選項滿足的需求)
                        Unique (唯一)           Scalable (可擴展)     Available (可用)      64-bit numeric ID (64 位數字 ID)      
  Using UUID             ✖️                      ✔️                   ✔️                     ✖️
Using a database         ✖️                      ✖️                   ✔️                     ✔️
Using a range handler    ✔️                      ✔️                   ✔️                     ✔️
```

我們開發了一個解決方案，為我們提供了一個唯一 ID，我們可以將其分配給各種事件，甚至可以用作主鍵。但是，如果我們添加 ID 也必須是可按時間排序的要求會怎樣？
