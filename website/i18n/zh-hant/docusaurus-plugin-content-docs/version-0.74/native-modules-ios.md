---
id: native-modules-ios
title: iOS Native Modules
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<NativeDeprecated />

Welcome to Native Modules for iOS. Please start by reading the [Native Modules Intro](native-modules-intro) for an intro to what native modules are.

## Create a Calendar Native Module

In the following guide you will create a native module, `CalendarModule`, that will allow you to access Apple's calendar APIs from JavaScript. By the end you will be able to call `CalendarModule.createCalendarEvent('Dinner Party', 'My House');` from JavaScript, invoking a native method that creates a calendar event.

### Setup

To get started, open up the iOS project within your React Native application in Xcode. You can find your iOS project here within a React Native app:

<figure>
  <img src="/docs/assets/native-modules-ios-open-project.png" width="500" alt="Image of opening up an iOS project within a React Native app inside of xCode." />
  <figcaption>Image of where you can find your iOS project</figcaption>
</figure>

We recommend using Xcode to write your native code. Xcode is built for iOS development, and using it will help you to quickly resolve smaller errors like code syntax.

### Create Custom Native Module Files

The first step is to create our main custom native module header and implementation files. Create a new file called `RCTCalendarModule.h`

<figure>
  <img src="/docs/assets/native-modules-ios-add-class.png" width="500" alt="Image of creating a class called  RCTCalendarModule.h." />
  <figcaption>Image of creating a custom native module file within the same folder as AppDelegate</figcaption>
</figure>

and add the following to it:

```objectivec
//  RCTCalendarModule.h
#import <React/RCTBridgeModule.h>
@interface RCTCalendarModule : NSObject <RCTBridgeModule>
@end

```

You can use any name that fits the native module you are building. Name the class `RCTCalendarModule` since you are creating a calendar native module. Since ObjC does not have language-level support for namespaces like Java or C++, convention is to prepend the class name with a substring. This could be an abbreviation of your application name or your infra name. RCT, in this example, refers to React.

As you can see below, the CalendarModule class implements the `RCTBridgeModule` protocol. A native module is an Objective-C class that implements the `RCTBridgeModule` protocol.

Next up, let’s start implementing the native module. Create the corresponding implementation file using cocoa touch class in xcode, `RCTCalendarModule.m`, in the same folder and include the following content:

```objectivec
// RCTCalendarModule.m
#import "RCTCalendarModule.h"

@implementation RCTCalendarModule

// To export a module named RCTCalendarModule
RCT_EXPORT_MODULE();

@end

```

### Module Name

For now, your `RCTCalendarModule.m` native module only includes a `RCT_EXPORT_MODULE` macro, which exports and registers the native module class with React Native. The `RCT_EXPORT_MODULE` macro also takes an optional argument that specifies the name that the module will be accessible as in your JavaScript code.

This argument is not a string literal. In the example below `RCT_EXPORT_MODULE(CalendarModuleFoo)` is passed, not `RCT_EXPORT_MODULE("CalendarModuleFoo")`.

```objectivec
// To export a module named CalendarModuleFoo
RCT_EXPORT_MODULE(CalendarModuleFoo);
```

The native module can then be accessed in JS like this:

```tsx
const {CalendarModuleFoo} = ReactNative.NativeModules;
```

If you do not specify a name, the JavaScript module name will match the Objective-C class name, with any "RCT" or "RK" prefixes removed.

Let's follow the example below and call `RCT_EXPORT_MODULE` without any arguments. As a result, the module will be exposed to React Native using the name `CalendarModule`, since that is the Objective-C class name, with RCT removed.

```objectivec
// Without passing in a name this will export the native module name as the Objective-C class name with “RCT” removed
RCT_EXPORT_MODULE();
```

The native module can then be accessed in JS like this:

```tsx
const {CalendarModule} = ReactNative.NativeModules;
```

### Export a Native Method to JavaScript

