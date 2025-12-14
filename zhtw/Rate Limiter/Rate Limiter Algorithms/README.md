# Rate Limiter Algorithms (速率限制器算法)

## Algorithms for rate limiting (速率限制算法)
速率限制器的任務由高效的算法指導，每種算法都有明顯的優點和缺點。但是，始終可以根據我們在給定時間的需求選擇算法或算法組合。雖然除了下面的算法之外還使用了不同的算法，但我們將看一看以下流行的算法。

- **Token bucket (令牌桶)**
- **Leaking bucket (漏桶)**
- **Fixed window counter (固定窗口計數器)**
- **Sliding window log (滑動窗口日誌)**
- **Sliding window counter (滑動窗口計數器)**

### Token bucket algorithm (令牌桶算法)
該算法使用具有預定義令牌容量的桶的類比。桶以恆定速率定期填充令牌。令牌可以被認為是具有特定大小的資料包。因此，該算法在我們每次收到請求時都會檢查桶中的令牌。至少應該有一個令牌來進一步處理請求。

令牌桶算法的流程如下：

假設我們有一個預定義的速率限制 R，桶的總容量為 C

1. 算法每 1/R 秒向桶中添加一個新令牌。
2. 當桶中的令牌數量等於桶的總容量 C 時，算法會丟棄新傳入的令牌。
3. 如果有 N 個傳入請求並且桶中至少有 N 個令牌，則消耗令牌，並將請求轉發以進行進一步處理。
4. 如果有 N 個傳入請求並且桶中的令牌數量較少，則接受的請求數量等於桶中可用令牌的數量。

下圖表示令牌桶算法的工作原理。

[令牌桶算法如何工作](./token_bucket.jpg)

下圖演示了令牌消耗和速率限制邏輯的工作原理。在此示例中，令牌桶的容量為 3，並且以每分鐘 3 個令牌的速率重新填充。

[令牌桶](./token_bucket)

#### Essential parameters (基本參數)
我們需要以下基本參數來實現令牌桶算法：

- **Bucket capacity(C) (桶容量)**: The maximum number of tokens that can reside in the bucket.
- **桶容量(C)**：桶中可以駐留的最大令牌數。
- **Rate limit (R) (速率限制)**: The number of requests we want to limit per unit time.
- **速率限制 (R)**：我們希望每單位時間限制的請求數。
- **Refill rate (1/R) (填充率)**: The number of tokens put into the bucket per unit time.
- **填充率 (1/R)**：每單位時間放入桶中的令牌數。
- **Requests count (N) (請求計數)**: This parameter tracks the number of incoming requests and compares them with the bucket’s capacity.
- **請求計數 (N)**：此參數跟踪傳入請求的數量並將其與桶的容量進行比較。

#### Advantages (優點)
- 只要桶中有足夠的令牌，該算法就可以導致突發流量。
- 它具有空間效率。由於狀態有限，算法所需的內存很少。

#### Disadvantages (缺點)
- 從實現的角度來看，鎖可能需要從桶中獲取令牌，如果鎖上的爭用增加，這可能會增加請求的處理延遲。
- 為基本參數選擇最佳值是一項艱鉅的任務。

### The leaking bucket algorithm (漏桶算法)
漏桶算法是令牌桶算法的變體，稍作修改。漏桶算法不使用令牌，而是使用桶來包含傳入的請求並以恆定的傳出速率處理它們。該算法使用以恆定速率洩漏的水桶的類比。同樣，在該算法中，請求以可變速率到達。該算法以先進先出 (FIFO) 的順序以恆定速率處理這些請求。

讓我們看下圖中漏桶算法是如何工作的：

#### Essential parameters (基本參數)
漏桶算法需要以下參數。

- **Bucket capacity (C) (桶容量)**: This determines the maximum capacity of the bucket. The algorithm will discard the incoming requests when the bucket reached its maximum limit of C.
- **桶容量 (C)**：這決定了桶的最大容量。當桶達到其最大限制 C 時，算法將丟棄傳入的請求。
- **Inflow rate (R_{in}) (流入率)**: This parameter shows the inflow rate of requests. This is a varying quantity that depends on the application and nature of requests. We use this parameter to find the initial capacity of the bucket.
- **流入率 (R_{in})**：此參數顯示請求的流入率。這是一個變化的量，取決於應用程式和請求的性質。我們使用此參數來查找桶的初始容量。
- **Outflow rate (R_{out}) (流出率)**: This determines the number of requests processed per unit time.
- **流出率 (R_{out})**：這決定了每單位時間處理的請求數。

