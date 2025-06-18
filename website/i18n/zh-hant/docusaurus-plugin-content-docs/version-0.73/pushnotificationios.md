---
id: pushnotificationios
title: '🚧 PushNotificationIOS'
---

> **已棄用。** 請改用 [社群套件](https://reactnative.directory/?search=push+notification) 中的其中一個。

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3>
  <p>The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/versions/latest/sdk/notifications/">Notifications</a> in the Expo documentation for the appropriate alternative.</p>
</div>

處理應用程式的推播通知，包括權限處理和應用程式圖示的徽章數字。

要開始使用，請先[向 Apple 設定您的通知](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/AddingCapabilities/AddingCapabilities.html#//apple_ref/doc/uid/TP40012582-CH26-SW6)以及您的伺服器端系統。

React Native 版本 0.60.0 或更高：

- 0.60.0 版本中的自動連結功能會為您處理連結！

React Native 版本低於 0.60.0：

將 PushNotificationIOS 函式庫加入您的 Podfile：./ios/Podfile

- CocoaPods：

  - 將 PushNotificationIOS 函式庫加入您的 Podfile：./ios/Podfile

    ```ruby
    target 'myAwesomeApp' do
      # Pods for myAwesomeApp
      pod 'React-RCTPushNotification', :path => '../node_modules/react-native/Libraries/PushNotificationIOS'
    end
    ```

- [手動連結](linking-libraries-ios.md#manual-linking) PushNotificationIOS 函式庫：
  - 將以下內容加入您的專案：`node_modules/react-native/Libraries/PushNotificationIOS/RCTPushNotification.xcodeproj`
  - 將以下內容加入 `Link Binary With Libraries`：`libRCTPushNotification.a`

最後，為了啟用對 `notification` 和 `register` 事件的支援，您需要擴充您的 AppDelegate。

在您的 `AppDelegate.m` 頂部：

`#import <React/RCTPushNotificationManager.h>`

然後在您的 AppDelegate 實作中加入以下內容：

```objectivec
 // Required to register for notifications
 - (void)application:(UIApplication *)application didRegisterUserNotificationSettings:(UIUserNotificationSettings *)notificationSettings
 {
  [RCTPushNotificationManager didRegisterUserNotificationSettings:notificationSettings];
 }
 // Required for the register event.
 - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
 {
  [RCTPushNotificationManager didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
 }
 // Required for the notification event. You must call the completion handler after handling the remote notification.
 - (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo
                                                        fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
 {
   [RCTPushNotificationManager didReceiveRemoteNotification:userInfo fetchCompletionHandler:completionHandler];
 }
 // Required for the registrationError event.
 - (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
 {
  [RCTPushNotificationManager didFailToRegisterForRemoteNotificationsWithError:error];
 }
 // Required for the localNotification event.
 - (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification
 {
  [RCTPushNotificationManager didReceiveLocalNotification:notification];
 }
```

要在前景顯示通知（從 iOS 10 開始可用），請加入以下幾行：

在您的 `AppDelegate.m` 頂部：

`#import <UserNotifications/UserNotifications.h>`

然後在您的 AppDelegate 實作中加入以下內容：

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  ...
  // Define UNUserNotificationCenter
  UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
  center.delegate = self;

  return YES;
}

//Called when a notification is delivered to a foreground app.
-(void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
{
  completionHandler(UNAuthorizationOptionSound | UNAuthorizationOptionAlert | UNAuthorizationOptionBadge);
}
```

接著啟用背景模式/遠端通知，以便正確使用遠端通知。最簡單的方法是透過專案設定。導航至 Targets -> 您的應用程式 -> Capabilities -> Background Modes 並勾選 Remote notifications。這將自動啟用所需的設定。

---

# 參考

## 方法

### `presentLocalNotification()`

```tsx
static presentLocalNotification(details: PresentLocalNotificationDetails);
```

安排本地通知立即顯示。

**參數：**

| Name    | Type   | Required | Description |
| ------- | ------ | -------- | ----------- |
| details | object | Yes      | See below.  |

details 是一個包含以下內容的物件：

- `alertBody` : 在通知警示中顯示的訊息。
- `alertAction` : 在可操作通知下方顯示的「動作」。預設為「view」。請注意，Apple 從 iOS 10 開始不再顯示此內容。
- `alertTitle` : 在通知警示中顯示的標題文字。
- `soundName` : 觸發通知時播放的聲音（可選）。
- `isSilent` : 如果為 true，通知將無聲顯示（可選）。
- `category` : 此通知的類別，用於可操作通知（可選）。
- `userInfo` : 包含額外通知資料的物件（可選）。
- `applicationIconBadgeNumber` : 顯示在應用程式圖示上的徽章數字。此屬性的預設值為 0，表示不顯示徽章（可選）。

---

### `scheduleLocalNotification()`

```tsx
static scheduleLocalNotification(details: ScheduleLocalNotificationDetails);
```

安排本地通知在未來顯示。

**參數：**

| Name    | Type   | Required | Description |
| ------- | ------ | -------- | ----------- |
| details | object | Yes      | See below.  |

details 是一個包含以下屬性的物件：

- `fireDate` : 系統應觸發通知的日期與時間。
- `alertTitle` : 顯示為通知警示標題的文字。
- `alertBody` : 顯示在通知警示中的訊息內容。
- `alertAction` : 可操作通知下方顯示的「動作」，預設為「檢視」。注意：Apple 自 iOS 10 起不再顯示此選項。
- `soundName` : 觸發通知時播放的音效（可選）。
- `isSilent` : 若為 true，通知將以靜音方式顯示（可選）。
- `category` : 此通知的分類，用於可操作通知（可選）。
- `userInfo` : 包含額外通知資料的物件（可選）。
- `applicationIconBadgeNumber` : 顯示在應用程式圖示上的徽章數字。設為 0 可移除徽章（可選）。
- `repeatInterval` : 以字串指定的重複間隔。可能值：`minute`、`hour`、`day`、`week`、`month`、`year`（可選）。

---

### `cancelAllLocalNotifications()`

```tsx
static cancelAllLocalNotifications();
```

取消所有已排程的本地通知

---

### `removeAllDeliveredNotifications()`

```tsx
static removeAllDeliveredNotifications();
```

從通知中心移除所有已送達的通知

---

### `getDeliveredNotifications()`

```tsx
static getDeliveredNotifications(callback: (notifications: Object[]) => void);
```

提供仍顯示在通知中心內的應用程式通知清單

**參數：**

| Name     | Type     | Required | Description                                                 |
| -------- | -------- | -------- | ----------------------------------------------------------- |
| callback | function | Yes      | Function which receive an array of delivered notifications. |

已送達通知為包含以下屬性的物件：

- `identifier` : 此通知的唯一識別碼。
- `title` : 此通知的標題。
- `body` : 此通知的內容正文。
- `category` : 此通知的分類（可選）。
- `userInfo` : 包含額外通知資料的物件（可選）。
- `thread-id` : 此通知的執行緒識別碼（若存在）。

---

### `removeDeliveredNotifications()`

```tsx
static removeDeliveredNotifications(identifiers: string[]);
```

從通知中心移除指定的通知

**參數：**

| Name        | Type  | Required | Description                        |
| ----------- | ----- | -------- | ---------------------------------- |
| identifiers | array | Yes      | Array of notification identifiers. |

---

### `setApplicationIconBadgeNumber()`

```tsx
static setApplicationIconBadgeNumber(number: number);
```

設定主畫面應用程式圖示的徽章數字

**參數：**

| Name   | Type   | Required | Description                    |
| ------ | ------ | -------- | ------------------------------ |
| number | number | Yes      | Badge number for the app icon. |

---

### `getApplicationIconBadgeNumber()`

```tsx
static getApplicationIconBadgeNumber(callback: (badge: number) => void);
```

取得主畫面應用程式圖示的當前徽章數字

**參數：**

| Name     | Type     | Required | Description                                              |
| -------- | -------- | -------- | -------------------------------------------------------- |
| callback | function | Yes      | A function that will be passed the current badge number. |

---

### `cancelLocalNotifications()`

```tsx
static cancelLocalNotifications(userInfo: Object);
```

取消本地通知。

可選限制條件：僅取消 `userInfo` 欄位與參數 `userInfo` 相符的通知。

**參數：**

| Name     | Type   | Required | Description |
| -------- | ------ | -------- | ----------- |
| userInfo | object | No       |             |

---

### `getScheduledLocalNotifications()`

```tsx
static getScheduledLocalNotifications(
  callback: (notifications: ScheduleLocalNotificationDetails[]) => void,
);
```

取得當前已排程的本地通知清單。

**參數：**

| Name     | Type     | Required | Description                                                                        |
| -------- | -------- | -------- | ---------------------------------------------------------------------------------- |
| callback | function | Yes      | A function that will be passed an array of objects describing local notifications. |

---

### `addEventListener()`

```tsx
static addEventListener(
  type: PushNotificationEventName,
  handler:
    | ((notification: PushNotification) => void)
    | ((deviceToken: string) => void)
    | ((error: {message: string; code: number; details: any}) => void),
);
```

在應用程式於前景或背景執行時，附加監聽器至遠端或本地通知事件。

**參數：**

| Name    | Type     | Required | Description |
| ------- | -------- | -------- | ----------- |
| type    | string   | Yes      | Event type. |
| handler | function | Yes      | Listener.   |

有效事件包括：

- `notification`：當收到遠端通知時觸發。處理程序將以 `PushNotificationIOS` 的實例調用。
- `localNotification`：當收到本地通知時觸發。處理程序將以 `PushNotificationIOS` 的實例調用。
- `register`：當使用者註冊遠端通知時觸發。處理程序將以代表裝置令牌的十六進位字串調用。
- `registrationError`：當使用者註冊遠端通知失敗時觸發。通常發生於 APNS 出現問題或裝置為模擬器時。處理程序將以 `{message: string, code: number, details: any}` 調用。

---

### `removeEventListener()`

```tsx
static removeEventListener(
  type: PushNotificationEventName,
  handler:
    | ((notification: PushNotification) => void)
    | ((deviceToken: string) => void)
    | ((error: {message: string; code: number; details: any}) => void),
);
```

移除事件監聽器。請在 `componentWillUnmount` 中執行此操作以避免記憶體洩漏。

**參數：**

| Name    | Type     | Required | Description |
| ------- | -------- | -------- | ----------- |
| type    | string   | Yes      | Event type. |
| handler | function | Yes      | Listener.   |

---

### `requestPermissions()`

```tsx
static requestPermissions(permissions?: PushNotificationPermissions[]);
```

向 iOS 請求通知權限，觸發使用者對話框。預設情況下會請求所有通知權限，但可透過傳遞請求權限的映射來僅請求部分權限。支援的權限包括：

- `alert`
- `badge`
- `sound`

若提供映射給此方法，僅會請求值為真（truthy）的權限。

此方法返回一個 Promise，當使用者接受、拒絕或先前已拒絕權限時會解析。Promise 解析結果為權限的當前狀態。

**參數：**

| Name        | Type  | Required | Description            |
| ----------- | ----- | -------- | ---------------------- |
| permissions | array | No       | alert, badge, or sound |

---

### `abandonPermissions()`

```tsx
static abandonPermissions();
```

取消註冊透過 Apple Push Notification 服務接收的所有遠端通知。

僅在極少數情況下應呼叫此方法，例如當應用程式新版本移除對所有遠端通知類型的支援時。使用者可透過設定應用程式的通知選項暫時阻止應用程式接收遠端通知。透過此方法取消註冊的應用程式隨時可重新註冊。

---

### `checkPermissions()`

```tsx
static checkPermissions(
  callback: (permissions: PushNotificationPermissions) => void,
);
```

檢查當前啟用的推送權限狀態。

**參數：**

| Name     | Type     | Required | Description |
| -------- | -------- | -------- | ----------- |
| callback | function | Yes      | See below.  |

`callback` 將以 `permissions` 物件調用：

- `alert: boolean`
- `badge: boolean`
- `sound: boolean`

---

### `getInitialNotification()`

```tsx
static getInitialNotification(): Promise<PushNotification | null>;
```

此方法返回一個 Promise。若應用程式由推送通知啟動，Promise 將解析為 `PushNotificationIOS` 類型的物件；否則解析為 `null`。

---

### `finish()`

```tsx
finish(result: string);
```

此方法適用於透過 `application:didReceiveRemoteNotification:fetchCompletionHandler:` 接收的遠端通知（參見：https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application?language=objc）。

呼叫此方法以在遠端通知處理完成時執行。呼叫此區塊時，傳入最能描述操作結果的 fetch 結果值。您「必須」呼叫此處理程序，且應盡快執行。可能的數值清單請參見 `PushNotificationIOS.FetchResult`。

若不調用此方法，您的背景遠端通知可能會被節流，詳細資訊請參閱上述文件連結。

---

### `getMessage()`

```tsx
getMessage(): string | Object;
```

`getAlert` 的別名，用於取得通知的主要訊息字串

---

### `getSound()`

```tsx
getSound(): string;
```

從 `aps` 物件中取得聲音字串

---

### `getCategory()`

```tsx
getCategory(): string;
```

從 `aps` 物件中取得分類字串

---

### `getAlert()`

```tsx
getAlert(): string | Object;
```

從 `aps` 物件中取得通知的主要訊息

---

### `getContentAvailable()`

```tsx
getContentAvailable(): number;
```

從 `aps` 物件中取得 content-available 數值

---

### `getBadgeCount()`

```tsx
getBadgeCount(): number;
```

從 `aps` 物件中取得徽章計數數值

---

### `getData()`

```tsx
getData(): Object;
```

取得通知上的資料物件

---

### `getThreadID()`

```tsx
getThreadID();
```

取得通知上的執行緒 ID