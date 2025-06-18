---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from '../the-new-architecture/\_markdown_native_deprecation.mdx'
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<NativeDeprecated />

歡迎來到 iOS 原生模組開發。請先閱讀 [原生模組簡介](native-modules-intro) 了解什麼是原生模組。

## 建立日曆原生模組

在本指南中，您將建立一個名為 `CalendarModule` 的原生模組，該模組將允許您從 JavaScript 存取 Apple 的日曆 API。最終您將能夠從 JavaScript 調用 `CalendarModule.createCalendarEvent('晚餐派對', '我家');`，從而觸發建立日曆事件的原生方法。

### 設定

首先，在 Xcode 中打開您的 React Native 應用程式內的 iOS 專案。您可以在以下路徑找到 React Native 應用程式中的 iOS 專案：

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

我們建議使用 Xcode 來編寫您的原生代碼。Xcode 專為 iOS 開發而設計，使用它將幫助您快速解決代碼語法等小錯誤。

### 建立自訂原生模組檔案

第一步是建立我們的主要自訂原生模組標頭檔和實作檔。建立一個名為 `RCTCalendarModule.h` 的新檔案：

<figure>
  <img src="/docs/assets/native-modules-ios-add-class.png" width="500" alt="Image of creating a class called  RCTCalendarModule.h." />
  <figcaption>Image of creating a custom native module file within the same folder as AppDelegate</figcaption>
</figure>

並添加以下內容：

```objectivec
//  RCTCalendarModule.h
#import <React/RCTBridgeModule.h>
@interface RCTCalendarModule : NSObject <RCTBridgeModule>
@end

```

您可以使用任何符合您正在建立的原生模組的名稱。由於您正在建立一個日曆原生模組，因此將類別命名為 `RCTCalendarModule`。由於 Objective-C 不像 Java 或 C++ 那樣具有語言層級的命名空間支持，慣例是在類別名稱前加上一個子字串。這可以是您應用程式名稱或基礎架構名稱的縮寫。在此範例中，RCT 代表 React。

如下所示，CalendarModule 類別實現了 `RCTBridgeModule` 協議。原生模組是一個實現 `RCTBridgeModule` 協議的 Objective-C 類別。

接下來，讓我們開始實作原生模組。在 Xcode 中使用 cocoa touch class 在同一個資料夾中建立對應的實作檔 `RCTCalendarModule.m`，並包含以下內容：

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### 模組名稱

目前，您的 `RCTCalendarModule.m` 原生模組僅包含一個 `RCT_EXPORT_MODULE` 宏，該宏將原生模組類別導出並註冊到 React Native。`RCT_EXPORT_MODULE` 宏還可以接受一個可選參數，指定模組在 JavaScript 代碼中可存取的名稱。

此參數不是字串字面量。在下面的範例中，傳遞的是 `RCT_EXPORT_MODULE(CalendarModuleFoo)`，而不是 `RCT_EXPORT_MODULE("CalendarModuleFoo")`。

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

然後，可以在 JS 中這樣存取原生模組：

```tsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

如果您不指定名稱，JavaScript 模組名稱將與 Objective-C 類別名稱匹配，並移除任何 "RCT" 或 "RK" 前綴。

讓我們遵循下面的範例，不帶任何參數調用 `RCT_EXPORT_MODULE`。因此，模組將使用名稱 `CalendarModule` 暴露給 React Native，因為這是移除 RCT 後的 Objective-C 類別名稱。

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

然後，可以在 JS 中這樣存取原生模組：

```tsx
const {CalendarModule} = ReactNative.NativeModules;
```

### 將原生方法導出到 JavaScript

React Native 不會自動將原生模組中的方法暴露給 JavaScript，除非明確使用 `RCT_EXPORT_METHOD` 巨集進行宣告。透過 `RCT_EXPORT_METHOD` 編寫的方法均為非同步，其回傳類型永遠為 void。若要將結果從 `RCT_EXPORT_METHOD` 方法傳遞至 JavaScript，可使用回調函數或發送事件（後續說明）。現在我們使用 `RCT_EXPORT_METHOD` 巨集為 `CalendarModule` 原生模組建立一個原生方法，命名為 `createCalendarEvent()`，暫時讓它接收名稱和位置兩個字串參數，參數類型的選項將稍後說明。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
}
```

> 請注意，除非您的方法依賴 RCT 參數轉換（參見下方參數類型說明），否則 TurboModules 將不再需要 `RCT_EXPORT_METHOD` 巨集。React Native 最終會移除 `RCT_EXPORT_MACRO`，因此我們不建議使用 `RCTConvert`，而應在方法體內自行處理參數轉換。

