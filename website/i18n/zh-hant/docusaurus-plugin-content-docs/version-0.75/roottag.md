---
id: roottag
title: RootTag
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`RootTag` 是一個不透明的識別符，用於標識 React Native 介面的原生根視圖——即 Android 的 `ReactRootView` 或 iOS 的 `RCTRootView` 實例。簡而言之，它是一個介面識別符。

## 何時需要使用 RootTag？

對於大多數 React Native 開發者來說，您可能不需要直接處理 `RootTag`。

`RootTag` 在應用程式渲染**多個 React Native 根視圖**時非常有用，尤其是當您需要根據不同的介面來區分處理原生 API 呼叫時。一個典型的例子是當應用程式使用原生導航，並且每個畫面都是一個獨立的 React Native 根視圖時。

在原生的導航架構中，每個 React Native 根視圖都會被渲染到平台的原生導航視圖中（例如 Android 的 `Activity` 或 iOS 的 `UINavigationViewController`）。這樣，您可以充分利用平台的導航特性，例如原生的外觀和導航過場效果。與原生導航 API 互動的功能可以通過 [原生模組](https://reactnative.dev/docs/next/native-modules-intro) 暴露給 React Native。

舉例來說，如果您需要更新某個畫面的標題欄，您會呼叫導航模組的 API `setTitle("Updated Title")`，但這個 API 需要知道要更新導航堆疊中的哪一個畫面。這時就需要 `RootTag` 來識別根視圖及其所在的容器。

另一個 `RootTag` 的使用場景是當您的應用程式需要根據 JavaScript 呼叫的來源根視圖來區分處理原生邏輯時。`RootTag` 可以用來區分來自不同介面的呼叫來源。

## 如何存取 RootTag（如果您需要的話）

在 0.65 及以下版本中，RootTag 是通過 [舊版 Context](https://github.com/facebook/react-native/blob/v0.64.1/Libraries/ReactNative/AppContainer.js#L56) 存取的。為了讓 React Native 能夠支援 React 18 及更高版本中的 Concurrent 特性，我們在 0.66 版本中遷移到了最新的 [Context API](https://reactjs.org/docs/context.html#api)，並通過 `RootTagContext` 來提供 RootTag。0.65 版本同時支援舊版 Context 和推薦的 `RootTagContext`，以便開發者有時間遷移程式碼。請參閱重大變更摘要。

如何通過 `RootTagContext` 存取 `RootTag`。

```js
import {RootTagContext} from 'react-native';
import NativeAnalytics from 'native-analytics';
import NativeNavigation from 'native-navigation';

function ScreenA() {
  const rootTag = useContext(RootTagContext);

  const updateTitle = title => {
    NativeNavigation.setTitle(rootTag, title);
  };

  const handleOneEvent = () => {
    NativeAnalytics.logEvent(rootTag, 'one_event');
  };

  // ...
}

class ScreenB extends React.Component {
  static contextType: typeof RootTagContext = RootTagContext;

  updateTitle(title) {
    NativeNavigation.setTitle(this.context, title);
  }

  handleOneEvent() {
    NativeAnalytics.logEvent(this.context, 'one_event');
  }

  // ...
}
```

您可以從 React 文件中了解更多關於 [類別](https://reactjs.org/docs/context.html#classcontexttype) 和 [Hook](https://reactjs.org/docs/hooks-reference.html#usecontext) 的 Context API 用法。

### 0.65 版本的重大變更

在 0.65 版本中，`RootTagContext` 從原先的 `unstable_RootTagContext` 更名為 `RootTagContext`。請將程式碼中所有使用 `unstable_RootTagContext` 的地方更新為新名稱。

### 0.66 版本的重大變更

舊版 Context 存取 `RootTag` 的方式將被移除，並由 `RootTagContext` 取代。從 0.65 版本開始，我們建議開發者主動將 `RootTag` 的存取方式遷移到 `RootTagContext`。

## 未來計劃

隨著新的 React Native 架構不斷演進，`RootTag` 也會有未來的迭代更新，目的是保持 `RootTag` 類型的不透明性，並避免 React Native 程式碼庫中的頻繁變動。請不要依賴目前 RootTag 實際上是數值類型的這一事實！如果您的應用程式依賴 RootTag，請密切關注我們的版本變更日誌，您可以在 [這裡](https://github.com/facebook/react-native/blob/main/CHANGELOG.md) 找到相關資訊。