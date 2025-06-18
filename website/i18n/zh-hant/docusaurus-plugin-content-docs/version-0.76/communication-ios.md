---
id: communication-ios
title: Communication between native and React Native
---

In [Integrating with Existing Apps guide](integration-with-existing-apps) and [Native UI Components guide](legacy/native-components-ios) we learn how to embed React Native in a native component and vice versa. When we mix native and React Native components, we'll eventually find a need to communicate between these two worlds. Some ways to achieve that have been already mentioned in other guides. This article summarizes available techniques.

## Introduction

React Native is inspired by React, so the basic idea of the information flow is similar. The flow in React is one-directional. We maintain a hierarchy of components, in which each component depends only on its parent and its own internal state. We do this with properties: data is passed from a parent to its children in a top-down manner. If an ancestor component relies on the state of its descendant, one should pass down a callback to be used by the descendant to update the ancestor.

The same concept applies to React Native. As long as we are building our application purely within the framework, we can drive our app with properties and callbacks. But, when we mix React Native and native components, we need some specific, cross-language mechanisms that would allow us to pass information between them.

## Properties

Properties are the most straightforward way of cross-component communication. So we need a way to pass properties both from native to React Native, and from React Native to native.

### Passing properties from native to React Native

In order to embed a React Native view in a native component, we use `RCTRootView`. `RCTRootView` is a `UIView` that holds a React Native app. It also provides an interface between native side and the hosted app.

`RCTRootView` has an initializer that allows you to pass arbitrary properties down to the React Native app. The `initialProperties` parameter has to be an instance of `NSDictionary`. The dictionary is internally converted into a JSON object that the top-level JS component can reference.

```objectivec
NSArray *imageList = @[@"https://dummyimage.com/600x400/ffffff/000000.png",
                       @"https://dummyimage.com/600x400/000000/ffffff.png"];

NSDictionary *props = @{@"images" : imageList};

RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                 moduleName:@"ImageBrowserApp"
                                          initialProperties:props];
```

```tsx
import React from 'react';
import {View, Image} from 'react-native';

export default class ImageBrowserApp extends React.Component {
  renderImage(imgURI) {
    return <Image source={{uri: imgURI}} />;
  }
  render() {
    return <View>{this.props.images.map(this.renderImage)}</View>;
  }
}
```

`RCTRootView` also provides a read-write property `appProperties`. After `appProperties` is set, the React Native app is re-rendered with new properties. The update is only performed when the new updated properties differ from the previous ones.

```objectivec
NSArray *imageList = @[@"https://dummyimage.com/600x400/ff0000/000000.png",
                       @"https://dummyimage.com/600x400/ffffff/ff0000.png"];

rootView.appProperties = @{@"images" : imageList};
```

It is fine to update properties anytime. However, updates have to be performed on the main thread. You use the getter on any thread.

:::note
Currently, there is a known issue where setting appProperties during the bridge startup, the change can be lost. See https://github.com/facebook/react-native/issues/20115 for more information.
:::

There is no way to update only a few properties at a time. We suggest that you build it into your own wrapper instead.

### Passing properties from React Native to native

