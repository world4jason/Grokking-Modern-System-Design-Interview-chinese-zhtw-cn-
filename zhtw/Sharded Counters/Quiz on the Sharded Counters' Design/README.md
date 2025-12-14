# Quiz on the Sharded Counters' Design (分片計數器設計測驗)

1 如果我們的系統為擁有 100 萬粉絲的使用者的帖子創建了一個只有少量分片的計數器，會發生什麼問題？

A) High read contention (高讀取爭用)
B) High write contention (高寫入爭用)
C) Users face delays on a read request (使用者在讀取請求上面臨延遲)
D) Users get a quick response on the liked post (使用者對點讚的帖子得到快速響應)

2 (選擇所有適用的選項。) 如果按順序 (順序) 而不是隨機選擇分片，可能會出現什麼問題？

A) The write request queue increases. (寫入請求隊列增加。)
B) Maximum shard utilization decreases. (最大分片利用率降低。)
C) One-by-one selection of shards will take more time. (逐個選擇分片將花費更多時間。)
D) The user will not get quick responses. (使用者將無法獲得快速響應。)

3 (選擇所有適用的選項。) 哪種情況需要更大量的分片計數器來處理 YouTube 上的流量？

A) A video posted by a channel with millions of subscribers (擁有數百萬訂閱者的頻道發布的視頻)
B) A video with a few likes over a long period of time (長時間內點贊數很少的視頻)
C) A video with millions of views but a few dislikes (擁有數百萬次觀看但很少不喜歡的視頻)
D) Long videos posted by a channel with a few subscribers (擁有少量訂閱者的頻道發布的長視頻)

Answers (答案)
1. (B) 
2. (A) (B) (D)
3. (A) (C)
