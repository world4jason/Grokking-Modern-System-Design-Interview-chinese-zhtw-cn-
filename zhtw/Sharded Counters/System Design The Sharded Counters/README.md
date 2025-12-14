# System Design: The Sharded Counters (系統設計：分片計數器)

## Problem statement (問題陳述)
Facebook、Twitter 和 YouTube 等實時應用程式擁有高使用者流量。使用者與應用程式交互並根據應用程式的結構執行多種操作 (查看、點贊、評論等)。例如，在擁有數百萬粉絲的 Facebook 頁面上發布了一張圖片，並且該帖子的點贊數在每一毫秒後迅速增加。在這裡，計算這張單個圖片的讚數可能很容易，但是當許多擁有名人 (每個人都有數百萬粉絲) 同時上傳數千個此類圖片或視頻時，我們會怎麼做？這個問題被稱為重擊者 (heavy hitters) 問題。

上述場景顯示了簡單的計數操作如何變得難以精確和高性能地管理。下圖顯示了 2021 年 8 月 24 小時內數百萬使用者觀看的 YouTube 視頻：

[24 小時內的 YouTube 視頻觀看次數](./views.jpg)

平均而言，Twitter 上一秒鐘發送六千條推文，這相當於每分鐘 360,000 條推文，每天約 5 億條推文。一項艱鉅的任務是每天處理這 5 億條推文上的數十億個贊。下表顯示了截至 2022 年一天中最受歡迎的推文：

[一天中最受歡迎的推文](./likes.jpg)

我們將如何處理每分鐘針對數千條推文的點贊而來的數百萬個寫入請求？挑戰在於寫入比讀取花費更多時間，並且並發活動使這個問題更加困難。隨著某些計數器 (可能是駐留在節點內存中的變量) 的並發寫入數量的增加，鎖爭用非線性增加。在某個點之後，我們可能會花費大部分時間來獲取鎖，以便我們可以安全地更新計數器。

## How will we design sharded counters? (我們將如何設計分片計數器？)

我們將分片計數器的設計分為三課：

1. **High-level Design (高層設計)**: We’ll discuss the high-level design of sharded counters in this lesson. In addition, we’ll also briefly explain the API design.
1. **高層設計**：我們將在本課中討論分片計數器的高層設計。此外，我們還將簡要解釋 API 設計。
2. **Detailed Design (詳細設計)**: This lesson will dive deeply into the design of sharded counters. Moreover, we’ll also evaluate our proposed design.
2. **詳細設計**：本課將深入探討分片計數器的設計。此外，我們還將評估我們提出的設計。
3. **Quiz (測驗)**: We’ll review major concepts of sharded counters design with a quiz.
3. **測驗**：我們將通過測驗回顧分片計數器設計的主要概念。

讓我們從分片計數器的高級解決方案草圖開始。
