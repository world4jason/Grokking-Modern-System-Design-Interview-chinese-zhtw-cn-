# System Design: YouTube (系統設計：YouTube)
## What is YouTube? (什麼是 YouTube？)
YouTube 是一個流行的影片流媒體服務，使用者可以在其中上傳、流式傳輸、收看、評論、分享和點讚或不點讚影片。大型企業和個人都維護著自己的頻道來託管他們的影片。YouTube 允許免費託管影片內容並與全球使用者分享。YouTube 被認為是主要的娛樂來源，尤其是在年輕人中，截至 2022 年，它被維基百科列為僅次於 Google 的第二大瀏覽量網站。

[一個內容創作者上傳被許多觀眾流式傳輸的影片到 YouTube 的描繪](./youtube.jpg)

## YouTube’s popularity (YouTube 的受歡迎程度)
多年來 YouTube 越來越受歡迎的一些原因包括：

- **Simplicity (簡單)**：YouTube 擁有簡單而強大的介面。
- **Rich content (豐富的內容)**：其簡單的介面和免費託管吸引了大量的內容創作者。
- **Continuous improvement (持續改進)**：YouTube 的開發團隊不懈努力，以滿足隨著時間推移而來的可擴展性需求。此外，Google 於 2007 年收購了 YouTube，這增加了其可信度。
- **Source of income (收入來源)**：YouTube 制定了一​​項合作夥伴計劃，允許內容創作者通過其病毒式傳播的內容賺錢。
除了上述原因外，YouTube 還與 CNN 和 NBC 等成熟的市場巨頭合作，以建立更穩固的立足點。

## YouTube’s growth (YouTube 的成長)
讓我們看看一些關於 YouTube 及其受歡迎程度的有趣統計資料。

自 2005 年 2 月成立以來，YouTube 使用者的數量成倍增長。截至目前，YouTube 的月活躍使用者已超過 25 億。讓我們看下面的圖表，看看過去十年 YouTube 使用者是如何增加的。

[每年 YouTube 月活躍使用者數的增長](./growth.jpg)

YouTube 是繼 Netflix 之後流媒體播放量第二高的網站。每分鐘在 YouTube 上流式傳輸的影片內容總計 694,000 小時！

```
Source: https://www.statista.com/statistics/195140/new-user-generated-content-uploaded-by-users-per-minute/
```

現在我們意識到 YouTube 有多成功，讓我們了解我們如何設計它。

## How will we design YouTube? (我們將如何設計 YouTube？)
我們將 YouTube 的設計分為五課：

1. **Requirements (需求)**：這是我們確定功能和非功能需求的地方。我們還估算了每天為數百萬使用者提供服務所需的資源。本課回答了諸如 YouTube 需要多少存儲空間來存儲每天上傳到 YouTube 的 500 小時影片內容等問題。

2. **Design (設計)**：在本課中，我們將解釋如何設計 YouTube 服務。我們還簡要解釋了 API 設計和資料庫模式。最後，我們還將簡要介紹 YouTube 的搜索是如何工作的。

3. **Evaluation (評估)**：本課解釋了 YouTube 如何通過擬議的設計滿足所有需求。它還著眼於未來的擴展如何影響系統，以及需要哪些解決方案來處理擴展問題。

4. **Reality is more complicated (現實更複雜)**：在本課中，我們將探討 YouTube 採用的不同技術，以有效地向客戶端傳遞內容並避免網路擁塞。

5. **Quiz (測驗)**：我們通過考慮如何設計 Netflix 的系統來加強我們在設計 YouTube 時學到的主要概念。

我們對設計中各種構建塊使用的討論將是有限的，因為我們已經在構建塊章節中詳細探討了它們。
