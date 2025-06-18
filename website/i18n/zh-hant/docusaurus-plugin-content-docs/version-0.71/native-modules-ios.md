---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

歡迎來到 iOS 原生模組開發。請先閱讀 [原生模組介紹](native-modules-intro) 了解原生模組的基本概念。

## 建立日曆原生模組

本指南將帶您建立一個名為 `CalendarModule` 的原生模組，該模組可讓您從 JavaScript 存取蘋果的日曆 API。最終您將能透過 JavaScript 呼叫 `CalendarModule.createCalendarEvent('晚餐派對', '我家');` 來觸發建立日曆事件的原生方法。

### 環境設定

首先，請在 Xcode 中開啟 React Native 應用程式的 iOS 專案。您可以在以下路徑找到 iOS 專案：

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

我們建議使用 Xcode 來編寫原生程式碼。Xcode 專為 iOS 開發設計，能幫助您快速解決程式語法等小錯誤。

### 建立自訂原生模組檔案

第一步是建立主要的自訂原生模組標頭檔與實作檔。請建立一個名為 `RCTCalendarModule.h` 的新檔案：

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

您可以根據開發的原生模組特性自由命名。此處命名為 `RCTCalendarModule` 是因為要建立日曆原生模組。由於 Objective-C 不像 Java 或 C++ 具有語言層級的命名空間支援，慣例是在類別名稱前加上前綴字串（例如應用程式名稱或基礎架構名稱的縮寫）。本例中的 RCT 代表 React。

如下所示，CalendarModule 類別實現了 `RCTBridgeModule` 協議。原生模組就是實現了 `RCTBridgeModule` 協議的 Objective-C 類別。

接著開始實作原生模組。請在同一目錄下建立對應的實作檔 `RCTCalendarModule.m`，並包含以下內容：

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### 模組名稱

目前您的 `RCTCalendarModule.m` 原生模組僅包含 `RCT_EXPORT_MODULE` 宏，該宏會將原生模組類別導出並註冊到 React Native。`RCT_EXPORT_MODULE` 宏還可接受一個可選參數，用於指定模組在 JavaScript 程式碼中的存取名稱。

請注意此參數不是字串常值。下方範例中傳遞的是 `RCT_EXPORT_MODULE(CalendarModuleFoo)` 而非 `RCT_EXPORT_MODULE("CalendarModuleFoo")`。

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

之後即可透過以下方式在 JS 中存取該原生模組：

```tsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

若未指定名稱，JavaScript 模組名稱將與 Objective-C 類別名稱相同（自動移除 "RCT" 或 "RK" 前綴）。

請參考下方範例，不帶參數呼叫 `RCT_EXPORT_MODULE`。如此一來，模組將以 `CalendarModule` 名稱暴露給 React Native（這是移除 RCT 前綴後的 Objective-C 類別名稱）。

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

之後即可透過以下方式在 JS 中存取該原生模組：

```tsx
const {CalendarModule} = ReactNative.NativeModules;
```

### 將原生方法導出至 JavaScript

React Native 不會自動將原生模組中的方法暴露給 JavaScript，除非明確告知。這可以通過使用 `RCT_EXPORT_METHOD` 宏來實現。在 `RCT_EXPORT_METHOD` 宏中編寫的方法是異步的，因此返回類型始終為 void。若要將 `RCT_EXPORT_METHOD` 方法的結果傳遞給 JavaScript，可以使用回調或發送事件（後文會介紹）。現在，讓我們使用 `RCT_EXPORT_METHOD` 宏為我們的 `CalendarModule` 原生模組設置一個原生方法。將其命名為 `createCalendarEvent()`，並暫時讓它接收名稱和位置作為字串參數。參數類型的選項將在稍後介紹。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
}
```

> 請注意，除非你的方法依賴於 RCT 參數轉換（參見下方的參數類型），否則 `RCT_EXPORT_METHOD` 宏在 TurboModules 中將不再需要。最終，React Native 會移除 `RCT_EXPORT_MACRO`，因此我們不建議使用 `RCTConvert`。相反，你可以在方法體內進行參數轉換。

在構建 `createCalendarEvent()` 方法的功能之前，先在方法中添加一個控制台日誌，以便確認它已從 JavaScript 調用。使用 React 的 `RCTLog` API。讓我們在文件頂部導入該標頭文件，然後添加日誌調用。

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