React Native 不會自動將原生模組中的方法暴露給 JavaScript，除非明確使用 `RCT_EXPORT_METHOD` 巨集進行宣告。透過 `RCT_EXPORT_METHOD` 編寫的方法都是非同步的，因此回傳類型永遠是 void。若要將 `RCT_EXPORT_METHOD` 方法的結果傳遞給 JavaScript，可以使用回調函數或發送事件（後續會說明）。現在我們使用 `RCT_EXPORT_METHOD` 巨集為 `CalendarModule` 原生模組建立一個原生方法，命名為 `createCalendarEvent()`，暫時讓它接收名稱和位置兩個字串參數，參數類型的選項稍後會詳細說明。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
}
```

> 請注意，除非你的方法依賴 RCT 參數轉換（參見下方的參數類型說明），否則 TurboModules 將不需要使用 `RCT_EXPORT_METHOD` 巨集。最終 React Native 會移除 `RCT_EXPORT_MACRO`，因此我們不建議使用 `RCTConvert`。你可以在方法體內自行處理參數轉換。

在完善 `createCalendarEvent()` 方法的功能之前，先在方法內添加一個 console log，以便確認它已從 React Native 應用的 JavaScript 端被調用。使用 React 提供的 `RCTLog` API。首先在檔案頂部引入該標頭檔，然後添加 log 呼叫。

```objectivec
#import <React/RCTLog.h>
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)name location:(NSString *)location)
{
 RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);
}
```

### 同步方法

你可以使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD` 來建立同步的原生方法。

```objectivec
RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD(getName)
{
return [[UIDevice currentDevice] name];
}
```

該方法的回傳類型必須是物件類型（id）且可序列化為 JSON。這意味著該方法只能回傳 nil 或 JSON 值（例如 NSNumber、NSString、NSArray、NSDictionary）。

目前我們不建議使用同步方法，因為同步呼叫可能導致嚴重的效能問題，並在原生模組中引入與線程相關的錯誤。此外，請注意若使用 `RCT_EXPORT_BLOCKING_SYNCHRONOUS_METHOD`，你的應用將無法使用 Google Chrome 調試器。這是因為同步方法要求 JS 虛擬機與應用共享記憶體，而 Google Chrome 調試器中，React Native 運行在 Chrome 的 JS 虛擬機內，並透過 WebSockets 與移動設備進行非同步通訊。

### 測試已建構的功能

至此你已完成 iOS 原生模組的基礎架構。現在可以透過 JavaScript 存取原生模組並呼叫其導出的方法來進行測試。

在應用中找到適合呼叫原生模組 `createCalendarEvent()` 方法的位置。以下是一個範例元件 `NewModuleButton`，你可以將其添加到應用中，並在 `onPress()` 函數內呼叫原生模組。

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

要從 JavaScript 存取原生模組，首先需要從 React Native 導入 `NativeModules`：

```tsx
import {NativeModules} from 'react-native';
```

接著可以從 `NativeModules` 中取得 `CalendarModule` 原生模組。

```tsx
const {CalendarModule} = NativeModules;
```

現在你可以透過 `CalendarModule` 原生模組呼叫原生方法 `createCalendarEvent()`。以下是將其添加到 `NewModuleButton` 的 `onPress()` 方法中的範例：

```tsx
const onPress = () => {
  CalendarModule.createCalendarEvent('testName', 'testLocation');
};
```

最後一步是重新建構 React Native 應用，以確保最新的原生程式碼（包含你的新原生模組！）已就緒。在 react native 應用所在的命令列中執行以下指令：

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

當你依照這些指南進行開發並迭代你的原生模組時，你需要對應用程式進行原生重建，才能從 JavaScript 存取最新的變更。這是因為你所編寫的程式碼位於應用程式的原生部分。雖然 React Native 的 Metro 打包工具可以監控 JavaScript 的變更並即時重建 JS 套件，但它不會對原生程式碼這麼做。因此，如果你想測試最新的原生變更，就需要使用上述命令進行重建。

### 回顧✨