在實作 `createCalendarEvent()` 方法的功能前，先在其中加入 console log 以確認該方法已從 React Native 應用的 JavaScript 端被呼叫。請使用 React 提供的 `RCTLog` API，先在檔案頂部匯入該標頭檔，然後加入 log 呼叫。

```objectivec
#import <React/RCTLog.h>
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
 RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);
}
```

### 同步方法

您可以使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD` 來建立同步的原生方法。

```objectivec
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getName)
{
return [[UIDevice currentDevice] name];
}
```

此方法的回傳類型必須為物件類型 (id) 且可序列化為 JSON，這表示該方法僅能回傳 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前我們不建議使用同步方法，因為同步呼叫可能導致嚴重的效能損耗，並為原生模組引入執行緒相關的錯誤。此外請注意，若選擇使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，您的應用將無法繼續使用 Google Chrome 除錯器，這是因為同步方法要求 JS 虛擬機器與應用共享記憶體，而 Google Chrome 除錯器中 React Native 運行於 Chrome 的 JS 虛擬機器內，需透過 WebSockets 與行動裝置非同步通訊。

### 測試已建構的功能

至此您已完成 iOS 原生模組的基礎架構，現在可透過 JavaScript 存取原生模組並呼叫其導出的方法來進行測試。

在應用中選擇合適位置加入對原生模組 `createCalendarEvent()` 方法的呼叫。以下是一個可加入應用中的 `NewModuleButton` 元件範例，您可在該元件的 `onPress()` 函數內呼叫原生模組。

```tsx
import React from 'react';
import {Button} from 'react-native';

const NewModuleButton = () => {
  const onPress = () => {
    console.log('We will invoke the native module here!');
  };

  return (
    <Button
      title="Click to invoke your native module!"
      color="#841584"
      onPress={onPress}
    />
  );
};

export default NewModuleButton;
```

要從 JavaScript 存取原生模組，需先從 React Native 匯入 `NativeModules`：

```tsx
import {NativeModules} from 'react-native';
```

接著即可從 `NativeModules` 取得 `CalendarModule` 原生模組。

```tsx
const {CalendarModule} = NativeModules;
```

現在您已可存取 CalendarModule 原生模組，並呼叫其原生方法 `createCalendarEvent()`。以下示範如何在 `NewModuleButton` 的 `onPress()` 方法中加入呼叫：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新建構 React Native 應用，以確保最新的原生程式碼（包含您的新原生模組！）已生效。在 react native 應用所在的命令列中執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios
```

</TabItem>
</Tabs>

### 迭代開發時的建構流程

當你依照這些指南進行開發並反覆修改原生模組時，需要重新建置應用程式才能讓 JavaScript 端取得最新的變更。這是因為你所編寫的程式碼位於應用的原生部分。雖然 React Native 的 Metro 打包工具能監控 JavaScript 變更並即時重建 JS 套件，但不會自動處理原生程式碼。因此若要測試最新的原生修改，必須使用上述指令手動重建。

### 重點回顧 ✨

你現在應該能透過 JavaScript 呼叫原生模組的 `createCalendarEvent()` 方法。由於函式中使用了 `RCTLog`，可透過[啟用應用除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)，在 Chrome 的 JS 控制台或行動應用除錯工具 Flipper 中確認原生方法是否被觸發。每次呼叫原生模組方法時，都應看到 `RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);` 的記錄訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此你已建立 iOS 原生模組，並從 React Native 應用的 JavaScript 端呼叫其方法。可繼續閱讀以了解更多進階主題，例如原生模組方法的參數類型設定，以及如何在原生模組中配置回調函式與 Promise。

## 日曆原生模組的進階應用

### 優化原生模組匯出方式

透過 `NativeModules` 提取原生模組的現行方式較為笨拙。

為避免模組使用者每次都要重複此操作，可為模組建立 JavaScript 封裝層。新建名為 NativeCalendarModule.js 的 JavaScript 檔案，內容如下：

```tsx
/**
* This exposes the native CalendarModule module as a JS module. This has a
* function 'createCalendarEvent' which takes the following parameters:

* 1. String name: A string representing the name of the event
* 2. String location: A string representing the location of the event
*/
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
export default CalendarModule;
```

此 JavaScript 檔案也適合加入 JavaScript 端功能。例如若使用 TypeScript 等類型系統，可在此為原生模組添加類型註解。雖然 React Native 尚未支援原生到 JS 的類型安全，但這些註解能確保所有 JS 程式碼具備類型安全，並為未來轉換至類型安全的原生模組預做準備。以下是為日曆模組添加類型安全的範例：

