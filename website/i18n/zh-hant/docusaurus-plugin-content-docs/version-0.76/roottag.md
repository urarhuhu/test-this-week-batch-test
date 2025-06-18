---
id: roottag
title: RootTag
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`RootTag` 是一個不透明的識別符，用於標記 React Native 介面的原生根視圖——即 Android 的 `ReactRootView` 或 iOS 的 `RCTRootView` 實例。簡而言之，它是一個介面識別符。

## 何時需要使用 RootTag？

對於大多數 React Native 開發者來說，您可能不需要直接處理 `RootTag`。

當應用程式渲染**多個 React Native 根視圖**，且需要根據不同介面區分處理原生 API 呼叫時，`RootTag` 會非常有用。例如，當應用程式使用原生導航且每個畫面都是獨立的 React Native 根視圖時。

在原生導航中，每個 React Native 根視圖都會渲染在平台的導航視圖中（例如 Android 的 `Activity` 或 iOS 的 `UINavigationViewController`）。這樣您就能利用平台的導航特性，例如原生外觀與導航過場效果。與原生導航 API 互動的功能可以透過[原生模組](https://reactnative.dev/docs/next/native-modules-intro)暴露給 React Native。

舉例來說，若要更新某個畫面的標題列，您會呼叫導航模組的 API `setTitle("Updated Title")`，但該 API 需要知道要更新導航堆疊中的哪個畫面。此時就需要 `RootTag` 來識別根視圖及其宿主容器。

另一個 `RootTag` 的使用情境是當應用程式需要根據 JavaScript 呼叫的來源根視圖來區分原生行為時。此時需要 `RootTag` 來辨別不同介面的呼叫來源。

## 如何存取 RootTag（若您需要）

在 0.65 及以下版本中，RootTag 透過[舊版 Context](https://github.com/facebook/react-native/blob/v0.64.1/Libraries/ReactNative/AppContainer.js#L56) 存取。為了讓 React Native 能支援 React 18 及更高版本的 Concurrent 特性，我們在 0.66 版改為透過 `RootTagContext` 使用最新的 [Context API](https://reactjs.org/docs/context.html#api)。0.65 版同時支援舊版 Context 和建議的 `RootTagContext`，讓開發者有時間遷移程式碼。詳見重大變更摘要。

如何透過 `RootTagContext` 存取 `RootTag`。

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

請參閱 React 文件了解 [類別元件](https://reactjs.org/docs/context.html#classcontexttype) 和 [Hook](https://reactjs.org/docs/hooks-reference.html#usecontext) 的 Context API 使用方式。

### 0.65 版的重大變更

`RootTagContext` 在 0.65 版之前名為 `unstable_RootTagContext`，現已更名為 `RootTagContext`。請更新程式碼中所有 `unstable_RootTagContext` 的引用。

### 0.66 版的重大變更

舊版 Context 存取 `RootTag` 的方式將被移除，改由 `RootTagContext` 取代。從 0.65 版開始，我們建議開發者主動將 `RootTag` 的存取方式遷移至 `RootTagContext`。

## 未來計劃

隨著新的 React Native 架構持續發展，`RootTag` 將會進行後續迭代，目標是保持 `RootTag` 類型的不透明性，避免 React Native 程式碼庫頻繁變動。請勿依賴目前 RootTag 實際為數字類型的實作細節！若您的應用程式依賴 RootTag，請密切關注我們的版本變更日誌，您可以在[此處](https://github.com/facebook/react-native/blob/main/CHANGELOG.md)找到相關資訊。