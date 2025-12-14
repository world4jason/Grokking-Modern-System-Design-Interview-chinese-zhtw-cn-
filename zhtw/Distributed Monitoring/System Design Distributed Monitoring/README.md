# System Design: Distributed Monitoring (系統設計：分佈式監控)

## Monitoring (監控)
現代經濟依賴於 IT 基礎設施的持續運行。此類基礎設施包含硬體、分佈式服務和網絡資源。這些組件在此類基礎設施中相互關聯，使得保持一切順利運行且沒有應用程式停機時間變得具有挑戰性。

當我們的基礎設施分佈在多個位置並且包括許多伺服器時，很難知道硬體或應用程式級別發生了什麼。組件可能會遇到故障、響應延遲超調、硬體過載或不可達，以及容器資源耗盡等問題。此類基礎設施中運行著多個服務，任何事情都可能出錯。

當其中一項服務宕機時，這可能是其他服務崩潰的原因，結果是使用者無法使用該應用程式。如果我們不及早知道出了什麼問題，我們可能需要花費大量時間和精力手動調試系統。此外，對於大型服務，我們需要確保我們的服務在我們商定的服務級別協議 (Service-Level Agreements) 內工作。我們需要捕捉重要趨勢和即將發生的故障信號作為早期預警，以便解決任何擔憂或問題。

監控有助於分析這種不斷發生故障的複雜基礎設施。監控分佈式系統需要收集、解釋和顯示有關同時運行的進程之間交互的資料。它有助於調試、測試、性能評估以及鳥瞰多個服務。

## How will we design a distributed monitoring system? (我們將如何設計分佈式監控系統？)
我們將分佈式監控系統設計分為以下章節和課程：

1. **Distributed Monitoring (分佈式監控)**
- **Introduction to Distributed Monitoring (分佈式監控介紹)**：了解為什麼分佈式系統中的監控至關重要、停機成本有多高以及監控的類型。
- **Prerequisites for a Monitoring System (監控系統的先決條件)**：探索有關監控系統中指標和警報的一些基本概念。

2. **Monitoring Server-side Errors (監控伺服器端錯誤)**
- **Designing a Monitoring System (設計監控系統)**：定義監控系統的需求和高級設計。
- **A Detailed Design of the Monitoring System (監控系統的詳細設計)**：深入探討設計監控系統的細節，並探索涉及的組件。
- **Visualize Data in a Monitoring System (在監控系統中可視化資料)**：學習一種可視化大量監控資料的獨特方法。

3. **Monitor Client-side Errors (監控客戶端錯誤)**
- **Focus on Client-side Errors (關注客戶端錯誤)**：了解客戶端錯誤以及為什麼監控它們很重要。
- **Design a Client-side Monitoring System (設計客戶端監控系統)**：學習設計一個監控客戶端錯誤的系統。

在下一課中，我們將通過一個例子來看看為什麼監控在分佈式系統中至關重要。我們還將查看故障的停機成本和監控類型。
