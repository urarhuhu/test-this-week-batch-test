---
id: debugging
title: Debugging Basics
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

:::note
除錯功能（如開發者選單、LogBox 和 React Native DevTools）在正式（生產）版本中會被停用。
:::

## 開啟開發者選單

React Native 提供了一個內建的開發者選單，可存取各項除錯功能。您可以透過搖動裝置或使用鍵盤快捷鍵來開啟開發者選單：

- iOS 模擬器：<kbd>Ctrl</kbd> + <kbd>Cmd ⌘</kbd> + <kbd>Z</kbd>（或選擇 Device > Shake）
- Android 模擬器：<kbd>Cmd ⌘</kbd> + <kbd>M</kbd>（macOS）或 <kbd>Ctrl</kbd> + <kbd>M</kbd>（Windows 和 Linux）

替代方案（Android）：`adb shell input keyevent 82`。

![React Native 開發者選單](/docs/assets/debugging-dev-menu-076.jpg)

## 開啟 DevTools

[React Native DevTools](./react-native-devtools) 是我們為 React Native 內建的除錯工具。它能讓您檢查並理解 JavaScript 程式碼的執行狀況，類似於網頁瀏覽器的開發者工具。

開啟 DevTools 的方式如下：

- 在開發者選單中選擇「Open DevTools」。
- 在 CLI 中按下 <kbd>j</kbd> 鍵（`npx react-native start`）。

首次啟動時，DevTools 會開啟歡迎面板，並同時展開控制台抽屜，您可以在這裡查看日誌並與 JavaScript 執行環境互動。從視窗頂部，您可以導覽至其他面板，包括整合的 React 元件檢查器和效能分析工具。

![React Native DevTools 開啟至「Welcome」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 由 React Native 內建的專用除錯架構驅動，並使用客製化的 [Chrome DevTools](https://developer.chrome.com/docs/devtools) 前端版本。這讓我們能提供熟悉且與瀏覽器一致的除錯功能，並實現深度整合與端到端的可靠性。

詳情請參閱我們的 [React Native DevTools 指南](./react-native-devtools)。

:::note
React Native DevTools 僅在使用 Hermes 引擎時可用，且需要安裝 Google Chrome 或 Microsoft Edge。
:::

:::info

#### Flipper 與其他替代除錯工具

React Native DevTools 取代了先前的 Flipper、實驗性除錯工具和 Hermes 除錯工具（Chrome）前端。如果您使用的是舊版 React Native，請參閱[對應版本的說明文件](/versions)。

對於使用 JavaScriptCore 而非 Hermes 的應用程式，仍可使用直接 JSC 除錯功能（請參閱[其他除錯方法](./other-debugging-methods)）。

React Native DevTools 專為除錯 React 應用程式問題而設計，並非用來取代原生工具。如果您需要檢查 React Native 底層平台（例如開發原生模組時），請使用 Xcode 和 Android Studio 提供的除錯工具（請參閱[除錯原生程式碼](/docs/next/debugging-native-code)）。

其他實用連結：

- <a href="https://shift.infinite.red/why-you-dont-need-flipper-in-your-react-native-app-and-how-to-get-by-without-it-3af461955109" target="_blank">為什麼您的 React Native 應用程式不需要 Flipper……以及如何不使用它也能順利開發&nbsp;↗</a>

:::

## LogBox

LogBox 是一個內建工具，當您的應用程式記錄了警告或錯誤時會顯示通知。

![LogBox 警告與展開的 LogBox 語法錯誤](/docs/assets/debugging-logbox-076.jpg)

### 致命錯誤

當發生無法恢復的錯誤時（例如 JavaScript 語法錯誤），LogBox 會開啟並顯示錯誤位置。在此狀態下，LogBox 無法關閉，因為您的程式碼無法執行。LogBox 會在語法錯誤修正後自動關閉——無論是透過 Fast Refresh 或手動重新載入。

### 主控台錯誤與警告

主控台錯誤與警告會以螢幕通知形式顯示，並帶有紅色或黃色標記。

- **錯誤**會顯示通知計數。點擊通知可查看詳細資訊並瀏覽其他日誌。
- **警告**會顯示不含細節的通知橫幅，提示您開啟 React Native DevTools。

當 React Native DevTools 開啟時，除致命錯誤外的所有錯誤都會對 LogBox 隱藏。由於 LogBox 有多種選項可能隱藏或調整特定日誌的等級，我們建議使用 React Native DevTools 中的主控台面板作為主要參考來源。

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

在 Android 和 iOS 上，開發期間可透過開發者選單選擇 **「Perf Monitor」** 來切換應用內效能疊加層。詳細功能說明請參閱[此處](/docs/performance)。

![iOS 和 Android 上的效能監控器疊加層](/docs/assets/debugging-performance-monitor.jpg)

:::info
效能監控器在應用內執行，僅供參考。我們建議使用 Android Studio 和 Xcode 的原生工具進行精確的效能測量。
:::