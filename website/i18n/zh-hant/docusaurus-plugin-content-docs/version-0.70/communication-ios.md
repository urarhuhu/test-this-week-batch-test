---
id: communication-ios
title: Communication between native and React Native
---

在[與現有應用整合指南](integration-with-existing-apps)和[原生UI元件指南](native-components-ios)中，我們學習了如何將React Native嵌入原生元件，反之亦然。當我們混合使用原生與React Native元件時，最終會需要在這兩個世界之間建立溝通管道。其他指南中已提及部分實現方法，本文將總結現有技術方案。

## 簡介

React Native的設計靈感源自React，因此資訊流的基本概念相似。React中的資料流是單向的，我們維護一個元件層級結構，其中每個元件僅依賴其父元件和自身內部狀態。這通過屬性(properties)實現：資料以自上而下的方式從父元件傳遞給子元件。若祖先元件需要依賴後代元件的狀態，則應向下傳遞回調函數供後代元件更新祖先狀態。

相同概念適用於React Native。只要我們完全在框架內構建應用程式，就可以通過屬性和回調驅動應用。但當混合使用React Native與原生元件時，我們需要特定的跨語言機制來實現它們之間的資訊傳遞。

## 屬性

屬性是跨元件通信最直接的方式。因此我們需要實現從原生到React Native，以及從React Native到原生的雙向屬性傳遞。

### 從原生傳遞屬性到React Native

要在原生元件中嵌入React Native視圖，我們使用`RCTRootView`。`RCTRootView`是承載React Native應用的`UIView`，同時也提供原生端與宿主應用之間的接口。

`RCTRootView`的初始化方法允許您向React Native應用傳遞任意屬性。`initialProperties`參數必須是`NSDictionary`實例，該字典會在內部轉換為頂層JS元件可引用的JSON物件。

```objectivec
NSArray *imageList = @[@"http://foo.com/bar1.png",
                       @"http://foo.com/bar2.png"];

NSDictionary *props = @{@"images" : imageList};

RCTRootView *rootView = [[RCTRootView alloc] initWithBridge:bridge
                                                 moduleName:@"ImageBrowserApp"
                                          initialProperties:props];
```

```jsx
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

`RCTRootView`還提供可讀寫屬性`appProperties`。設置`appProperties`後，React Native應用會使用新屬性重新渲染。僅當新屬性與先前屬性不同時才會執行更新。

```objectivec
NSArray *imageList = @[@"http://foo.com/bar3.png",
                       @"http://foo.com/bar4.png"];

