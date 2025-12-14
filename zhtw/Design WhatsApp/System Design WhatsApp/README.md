# System Design: WhatsApp (系統設計：WhatsApp)
## WhatsApp
在當今的技術世界中，WhatsApp 是一個重要的訊息應用程式，連接全球數十億人。許多使用者通過閱讀或向朋友和家人發送 WhatsApp 訊息開始新的一天。根據 2021 年 7 月的估計，WhatsApp 在全球擁有 20 億活躍使用者。此外，平均每個 WhatsApp 使用者每月在該應用程式上花費約 19.4 小時。

2020 年 12 月，WhatsApp CEO 發推文稱，WhatsApp 使用者每天分享超過 1000 億條訊息，自 2018 年以來增加了約 54%。下圖描述了全球每日發送訊息的增加情況：

## Design problem (設計問題)
作為系統設計師，我們應該意識到使用者的增長率。關於 WhatsApp 有許多有趣的問題：

- **How is this application designed? (這個應用程式是如何設計的？)**
- **How does it work? (它是如何工作的？)**
- **What are the different types of components involved in it? (其中涉及哪些不同類型的組件？)**
- **How does WhatsApp enable billions of users to communicate with each other? (WhatsApp 如何讓數十億使用者相互通訊？)**
- **How does WhatsApp keep all that data secure? (WhatsApp 如何保證所有資料的安全？)**
在本章中，我們將重點介紹 WhatsApp 應用程式的高層和詳細設計，以回答上述問題。

## How will we design WhatsApp? (我們將如何設計 WhatsApp？)
我們將 WhatsApp Messenger 的設計分為以下五課：

1. **Requirements (需求)**：在本課中，我們將確定功能和非功能需求。我們還將討論為擬議的 WhatsApp 設計更好、更順暢的運行所需的資源估算。
2. **High-level Design (高層設計)**：我們將重點介紹我們 WhatsApp 版本的高層設計。我們還將討論我們 WhatsApp 設計的基本 API。
3. **Detailed Design (詳細設計)**：在本課中，我們將詳細描述我們的 WhatsApp messenger 的設計。最初，我們將解釋每個微服務的設計，包括與伺服器的連接、發送和接收訊息和媒體內容以及群組訊息。最後，將每個微服務的設計組合成 WhatsApp 的詳細設計。
4. **Evaluation (評估)**：本課將解釋我們的 WhatsApp 版本如何滿足非功能需求。我們還將評估我們設計的一些權衡。
5. **Quiz (測驗)**：在這裡，我們將通過測驗評估我們在本章中學到的內容。
讓我們從討論我們版本的 WhatsApp 的需求開始。
