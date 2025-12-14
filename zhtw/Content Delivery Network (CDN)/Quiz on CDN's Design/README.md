# Quiz on CDN's Design (CDN 設計測驗)
```
當大多數 Web 內容是靜態時，哪種 CDN 方法最適合使用？

Pull CDN (拉取 CDN)

Push CDN (推送 CDN)
```

```
(選擇所有適用的選項。) 使用公共 CDN 的局限性是什麼？

A)
額外的故障點

B)
資料洩露的風險

C)
失去控制權

D)
額外的 DNS 查詢

E)
增加延遲
```

```
(選擇所有適用的選項。) 建立專用 CDN 的主要原因是什麼？

A)
它優化了內容傳遞。

B)
它有助於我們避免 CDN 提供者成本的增加。

C)
它防止 ISP 干預。

D)
它降低了命中率。
```

```
假設您被要求擴展 Quora 系統以減少源伺服器上的資料分發負擔。您將使用哪種技術來做到這一點？

A)
單層 CDN

B)
多層 CDN

C)
公共 CDN

D)
多種 CDN
```

```
(選擇所有適用的選項。) 為什麼大多數網站使用 CDN？

A)
CDN 確保改進的可擴展性。

B)
CDN 提供針對 DDoS 攻擊的保護。

C)
CDN 為所有被驅逐的內容建立備份。

D)
CDN 減少了源伺服器上的負載。

E)
CDN 保證更好的效能。
```

```
哪對指標最適合尋找最近的代理伺服器？

A)
流量負載，快取容量

B)
請求負載，網路距離

C)
位置，頻寬

D)
快取容量，網路距離
```

Answers (答案)
```
當大多數 Web 內容是靜態時，哪種 CDN 方法最適合使用？

Pull CDN (拉取 CDN)

Explanation
解釋
在 Web 內容頻繁更改的地方使用拉取 CDN。

Correct Answer
Push CDN (推送 CDN)

Explanation
解釋
在 Web 內容不經常更改並在邊緣代理伺服器中快取很長時間的地方使用推送 CDN。
```

```
(選擇所有適用的選項。) 使用公共 CDN 的局限性是什麼？

A)
額外的故障點

Explanation
解釋
託管在 CDN 上的內容將不可用。

B)
資料洩露的風險

Explanation
解釋
第三方管理資料總是有風險。

C)
失去控制權

Explanation
解釋
當內容發布到代理伺服器時，CDN 提供者控制它。

D)
額外的 DNS 查詢

Explanation
解釋
CDN 網路層增加了 DNS 查詢。

E)
增加延遲

Explanation
解釋
CDN 用於減少終端使用者和內容之間的延遲。因此，此選項不能是局限性。
```

```
(選擇所有適用的選項。) 建立專用 CDN 的主要原因是什麼？

A)
它優化了內容傳遞。

Explanation
解釋
指定的 CDN 致力於優化請求內容傳遞的特定內容提供者。

B)
它有助於我們避免 CDN 提供者成本的增加。

Explanation
解釋
當我們增加快取大小或新增更多組件時，CDN 提供者的成本會增加。

C)
它防止 ISP 干預。

Explanation
解釋
每個請求都使用加密連接通過 ISP。所以，ISP 無法干預。

D)
它降低了命中率。

Explanation
解釋
專用 CDN 將增加命中率而不是降低命中率，因為它是專門針對特定內容的。
```

```
假設您被要求擴展 Quora 系統以減少源伺服器上的資料分發負擔。您將使用哪種技術來做到這一點？

A)
單層 CDN

Explanation
解釋
單層並不能減輕負擔，因為源伺服器必須將內容分發到所有 CDN 伺服器。

Correct Answer
B)
多層 CDN

Explanation
解釋
使用這種方法，資料分發被分為 CDN 層，例如從源到父 CDN 伺服器，以及從父到下一層 CDN 伺服器。

C)
公共 CDN

Explanation
解釋
源伺服器需要向 CDN 提供資料，無論是公共還是私有的。

D)
多種 CDN

Explanation
解釋
多種 CDN 將來自不同 CDN 提供者的多個 CDN 結合成一個單一網路。通常這樣的策略既複雜又昂貴。
```

```
(選擇所有適用的選項。) 為什麼大多數網站使用 CDN？

A)
CDN 確保改進的可擴展性。

Explanation
解釋
水平擴展對於 CDN 來說不是問題。

B)
CDN 提供針對 DDoS 攻擊的保護。

Explanation
解釋
清洗伺服器用於防止 CDN 網路中的 DDoS 攻擊。

C)
CDN 為所有被驅逐的內容建立備份。

Explanation
解釋
內容從指定 CDN 伺服器的所有儲存中刪除，以確保驅逐過程中的一致性。

D)
CDN 減少了源伺服器上的負載。

Explanation
解釋
CDN 減輕了源伺服器的負擔，如資料分發、處理大量流量等。

E)
CDN 保證更好的效能。

Explanation
解釋
CDN 位於終端使用者附近，因此通過減少延遲和加速請求/回應過程來提高效能。
```

```
哪對指標最適合尋找最近的代理伺服器？

A)
流量負載，快取容量

Explanation
解釋
快取容量不是相關因素，但流量負載是。

Correct Answer
B)
請求負載，網路距離

Explanation
解釋
這是尋找最近代理伺服器的最合適對。

C)
位置，頻寬

Explanation
解釋
位置不描述任何細節，但兩個位置之間的距離可以。

D)
快取容量，網路距離

Explanation
解釋
快取容量不是相關因素，但網路距離是。
```
