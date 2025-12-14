# High-level Design of WhatsApp (WhatsApp 的高層設計)
## High-level design (高層設計)
在抽象層面上，高層設計由負責發送者和接收者之間通訊的聊天伺服器組成。當一個使用者想向另一個使用者發送訊息時，他們都連接到聊天伺服器。兩個使用者都將他們的訊息發送到聊天伺服器。然後聊天伺服器將訊息發送給另一個預期使用者，並將訊息儲存在資料庫中。

[WhatsApp messenger 的高層設計](./design.jpg)

以下步驟描述了兩個客戶端之間的通訊：

1. User A (使用者 A) 和 User B (使用者 B) 與聊天伺服器建立通訊通道。
2. User A 向聊天伺服器發送訊息。
3. 收到訊息後，聊天伺服器向 User A 確認。
4. 如果接收者的狀態為離線，聊天伺服器將訊息發送給 User B 並將訊息儲存在資料庫中。
5. User B 向聊天伺服器發送確認。
6. 聊天伺服器通知 User A 訊息已成功送達。
7. 當 User B 閱讀訊息時，應用程式通知聊天伺服器。
8. 聊天伺服器通知 User A，User B 已閱讀訊息。

該過程如下圖所示：

[過程](./process)

## API design (API 設計)
WhatsApp 通過不同的 API 為使用者提供了大量功能。下面提到了一些功能：

- 發送訊息
- 獲取訊息或接收訊息
- 上傳媒體文件或文件
- 下載文件或媒體文件
- 發送位置
- 發送聯繫人
- 創建狀態
但是，我們將討論與前四個功能相關的基本 API。

### Send message (發送訊息)
sendMessage API 如下：
```
sendMessage(sender_ID, reciever_ID, type, text=none, media_object=none, document=none)
此 API 用於通過向 /messages API 端點發出 POST API 調用，將文本訊息從發送者發送到接收者。通常，發送者和接收者的 ID 是他們的電話號碼。此 API 調用中使用的參數在下表中描述：
Parameter (參數)        Description (描述)

sender_ID             This is a unique identifier of the user who sends the message. (這是發送訊息的使用者的唯一標識符。)

reciever_ID           This is a unique identifier of the user who receives the message. (這是接收訊息的使用者的唯一標識符。)

type                  The default message type is text. This represents whether the sender sends a media file or a document. (預設訊息類型為文本。這表示發送者是否發送媒體文件或文件。)

text                  This feild contains the text that has to be sent as a message. (此欄位包含必須作為訊息發送的文本。)

media_object          This parameter is defined based on the type parameter. It represents the media file to be sent. (此參數根據 type 參數定義。它表示要發送的媒體文件。)

document              This represents the document file to be sent. (這表示要發送的文件檔案。)
```

### Get message (獲取訊息)
getMessage API 如下：
```
getMessage(user_Id)
使用此 API 調用，使用者可以在離線一段時間後上線時獲取所有未讀訊息。
Parameter (參數)       Description (描述)

user_Id                This is a unique identifier representing the user who has to fetch all unread messages. (這是代表必須獲取所有未讀訊息的使用者的唯一標識符。)
```

### Upload media or document file (上傳媒體或文件檔案)
uploadFile API 如下：
```
uploadFile(file_type, file)
我們可以通過向 /v1/media API 端點發出 POST 請求，通過 uploadFile API 上傳媒體文件。成功響應返回一個轉發給接收者的 ID。可以上傳的媒體文件的最大大小為 16 MB，而文件的限制為 100 MB。
Parameter (參數)     Description (描述)

file_type            This represents the type of file uploaded via the API call. (這表示通過 API 調用上傳的文件類型。)

file                 This contains the file being uploaded via the API call. (這包含通過 API 調用上傳的文件。)
```

### Download a document or media file (下載文件或媒體檔案)
downloadFile API 如下：
```
downloadFile(user_id, file_id)
此 API 調用的參數在下表中解釋：
Parameter (參數)     Description (描述)

user_id              This is a unique identifier of the user who will download a file. (這是將下載文件的使用者的唯一標識符。)

file_id              This is a unique identifier of a file. It’s generated while uploading a file via uploadFile() API call. The downloadFile() API call downloads the media file through this identifier. The client can find the file_id by providing the file name to the server. That API call is not shown here. (這是一個文件的唯一標識符。它是在通過 uploadFile() API 調用上傳文件時生成的。downloadFile() API 調用通過此標識符下載媒體文件。客戶端可以通過向伺服器提供檔名來找到 file_id。此處未顯示該 API 調用。)
```

在下一課中，我們將重點介紹 WhatsApp 系統的詳細設計。