該方法的返回類型必須是對象類型（id）並且可序列化為 JSON。這意味著該方法只能返回 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前，我們不建議使用同步方法，因為同步調用方法可能會帶來嚴重的性能損失，並在原生模組中引入與線程相關的錯誤。此外，請注意，如果你選擇使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，你的應用將無法再使用 Google Chrome 調試器。這是因為同步方法要求 JS VM 與應用共享內存。對於 Google Chrome 調試器，React Native 在 Google Chrome 的 JS VM 中運行，並通過 WebSockets 與移動設備進行異步通信。

### 測試你構建的內容

此時，你已經在 iOS 中為原生模組設置了基本的框架。通過訪問原生模組並在 JavaScript 中調用其導出的方法來測試它。

在你的應用中找到一個你想要調用原生模組的 `createCalendarEvent()` 方法的地方。以下是一個組件 `NewModuleButton` 的示例，你可以將其添加到你的應用中。你可以在 `NewModuleButton` 的 `onPress()` 函數中調用原生模組。

```tsx
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

為了從 JavaScript 訪問你的原生模組，你需要首先從 React Native 導入 `NativeModules`：

```tsx
import {NativeModules} from 'react-native';
```

然後，你可以從 `NativeModules` 中訪問 `CalendarModule` 原生模組。

```tsx
const {CalendarModule} = NativeModules;
```

現在，你已經可以使用 `CalendarModule` 原生模組，可以調用你的原生方法 `createCalendarEvent()`。以下是將其添加到 `NewModuleButton` 的 `onPress()` 方法中的示例：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新構建 React Native 應用，以便獲得最新的原生代碼（包含你的新原生模組！）。在你的命令行中，進入 react native 應用所在的目錄，運行以下命令：

```shell
npx react-native run-ios
```

### 迭代構建

當你按照這些指南進行開發並迭代你的原生模組時，你需要重新編譯應用程式才能從 JavaScript 存取最新的變更。這是因為你所編寫的程式碼位於應用程式的原生部分。雖然 React Native 的 Metro 打包工具可以監控 JavaScript 的變更並即時重新打包 JS 套件，但這並不適用於原生程式碼。因此，如果你想測試最新的原生變更，你需要使用 `npx react-native run-ios` 指令重新編譯。

### 回顧✨

你現在應該能夠在 JavaScript 中呼叫原生模組的 `createCalendarEvent()` 方法。由於你在函式中使用了 `RCTLog`，你可以透過[在應用程式中啟用除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)並查看 Chrome 的 JS 控制台或行動應用除錯工具 Flipper 來確認原生方法是否被呼叫。每次呼叫原生模組方法時，你都應該會看到 `RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);` 的訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此，你已經建立了一個 iOS 原生模組，並從 React Native 應用程式的 JavaScript 中呼叫了它的方法。你可以繼續閱讀，了解更多關於原生模組方法的參數類型，以及如何在原生模組中設定回調和 Promise 的內容。

## 超越日曆原生模組

### 更好的原生模組匯出方式

像上面那樣從 `NativeModules` 中提取原生模組的方式有點笨拙。

為了讓你的原生模組的使用者不必每次存取原生模組時都這樣做，你可以為模組建立一個 JavaScript 包裝器。建立一個名為 NativeCalendarModule.js 的新 JavaScript 檔案，內容如下：

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

這個 JavaScript 檔案也是你新增 JavaScript 端功能的理想位置。例如，如果你使用像 TypeScript 這樣的類型系統，你可以在這裡為你的原生模組新增類型註解。雖然 React Native 目前還不支援原生到 JS 的類型安全，但有了這些類型註解，你的所有 JS 程式碼都將是類型安全的。這些註解也將幫助你在未來更容易地切換到類型安全的原生模組。以下是一個為日曆模組新增類型安全的範例：

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

在你的其他 JavaScript 檔案中，你可以這樣存取原生模組並呼叫其方法：

```tsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意，這假設你匯入 `CalendarModule` 的位置與 `NativeCalendarModule.js` 位於相同的層級。請根據需要更新相對匯入路徑。

### 參數類型

當在 JavaScript 中呼叫原生模組方法時，React Native 會將參數從 JS 物件轉換為對應的 Objective-C/Swift 物件。因此，舉例來說，如果你的 Objective-C 原生模組方法接受一個 NSNumber，在 JS 中你需要用數字呼叫該方法。React Native 會為你處理轉換。以下是原生模組方法支援的參數類型及其對應的 JavaScript 等效類型的列表。

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

