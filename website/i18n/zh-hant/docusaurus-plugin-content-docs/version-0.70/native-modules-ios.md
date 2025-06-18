---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

歡迎來到 iOS 原生模組開發。請先閱讀 [原生模組介紹](native-modules-intro) 了解原生模組的基本概念。

## 建立日曆原生模組

在本指南中，您將建立一個名為 `CalendarModule` 的原生模組，用於從 JavaScript 存取蘋果的日曆 API。最終您將能透過 JavaScript 呼叫 `CalendarModule.createCalendarEvent('晚餐派對', '我家');` 來觸發建立日曆事件的原生方法。

### 設定

首先，請在 Xcode 中開啟您 React Native 應用程式的 iOS 專案。您可以在以下路徑找到 iOS 專案：

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

我們建議使用 Xcode 來編寫原生程式碼。Xcode 專為 iOS 開發設計，能幫助您快速解決程式碼語法等小錯誤。

### 建立自訂原生模組檔案

第一步是建立我們的主要自訂原生模組標頭檔與實作檔。請建立一個名為 `RCTCalendarModule.h` 的新檔案

<figure>
  <img src="/docs/assets/native-modules-ios-add-class.png" width="500" alt="Image of creating a class called  RCTCalendarModule.h." />
  <figcaption>Image of creating a custom native module file within the same folder as AppDelegate</figcaption>
</figure>

並加入以下內容：

```objectivec
//  RCTCalendarModule.h
#import <React/RCTBridgeModule.h>
@interface RCTCalendarModule : NSObject <RCTBridgeModule>
@end

```

您可以為原生模組使用任何合適的名稱。此處命名為 `RCTCalendarModule` 是因為您正在建立日曆原生模組。由於 Objective-C 不像 Java 或 C++ 具有語言層級的命名空間支援，慣例是在類別名稱前加上前綴字串。這可以是應用程式名稱或基礎架構名稱的縮寫。本例中的 RCT 代表 React。

如下所示，CalendarModule 類別實現了 `RCTBridgeModule` 協議。原生模組就是實現 `RCTBridgeModule` 協議的 Objective-C 類別。

接下來，讓我們開始實作原生模組。請在同個資料夾建立對應的實作檔 `RCTCalendarModule.m`，並包含以下內容：

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### 模組名稱

目前您的 `RCTCalendarModule.m` 原生模組僅包含 `RCT_EXPORT_MODULE` 宏，該宏會將原生模組類別匯出並註冊到 React Native。`RCT_EXPORT_MODULE` 宏還接受一個可選參數，用於指定模組在 JavaScript 程式碼中的存取名稱。

請注意此參數不是字串常值。在以下範例中傳遞的是 `RCT_EXPORT_MODULE(CalendarModuleFoo)`，而非 `RCT_EXPORT_MODULE("CalendarModuleFoo")`。

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

接著就能在 JS 中這樣存取原生模組：

```jsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

若未指定名稱，JavaScript 模組名稱將匹配 Objective-C 類別名稱，並移除 "RCT" 或 "RK" 前綴。

讓我們遵循以下範例，不帶任何參數呼叫 `RCT_EXPORT_MODULE`。結果模組將以 `CalendarModule` 名稱暴露給 React Native，因為這是移除 RCT 前綴後的 Objective-C 類別名稱。

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

接著就能在 JS 中這樣存取原生模組：

```jsx
const {CalendarModule} = ReactNative.NativeModules;
```

### 將原生方法匯出至 JavaScript

React Native 不會自動將原生模組中的方法暴露給 JavaScript，除非明確使用 `RCT_EXPORT_METHOD` 巨集進行宣告。透過 `RCT_EXPORT_METHOD` 編寫的方法都是非同步的，因此返回值類型始終為 void。若要將 `RCT_EXPORT_METHOD` 方法的結果傳回 JavaScript，可以使用回調函數或發送事件（後文會說明）。現在我們為 `CalendarModule` 原生模組設置一個使用 `RCT_EXPORT_METHOD` 巨集的原生方法，命名為 `createCalendarEvent()`，暫時讓它接收名稱和位置兩個字串參數。參數類型的選項稍後會詳細說明。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
}
```

> 請注意，在 TurboModules 中 `RCT_EXPORT_METHOD` 巨集將不再必要，除非你的方法依賴於 RCT 參數轉換（參見下方的參數類型說明）。最終，React Native 會移除 `RCT_EXPORT_MACRO`，因此我們不建議使用 `RCTConvert`。相反，你可以在方法體內進行參數轉換。

在完善 `createCalendarEvent()` 方法的功能之前，先在方法中添加一個控制台日誌，以便確認它已從 React Native 應用程序的 JavaScript 端被調用。使用 React 提供的 `RCTLog` API。首先在文件頂部導入該標頭文件，然後添加日誌調用。