#### Advantages (優點)
- 由於恆定的流出率 (R_{out})，它避免了請求的爆發，這與令牌桶算法不同。
- 該算法也具有空間效率，因為它只需要三個狀態：流入率 (R_{in})、流出率 (R_{out}) 和桶容量 (C)。
- 由於請求以固定速率處理，因此適用於具有穩定流出率的應用程式。

#### Disadvantages (缺點)
- 請求的爆發可能會填滿桶，如果在指定時間內未處理，最近的請求可能會受到影響。
- 確定最佳桶大小和流出率是一個挑戰。

### Fixed window counter algorithm (固定窗口計數器算法)
該算法將時間劃分為稱為窗口的固定間隔，並為每個窗口分配一個計數器。當特定窗口收到請求時，計數器加一。一旦計數器達到其限制，就會在該窗口中丟棄新請求。

如下圖所示，虛線表示每個窗口中的限制。如果計數器低於限制，則轉發請求；否則，丟棄請求。

[固定窗口計數器算法：丟棄超過限制的請求](./fixed_window.jpg)

該算法有一個重大問題。在窗口的邊緣可能會出現大於允許請求的流量突發。在下圖中，系統每分鐘最多允許 10 個請求。但是，從 01:30 到 02:30 的一分鐘窗口中的請求數為 20，大於允許的請求數。

[固定窗口計數器算法中的邊緣情況問題。從 01:30 到 02:30 的一分鐘內的請求數超過了每分鐘 10 個請求的預定義限制](./edge_cases.jpg)

#### Essential parameters (基本參數)
固定窗口計數器算法需要以下參數：

- **Window size (W) (窗口大小)**: It represents the size of the time window. It can be a minute, an hour, or any other suitable time slice.
- **窗口大小 (W)**：它表示時間窗口的大小。可以是一分鐘，一小時或任何其他合適的時間片。
- **Rate limit (R) (速率限制)**: It shows the number of requests allowed per time window.
- **速率限制 (R)**：它顯示每個時間窗口允許的請求數。
- **Requests count (N) (請求計數)**: This parameter shows the number of incoming requests per window. The incoming requests are allowed if N is less than or equal to R.
- **請求計數 (N)**：此參數顯示每個窗口的傳入請求數。如果 N 小於或等於 R，則允許傳入請求。

#### Advantages (優點)
- 由於對請求速率的限制，它也具空間效率。
- 與令牌桶式算法 (如果沒有足夠的令牌則丟棄新請求) 相比，該算法為新請求提供服務。

#### Disadvantages (缺點)
- 在窗口邊緣持續的流量突發 (允許的每個窗口請求數的兩倍) 可能會導致性能潛在下降。

### Sliding window log algorithm (滑動窗口日誌算法)
滑動窗口日誌算法跟踪每個傳入的請求。當請求到達時，其到達時間存儲在哈希映射中，通常稱為日誌。日誌根據傳入請求的時間戳進行排序。根據日誌的大小和到達時間允許請求。

該算法的主要優點是，與固定窗口計數器算法相比，它不受邊緣條件的影響。

讓我們通過下圖了解滑動窗口日誌算法的工作原理。假設我們每分鐘最多限制兩個請求。

[滑動窗口日誌算法](./sliding_window)


#### Essential parameters (基本參數)
實現滑動窗口日誌算法需要以下參數：

- **Log size (L) (日誌大小)**: This parameter is similar to the rate limit (R) as it determines the number of requests allowed in a specific time frame.
- **日誌大小 (L)**：此參數類似於速率限制 (R)，因為它決定了特定時間範圍內允許的請求數。
- **Arrival time (T) (到達時間)**: This parameter tracks incoming requests’ time stamps and determines their count.
- **到達時間 (T)**：此參數跟踪傳入請求的時間戳並確定其計數。
- **Time range (T_{r}) (時間範圍)**: This parameter determines the time frame. The time stamps of the old requests are deleted if they do not fall in this range. The start time of the window is defined based on the first incoming request and expires after one minute. Similarly, when another request after the expiry time arrives the window ranges are updated accordingly.
- **時間範圍 (T_{r})**：此參數確定時間範圍。如果舊請求的時間戳不在此範圍內，則將其刪除。窗口的開始時間基於第一個傳入請求定義，並在一分鐘後過期。同樣，當過期時間之後的另一個請求到達時，窗口範圍會相應更新。