> 以下類型目前受支援，但在 TurboModules 中將不再支援。請避免使用它們。
>
> - Function (failure) -> RCTResponseErrorBlock
> - Number -> NSInteger
> - Number -> CGFloat
> - Number -> float

對於 iOS，你也可以使用 `RCTConvert` 類支援的任何參數類型來編寫原生模組方法（詳見 [RCTConvert](https://github.com/facebook/react-native/blob/0.71-stable/React/Base/RCTConvert.h) 以了解支援的內容）。RCTConvert 輔助函式都接受一個 JSON 值作為輸入，並將其映射到原生的 Objective-C 類型或類別。

### 匯出常數

原生模組可以透過覆寫原生方法 `constantsToExport()` 來匯出常數。以下覆寫了 `constantsToExport()`，並返回一個字典，其中包含一個你可以在 JavaScript 中存取的預設事件名稱屬性，如下所示：

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 來存取該常數：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

技術上來說，直接從 `NativeModule` 物件存取 `constantsToExport()` 匯出的常數是可行的。但這在 TurboModules 中將不再支援，因此我們鼓勵開發社群改用上述方式，以避免未來不必要的遷移工作。

> 請注意常數僅在初始化時匯出，若在執行階段變更 `constantsToExport()` 的值，將不會影響 JavaScript 環境。

在 iOS 上，若您覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup` 來告知 React Native 您的模組是否需要先於任何 JavaScript 程式碼執行前，在主執行緒上初始化。否則您會看到警告，提示未來您的模組可能會在背景執行緒初始化，除非您明確透過 `+ requiresMainQueueSetup:` 選擇退出。若您的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式(callback)。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞到 JavaScript，也可用於從原生端非同步執行 JavaScript。

在 iOS 中，回呼函式是透過 `RCTResponseSenderBlock` 類型實作的。以下將回呼參數 `myCallBack` 加入 `createCalendarEventMethod()`：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著您可以在原生函式中呼叫回呼函式，透過陣列傳遞任何想傳給 JavaScript 的結果。請注意 `RCTResponseSenderBlock` 僅接受一個參數——傳遞給 JavaScript 回呼函式的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需特別強調的是，回呼函式並非在原生函式完成後立即呼叫——請記住通訊是非同步進行的。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

接著可透過以下方式在 JavaScript 中存取此方法：

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

原生模組應僅呼叫其回呼函式一次。不過，它可以儲存回呼函式並稍後呼叫。此模式常用於封裝需要委派(delegate)的 iOS API——參見 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/3a11f0536ea65b87dc0f006665f16a87cfa14b5e/React/CoreModules/RCTAlertManager.mm) 的範例。若回呼函式從未被呼叫，會導致部分記憶體洩漏。

回呼函式的錯誤處理有兩種方式。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，您可以檢查第一個參數來判斷是否有錯誤傳入：

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

另一種選項是使用兩個獨立的回呼函式：onFailure 和 onSuccess。

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

接著在 JavaScript 中可為錯誤和成功回應分別加入回呼函式：

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

若要將類錯誤物件傳遞給 JavaScript，請使用 [`RCTUtils.h`](https://github.com/facebook/react-native/blob/0.71-stable/React/Base/RCTUtils.h) 中的 `RCTMakeError`。目前這僅會將 Error 形態的字典傳給 JavaScript，但 React Native 目標是未來能自動產生真正的 JavaScript Error 物件。您也可以提供 `RCTResponseErrorBlock` 參數，用於錯誤回呼並接受 `NSError *` 物件。請注意此參數類型在 TurboModules 中將不受支援。

### Promise

原生模組也能實現 Promise，這可簡化您的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳 JS Promise 物件。

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

該方法在 JavaScript 端的對應會回傳一個 Promise。這意味著您可以在非同步函數中使用 `await` 關鍵字來呼叫它並等待其結果：

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

原生模組可以直接觸發事件通知 JavaScript，而無需被直接呼叫。例如，您可能希望通知 JavaScript 原生 iOS 日曆應用中的某個日曆事件即將發生。推薦的做法是繼承 `RCTEventEmitter`、實作 `supportedEvents` 並呼叫 `self sendEventWithName`：

更新您的標頭檔類別以導入 `RCTEventEmitter` 並繼承 `RCTEventEmitter`：

```objectivec
//  CalendarModule.h

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

```

JavaScript 程式碼可以透過建立一個圍繞您模組的 `NativeEventEmitter` 實例來訂閱這些事件。

若在沒有監聽器的情況下發送事件，您會收到警告。為避免此情況並優化模組工作負載（例如取消訂閱上游通知或暫停背景任務），您可以在 `RCTEventEmitter` 子類別中覆寫 `startObserving` 和 `stopObserving` 方法。

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
  if (hasListeners) {// Only send events if anyone is listening
    [self sendEventWithName:@"EventReminder" body:@{@"name": eventName}];
  }
}

```

### 執行緒處理

除非原生模組提供自己的方法佇列，否則不應對呼叫執行緒做出任何假設。目前若未提供方法佇列，React Native 會為其建立獨立的 GCD 佇列並在其中呼叫方法。請注意這是實作細節且可能變更。若要明確提供方法佇列，請在原生模組中覆寫 `(dispatch_queue_t) methodQueue` 方法。例如需使用僅限主執行緒的 iOS API 時：

```objectivec
- (dispatch_queue_t)methodQueue
{
  return dispatch_get_main_queue();
}
```

同樣地，若操作可能耗時，原生模組可指定自己的執行佇列。再次強調，目前 React Native 會為您的模組提供獨立方法佇列，但這是不應依賴的實作細節。若未提供自訂佇列，未來長時間運算可能阻塞其他無關模組的非同步呼叫。例如 `RCTAsyncLocalStorage` 模組建立自己的佇列，避免 React 佇列被潛在的慢速磁碟存取阻塞。

```objectivec
- (dispatch_queue_t)methodQueue
{
 return dispatch_queue_create("com.facebook.React.AsyncLocalStorageQueue", DISPATCH_QUEUE_SERIAL);
}
```

指定的 `methodQueue` 會被模組中所有方法共享。若僅單一方法需長時間執行（或需在不同佇列運行），可在方法內使用 `dispatch_async` 將特定程式碼移至其他佇列執行，不影響其他方法：

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
> `methodQueue` 方法會在模組初始化時被呼叫一次，之後由 React Native 保留，因此除非需在模組內部使用，否則無需自行保留佇列參考。但若需在多個模組間共享相同佇列，則必須確保為每個模組保留並回傳相同的佇列實例。

### 依賴注入

React Native 會自動建立並初始化所有註冊的原生模組。但您可能希望自行建立模組實例以實現例如依賴注入等功能。

可透過建立實作 `RCTBridgeDelegate` 協定的類別，以該委派作為參數初始化 `RCTBridge`，再用初始化後的橋接器建立 `RCTRootView` 來達成：

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 匯出 Swift 模組

由於 Swift 不支援巨集，要在 React Native 中將原生模組及其方法公開給 JavaScript 需要更多設定步驟，但基本原理相同。假設您有相同的 `CalendarModule` 但改為 Swift 類別：

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

> 務必使用 `@objc` 修飾符，以確保類別和函式能正確導出至 Objective-C 運行環境。

接著建立一個私有實作檔案，用於向 React Native 註冊必要的資訊：

```objectivec
// CalendarManagerBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarManager, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

對於初次接觸 Swift 與 Objective-C 的開發者，當你在 iOS 專案中[混合使用這兩種語言](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)時，還需要一個額外的橋接檔案（稱為橋接標頭檔）來將 Objective-C 檔案暴露給 Swift。若你透過 Xcode 的 `檔案>新增檔案` 選單將 Swift 檔案加入專案，Xcode 會提示你建立此標頭檔。你需要在該標頭檔中導入 `RCTBridgeModule.h`。

```objectivec
// CalendarManager-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

你也可以使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來調整導出模組或方法在 JavaScript 中的名稱。更多資訊請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/0.71-stable/React/Base/RCTBridgeModule.h)。

> 開發第三方模組時的重要注意事項：僅 Xcode 9 及以上版本支援包含 Swift 的靜態函式庫。若你在模組的 iOS 靜態函式庫中使用 Swift，為了讓 Xcode 專案能成功建置，你的主應用程式專案必須包含 Swift 程式碼和橋接標頭檔。若應用程式專案不含任何 Swift 程式碼，可透過建立一個空的 .swift 檔案和空白橋接標頭檔作為臨時解決方案。

### 保留方法名稱

#### invalidate()

iOS 平台的原生模組可透過實作 `invalidate()` 方法來遵循 [RCTInvalidating](https://github.com/facebook/react-native/blob/0.62-stable/React/Base/RCTInvalidating.h) 協議。當原生橋接失效時（例如：開發模式重新載入），[系統可能會呼叫](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456)此方法。請視需求使用此機制來為你的原生模組執行必要的清理工作。