```objectivec
#import <React/RCTLog.h>
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
 RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);
}
```

### 同步方法

你可以使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD` 來創建一個同步的原生方法。

```objectivec
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getName)
{
return [[UIDevice currentDevice] name];
}
```

該方法的返回類型必須是對象類型（id），並且應該可序列化為 JSON。這意味著該方法只能返回 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前，我們不建議使用同步方法，因為同步調用方法可能會導致嚴重的性能損失，並給你的原生模組引入線程相關的錯誤。此外，請注意，如果你選擇使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，你的應用程序將無法再使用 Google Chrome 調試器。這是因為同步方法要求 JS 虛擬機與應用程序共享內存。對於 Google Chrome 調試器，React Native 運行在 Google Chrome 的 JS 虛擬機中，並通過 WebSockets 與移動設備進行異步通信。

### 測試你構建的內容

此時，你已經在 iOS 中為你的原生模組設置了基本框架。通過在 JavaScript 中訪問原生模組並調用其導出的方法來測試它。

在你的應用程序中找到一個地方，添加對原生模組 `createCalendarEvent()` 方法的調用。下面是一個組件 `NewModuleButton` 的示例，你可以將其添加到你的應用中。你可以在 `NewModuleButton` 的 `onPress()` 函數中調用原生模組。

```jsx
import React from 'react';
import {NativeModules, Button} from 'react-native';

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

為了從 JavaScript 訪問你的原生模組，首先需要從 React Native 導入 `NativeModules`：

```jsx
import {NativeModules} from 'react-native';
```

然後，你可以從 `NativeModules` 中訪問 `CalendarModule` 原生模組。

```jsx
const {CalendarModule} = NativeModules;
```

現在你已經可以訪問 CalendarModule 原生模組，可以調用你的原生方法 `createCalendarEvent()`。下面將其添加到 `NewModuleButton` 的 `onPress()` 方法中：

```jsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新構建 React Native 應用程序，以便讓最新的原生代碼（包含你的新原生模組！）生效。在你的命令行中，進入 react native 應用程序所在的目錄，運行以下命令：

```shell
npx react-native run-ios
```

### 迭代開發時的構建

當您依照這些指南逐步開發並迭代原生模組時，需要重新建置應用程式才能從 JavaScript 存取最新的更動。這是因為您編寫的程式碼位於應用程式的原生部分。雖然 React Native 的 Metro 打包工具能監控 JavaScript 變更並即時重建 JS 套件包，但不會對原生程式碼執行相同操作。因此若要測試最新的原生端修改，必須透過執行 `npx react-native run-ios` 指令重新建置。

### 重點回顧 ✨

您現在應能透過 JavaScript 呼叫原生模組的 `createCalendarEvent()` 方法。由於函式中使用了 `RCTLog`，可透過[啟用應用程式的除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)，在 Chrome 的 JS 控制台或行動端除錯工具 Flipper 中確認原生方法是否被觸發。每次呼叫原生模組方法時，都應看到 `RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);` 的日誌訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此您已建立 iOS 原生模組，並從 React Native 應用程式的 JavaScript 端呼叫其方法。後續可進一步了解原生模組方法的參數類型設定，以及如何在原生模組中配置回調函式與 Promise。

## 日曆原生模組的進階應用

### 優化原生模組匯出方式

透過從 `NativeModules` 提取來匯入原生模組的做法稍顯笨拙。

為避免模組使用者每次存取時重複此操作，可為模組建立 JavaScript 封裝層。建立名為 NativeCalendarModule.js 的新 JavaScript 檔案，內容如下：

```jsx
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

此 JavaScript 檔案同時也是添加 JavaScript 端功能的理想位置。例如若使用 TypeScript 等型別系統，可在此為原生模組添加型別註解。雖然 React Native 尚未支援原生端到 JS 的型別安全，但透過這些型別註解能確保所有 JS 程式碼具備型別安全性。這些註解也將有助於未來遷移至型別安全的原生模組。以下是為日曆模組添加型別安全的範例：

```jsx
/**
* This exposes the native CalendarModule module as a JS module. This has a
* function 'createCalendarEvent' which takes the following parameters:
*
* 1. String name: A string representing the name of the event
* 2. String location: A string representing the location of the event
*/
import { NativeModules } from 'react-native';
const { CalendarModule } = NativeModules;
interface CalendarInterface {
   createCalendarEvent(name: string, location: string): void;
}
export default CalendarModule as CalendarInterface;
```

在其他 JavaScript 檔案中，可透過以下方式存取原生模組並呼叫其方法：

```jsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意：此處假設導入 `CalendarModule` 的位置與 `NativeCalendarModule.js` 處於相同目錄層級，請根據實際情況調整相對路徑。

### 參數類型

當 JavaScript 呼叫原生模組方法時，React Native 會將參數從 JS 物件轉換為對應的 Objective-C/Swift 物件。例如若 Objective-C 原生模組方法接受 NSNumber 類型，在 JS 中需傳入數字類型參數，React Native 會自動處理轉換。以下是原生模組方法支援的參數類型及其對應的 JavaScript 等效類型清單。

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

在 iOS 端，您還可使用任何 `RCTConvert` 類別支援的參數類型編寫原生模組方法（詳見 [RCTConvert](https://github.com/facebook/react-native/blob/0.70-stable/React/Base/RCTConvert.h)）。RCTConvert 輔助函式皆接受 JSON 值作為輸入，並將其映射至原生 Objective-C 類型或類別。

### 匯出常數

原生模組可透過覆寫 `constantsToExport()` 方法來匯出常數。以下範例覆寫該方法，回傳包含預設事件名稱屬性的字典，在 JavaScript 端可如此存取：

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 方法來存取該常數：

```objectivec
const { DEFAULT_EVENT_NAME } = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

技術上來說，直接從 `NativeModule` 物件存取 `constantsToExport()` 匯出的常數是可行的。但這在 TurboModules 中將不再支援，因此我們鼓勵社群改用上述方法，以避免未來不必要的遷移工作。

> 請注意，常數僅在初始化時匯出，因此若在執行階段變更 `constantsToExport()` 的值，並不會影響 JavaScript 環境。

在 iOS 上，若您覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup` 來告知 React Native 您的模組是否需要先於任何 JavaScript 程式碼執行前，在主執行緒上初始化。否則您會看到警告，提示未來您的模組可能會在背景執行緒初始化，除非您明確透過 `+ requiresMainQueueSetup:` 選擇退出。若您的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式（callback）。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞至 JavaScript，也可用於從原生端非同步執行 JavaScript。

在 iOS 上，回呼函式是透過 `RCTResponseSenderBlock` 類型實作的。以下範例將回呼參數 `myCallBack` 加入 `createCalendarEventMethod()`：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著您可以在原生函式中呼叫回呼，並透過陣列傳遞任何想送至 JavaScript 的結果。請注意，`RCTResponseSenderBlock` 僅接受一個參數——傳遞給 JavaScript 回呼的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需特別強調的是，回呼函式並非在原生函式完成後立即觸發——請記住通訊是非同步進行的。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

接著可透過以下方式在 JavaScript 中存取此方法：

```jsx
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

原生模組預期僅呼叫其回呼一次。不過，它可以儲存回呼並稍後再呼叫。此模式常用於封裝需要委派（delegate）的 iOS API——參見 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/3a11f0536ea65b87dc0f006665f16a87cfa14b5e/React/CoreModules/RCTAlertManager.mm) 範例。若回呼從未被呼叫，將會導致部分記憶體洩漏。

回呼的錯誤處理有兩種方式。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，您可以檢查第一個參數來判斷是否傳入了錯誤：

```jsx
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

另一種選項是使用兩個獨立的回呼：onFailure 和 onSuccess。

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

接著在 JavaScript 中可分別為錯誤和成功回應添加回呼：

```jsx
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

若您想將類錯誤物件傳遞至 JavaScript，請使用 [`RCTUtils.h`](https://github.com/facebook/react-native/blob/0.70-stable/React/Base/RCTUtils.h) 中的 `RCTMakeError`。目前這僅會將 Error 形態的字典傳給 JavaScript，但 React Native 目標是未來能自動產生真正的 JavaScript Error 物件。您也可以提供 `RCTResponseErrorBlock` 參數，用於錯誤回呼並接受 `NSError *` 物件。請注意，此參數類型在 TurboModules 中將不受支援。

### Promise

原生模組也能實現 Promise，這能簡化您的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳一個 JS Promise 物件。

將上述程式碼重構為使用 Promise 而非回調函數的範例如下：

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

該方法在 JavaScript 端的對應實現會回傳一個 Promise。這意味著您可以在非同步函數中使用 `await` 關鍵字來呼叫它並等待其結果：

```jsx
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

原生模組能夠不經直接呼叫就向 JavaScript 發出事件信號。舉例來說，您可能希望向 JavaScript 發出提醒，通知原生 iOS 日曆應用中的某個事件即將發生。建議的做法是繼承 `RCTEventEmitter`、實現 `supportedEvents` 並呼叫 `self sendEventWithName`：

更新您的標頭類別以導入 `RCTEventEmitter` 並繼承 `RCTEventEmitter`：

```objectivec
//  CalendarModule.h

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

```

JavaScript 程式碼可以透過在您的模組周圍建立新的 `NativeEventEmitter` 實例來訂閱這些事件。

若在沒有監聽器的情況下發出事件，您將會收到警告。為避免此情況並優化模組工作負載（例如取消訂閱上游通知或暫停背景任務），您可以覆寫 `RCTEventEmitter` 子類中的 `startObserving` 和 `stopObserving` 方法。

```objectivec
@implementation CalendarManager
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
  if (hasListeners) { // Only send events if anyone is listening
    [self sendEventWithName:@"EventReminder" body:@{@"name": eventName}];
  }
}