rootView.appProperties = @{@"images" : imageList};
```

隨時更新屬性是可接受的，但更新操作必須在主線程執行。而讀取操作可在任意線程進行。

:::note
請注意目前存在已知問題：若在橋接器啟動期間設置appProperties，更改可能會丟失。詳見https://github.com/facebook/react-native/issues/20115。
:::

無法實現部分屬性的局部更新。我們建議您自行構建封裝層來處理此需求。

### 從React Native傳遞屬性到原生

原生元件屬性的暴露問題在[本文](native-components-ios#properties)中有詳細說明。簡而言之，在自訂原生元件中使用`RCT_CUSTOM_VIEW_PROPERTY`宏導出屬性，即可在React Native中像使用普通React Native元件屬性那樣使用它們。

### 屬性的限制

跨語言屬性的主要缺陷是不支援回調機制，而回調是實現自下而上資料綁定的關鍵。例如當您需要通過JS操作移除原生父視圖中的某個小型RN視圖時，純屬性方案無法實現，因為這需要資訊自下而上傳遞。

雖然我們有一種跨語言回調方案([參見此處](native-modules-ios#callbacks))，但這些回調並非萬能解藥。主要問題在於它們不適合作為屬性傳遞。該機制實際實現的是從JS觸發原生操作，並在JS中處理操作結果。

## 其他跨語言交互方式（事件與原生模組）

如前一章所述，使用屬性存在一些限制。有時屬性不足以驅動應用程式的邏輯，我們需要更靈活的解決方案。本章涵蓋 React Native 中可用的其他通訊技術，這些技術可用於內部通訊（RN 中 JS 與原生層之間）以及外部通訊（RN 與應用程式的「純原生」部分之間）。

React Native 允許您執行跨語言函數呼叫。您可以從 JS 執行自訂原生程式碼，反之亦然。遺憾的是，根據我們所處的端點，我們會以不同方式實現相同目標。對於原生端，我們使用事件機制來排程在 JS 中執行處理函數；而對於 React Native 端，我們直接呼叫原生模組匯出的方法。

### 從原生端呼叫 React Native 函數（事件）

事件在[這篇文章](native-components-ios#events)中有詳細說明。請注意，使用事件無法保證執行時間，因為事件是在獨立執行緒中處理的。

事件功能強大，因為它們允許我們無需持有元件的參照即可修改 React Native 元件。然而，使用時可能會遇到以下陷阱：

- 由於事件可以從任何地方發送，可能會在專案中引入義大利麵條式的依賴關係。
- 事件共享命名空間，這意味著可能會遇到名稱衝突。衝突無法靜態檢測，因此難以除錯。
- 如果使用多個相同 React Native 元件的實例，並希望從事件角度區分它們，可能需要引入識別符並隨事件傳遞（可使用原生視圖的 `reactTag` 作為識別符）。

我們在將原生元件嵌入 React Native 時常用的模式，是讓原生元件的 RCTViewManager 作為視圖的委派，透過橋接將事件發送回 JavaScript。這能將相關事件呼叫集中在一處。

### 從 React Native 呼叫原生函數（原生模組）

原生模組是可在 JS 中使用的 Objective-C 類別。通常每個 JS 橋接會建立每個模組的一個實例。它們可以向 React Native 匯出任意函數和常數。[這篇文章](native-modules-ios#content)對此有詳細說明。

原生模組是單例的性質在嵌入情境中限制了此機制。假設我們有一個嵌入原生視圖的 React Native 元件，且我們想更新原生父視圖。使用原生模組機制，我們需要匯出一個不僅接收預期參數，還需接收父原生視圖識別符的函數。該識別符將用於取得父視圖的參照以進行更新。也就是說，我們需要在模組中維護從識別符到原生視圖的映射。

雖然此解決方案較複雜，但 React Native 內部管理所有 React Native 視圖的類別 `RCTUIManager` 正是採用此方式。

原生模組也可用於向 JS 公開現有的原生函式庫。[Geolocation 函式庫](https://github.com/michalchudziak/react-native-geolocation)就是此概念的實際範例。

:::caution
所有原生模組共享相同的命名空間。建立新模組時請注意名稱衝突問題。
:::

## 佈局計算流程

整合原生與 React Native 時，我們還需要統一兩種不同佈局系統的方法。本節涵蓋常見佈局問題，並簡要說明解決這些問題的機制。

### 嵌入 React Native 的原生元件佈局

[這篇文章](native-components-ios#styles)說明了此情況。總結來說，由於所有原生 React 視圖都是 `UIView` 的子類別，大多數樣式和尺寸屬性都能如預期般直接使用。

### 嵌入原生端的 React Native 元件佈局

#### 固定尺寸的 React Native 內容

常見情境是當我們有一個尺寸固定且原生端已知的 React Native 應用程式。特別是全螢幕的 React Native 視圖就屬於此類。如果我們需要較小的根視圖，可以明確設定 RCTRootView 的框架。

舉例來說，若要讓一個 React Native 應用程式高度為 200（邏輯）像素，寬度與宿主視圖同寬，我們可以這樣做：

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

當我們有一個固定尺寸的根視圖時，需要在 JavaScript 端尊重其邊界。換句話說，我們需要確保 React Native 內容能被限制在固定尺寸的根視圖內。最簡單的方法是使用 Flexbox 佈局。如果使用絕對定位，且 React 元件在根視圖邊界外可見，將會與原生視圖產生重疊，導致某些功能出現非預期行為。例如，'TouchableHighlight' 將不會在根視圖邊界外顯示觸控高亮效果。

動態更新根視圖尺寸（透過重新設定其 frame 屬性）是完全可行的，React Native 會自動處理內容的佈局調整。

#### 彈性尺寸的 React Native 內容

某些情況下，我們需要渲染初始尺寸未知的內容。假設尺寸將由 JavaScript 動態決定，我們有兩種解決方案：

1. 可以用 `ScrollView` 元件包裹 React Native 視圖，這能確保內容始終可存取且不會與原生視圖重疊。
2. React Native 允許在 JavaScript 中計算應用程式尺寸，並將結果傳遞給宿主 `RCTRootView` 的所有者。所有者需負責重新佈局子視圖以保持介面一致性，這是透過 `RCTRootView` 的彈性模式實現的。

`RCTRootView` 支援 4 種不同的尺寸彈性模式：

```objectivec title='RCTRootView.h'
typedef NS_ENUM(NSInteger, RCTRootViewSizeFlexibility) {
  RCTRootViewSizeFlexibilityNone = 0,
  RCTRootViewSizeFlexibilityWidth,
  RCTRootViewSizeFlexibilityHeight,
  RCTRootViewSizeFlexibilityWidthAndHeight,
};
```

預設值 `RCTRootViewSizeFlexibilityNone` 會固定根視圖尺寸（但仍可透過 `setFrame:` 更新）。其他三種模式能追蹤 React Native 內容的尺寸變化。例如，設定為 `RCTRootViewSizeFlexibilityHeight` 時，React Native 會測量內容高度並將資訊回傳給 `RCTRootView` 的委派對象。委派方法中可執行任何操作（包括調整根視圖 frame 以適應內容），且僅在內容尺寸變化時觸發。

:::caution
若同時在 JavaScript 和原生端設定彈性尺寸會導致未定義行為。例如：當宿主 `RCTRootView` 使用 `RCTRootViewSizeFlexibilityWidth` 時，請勿將頂層 React 元件的寬度設為彈性（使用 `flexbox`）。
:::

以下是一個範例：

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

範例中的 `FlexibleSizeExampleView` 包含一個根視圖。我們建立並初始化根視圖後設定委派對象來處理尺寸更新。將尺寸彈性模式設為 `RCTRootViewSizeFlexibilityHeight` 後，每當 React Native 內容高度變化時，就會呼叫 `rootViewDidChangeIntrinsicSize:` 方法。最後設定根視圖的寬度與位置（注意高度設定此時無效，因為高度已交由 React Native 決定）。

完整範例程式碼可參考[此處](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/RNTester/NativeExampleViews/FlexibleSizeExampleView.m)。

動態變更根視圖的尺寸彈性模式是可行的。變更後會觸發佈局重新計算，待內容尺寸確定後即呼叫委派方法 `rootViewDidChangeIntrinsicSize:`。

:::note
需注意：React Native 的佈局計算在獨立線程執行，而原生 UI 更新則在主線程進行。這可能導致原生與 React Native 間的暫時性介面不一致。此為已知問題，我們的團隊正在努力同步不同來源的 UI 更新。
:::

:::note
React Native 在根視圖成為其他視圖的子視圖之前，不會執行任何佈局計算。
若您希望在知道尺寸前隱藏 React Native 視圖，可先將根視圖添加為子視圖並設定為初始隱藏（使用 `UIView` 的 `hidden` 屬性）。然後在委派方法中變更其可見性。
:::