你現在應該能夠在 JavaScript 中呼叫原生模組的 `createCalendarEvent()` 方法了。由於你在函式中使用了 `RCTLog`，你可以透過[在應用程式中啟用除錯模式](https://reactnative.dev/docs/debugging#chrome-developer-tools)並查看 Chrome 的 JS 控制台或行動應用除錯工具 Flipper 來確認原生方法是否被呼叫。每次呼叫原生模組方法時，你都應該會看到 `RCTLogInfo(@"Pretending to create an event %@ at %@", name, location);` 的訊息。

<figure>
  <img src="/docs/assets/native-modules-ios-logs.png" width="1000" alt="Image of logs." />
  <figcaption>Image of iOS logs in Flipper</figcaption>
</figure>

至此，你已經建立了一個 iOS 原生模組，並從 React Native 應用程式的 JavaScript 中呼叫了它的方法。你可以繼續閱讀，了解更多關於原生模組方法接受的參數類型，以及如何在原生模組中設定回調和 Promise 的內容。

## 超越日曆原生模組

### 更好的原生模組匯出方式

像上面那樣從 `NativeModules` 中拉取原生模組的方式有點笨拙。

為了讓你的原生模組的使用者不必每次存取原生模組時都這麼做，你可以為模組建立一個 JavaScript 包裝器。建立一個名為 NativeCalendarModule.js 的新 JavaScript 檔案，內容如下：

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

這個 JavaScript 檔案也成為你新增 JavaScript 端功能的理想位置。例如，如果你使用像 TypeScript 這樣的類型系統，你可以在這裡為你的原生模組新增類型註解。雖然 React Native 目前還不支援原生到 JS 的類型安全，但有了這些類型註解，你所有的 JS 程式碼都將是類型安全的。這些註解也會讓你在未來切換到類型安全的原生模組時更加容易。以下是一個為日曆模組新增類型安全的範例：

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

在你的其他 JavaScript 檔案中，你可以這樣存取原生模組並呼叫它的方法：

```tsx
import NativeCalendarModule from './NativeCalendarModule';
NativeCalendarModule.createCalendarEvent('foo', 'bar');
```

> 注意：這假設你匯入 `CalendarModule` 的位置與 `NativeCalendarModule.js` 位於相同的層級。請根據需要更新相對匯入路徑。

### 參數類型

當在 JavaScript 中呼叫原生模組方法時，React Native 會將參數從 JS 物件轉換為它們對應的 Objective-C/Swift 物件。因此，舉例來說，如果你的 Objective-C 原生模組方法接受一個 NSNumber，在 JS 中你需要用一個數字來呼叫該方法。React Native 會為你處理轉換。以下列出了原生模組方法支援的參數類型及其對應的 JavaScript 等效類型。

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

對於 iOS，你也可以使用 `RCTConvert` 類別支援的任何參數類型來編寫原生模組方法（詳見 [RCTConvert](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTConvert.h) 以了解支援的內容）。RCTConvert 輔助函式都接受一個 JSON 值作為輸入，並將其映射到原生的 Objective-C 類型或類別。

### 匯出常數

原生模組可以透過覆寫原生方法 `constantsToExport()` 來匯出常數。以下覆寫了 `constantsToExport()`，並返回一個字典，其中包含一個預設事件名稱屬性，你可以在 JavaScript 中這樣存取：

```objectivec
- (NSDictionary *)constantsToExport
{
 return @{ @"DEFAULT_EVENT_NAME": @"New Event" };
}
```

接著可以透過在 JavaScript 中呼叫原生模組的 `getConstants()` 方法來存取這個常數：

```tsx
const {DEFAULT_EVENT_NAME} = CalendarModule.getConstants();
console.log(DEFAULT_EVENT_NAME);
```

技術上來說，直接從 `NativeModule` 物件存取 `constantsToExport()` 匯出的常數是可行的。但這在 TurboModules 中將不再支援，因此我們鼓勵社群改用上述方法，以避免未來不必要的遷移工作。

> 請注意，常數僅在初始化時匯出，因此若在執行階段變更 `constantsToExport()` 的值，將不會影響 JavaScript 環境。

在 iOS 上，若您覆寫了 `constantsToExport()`，則應同時實作 `+ requiresMainQueueSetup` 來告知 React Native 您的模組是否需要先於任何 JavaScript 程式碼執行前在主執行緒初始化。否則您會看到警告，提示未來您的模組可能會在背景執行緒初始化，除非您明確透過 `+ requiresMainQueueSetup:` 選擇退出。若您的模組不需要存取 UIKit，則應對 `+ requiresMainQueueSetup` 回傳 NO。

### 回呼函式

原生模組還支援一種特殊的參數類型——回呼函式(callback)。回呼函式用於在非同步方法中將資料從 Objective-C 傳遞至 JavaScript，也可用於從原生端非同步執行 JavaScript。

在 iOS 中，回呼函式是透過 `RCTResponseSenderBlock` 類型實作的。以下範例將回呼參數 `myCallBack` 加入 `createCalendarEventMethod()`：

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title
                location:(NSString *)location
                myCallback:(RCTResponseSenderBlock)callback)