```

### 執行緒處理

除非原生模組提供自己的方法佇列，否則不應對其被呼叫的執行緒做出任何假設。目前，若原生模組未提供方法佇列，React Native 會為其建立獨立的 GCD 佇列並在其中呼叫方法。請注意這是實作細節且可能變更。若要明確為原生模組提供方法佇列，請覆寫模組中的 `(dispatch_queue_t) methodQueue` 方法。例如，若需使用僅限主執行緒的 iOS API，應透過以下方式指定：

```objectivec
- (dispatch_queue_t)methodQueue
{
  return dispatch_get_main_queue();
}
```

同樣地，若某操作可能耗時較長，原生模組可指定自己的佇列來執行操作。再次強調，目前 React Native 會為您的模組提供獨立方法佇列，但這是不應依賴的實作細節。若未提供自訂佇列，未來您的模組長時間運算可能導致其他無關原生模組的非同步呼叫被阻塞。例如 `RCTAsyncLocalStorage` 模組會建立自己的佇列，避免 React 佇列被可能緩慢的磁碟存取阻塞。

```objectivec
- (dispatch_queue_t)methodQueue
{
 return dispatch_queue_create("com.facebook.React.AsyncLocalStorageQueue", DISPATCH_QUEUE_SERIAL);
}
```

指定的 `methodQueue` 將由模組中所有方法共享。若僅某個方法需長時間運行（或需在不同佇列執行），可在方法內部使用 `dispatch_async` 將特定程式碼移至其他佇列執行，而不影響其他方法：

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

> 模組間共享調度佇列
>
> `methodQueue` 方法會在模組初始化時被呼叫一次，隨後由 React Native 保留，因此除非需在模組內部使用，否則無需自行保留佇列參考。但若需在多個模組間共享相同佇列，則必須確保為每個模組保留並回傳相同的佇列實例。

### 依賴注入

React Native 會自動建立並初始化所有註冊的原生模組。但您可能希望自行建立並初始化模組實例，例如為了注入依賴項。

可透過建立實作 `RCTBridgeDelegate` 協定的類別，以該委派作為參數初始化 `RCTBridge`，再用初始化後的橋接器建立 `RCTRootView` 來實現。

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 匯出 Swift 模組

由於 Swift 不支援巨集，要在 React Native 中將原生模組及其方法暴露給 JavaScript 需要更多設定步驟，但運作原理基本相同。假設您有一個作為 Swift 類別的相同 `CalendarModule`：

```swift
// CalendarManager.swift

@objc(CalendarManager)
class CalendarManager: NSObject {

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

> 必須使用 `@objc` 修飾符以確保類別和函式能正確導出至 Objective-C 運行時環境。

接著建立一個私有實作檔案，用於向 React Native 註冊必要的資訊：

```objectivec
// CalendarManagerBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarManager, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

對於剛接觸 Swift 和 Objective-C 的開發者，當你在 iOS 專案中[混合使用這兩種語言](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)時，還需要一個額外的橋接檔案（稱為橋接標頭檔）來將 Objective-C 檔案暴露給 Swift。若你透過 Xcode 的「檔案 > 新增檔案」選單將 Swift 檔案加入專案，Xcode 會提示你建立此標頭檔。你需要在該標頭檔中導入 `RCTBridgeModule.h`。

```objectivec
// CalendarManager-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

你也可以使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來變更導出的模組或方法在 JavaScript 中的名稱。更多資訊請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/0.70-stable/React/Base/RCTBridgeModule.h)。

> 開發第三方模組時的重要注意事項：僅 Xcode 9 及以上版本支援包含 Swift 的靜態函式庫。若你在模組中包含的 iOS 靜態函式庫使用 Swift，為了讓 Xcode 專案能成功建置，你的主應用程式專案必須包含 Swift 程式碼和橋接標頭檔。若你的應用程式專案不含任何 Swift 程式碼，解決方法可以是加入一個空的 .swift 檔案和空的橋接標頭檔。

### 保留方法名稱

#### invalidate()

原生模組可透過實作 `invalidate()` 方法，在 iOS 上遵循 [RCTInvalidating](https://github.com/facebook/react-native/blob/0.62-stable/React/Base/RCTInvalidating.h) 協議。當原生橋接失效時（例如：開發模式重新載入時），[可能會呼叫](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456)此方法。請視需求使用此機制來為你的原生模組執行必要的清理工作。