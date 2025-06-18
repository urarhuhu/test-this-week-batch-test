---
id: debugging
title: Debugging Basics
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

:::note
除錯功能（如開發者選單、LogBox 和 React Native DevTools）在正式版（生產環境）建置中會被停用。
:::

## 開啟開發者選單

React Native 提供內建的開發者選單，可存取各項除錯功能。您可以透過搖動裝置或鍵盤快捷鍵開啟開發者選單：

- iOS 模擬器：<kbd>Ctrl</kbd> + <kbd>Cmd ⌘</kbd> + <kbd>Z</kbd>（或選擇 Device > Shake）
- Android 模擬器：<kbd>Cmd ⌘</kbd> + <kbd>M</kbd>（macOS）或 <kbd>Ctrl</kbd> + <kbd>M</kbd>（Windows 與 Linux）

替代方案（Android）：`adb shell input keyevent 82`。

![React Native 開發者選單](/docs/assets/debugging-dev-menu-076.jpg)

## 開啟 DevTools

[React Native DevTools](./react-native-devtools) 是我們為 React Native 內建的除錯工具。它能讓您檢查並理解 JavaScript 程式碼的執行狀況，類似網頁瀏覽器的開發者工具。

開啟 DevTools 的方式：

- 在開發者選單中選擇「Open DevTools」。
- 在 CLI（`npx react-native start`）中按下 <kbd>j</kbd>。

首次啟動時，DevTools 會顯示歡迎面板，並開啟控制台抽屜供您查看日誌與互動式 JavaScript 執行環境。您可從視窗頂部切換至其他面板，包括整合式 React 元件檢查器與分析器。

![React Native DevTools 開啟至「Welcome」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 採用專為 React Native 打造的除錯架構，並使用客製化版本的 [Chrome DevTools](https://developer.chrome.com/docs/devtools) 前端介面。這讓我們能提供與瀏覽器操作一致且深度整合的可靠除錯功能。

詳見我們的 [React Native DevTools 指南](./react-native-devtools)。

:::note
React Native DevTools 僅支援 Hermes 引擎，且需安裝 Google Chrome 或 Microsoft Edge。
:::

:::info

#### Flipper 與替代除錯工具

React Native DevTools 已取代先前的 Flipper、實驗性除錯工具與 Hermes 除錯器（Chrome）前端介面。若您使用舊版 React Native，請查閱[對應版本文件](/versions)。

使用 JavaScriptCore（非 Hermes）的應用程式仍可使用直接 JSC 除錯功能（參見[其他除錯方法](./other-debugging-methods)）。

React Native DevTools 專為 React 應用程式除錯設計，並非用來取代原生工具。如需檢查 React Native 底層平台（例如開發原生模組時），請使用 Xcode 與 Android Studio 的除錯工具（參見[原生程式碼除錯](/docs/next/debugging-native-code)）。

其他實用連結：

- <a href="https://shift.infinite.red/why-you-dont-need-flipper-in-your-react-native-app-and-how-to-get-by-without-it-3af461955109" target="_blank">為何您的 React Native 應用不需要 Flipper…以及替代方案&nbsp;↗</a>

:::

## LogBox

LogBox 是應用程式內建工具，當應用程式記錄警告或錯誤時會顯示相關資訊。

![LogBox 警告與展開的語法錯誤畫面](/docs/assets/debugging-logbox-076.jpg)

### 致命錯誤

當發生無法恢復的錯誤時（例如 JavaScript 語法錯誤），LogBox 會開啟並顯示錯誤位置。在此狀態下，由於程式碼無法執行，LogBox 無法關閉。只有當語法錯誤被修正後（透過 Fast Refresh 或手動重新載入），LogBox 才會自動關閉。

### 主控台錯誤與警告

主控台錯誤與警告會以螢幕通知形式顯示，並標有紅色或黃色徽章。

- **錯誤**會顯示通知計數。點擊通知可查看詳細資訊並瀏覽其他日誌。
- **警告**會顯示不含細節的通知橫幅，提示您開啟 React Native DevTools。

當 React Native DevTools 開啟時，除致命錯誤外的所有錯誤都會對 LogBox 隱藏。由於 LogBox 有多種選項可能隱藏或調整特定日誌的顯示層級，建議將 React Native DevTools 中的主控台面板作為主要參考來源。

<details>
<summary>**💡 Ignoring logs**</summary>

LogBox can be configured via the `LogBox` API.

```js
import {LogBox} from 'react-native';
```

#### Ignore all logs

LogBox notifications can be disabled using `LogBox.ignoreAllLogs()`. This can be useful in situations such as giving product demos.

```js
LogBox.ignoreAllLogs();
```

#### Ignore specific logs

Notifications can be disabled on a per-log basis via `LogBox.ignoreLogs()`. This can be useful for noisy warnings or those that cannot be fixed, e.g. in a third-party dependency.

```js
LogBox.ignoreLogs([
  // Exact message
  'Warning: componentWillReceiveProps has been renamed',

  // Substring or regex match
  /GraphQL error: .*/,
]);
```

:::note

LogBox will treat certain errors from React as warnings, which will mean they don't display as an in-app error notification. Advanced users can change this behaviour by customising LogBox's warning filter using [`LogBoxData.setWarningFilter()`](https://github.com/facebook/react-native/blob/d334f4d77eea538dff87fdcf2ebc090246cfdbb0/packages/react-native/Libraries/LogBox/Data/LogBoxData.js#L338).

:::

</details>

## 效能監控器

在 Android 和 iOS 上，開發期間可透過開發者選單中的**「效能監控器」**選項切換應用內效能疊加層。詳細功能說明請參閱[此文件](/docs/performance)。

![iOS 與 Android 上的效能監控器疊加層](/docs/assets/debugging-performance-monitor.jpg)

:::info
效能監控器在應用內運行，僅供參考。建議使用 Android Studio 和 Xcode 的原生工具進行精確的效能測量。
:::