```

接著您可以在原生函式中呼叫回呼函式，並透過陣列傳遞任何想傳給 JavaScript 的結果。請注意 `RCTResponseSenderBlock` 僅接受一個參數——傳遞給 JavaScript 回呼函式的參數陣列。以下範例將回傳先前呼叫中建立的事件 ID。

> 需特別強調的是，回呼函式不會在原生函式完成後立即被呼叫——請記住這種通訊是非同步的。

```objectivec
RCT_EXPORT_METHOD(createCalendarEvent:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
 NSInteger eventId = ...
 callback(@[@(eventId)]);

 RCTLogInfo(@"Pretending to create an event %@ at %@", title, location);
}

```

接著可以透過以下方式在 JavaScript 中存取這個方法：

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

原生模組應僅呼叫其回呼函式一次。不過，它可以儲存回呼函式並稍後再呼叫。這種模式常用於封裝需要委派(delegate)的 iOS API——參見 [`RCTAlertManager`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/CoreModules/RCTAlertManager.mm) 的範例。若回呼函式從未被呼叫，將會導致部分記憶體洩漏。

使用回呼函式進行錯誤處理有兩種方法。第一種是遵循 Node 的慣例，將傳遞給回呼陣列的第一個參數視為錯誤物件。

```objectivec
RCT_EXPORT_METHOD(createCalendarEventCallback:(NSString *)title location:(NSString *)location callback: (RCTResponseSenderBlock)callback)
{
  NSNumber *eventId = [NSNumber numberWithInt:123];
  callback(@[[NSNull null], eventId]);
}
```

在 JavaScript 中，您可以檢查第一個參數來判斷是否傳遞了錯誤：

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

接著在 JavaScript 中可以為錯誤和成功回應分別添加回呼函式：

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

若要將類似錯誤的物件傳遞給 JavaScript，請使用 [`RCTUtils.h.`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTUtils.h) 中的 `RCTMakeError`。目前這僅會將 Error 形狀的字典傳遞給 JavaScript，但 React Native 的目標是在未來自動產生真正的 JavaScript Error 物件。您也可以提供 `RCTResponseErrorBlock` 參數，用於錯誤回呼並接受 `NSError \* 物件`。請注意此參數類型在 TurboModules 中將不受支援。

### Promise

原生模組也可以實現 Promise，這能簡化您的 JavaScript 程式碼，特別是在使用 ES2016 的 `async/await` 語法時。當原生模組方法的最後一個參數是 `RCTPromiseResolveBlock` 和 `RCTPromiseRejectBlock` 時，其對應的 JS 方法將回傳一個 JS Promise 物件。

將上述程式碼重構為使用 Promise 而非回呼函式的範例如下：

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

該方法的 JavaScript 對應部分會回傳一個 Promise。這意味著您可以在 async 函數中使用 `await` 關鍵字來調用它並等待其結果：

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

原生模組可以直接向 JavaScript 發送事件信號，而無需被直接調用。例如，您可能希望向 JavaScript 發出信號，提醒原生 iOS 日曆應用中的某個日曆事件即將發生。推薦的做法是繼承 `RCTEventEmitter`，實現 `supportedEvents` 並調用 `self sendEventWithName`：

更新您的頭文件類以導入 `RCTEventEmitter` 並繼承 `RCTEventEmitter`：

```objectivec
//  CalendarModule.h

#import <React/RCTBridgeModule.h>
#import <React/RCTEventEmitter.h>

@interface CalendarModule : RCTEventEmitter <RCTBridgeModule>
@end

