# System Design: Google Docs (系統設計：Google Docs)
## Problem statement (問題陳述)
想像兩個學生正在準備一份關於他們剛剛完成的專案的報告。由於學生分開居住，第一個學生要求第二個學生開始寫報告，第一個學生會改進收到的副本。雖然很有動力，但學生們很快明白，這種合作是無組織的。下圖顯示了這個過程會變得多麼乏味。

[沒有協同文件編輯服務時出現的問題](./demo.jpg)

上面的場景是一個例子，說明當使用者通過相互交換檔案來協作處理文件時，會導致時間浪費和沮喪。

為了解決上述問題，我們可以使用像 Google Docs 這樣的線上協同文件編輯服務。使用線上文件編輯服務而不是桌面應用程式的一些優點如下：

- 使用者可以在編輯文件的同時查看和評論文件。
- 不需要特殊的硬體規格即可獲得最新功能。一台可以運行瀏覽器的機器就足夠了。
- 可以在任何地點工作。
- 與本地桌面編輯器不同，使用者可以查看長期的文件歷史記錄，並在需要時恢復舊版本。
- 該服務免費。

除 Google Docs 外，一些流行的線上編輯服務包括 Etherpad、Microsoft Office 365、Slite 等。

### Designing Google Docs (設計 Google Docs)
協同文件編輯服務可以通過兩種方式設計：

- 它可以設計為使用客戶端-伺服器 (client-server) 架構的集中式設施，為所有使用者提供文件編輯服務。
- 可以使用點對點 (peer-to-peer) 技術進行設計，以協作處理單個文件。

大多數商業解決方案都專注於客戶端-服務架構，以便進行更精細的控制。因此，我們將專注於使用客戶端服務架構設計服務。讓我們看看我們將在本章中如何進展。

注意：根據一項調查，64% 的人每週至少使用一次 Google Docs 進行文件編輯。
參見：https://www.statista.com/forecasts/1011649/frequency-of-using-google-docs-in-the-us
```

## How will we design Google Docs? (我們將如何設計 Google Docs？)
我們將設計問題分為四個階段：

1. **Requirements for Google Docs’ Design (Google Docs 設計需求)**：本課將重點關注建立設計協同文件編輯服務的需求。我們還將在這個階段量化基礎設施需求。
2. **Google Docs’ Design (Google Docs 設計)**：本課的目標是提出一個滿足服務需求的設計。本課將解釋為什麼使用某個組件以及它如何與其他組件整合以滿足功能需求。
3. **Concurrency in Collaborative Editing (協同編輯中的併發)**：線上文件編輯服務必須解決編輯文件同一部分的使用者之間的衝突。本課涵蓋了可能出現的問題類型以及用於解決此類衝突的技術。
4. **Evaluating Google Docs’ Design (評估 Google Docs 的設計)**：本課的主要目標是評估我們設計的非功能需求。主要是看我們的設計是否高效能、一致、可用和可擴展。