The problem exposing properties of native components is covered in detail in [this article](legacy/native-components-ios#properties). In short, export properties with `RCT_CUSTOM_VIEW_PROPERTY` macro in your custom native component, then use them in React Native as if the component was an ordinary React Native component.

### Limits of properties

The main drawback of cross-language properties is that they do not support callbacks, which would allow us to handle bottom-up data bindings. Imagine you have a small RN view that you want to be removed from the native parent view as a result of a JS action. There is no way to do that with props, as the information would need to go bottom-up.

Although we have a flavor of cross-language callbacks ([described here](legacy/native-modules-ios#callbacks)), these callbacks are not always the thing we need. The main problem is that they are not intended to be passed as properties. Rather, this mechanism allows us to trigger a native action from JS, and handle the result of that action in JS.

## Other ways of cross-language interaction (events and native modules)

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要一個更靈活的解決方案。本章涵蓋了 React Native 中可用的其他通訊技術。這些技術既可用於內部通訊（RN 中 JS 與原生層之間的互動），也可用於外部通訊（RN 與應用程式中「純原生」部分之間的互動）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 呼叫自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的環境，實現相同目標的方式有所不同。對於原生端，我們使用事件機制來安排在 JS 中執行處理函數；而對於 React Native，我們則直接呼叫原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](legacy/native-components-ios#events)中有詳細說明。請注意，使用事件無法保證執行時間，因為事件是在單獨的執行緒上處理的。

事件非常強大，因為它們允許我們在不需持有 React Native 元件參考的情況下變更這些元件。然而，使用時可能會遇到一些陷阱：

- 由於事件可以從任何地方發送，它們可能會在專案中引入混亂的依賴關係。
- 事件共享命名空間，這意味著您可能會遇到名稱衝突。這些衝突無法靜態檢測，因此難以除錯。
- 如果您使用多個相同 React Native 元件的實例，並希望從事件的角度區分它們，則可能需要引入標識符並與事件一起傳遞（可以使用原生視圖的 `reactTag` 作為標識符）。

我們在將原生元件嵌入 React Native 時常用的模式是讓原生元件的 RCTViewManager 作為視圖的委託，透過橋接器將事件發送回 JavaScript。這將相關的事件呼叫集中在一個地方。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是可在 JS 中使用的 Objective-C 類別。通常每個模組在每個 JS 橋接器中會建立一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](legacy/native-modules-ios#content)對此有詳細說明。

原生模組是單例的這一事實限制了在嵌入情境下的使用。假設我們有一個嵌入原生視圖中的 React Native 元件，並且我們想要更新原生的父視圖。使用原生模組機制，我們需要匯出一個函數，該函數不僅接受預期的參數，還需要接受父原生視圖的標識符。該標識符將用於取得父視圖的參考以進行更新。也就是說，我們需要在模組中維護一個從標識符到原生視圖的映射。

儘管這個解決方案較為複雜，但它被用於 `RCTUIManager` 中，這是 React Native 內部管理所有 React Native 視圖的類別。

原生模組還可用於將現有的原生函式庫暴露給 JS。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是這個概念的實際範例。

:::caution
所有原生模組共享相同的命名空間。在建立新模組時請注意名稱衝突。
:::

## 佈局計算流程

在整合原生與 React Native 時，我們還需要一種方法來協調兩種不同的佈局系統。本節涵蓋常見的佈局問題，並提供解決這些問題的機制簡要說明。

### 嵌入 React Native 的原生元件佈局

[這篇文章](legacy/native-components-ios#styles)涵蓋了這種情況。總結來說，由於我們所有的原生 React 視圖都是 `UIView` 的子類別，大多數樣式和尺寸屬性都會如預期般運作。

### 嵌入原生端的 React Native 元件佈局

#### 固定尺寸的 React Native 內容

一般情境是當我們有一個尺寸固定的 React Native 應用程式，且原生端已知其尺寸。特別是全螢幕的 React Native 視圖就屬於這種情況。如果我們想要一個較小的根視圖，可以明確設定 RCTRootView 的框架。

舉例來說，若要讓一個 RN 應用程式的高度為 200（邏輯）像素，並與宿主視圖同寬，我們可以這樣做：

```objectivec title='SomeViewController.m'
- (void)viewDidLoad
{
  [...]
  RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                   moduleName:appName
                                            initialProperties:props];
  rootView.frame = CGRectMake(0, 0, self.view.width, 200);
  [self.view addSubview:rootView];
}
```

當我們有一個固定尺寸的根視圖時，需要在 JS 端尊重其邊界。換句話說，我們需要確保 React Native 內容能被包含在固定尺寸的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖產生重疊，導致某些功能表現異常。例如，'TouchableHighlight' 將不會在根視圖邊界外高亮觸控區域。

動態更新根視圖尺寸（透過重新設定其 frame 屬性）是完全可行的，React Native 會自動處理內容的佈局調整。

#### 彈性尺寸的 React Native 內容

某些情況下，我們需要渲染初始尺寸未知的內容。假設尺寸將由 JS 動態決定，我們有兩種解決方案：

1. 可用 `ScrollView` 元件包裹 React Native 視圖，這能確保內容始終可存取且不會與原生視圖重疊。
2. React Native 允許在 JS 中計算 RN 應用的尺寸並提供給宿主 `RCTRootView` 的所有者。所有者需負責重新佈局子視圖以維持 UI 一致性，這透過 `RCTRootView` 的彈性尺寸模式實現。

`RCTRootView` 支援 4 種尺寸彈性模式：

```objectivec title='RCTRootView.h'
typedef NS_ENUM(NSInteger, RCTRootViewSizeFlexibility) {
  RCTRootViewSizeFlexibilityNone = 0,
  RCTRootViewSizeFlexibilityWidth,
  RCTRootViewSizeFlexibilityHeight,
  RCTRootViewSizeFlexibilityWidthAndHeight,
};
```

預設值 `RCTRootViewSizeFlexibilityNone` 使根視圖尺寸固定（但仍可透過 `setFrame:` 更新）。其他三種模式允許追蹤 React Native 內容的尺寸更新。例如，設定模式為 `RCTRootViewSizeFlexibilityHeight` 時，React Native 會測量內容高度並將資訊回傳給 `RCTRootView` 的委派。委派可執行任意操作（包括調整根視圖 frame 以適應內容），且僅在內容尺寸變化時觸發。

:::caution
若在 JS 與原生端同時設定維度為彈性，將導致未定義行為。例如：不應在頂層 React 元件使用 `flexbox` 彈性寬度的同時，對宿主 `RCTRootView` 啟用 `RCTRootViewSizeFlexibilityWidth`。
:::

以下為範例說明：

```objectivec title='FlexibleSizeExampleView.m'
- (instancetype)initWithFrame:(CGRect)frame
{
  [...]

  _rootView = [[RCTRootView alloc] initWithBridge:bridge
  moduleName:@"FlexibilityExampleApp"
  initialProperties:@{}];

  _rootView.delegate = self;
  _rootView.sizeFlexibility = RCTRootViewSizeFlexibilityHeight;
  _rootView.frame = CGRectMake(0, 0, self.frame.size.width, 0);
}

#pragma mark - RCTRootViewDelegate
- (void)rootViewDidChangeIntrinsicSize:(RCTRootView *)rootView
{
  CGRect newFrame = rootView.frame;
  newFrame.size = rootView.intrinsicContentSize;

  rootView.frame = newFrame;
}
```

範例中的 `FlexibleSizeExampleView` 視圖持有根視圖。我們建立並初始化根視圖後設定委派來處理尺寸更新。將根視圖尺寸彈性設為 `RCTRootViewSizeFlexibilityHeight` 意味著每次 React Native 內容高度變化時，`rootViewDidChangeIntrinsicSize:` 方法會被呼叫。最後設定根視圖的寬度與位置（注意高度設定在此無效，因其已改為依賴 RN 計算）。

完整範例程式碼可參閱[此處](https://github.com/facebook/react-native/blob/main/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.mm)。

動態變更根視圖尺寸彈性模式是可行的。變更後會觸發佈局重新計算，並在內容尺寸確定後呼叫委派的 `rootViewDidChangeIntrinsicSize:` 方法。

:::note
React Native 的佈局計算在獨立線程執行，而原生 UI 更新則在主線程進行。
這可能導致原生與 React Native 間的暫時性 UI 不一致。此為已知問題，團隊正致力於同步不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
如果您希望在知道尺寸前隱藏 React Native 視圖，請先將根視圖添加為子視圖並將其設為初始隱藏狀態（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中更改其可見性。
:::