```tsx
/**
 * This exposes the native CalendarModule module as a JS module. This has a
 * function 'createCalendarEvent' which takes the following parameters:
 *
 * 1. String name: A string representing the name of the event
 * 2. String location: A string representing the location of the event
 */
import {NativeModules} from 'react-native';
const {CalendarModule} = NativeModules;
interface CalendarInterface {
  createCalendarEvent(name: string, location: string): void;
}
export default CalendarModule as CalendarInterface;
```

在其他 JavaScript 檔案中，可透過以下方式存取原生模組並呼叫其方法：

```tsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意：此處假設導入 `CalendarModule` 的位置與 `NativeCalendarModule.js` 處於相同目錄層級，請根據實際情況調整相對路徑。

### 參數類型對照

當 JavaScript 呼叫原生模組方法時，React Native 會將參數從 JS 物件轉換為對應的 Objective-C/Swift 物件。例如若 Objective-C 原生模組方法接受 NSNumber，在 JS 中需傳入數字類型，React Native 會自動處理轉換。以下是原生模組方法支援的參數類型與其 JavaScript 對應型態清單：

| Objective-C                                   | JavaScript         |
| --------------------------------------------- | ------------------ |
| NSString                                      | string, ?string    |
| BOOL                                          | boolean            |
| double                                        | number             |
| NSNumber                                      | ?number            |
| NSArray                                       | Array, ?Array      |
| NSDictionary                                  | Object, ?Object    |
| RCTResponseSenderBlock                        | Function (success) |
| RCTResponseSenderBlock, RCTResponseErrorBlock | Function (failure) |
| RCTPromiseResolveBlock, RCTPromiseRejectBlock | Promise            |

> 以下類型目前雖受支援，但 TurboModules 將不再相容，請避免使用：
>
> - Function (failure) -> RCTResponseErrorBlock
> - Number -> NSInteger
> - Number -> CGFloat
> - Number -> float

在 iOS 端，還可使用任何 `RCTConvert` 類別支援的參數類型編寫原生模組方法（詳見 [RCTConvert](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTConvert.h)）。RCTConvert 輔助函式皆接受 JSON 值作為輸入，並將其映射至原生 Objective-C 類型或類別。

### 匯出常數

原生模組可透過覆寫 `constantsToExport()` 方法來匯出常數。以下範例覆寫該方法，回傳包含預設事件名稱屬性的字典，即可在 JavaScript 端如此存取：

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 來存取這個常數，如下所示：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

技術上來說，可以直接從 `NativeModule` 物件存取透過 `constantsToExport()` 匯出的常數。但這在 TurboModules 中將不再支援，因此我們鼓勵開發社群改用上述方法，以避免未來不必要的遷移工作。

> 請注意，常數僅在初始化時匯出，因此若在執行階段變更 `constantsToExport()` 的值，將不會影響 JavaScript 環境。

在 iOS 上，若您覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup` 來告知 React Native 您的模組是否需要先於任何 JavaScript 程式碼執行前在主執行緒初始化。否則您會看到警告，提示未來您的模組可能會在背景執行緒初始化，除非您透過 `+ requiresMainQueueSetup:` 明確選擇退出。若您的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式(callback)。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞到 JavaScript，也可用於從原生端非同步執行 JavaScript。

在 iOS 中，回呼函式是透過 `RCTResponseSenderBlock` 類型實作的。以下範例在 `createCalendarEventMethod()` 中新增了回呼參數 `myCallBack`：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著您可以在原生函式中呼叫回呼函式，透過陣列傳遞任何想提供給 JavaScript 的結果。請注意，`RCTResponseSenderBlock` 僅接受一個參數——要傳遞給 JavaScript 回呼函式的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需特別強調的是，回呼函式不會在原生函式完成後立即被呼叫——請記住這種通訊是非同步的。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

然後可以透過以下方式在 JavaScript 中存取這個方法：

```tsx
const onSubmit = () => {
  CalendarModule.createCalendarEvent(
    'Party',
    '04-12-2020',
    eventId => {
      console.log(`Created a new event with id ${eventId}`);
    },
  );
};
```

原生模組應該只呼叫其回呼函式一次。不過，它可以儲存回呼函式並稍後再呼叫。這種模式常用於封裝需要委派(delegate)的 iOS API——請參閱 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/CoreModules/RCTAlertManager.mm) 的範例。若回呼函式從未被呼叫，會導致一些記憶體洩漏。