```

JavaScript 代碼可以通過在您的模組周圍創建一個新的 `NativeEventEmitter` 實例來訂閱這些事件。

如果在沒有監聽器的情況下發送事件，您將收到警告。為了避免這種情況，並優化模組的工作負載（例如通過取消訂閱上游通知或暫停後台任務），您可以在 `RCTEventEmitter` 子類中覆寫 `startObserving` 和 `stopObserving` 方法。

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

### 線程處理

除非原生模組提供了自己的方法隊列，否則它不應對自己被調用的線程做出任何假設。目前，如果原生模組沒有提供方法隊列，React Native 會為其創建一個單獨的 GCD 隊列並在那裡調用其方法。請注意，這是一個實現細節，可能會發生變化。如果您想為原生模組明確提供一個方法隊列，請在原生模組中覆寫 `(dispatch_queue_t) methodQueue` 方法。例如，如果需要使用僅限主線程的 iOS API，則應通過以下方式指定：

```objectivec
- (dispatch_queue_t)methodQueue
{
  return dispatch_get_main_queue();
}
```

同樣，如果某個操作可能需要很長時間才能完成，原生模組可以指定自己的隊列來運行操作。再次強調，目前 React Native 會為您的原生模組提供一個單獨的方法隊列，但這是一個您不應依賴的實現細節。如果您不提供自己的方法隊列，將來您的原生模組的長時間運行操作可能會阻塞在其他不相關原生模組上執行的異步調用。例如，這裡的 `RCTAsyncLocalStorage` 模組創建了自己的隊列，這樣 React 隊列就不會被潛在的慢速磁盤訪問阻塞。

```objectivec
- (dispatch_queue_t)methodQueue
{
 return dispatch_queue_create("com.facebook.React.AsyncLocalStorageQueue", DISPATCH_QUEUE_SERIAL);
}
```

指定的 `methodQueue` 將由模組中的所有方法共享。如果只有一個方法是長時間運行的（或由於某些原因需要與其他方法在不同的隊列上運行），您可以在方法內部使用 `dispatch_async` 將該特定方法的代碼放在另一個隊列上執行，而不影響其他方法：

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

> 在模組之間共享調度隊列
>
> `methodQueue` 方法將在模組初始化時調用一次，然後由 React Native 保留，因此除非您希望在模組內部使用該隊列，否則無需自己保留對隊列的引用。但是，如果您希望在多個模組之間共享同一個隊列，則需要確保為每個模組保留並返回相同的隊列實例。

### 依賴注入

React Native 會自動創建並初始化所有已註冊的原生模組。但是，您可能希望創建並初始化自己的模組實例，以便注入依賴項。

您可以通過創建一個實現 `RCTBridgeDelegate` 協議的類，使用該委託作為參數初始化一個 `RCTBridge`，然後使用初始化的橋接器初始化一個 `RCTRootView` 來實現這一點。

```objectivec
id<RCTBridgeDelegate> moduleInitialiser = [[classThatImplementsRCTBridgeDelegate alloc] init];

RCTBridge *bridge = [[RCTBridge alloc] initWithDelegate:moduleInitialiser launchOptions:nil];

RCTRootView *rootView = [[RCTRootView alloc]
                        initWithBridge:bridge
                            moduleName:kModuleName
                     initialProperties:nil];
```

### 導出 Swift

Swift 不支持宏，因此將原生模組及其方法暴露給 React Native 中的 JavaScript 需要更多的設置。不過，其工作原理相對相同。假設您有一個相同的 `CalendarModule`，但作為 Swift 類：

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

> 務必使用 `@objc` 修飾符，以確保類別和函式能正確導出至 Objective-C 運行環境。

接著建立一個私有實作檔案，用於向 React Native 註冊必要資訊：

```objectivec
// CalendarModuleBridge.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarModule, NSObject)

RCT_EXTERN_METHOD(addEvent:(NSString *)name location:(NSString *)location date:(nonnull NSNumber *)date)

@end
```

若您初次接觸 Swift 與 Objective-C，請注意當您在 iOS 專案中[混用這兩種語言](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/MixandMatch.html)時，需額外建立橋接標頭檔（bridging header）來將 Objective-C 檔案暴露給 Swift。若您透過 Xcode 的 `檔案>新增檔案` 選單加入 Swift 檔案，Xcode 會主動提示建立此標頭檔。您需在此標頭檔中導入 `RCTBridgeModule.h`。

```objectivec
// CalendarModule-Bridging-Header.h
#import <React/RCTBridgeModule.h>
```

您亦可使用 `RCT_EXTERN_REMAP_MODULE` 和 `_RCT_EXTERN_REMAP_METHOD` 來調整導出模組或方法在 JavaScript 中的名稱。詳情請參閱 [`RCTBridgeModule`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridgeModule.h)。

> 開發第三方模組的重要注意事項：僅 Xcode 9 及以上版本支援含 Swift 的靜態函式庫。若您在模組中包含的 iOS 靜態函式庫使用 Swift，主應用專案必須包含 Swift 程式碼及橋接標頭檔才能成功建置 Xcode 專案。若應用專案不含任何 Swift 程式碼，可暫時以單一空白 .swift 檔案與空白橋接標頭檔作為解決方案。

### 保留方法名稱

#### invalidate()

iOS 平台的原生模組可透過實作 `invalidate()` 方法來遵循 [RCTInvalidating](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTInvalidating.h) 協議。當原生橋接失效時（例如：開發模式重新載入），[系統可能調用](https://github.com/facebook/react-native/blob/0.62-stable/ReactCommon/turbomodule/core/platform/ios/RCTTurboModuleManager.mm#L456)此方法。請視需求運用此機制執行原生模組所需的清理工作。