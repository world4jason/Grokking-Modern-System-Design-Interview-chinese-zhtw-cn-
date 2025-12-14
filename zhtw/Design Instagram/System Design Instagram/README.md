# System Design: Instagram (系統設計：Instagram)
## What is Instagram? (Instagram 是什麼？)
Instagram 是一個免費的社交網路應用程式，允許使用者發布照片和短片。使用者可以為每個貼文新增標題，並使用主題標籤或基於位置的地理標籤對其進行索引，並使其在應用程式中可被搜尋。使用者的 Instagram 貼文顯示在其關注者的動態消息中，如果標有主題標籤或地理標籤，公眾也可以看到。或者，使用者可以選擇將其個人資料設為私有，這將限制只有選擇關注他們的人才能訪問。

使用者數量的擴展需要更多的資源 (伺服器、資料庫等)。了解使用者的增長率有助於我們預測資源以相應地擴展我們的系統。下圖顯示了截至 2022 年 1 月不同國家的 Instagram 使用者群 (來源：Statista)。

[擁有最多 Instagram 使用者的國家](./countries.jpg)

## How will we design Instagram? (我們將如何設計 Instagram？)
我們將 Instagram 的設計分為四節課：

1. **Requirements (需求)**：本課將提出 Instagram 的功能和非功能需求。它還將估算實現這些需求所需的資源。
2. **Design (設計)**：本課將解釋每個組件的工作流程和用法、API 設計和資料庫模式。
3. **Detailed design (詳細設計)**：在本課中，我們將詳細探討我們 Instagram 設計的組件，並討論生成時間線的各種方法。此外，我們還將評估我們提出的設計。
4. **Quiz (測驗)**：本課將測試我們對 Instagram 設計的理解。

讓我們從理解設計我們的 Instagram 系統的需求開始，並在下一課中提供資源估算。
