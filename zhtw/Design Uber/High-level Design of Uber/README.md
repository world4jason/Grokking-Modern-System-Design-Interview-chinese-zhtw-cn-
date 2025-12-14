# High-level Design of Uber (Uber 的高層設計)
## Workflow of our application (我們應用程式的工作流程)
在深入研究設計之前，讓我們了解一下我們的應用程式是如何工作的。以下步驟顯示了我們應用程式的工作流程：

1. 當乘客啟動我們的應用程式時，可以看到除了已經提供服務的司機之外的所有附近司機。

2. 乘客輸入下車地點並請求乘車。

3. 應用程式接收請求並找到合適的司機。

4. 在找到匹配的司機之前，狀態將為“Waiting for the driver to respond (等待司機響應)”。

5. 司機每四秒報告一次位置。應用程式查找行程資訊並將其返回給司機。

6. 司機接受或拒絕請求：

  - 司機接受請求，並且在乘客和司機的應用程式上修改狀態資訊。乘客發現他們已成功匹配並獲得司機的資訊。

  - 司機拒絕乘車請求。乘客從步驟 2 重新開始並重新匹配到另一位司機。
  
[乘車](./uber)

## High-level design of Uber (Uber 的高層設計)
在高層次上，我們的系統應該能夠接受來自乘客的乘車請求，並將匹配的司機資訊和行程資訊返回給乘客。它還定期獲取司機的位置。此外，當司機與乘客匹配時，它將行程和乘客資訊返回給司機。

[高層設計](./hld.jpg)

## API design (API 設計)
讓我們根據我們提供的功能討論 API 的設計。我們將設計 API 以將我們的功能集轉化為技術規範。

我們不會在以下 API 中重複重複參數的描述。

### Update driver location (更新司機位置)
```
updateDriverLocation(driverID, oldlat, oldlong, newlat, newlong )
```

Parameter (參數)       Description (描述)

driverID               The ID of the driver (司機的 ID)

oldlat                 The previous latitude of the driver (司機以前的緯度)

oldlong                The previous longitude of the driver (司機以前的經度)

newlat                 The new latitude of the driver (司機的新緯度)

newlong                The new longitude of the driver (司機的新經度)
```

updateDriverLocation API 用於將司機的坐標發送到司機位置伺服器。這是更新司機位置並將其傳達給乘客的地方。


### Find nearby drivers (尋找附近的司機)
```
findNearbyDrivers(riderID, lat, long)
```

Parameter (參數)    Description (描述)

riderID             The ID of the rider (乘客的 ID)

lat                 The latitude of the rider (乘客的緯度)

long                The longitude of the rider (乘客的經度)
findNearbyDrivers API 用於發送我們要查找附近司機的乘客的位置。

### Request a ride (請求乘車)
```
requestRide(riderID, lat, long, dropOfflat,dropOfflong, typeOfVehicle)
```

Parameter (參數)     Description (描述)

lat                  The current latitude of the rider (乘客的當前緯度)

long                 The current longitude of the rider (乘客的當前經度)

dropOfflat           The latitude of the rider’s drop-off location (乘客下車地點的緯度)

dropOfflong          The longitude of the rider’s drop-off location (乘客下車地點的經度)

typeOfVehicle        The type of vehicle required by the rider—for example, business, economy, and so on. (乘客所需的車輛類型——例如商務、經濟等。)
requestRide API 用於發送乘客的位置和乘客所需的車輛類型。

### Show driver ETA (顯示司機 ETA)
```
showETA(driverID, eta)
```

Parameter (參數)     Description (描述)

eta                  The estimated time of arrival of the driver (司機的預計到達時間)
showEta API 用於向乘客顯示預計到達時間。


### Confirm pickup (確認接載)
```
confirmPickup(driverID, riderID, timestamp)
```

Parameter (參數)   Description (描述)

timestamp          The time at which the driver picked up the rider (司機接載乘客的時間)
confirmPickup API 用於確定司機何時接載乘客。

### Show trip updates (顯示行程更新)
```
showTripUpdates(tripID, riderID, driverID, driverlat, driverlong, time_elapsed, time_remaining)
```

Parameter (參數)      Description (描述)

tripID                The ID of the trip (行程的 ID)

driverlat             The latitude of the driver (司機的緯度)

driverlong            The longitude of the driver (司機的經度)

time_elapsed          The total time of the trip (行程的總時間)

time_remaining        The time remaining (extract the current time from the ETA) to reach the destination (到達目的地的剩餘時間 (從 ETA 中提取當前時間))
showTripUpdates API 用於顯示行程的更新，包括司機的位置和到達目的地的剩餘時間。

### End the trip (結束行程)
```
endTrip(tripID, riderID, driverID ,time_elapsed, lat, long)
endTrip API 用於結束行程。