使用回呼函式進行錯誤處理有兩種方法。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，您可以檢查第一個參數來判斷是否有錯誤被傳遞：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEventCallback(
    'testName',
    'testLocation',
    (error, eventId) => {
      if (error) {
        console.error(`Error found! ${error}`);
      }
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

另一種選擇是使用兩個獨立的回呼函式：onFailure 和 onSuccess。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title
                  location:(NSString *)location
                  errorCallback: (RCTResponseSenderBlock)errorCallback
                  successCallback: (RCTResponseSenderBlock)successCallback)
{
  @try {
    NSNumber *eventId = [NSNumber numberWithInt:123];
    successCallback(@[eventId]);
  }

  @catch ( NSException *e ) {
    errorCallback(@[e]);
  }
}
```

然後在 JavaScript 中，您可以為錯誤和成功回應分別添加回呼函式：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEventCallback(
    'testName',
    'testLocation',
    error => {
      console.error(`Error found! ${error}`);
    },
    eventId => {
      console.log(`event id ${eventId} returned`);
    },
  );
};
```

若要將類似錯誤的物件傳遞給 JavaScript，請使用 [`RCTUtils.h`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTUtils.h) 中的 `RCTMakeError`。目前這只會將一個 Error 形狀的字典傳遞給 JavaScript，但 React Native 的目標是在未來自動生成真正的 JavaScript Error 物件。您也可以提供 `RCTResponseErrorBlock` 參數，用於錯誤回呼並接受 `NSError *` 物件。請注意，此參數類型在 TurboModules 中將不受支援。

### Promise

原生模組也可以實現 Promise，這能簡化您的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳一個 JS Promise 物件。

將上述程式碼重構為使用 Promise 而非回呼函式，看起來會像這樣：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                 location:(NSString *)location
                 resolver:(RCTPromiseResolveBlock)resolve
                 rejecter:(RCTPromiseRejectBlock)reject)
{
 NSInteger eventId = createCalendarEvent();
 if (eventId) {
    resolve(@(eventId));
  } else {
    reject(@"event_failure", @"no event id returned", nil);
  }
}

```

該方法的 JavaScript 對應部分會回傳一個 Promise。這意味著你可以在 async 函式中使用 `await` 關鍵字來呼叫它並等待其結果：

```tsx
const onSubmit = async () => {
  try {
    const eventId = await CalendarModule.createCalendarEvent(
      'Party',
      'my house',
    );
    console.log(`Created a new event with id ${eventId}`);
  } catch (e) {
    console.error(e);
  }
};
```

### 向 JavaScript 發送事件

原生模組可以直接向 JavaScript 發送事件訊號，而無需被直接呼叫。例如，你可能希望向 JavaScript 發出訊號，提醒來自原生 iOS 日曆應用程式的日曆事件即將發生。首選方式是繼承 `RCTEventEmitter`，實作 `supportedEvents` 並呼叫 `self sendEventWithName`：

更新你的標頭類別以導入 `RCTEventEmitter` 並繼承 `RCTEventEmitter`：

```objectivec
//  CalendarModule.h

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

```

JavaScript 程式碼可以透過在你的模組周圍建立一個新的 `NativeEventEmitter` 實例來訂閱這些事件。

如果你在沒有監聽器的情況下發送事件，將會收到警告。為了避免這種情況，並優化模組的工作負載（例如取消訂閱上游通知或暫停背景任務），你可以在 `RCTEventEmitter` 子類別中覆寫 `startObserving` 和 `stopObserving` 方法。

```objectivec
@implementation CalendarModule
{
  bool hasListeners;
}

// Will be called when this module's first listener is added.
-(void)startObserving {
    hasListeners = YES;
    // Set up any upstream listeners or background tasks as necessary
}

// Will be called when this module's last listener is removed, or on dealloc.
-(void)stopObserving {
    hasListeners = NO;
    // Remove upstream listeners, stop unnecessary background tasks
}

- (void)calendarEventReminderReceived:(NSNotification *)notification
{
  NSString *eventName = notification.userInfo[@"name"];
  if (hasListeners) {// Only send events if anyone is listening
    [self sendEventWithName:@"EventReminder" body:@{@"name": eventName}];
  }
}

