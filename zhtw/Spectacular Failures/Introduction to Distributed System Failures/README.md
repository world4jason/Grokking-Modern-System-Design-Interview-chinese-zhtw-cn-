# Introduction to Distributed System Failures (分佈式系統故障簡介)

## Introduction (簡介)
我們偶爾會遇到家喻戶曉的服務發生故障，個人和企業都會對此做出反應。作為系統設計人員，我們可能會想知道，由經驗豐富的團隊多年完善的精心設計的服務怎麼也會失敗。

本章討論知名服務的一些主要故障以及可以採取的緩解此類故障的措施。

以下兩個因素導致故障：

- **Diverse users (多樣化的使用者)**: Most services have a vibrant user community, and as their needs evolve, so do the software products. If a software doesn’t update in the way it provides new features and services, it will become stable over time. However, it might not have the features customers want.
- **多樣化的使用者**：大多數服務都有一個充滿活力的使用者社區，隨著他們需求的發展，軟體產品也在發展。如果軟體不更新其提供新功能和服務的方式，它會隨著時間的推移變得穩定。但是，它可能沒有客戶想要的功能。

- **Complex systems (複雜系統)**: Systems are complex, and they usually have emergent properties where the sum of system components is more complex than the individual pieces.
- **複雜系統**：系統很複雜，它們通常具有湧現特性，即系統組件的總和比各個部分更複雜。

## Types of failure in distributed systems (分佈式系統中的故障類型)
大多數現代服務的設計方式都是為了遏制故障，而其他故障可能只針對某些使用者。讓我們探索我們可以在分佈式系統中觀察到的故障類型。

- **System failure (系統故障)**: A software or hardware failure is the most common cause of system failure. The contents in the primary memory are lost when a system fails. However, the data in secondary storage or replicas remains unaffected. The system reboots during such a breakdown.
- **系統故障**：軟體或硬體故障是系統故障的最常見原因。當系統出現故障時，主存儲器中的內容會丟失。但是，二級存儲或副本中的資料不受影響。系統在此類故障期間會重新引導。

- **Method failure (方法故障)**: Such failures suspend the working of distributed systems. It may also make the system execute the processes incorrectly or enter a deadlock state.
- **方法故障**：此類故障會暫停分佈式系統的工作。它還可能使系統錯誤地執行進程或進入死鎖狀態。

- **Communication medium failure (通信介質故障)**: Such failures occur when one component or service of a system can’t reach the other internal or external entities.
- **通信介質故障**：當系統的一個組件或服務無法到達其他內部或外部實體時，就會發生此類故障。

- **Secondary storage failure (二級存儲故障)**: In such failures, the secondary storage or replicas are down. The data in these nodes becomes inaccessible, so primary nodes need to generate another replica to ensure reliability and fault tolerance.
- **二級存儲故障**：在此類故障中，二級存儲或副本已關閉。這些節點中的資料變得無法訪問，因此主節點需要生成另一個副本以確保可靠性和容錯性。

## Vantage points (有利位置)
大型服務總是會出現故障。最好進行優雅降級，以便只有一小部分使用者在短時間內受到影響。因此，我們需要全球分散的有利位置來獨立查看服務狀態。
```
注意：許多獨立服務 (例如 Downdetector) 用於眾包問題報告。有趣的是，如果我們訪問此類服務並查看我們喜歡的應用程式的狀態，我們會發現世界上總有人面臨某些服務問題。
```

[分佈式系統中的故障類型](./types.jpg)

### Importance of independent service providers (獨立服務提供商的重要性)
最初互聯網的設計目標之一是提供彈性，以便如果一部分出現故障，其餘部分仍可運行。

隨著過去十年少數服務提供商的出現，批評者對這種集中化以及故障的潛在影響表示擔憂。請參閱 Bruce Davie 的“是時候再次分散互聯網了：分佈式的東西現在由 Google、Facebook 等集中”。大多數公司提供某種儀表板，讓使用者查看服務狀態。但是，某些故障甚至可能使這些儀表板癱瘓。受影響的公司隨後通過 Twitter 等服務與客戶溝通以發布更新。獨立的第三方服務對於故障檢測和狀態傳播非常有價值。
```
注意：這裡更廣泛的概念是使用獨立的故障域。故障域是一個概念，其中在域或網絡內發生的任何故障都不應影響其他域中的其他組件和服務。有時，如果兩個域在彼此的爆炸半徑之外，我們會說這兩個域是獨立的。
```

[獨立服務的重要性](./independence.jpg)

在接下來的課程中，我們將討論巨頭公司知名服務的故障、故障原因以及可以用來避免這些故障的緩解技術。雖然故障是了解出了什麼問題以及原始設計者可以採取什麼措施來避免此類故障的好方法，但我們希望完全防止它們發生。
