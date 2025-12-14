# System Design: Web Crawler (系統設計：網絡爬蟲)

## Introduction (簡介)
網絡爬蟲是一種互聯網機器人，它從種子 URL 池開始運行，系統地搜索萬維網 (WWW) 以獲取內容。從 WWW 獲取內容的這個過程稱為爬取。它進一步將爬取的內容保存在資料存儲中。為後續使用有效保存資料的過程稱為存儲。

```
迅速去或移動，越過或穿過以尋找某物。
```

```
存儲的 URL 作為爬蟲的起點。
```
這是搜索引擎執行的第一步；存儲的資料用於索引和排名目的。此特定設計問題僅限於網絡爬蟲，不包括搜索引擎中後續索引和排名階段的解釋。

[網絡爬蟲系統概覽](./overview.jpg)

### Additional benefits (額外好處)
網絡爬蟲的其他實用程式如下：

- **Web pages testing (網頁測試)**: We use web crawlers to check the validity of the links and structures of web pages.
- **網頁測試**：我們使用網絡爬蟲來檢查網頁鏈接和結構的有效性。

- **Web page monitoring (網頁監控)**: We use web crawlers to monitor the content or structure updates on web pages.
- **網頁監控**：我們使用網絡爬蟲來監控網頁上的內容或結構更新。

- **Site mirroring (站點鏡像)**: Web crawlers are an effective way to mirror popular websites.
- **站點鏡像**：網絡爬蟲是鏡像熱門網站的有效方式。

- **Copyright infringement check (侵犯版權檢查)**: Web crawlers fetch content and check for copyright infringement issues.
- **侵犯版權檢查**：網絡爬蟲獲取內容並檢查侵犯版權問題。

在本章中，我們將設計一個網絡爬蟲並評估它如何滿足功能和非功能需求。

爬取過程的輸出是後續處理階段 (資料清洗、索引、使用頁面排名等算法的頁面相關性以及分析) 的輸入資料。要了解其中一些後續階段，請參閱我們關於分佈式搜索的章節。

```
鏡像是製作網站的動態複寫副本。鏡像是指通過任何協議 (例如 HTTP 或 FTP) 可用的網絡服務。這些站點的 URL 與原始站點不同，但內容相似或幾乎相同。
## How will we design a Web crawler? (我們將如何設計網絡爬蟲？)
本章由四個課程組成，涵蓋了網絡爬蟲系統的整體設計：

1. **Requirements (需求)**: This lesson enlists the functional and non-functional requirements of the system and estimates calculations for various system parameters.
1. **需求**：本課列出了系統的功能和非功能需求，並估計了各種系統參數的計算。
2. **Design (設計)**: This lesson analyzes a bottom-up approach for a web-crawling service. We get a detailed overview of all the individual components leading to a combined operational mechanism to meet the requirements.
2. **設計**：本課分析了網絡爬蟲服務的自下而上的方法。我們詳細了解所有各個組件，從而形成滿足要求的組合操作機制。
3. **Improvements (改進)**: This lesson provides all the design improvements required to counter shortcomings, especially the crawler traps. These crawler traps include links with query parameters, internal links redirection, links holding infinite calendar pages, links for dynamic content generation, and links containing cyclic directories.
3. **改進**：本課提供了應對缺點 (尤其是爬蟲陷阱) 所需的所有設計改進。這些爬蟲陷阱包括帶有查詢參數的鏈接、內部鏈接重定向、包含無限日曆頁面的鏈接、用於動態內容生成的鏈接以及包含循環目錄的鏈接。
4. **Evaluation (評估)**: This lesson provides an in-depth evaluation of our design choices to check if they meet all the standards and requirements we expect from our design.
4. **評估**：本課對我們的設計選擇進行深入評估，以檢查它們是否符合我們期望的所有標準和要求。

讓我們從定義網絡爬蟲的需求開始。