```

### 執行緒處理

除非原生模組提供自己的方法佇列，否則不應對其被呼叫的執行緒做出任何假設。目前，如果原生模組未提供方法佇列，React Native 會為其建立一個獨立的 GCD 佇列並在其中呼叫其方法。請注意，這是一個實作細節，可能會變更。如果你想明確為原生模組提供方法佇列，請在原生模組中覆寫 `(dispatch_queue_t) methodQueue` 方法。例如，如果需要使用僅限主執行緒的 iOS API，應透過以下方式指定：

```objectivec
- (dispatch_queue_t)methodQueue
{
  return dispatch_get_main_queue();
}
```

同樣地，如果某個操作可能需要很長時間才能完成，原生模組可以指定自己的佇列來執行操作。再次強調，目前 React Native 會為你的原生模組提供一個獨立的方法佇列，但這是一個你不應依賴的實作細節。如果你不提供自己的方法佇列，未來你的原生模組的長時間運行操作可能會阻塞在其他不相關原生模組上執行的非同步呼叫。例如，這裡的 `RCTAsyncLocalStorage` 模組建立了自己的佇列，這樣 React 佇列就不會被潛在的慢速磁碟存取阻塞。

```objectivec
- (dispatch_queue_t)methodQueue
{
 return dispatch_queue_create("com.facebook.React.AsyncLocalStorageQueue", DISPATCH_QUEUE_SERIAL);
}
```

指定的 `methodQueue` 將由模組中的所有方法共享。如果只有一個方法是長時間運行的（或由於某些原因需要與其他方法在不同的佇列上運行），你可以在方法內部使用 `dispatch_async` 將該特定方法的程式碼放在另一個佇列上執行，而不影響其他方法：

```objectivec
RCT_EXPORT_METHOD(doSomethingExpensive:(NSString *)param callback:(RCTResponseSenderBlock)callback)
{
 dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
   // Call long-running code on background thread
   ...
   // You can invoke callback from any thread/queue
   callback(@[...]);
 });
}

```

> 在模組之間共享調度佇列
>
> `methodQueue` 方法將在模組初始化時被呼叫一次，然後由 React Native 保留，因此除非你希望在模組內部使用該佇列，否則無需自行保留對佇列的引用。然而，如果你希望在多個模組之間共享同一個佇列，則需要確保為每個模組保留並回傳相同的佇列實例。

### 依賴注入

React Native 會自動建立並初始化所有已註冊的原生模組。但是，你可能希望建立並初始化自己的模組實例，以便注入依賴項。

你可以透過建立一個實作 `RCTBridgeDelegate` 協定的類別來實現這一點，使用該委派作為參數初始化一個 `RCTBridge`，然後使用初始化後的橋接器來初始化一個 `RCTRootView`。

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 匯出 Swift

Swift 不支援巨集，因此將原生模組及其方法暴露給 React Native 中的 JavaScript 需要更多的設定。不過，其運作方式大致相同。假設你有相同的 `CalendarModule`，但作為一個 Swift 類別：

```swift
// CalendarModule.swift

@objc(CalendarModule)
class CalendarModule: NSObject {

 @objc(addEvent:location:date:)
 func addEvent(_ name: String, location: String, date: NSNumber) -> Void {
   // Date is ready to use!
 }

 @objc
 func constantsToExport() -> [String: Any]! {
   return ["someKey": "someValue"]
 }

}
```

> 務必使用 `@objc` 修飾符，以確保類別和函式能正確匯出至 Objective-C 運行環境。

接著建立一個私有實作檔案，用於向 React Native 註冊必要的資訊：

```objectivec
// CalendarModuleBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarModule, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

對於初次接觸 Swift 和 Objective-C 的開發者，當你在 iOS 專案中[混用這兩種語言](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)時，還需要一個額外的橋接檔案（稱為橋接標頭檔）來將 Objective-C 檔案暴露給 Swift。若你透過 Xcode 的 `檔案>新增檔案` 選單將 Swift 檔案加入專案，Xcode 會提示你建立此標頭檔。你需要在該標頭檔中匯入 `RCTBridgeModule.h`。

```objectivec
// CalendarModule-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

你也可以使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來調整匯出模組或方法在 JavaScript 中的名稱。更多資訊請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridgeModule.h)。

> 開發第三方模組時的重要注意事項：僅 Xcode 9 及以上版本支援包含 Swift 的靜態函式庫。若你在模組的 iOS 靜態函式庫中使用 Swift，為了讓 Xcode 專案能成功建置，你的主應用程式專案必須包含 Swift 程式碼和橋接標頭檔。若應用程式專案不含任何 Swift 程式碼，可透過建立一個空的 .swift 檔案和空的橋接標頭檔作為臨時解決方案。

### 保留方法名稱

#### invalidate()

原生模組可透過實作 `invalidate()` 方法，在 iOS 上遵循 [RCTInvalidating](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTInvalidating.h) 協議。當原生橋接失效時（例如：開發模式重新載入），[可能會呼叫](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456)此方法。請視需求使用此機制為你的原生模組執行必要的清理工作。