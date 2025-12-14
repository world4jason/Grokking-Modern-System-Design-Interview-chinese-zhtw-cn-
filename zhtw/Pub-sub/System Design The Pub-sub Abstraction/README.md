# System Design: The Pub-sub Abstraction (系統設計：Pub-sub 抽象)

## What is a pub-sub system? (什麼是 pub-sub 系統？)
發布-訂閱消息傳遞，通常稱為 pub-sub 消息傳遞，是一種異步服務到服務通信方法，在無伺服器和微服務架構中很流行。使用 pub-sub 系統可以將消息異步發送到系統的不同子系統。所有訂閱 pub-sub 模型的服務都會收到推送到系統中的消息。

例如，當著名運動員 Cristiano Ronaldo 在 Instagram 上發帖或分享推文時，他的所有追隨者都會收到更新。在這裡，Cristiano Ronaldo 是發布者，他的帖子或推文是消息，他的所有追隨者都是訂閱者。

[Pub-sub 系統](./Pub-sub-sys.jpg)

## Motivation (動機)
分佈式系統的硬件基礎設施由數百萬台機器組成。使用 pub-sub 系統進行異步通信可提高可擴展性。生產者和消費者是斷開連接並獨立運行的，從而允許我們單獨擴展和開發它們。組件、生產者和消費者之間的解耦允許更大的可擴展性，因為添加或刪除任何組件都不會影響其他組件。

## How do we design a pub-sub system? (我們如何設計 pub-sub 系統？)
我們將 pub-sub 系統設計分為以下幾課：

1. **Introduction (簡介)**: In this lesson, we learn about the use cases of the pub-sub system, define its requirements, and design the API for it.
1. **簡介**：在本課中，我們將了解 pub-sub 系統的用例，定義其需求，並為其設計 API。
2. **Design (設計)**: In this lesson, we discuss two designs of the pub-sub system, one with messaging queues and the other with a broker.
2. **設計**：在本課中，我們討論 pub-sub 系統的兩種設計，一種使用消息隊列，另一種使用代理。
