# System Design: A Blob Store (系統設計：Blob 儲存)

## What is a blob store? (什麼是 Blob 儲存？)
Blob 儲存是一種用於非結構化資料的儲存解決方案。我們可以在 Blob 儲存中儲存相片、音訊、影片、二進位可執行程式碼或其他多媒體項目。每種類型的資料都儲存為 Blob。它遵循一種扁平的資料組織模式，沒有層級結構，即沒有目錄、子目錄等。

大多數情況下，它被具有特定業務需求的應用程式使用，稱為單寫多讀 (WORM - write once, read many)，該需求規定資料只能寫入一次，並且沒人能更改它。就像在 Microsoft Azure 中一樣，Blob 被建立一次並被讀取多次。此外，這些 Blob 在指定的間隔之前無法刪除，也不能修改，以保護關鍵資料。

Blob (二進位大型物件) 由作為單個單元儲存的二進位資料集合組成。
```

注意：並非所有應用程式都必須具有此 WORM 需求。然而，我們假設寫入的 Blob 無法修改。如果需要，我們可以上傳 Blob 的新版本來代替修改。
```

## Why do we use a blob store? (為什麼我們使用 Blob 儲存？)
Blob 儲存是許多資料密集型應用程式的重要組件，例如 YouTube、Netflix、Facebook 等。下表顯示了一些最著名的應用程式使用的 Blob 儲存。這些應用程式每天產生大量的非結構化資料。它們需要一個易於擴展、可靠且高可用的儲存解決方案，以便它們可以儲存大型媒體檔案。由於資料量不斷增加，這些應用程式需要儲存無限數量的 Blob。根據一些估計，YouTube 每天需要超過一個 PB 的額外儲存空間。在像 YouTube 這樣的系統中，影片以多種解析度儲存。此外，所有解析度的影片在不同的資料中心和區域被複製多次以用於可用性目的。這就是為什麼每個影片所需的總儲存空間不等於上傳影片的大小。

System (系統)         Blob Store (Blob 儲存)

Netflix              S3

YouTube              Google Cloud Storage

Facebook             Tectonic
```

## How do we design a blob store system? (我們如何設計 Blob 儲存系統？)
我們將 Blob 儲存的設計分為五課和一個測驗。

1. **Requirements (需求)**：在本課中，我們識別 Blob 儲存的功能和非功能需求。我們還估算我們的 Blob 儲存系統所需的資源。
2. **Design (設計)**：本課向我們介紹 Blob 儲存的高階設計、API 設計和詳細設計，同時解釋所有組件和工作流程的細節。
3. **Design considerations (設計考量)**：在本課中，我們討論設計的一些重要方面。例如，我們學習資料庫模式、分區策略、Blob 索引、分頁和複製。
4. **Evaluation (評估)**：在本課中，我們根據我們的需求評估我們的 Blob 儲存。
5. **Quiz (測驗)**：在本課中，我們評估對 Blob 儲存設計的理解。

讓我們從 Blob 儲存系統的需求開始。