#### Advantages (優點)
- 該算法不受固定窗口邊界條件的影響。

#### Disadvantages (缺點)
- 它消耗額外的內存來存儲附加資訊，即傳入請求的時間戳。它保留時間戳以提供動態窗口，即使請求被拒絕也是如此。

### Sliding window counter algorithm (滑動窗口計數器算法)
與以前的固定窗口算法不同，滑動窗口計數器算法不基於固定時間單位限制請求。該算法同時考慮了固定窗口計數器和滑動窗口日誌算法，使請求流更加平滑。讓我們看看下圖中算法的流程。

[滑動窗口計數器算法，其中綠色陰影區域顯示 1 分鐘的滾動窗口](./sliding_window_counter.jpg)

在上圖中，我們在上一個窗口中有 88 個請求，而在當前窗口中有 12 個請求。我們將速率限制設置為每分鐘 100 個請求。此外，滾動窗口與當前窗口重疊 15 秒。現在假設一個新請求在 02:15 到達。我們將使用數學公式決定接受或拒絕哪個請求：

```
Rate = Rp x (time frame - overlap time)/time frame + Rc
```
這裡，Rp 是上一個窗口中的請求數，為 88。Rc 是當前窗口中的請求數，為 12。在我們的例子中，時間範圍是 60 秒，重疊時間是 15 秒。

Rate=88 x (60-15)/60 + 12
Rate=78 <100
由於 78 小於 100，因此允許傳入請求。


#### Essential parameters (基本參數)
該算法比上述其他算法相對更複雜。它需要以下參數：

- **Rate limit (R) (速率限制)** It determines the number of maximum requests allowed per window.
- **速率限制 (R)**：它確定每個窗口允許的最大請求數。
- **Size of the window (W) (窗口大小)**: This parameter represents the size of a time window that can be a minute, an hour, or any time slice.
- **窗口大小 (W)**：此參數表示時間窗口的大小，可以是一分鐘、一小時或任何時間片。
- **The number of requests in the previous window (R_{p}) (上一個窗口中的請求數)**: It determines the total number of requests that have been received in the previous time window.
- **上一個窗口中的請求數 (R_{p})**：它確定在上一個時間窗口中接收到的請求總數。
- **The number of requests in the current window (R_{c}) (當前窗口中的請求數)**: It represents the number of requests received in the current window.
- **當前窗口中的請求數 (R_{c})**：它表示在當前窗口中接收到的請求數。
- **Overlap time (O_{t}) (重疊時間)**: This parameter shows the overlapping time of the rolling window with the current window.
- **重疊時間 (O_{t})**：此參數顯示滾動窗口與當前窗口的重疊時間。

#### Advantages (優點)
- 該算法也具有空間效率，因為狀態有限：當前窗口中的請求數、上一個窗口中的請求數、重疊百分比等。
- 它平滑了請求的爆發，並根據上一個窗口以近似平均速率處理它們。

#### Disadvantages (缺點)
- 該算法假設上一個窗口中的請求數量分佈均勻，這可能並不總是可能的。

## A comparison of rate-limiting algorithms (速率限制算法比較)
所有速率限制算法共有的兩個主要因素是：

- **Memory (內存)**: This feature refers to the number of states an algorithm requires to maintain for a normal operation. For example, if one algorithm requires fewer variables (states) than the other, it is more space efficient.
- **內存**：此功能是指算法維持正常操作所需的狀態數量。例如，如果一種算法比另一種算法需要更少的變量 (狀態)，則它的空間效率更高。
- **Burst (突發)**: This refers to an increase of traffic in a unit time exceeding the defined limit.
- **突發**：這是指單位時間內流量增加超過定義的限制。

下表顯示了本課中描述的所有算法的空間效率和流量突發。

A Comparison of Rate-limiting Algorithms (速率限制算法比較)
算法                      空間效率                                                                              允許突發？

令牌桶                    是                                                                                   是，它允許在定義的限制內突發流量。

漏桶                      是                                                                                   否

固定窗口計數器            是                                                                                   是，它允許在時間窗口邊緣突發，並且可能超過定義的限制。

滑動窗口日誌              否，維護日誌需要額外的存儲空間。                                                       否

滑動窗口計數器            是，但它比其他空間效率高的算法需要相對更多的空間。                                       平滑突發
```

## Conclusion (結論)

在本課中，我們探討了各種流行的速率限制算法。我們還闡明了這些算法的優缺點。每種算法都可以根據使用者選擇和用例類型進行部署。
