---
id: pushnotificationios
title: '🚧 PushNotificationIOS'
---

> **Deprecated.** Use the [community package](https://github.com/react-native-push-notification/ios) instead.

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3>
  <p>The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/versions/latest/sdk/notifications/">Notifications</a> in the Expo documentation for the appropriate alternative.</p>
</div>

Handle notifications for your app, including scheduling and permissions.

---

## Getting Started

To enable push notifications, [configure your notifications with Apple](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server) and your server-side system.

Then, [enable remote notifications](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/pushing_background_updates_to_your_app#2980038) in your project. This will automatically enable the required settings.

### Enable support for `register` events

In your `AppDelegate.m`, add:

```objectivec
#import <React/RCTPushNotificationManager.h>
```

Then implement the following in order to handle remote notification registration events:

```objectivec
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken
{
 // This will trigger 'register' events on PushNotificationIOS
 [RCTPushNotificationManager didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}
- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error
{
 // This will trigger 'registrationError' events on PushNotificationIOS
 [RCTPushNotificationManager didFailToRegisterForRemoteNotificationsWithError:error];
}
```

### Handle notifications

You'll need to implement `UNUserNotificationCenterDelegate` in your `AppDelegate`:

```objectivec
#import <UserNotifications/UserNotifications.h>

@interface YourAppDelegate () <UNUserNotificationCenterDelegate>
@end
```

Set the delegate on app launch:

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  ...
  UNUserNotificationCenter *center = [UNUserNotificationCenter currentNotificationCenter];
  center.delegate = self;

  return YES;
}
```

#### Foreground notifications

Implement `userNotificationCenter:willPresentNotification:withCompletionHandler:` to handle notifications that arrive when the app is in the foreground. Use the completionHandler to determine if the notification will be shown to the user and notify `RCTPushNotificationManager` accordingly:

```objectivec
// Called when a notification is delivered to a foreground app.
- (void)userNotificationCenter:(UNUserNotificationCenter *)center
       willPresentNotification:(UNNotification *)notification
         withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler
{
  // This will trigger 'notification' and 'localNotification' events on PushNotificationIOS
  [RCTPushNotificationManager didReceiveNotification:notification];
  // Decide if and how the notification will be shown to the user
  completionHandler(UNNotificationPresentationOptionNone);
}
```

#### Background notifications

Implement `userNotificationCenter:didReceiveNotificationResponse:withCompletionHandler:` to handle when a notification is tapped, typically called for background notifications which the user taps to open the app. However, if you had set foreground notifications to be shown in `userNotificationCenter:willPresentNotification:withCompletionHandler:`, this method will also be invoked on foreground notifications when tapped. In this case, you should only notify `RCTPushNotificationManager` in one of these callbacks.

If the tapped notification resulted in app launch, call `setInitialNotification:`. If the notification was not previously handled by `userNotificationCenter:willPresentNotification:withCompletionHandler:`, call `didReceiveNotification:` as well:

```objectivec
- (void)  userNotificationCenter:(UNUserNotificationCenter *)center
  didReceiveNotificationResponse:(UNNotificationResponse *)response
           withCompletionHandler:(void (^)(void))completionHandler
{
  // This condition passes if the notification was tapped to launch the app
  if ([response.actionIdentifier isEqualToString:UNNotificationDefaultActionIdentifier]) {
    // Allow the notification to be retrieved on the JS side using getInitialNotification()
    [RCTPushNotificationManager setInitialNotification:response.notification];
  }
  // This will trigger 'notification' and 'localNotification' events on PushNotificationIOS
  [RCTPushNotificationManager didReceiveNotification:response.notification];
  completionHandler();
}
```

---

# Reference

## Methods

### `presentLocalNotification()`

```tsx
static presentLocalNotification(details: PresentLocalNotificationDetails);
```

Schedules a local notification for immediate presentation.

**Parameters:**

| Name    | Type   | Required | Description |
| ------- | ------ | -------- | ----------- |
| details | object | Yes      | See below.  |

`details` is an object containing:

- `alertTitle` : The text displayed as the title of the notification alert.
- `alertBody` : The message displayed in the notification alert.
- `userInfo` : An object containing additional notification data (optional).
- `category` : The category of this notification, required for actionable notifications (optional). e.g. notifications with additional actions such as Reply or Like.
- `applicationIconBadgeNumber` The number to display as the app's icon badge. The default value of this property is 0, which means that no badge is displayed (optional).
- `isSilent` : If true, the notification will appear without sound (optional).
- `soundName` : The sound played when the notification is fired (optional).
- `alertAction` : DEPRECATED. This was used for iOS's legacy UILocalNotification.

---

### `scheduleLocalNotification()`

```tsx
static scheduleLocalNotification(details: ScheduleLocalNotificationDetails);
```

Schedules a local notification for future presentation.

**Parameters:**

| Name    | Type   | Required | Description |
| ------- | ------ | -------- | ----------- |
| details | object | Yes      | See below.  |

`details` is an object containing:

- `alertTitle` : The text displayed as the title of the notification alert.
- `alertBody` : The message displayed in the notification alert.
- `fireDate` : When the notification will be fired. Schedule notifications using either `fireDate` or `fireIntervalSeconds`, with `fireDate` taking precedence.
- `fireIntervalSeconds` : Seconds from now to display the notification.
- `userInfo` : An object containing additional notification data (optional).
- `category` : The category of this notification, required for actionable notifications (optional). e.g. notifications with additional actions such as Reply or Like.
- `applicationIconBadgeNumber` The number to display as the app's icon badge. The default value of this property is 0, which means that no badge is displayed (optional).
- `isSilent` : If true, the notification will appear without sound (optional).
- `soundName` : The sound played when the notification is fired (optional).
- `alertAction` : DEPRECATED. This was used for iOS's legacy UILocalNotification.
- `repeatInterval` : DEPRECATED. Use `fireDate` or `fireIntervalSeconds` instead.

---

### `cancelAllLocalNotifications()`

```tsx
static cancelAllLocalNotifications();
```

Cancels all scheduled local notifications.

---

### `removeAllDeliveredNotifications()`

```tsx
static removeAllDeliveredNotifications();
```

Removes all delivered notifications from Notification Center.

---

### `getDeliveredNotifications()`

```tsx
static getDeliveredNotifications(callback: (notifications: Object[]) => void);
```

Provides a list of the app’s notifications that are currently displayed in Notification Center.

**Parameters:**

| Name     | Type     | Required | Description                                                  |
| -------- | -------- | -------- | ------------------------------------------------------------ |
| callback | function | Yes      | Function which receives an array of delivered notifications. |

A delivered notification is an object containing:

- `identifier` : The identifier of this notification.
- `title` : The title of this notification.
- `body` : The body of this notification.
- `category` : The category of this notification (optional).
- `userInfo` : An object containing additional notification data (optional).
- `thread-id` : The thread identifier of this notification, if it has one.

---

### `removeDeliveredNotifications()`

```tsx
static removeDeliveredNotifications(identifiers: string[]);
```

Removes the specified notifications from Notification Center.

**Parameters:**

| Name        | Type  | Required | Description                        |
| ----------- | ----- | -------- | ---------------------------------- |
| identifiers | array | Yes      | Array of notification identifiers. |

---

### `setApplicationIconBadgeNumber()`

```tsx
static setApplicationIconBadgeNumber(num: number);
```

Sets the badge number for the app icon on the Home Screen.

**Parameters:**

| Name   | Type   | Required | Description                    |
| ------ | ------ | -------- | ------------------------------ |
| number | number | Yes      | Badge number for the app icon. |

---

### `getApplicationIconBadgeNumber()`

```tsx
static getApplicationIconBadgeNumber(callback: (num: number) => void);
```

Gets the current badge number for the app icon on the Home Screen.

**Parameters:**

| Name     | Type     | Required | Description                                        |
| -------- | -------- | -------- | -------------------------------------------------- |
| callback | function | Yes      | Function which processes the current badge number. |

---

### `cancelLocalNotifications()`

```tsx
static cancelLocalNotifications(userInfo: Object);
```

Cancels any scheduled local notifications which match the fields in the provided `userInfo`.

**Parameters:**

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

Gets the list of local notifications that are currently scheduled.

**Parameters:**

| Name     | Type     | Required | Description                                                                  |
| -------- | -------- | -------- | ---------------------------------------------------------------------------- |
| callback | function | Yes      | Function which processes an array of objects describing local notifications. |

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

Attaches a listener to notification events including local notifications, remote notifications, and notification registration results.

**參數：**

| Name    | Type     | Required | Description                         |
| ------- | -------- | -------- | ----------------------------------- |
| type    | string   | Yes      | Event type to listen to. See below. |
| handler | function | Yes      | Listener.                           |

有效的事件類型包括：

- `notification`：當收到遠端通知時觸發。處理程序將以 `PushNotificationIOS` 的實例調用。此事件會處理在前景收到的通知或從背景點擊開啟應用程式的通知。
- `localNotification`：當收到本地通知時觸發。處理程序將以 `PushNotificationIOS` 的實例調用。此事件會處理在前景收到的通知或從背景點擊開啟應用程式的通知。
- `register`：當用戶成功註冊遠端通知時觸發。處理程序將以代表 deviceToken 的十六進制字符串調用。
- `registrationError`：當用戶註冊遠端通知失敗時觸發。通常由於 APNS 問題或設備是模擬器導致。處理程序將以 `{message: string, code: number, details: any}` 調用。

---

### `removeEventListener()`

```tsx
static removeEventListener(
  type: PushNotificationEventName,
);
```

移除事件監聽器。請在 `componentWillUnmount` 中執行此操作以避免記憶體洩漏。

**參數：**

| Name | Type   | Required | Description                                       |
| ---- | ------ | -------- | ------------------------------------------------- |
| type | string | Yes      | Event type. See `addEventListener()` for options. |

---

### `requestPermissions()`

```tsx
static requestPermissions(permissions?: PushNotificationPermissions[]);
```

向 iOS 請求通知權限，並向用戶顯示對話框。預設情況下，此方法會請求所有通知權限，但您可以選擇性地指定要請求的權限。支援的權限如下：

- `alert`
- `badge`
- `sound`

如果提供了一個映射（map）給此方法，則僅會請求值為真（truthy）的權限。

此方法返回一個 Promise，當用戶接受或拒絕請求時，或權限先前已被拒絕時，該 Promise 會解析。Promise 解析為請求完成後的權限狀態。

**參數：**

| Name        | Type  | Required | Description            |
| ----------- | ----- | -------- | ---------------------- |
| permissions | array | No       | alert, badge, or sound |

---

### `abandonPermissions()`

```tsx
static abandonPermissions();
```

取消註冊所有透過 Apple Push Notification 服務接收的遠端通知。

您應僅在極少數情況下調用此方法，例如當應用程式的新版本移除了對所有類型遠端通知的支援時。用戶可以透過設定應用程式暫時阻止應用程式接收遠端通知。透過此方法取消註冊的應用程式隨時可以重新註冊。

---

### `checkPermissions()`

```tsx
static checkPermissions(
  callback: (permissions: PushNotificationPermissions) => void,
);
```

檢查當前啟用了哪些推送權限。

**參數：**

| Name     | Type     | Required | Description |
| -------- | -------- | -------- | ----------- |
| callback | function | Yes      | See below.  |

`callback` 將以一個 `permissions` 物件調用：

- `alert: boolean`
- `badge: boolean`
- `sound: boolean`

---

### `getInitialNotification()`

```tsx
static getInitialNotification(): Promise<PushNotification | null>;
```

此方法返回一個 Promise。如果應用程式是由推送通知啟動的，則此 Promise 會解析為被點擊通知的 `PushNotificationIOS` 類型物件。否則，解析為 `null`。

---

### `getAuthorizationStatus()`

```tsx
static getAuthorizationStatus(): Promise<number>;
```

此方法返回一個 Promise，該 Promise 解析為當前的通知授權狀態。可能的數值請參閱 [UNAuthorizationStatus](https://developer.apple.com/documentation/usernotifications/unauthorizationstatus?language=objc)。

---

### `finish()`

```tsx
finish(result: string);
```

This method is available for remote notifications that have been received via [`application:didReceiveRemoteNotification:fetchCompletionHandler:`](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application?language=objc). However, this is superseded by `UNUserNotificationCenterDelegate` and will no longer be invoked if both `application:didReceiveRemoteNotification:fetchCompletionHandler:` and the newer handlers from `UNUserNotificationCenterDelegate` are implemented.

If for some reason you're still relying on `application:didReceiveRemoteNotification:fetchCompletionHandler:`, you'll need to set up event handling on the iOS side:

```objectivec
- (void)           application:(UIApplication *)application
  didReceiveRemoteNotification:(NSDictionary *)userInfo
        fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))handler
{
  [RCTPushNotificationManager didReceiveRemoteNotification:userInfo fetchCompletionHandler:handler];
}
```

Call `finish()` to execute the native completion handlers once you're done handling the notification on the JS side. When calling this block, pass in the fetch result value that best describes the results of your operation. For a list of possible values, see `PushNotificationIOS.FetchResult`.

If you're using `application:didReceiveRemoteNotification:fetchCompletionHandler:`, you _must_ call this handler and should do so as soon as possible. See the [official documentation](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623013-application?language=objc) for more details.

---

### `getMessage()`

```tsx
getMessage(): string | Object;
```

An alias for `getAlert` to get the notification's main message string.

---

### `getSound()`

```tsx
getSound(): string;
```

Gets the sound string from the `aps` object. This will be `null` for local notifications.

---

### `getCategory()`

```tsx
getCategory(): string;
```

Gets the category string from the `aps` object.

---

### `getAlert()`

```tsx
getAlert(): string | Object;
```

Gets the notification's main message from the `aps` object. Also see the alias: `getMessage()`.

---

### `getContentAvailable()`

```tsx
getContentAvailable(): number;
```

Gets the content-available number from the `aps` object.

---

### `getBadgeCount()`

```tsx
getBadgeCount(): number;
```

Gets the badge count number from the `aps` object.

---

### `getData()`

```tsx
getData(): Object;
```

Gets the data object on the notification.

---

### `getThreadID()`

```tsx
getThreadID();
```

Gets the thread ID on the notification.