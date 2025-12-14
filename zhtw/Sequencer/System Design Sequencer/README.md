# System Design: Sequencer (系統設計：定序器)

## Motivation (動機)
在大型分佈式系統中，每秒可能發生數百萬個事件。在 Facebook 上評論帖子、分享推文以及在 Instagram 上發布圖片只是此類事件的幾個示例。我們需要一種機制來區分這些事件。一種這樣的機制是為每個事件分配全局唯一 ID。

為資料庫中的條目分配主鍵是唯一 ID 的另一個用例。通常，資料庫中的自動遞增功能滿足此要求。但是，該功能不適用於分佈式資料庫，在分佈式資料庫中，不同節點獨立生成標識符。對於這種情況，我們需要一個在分佈式設置中充當主鍵的唯一 ID 生成器——例如，水平分片的表。

唯一 ID 幫助我們識別日誌中事件的流向，這對於調試非常有用。唯一 ID 使用的一個真實示例是 Facebook 的端到端性能跟踪和分析系統 Canopy。Canopy 使用 TraceID 在執行路徑中唯一標識事件，該執行路徑可能會執行數百個微服務來滿足一個使用者請求。

```
Canopy：端到端性能跟踪和分析系統
```

[為每個事件分配唯一的 TraceID](./seq.jpg)

## How do we design a sequencer? (我們如何設計定序器？)
我們將定序器的綜合設計分為以下兩課：

1. **Design of a Unique ID Generator (唯一 ID 生成器的設計)**: After enlisting the requirements of the design, we discuss three ways to generate unique IDs: using UUID, using a database, and using a range handler.
1. **唯一 ID 生成器的設計**：在列出設計要求後，我們討論三種生成唯一 ID 的方法：使用 UUID，使用資料庫和使用範圍處理程序。
2. **Unique IDs with Causality (具有因果關係的唯一 ID)**: In this lesson, we incorporate an additional factor of time in the generation of IDs and explain the process by taking causality into consideration.
2. **具有因果關係的唯一 ID**：在本課中，我們在 ID 生成中加入了一個額外的時間因素，並通過考慮因果關係來解釋該過程。

唯一 ID 對於識別分佈式系統中的事件和對象非常重要。然而，在分佈式系統中設計唯一 ID 生成器具有挑戰性。在下一課中，讓我們看看分佈式唯一 ID 生成系統的要求。
