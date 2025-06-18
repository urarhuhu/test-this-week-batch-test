---
id: react-native-devtools
title: React Native DevTools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native DevTools 是我們全新打造的除錯體驗，對除錯器堆疊進行了全面重寫。其目標是比 React Native 先前的除錯方法更深度整合且從根本上更可靠。

![React Native DevTools 開啟至「歡迎」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 專為除錯 React 應用問題而設計，並非用來取代原生工具。若您需要檢查 React Native 底層平台（例如開發原生模組時），請使用 Android Studio 和 Xcode 提供的除錯工具（參見[除錯原生程式碼](/docs/next/debugging-native-code)）。

<details>
<summary>**💡 Compatibility** — released in 0.76</summary>

React Native DevTools supports all React Native apps running Hermes. It replaces the previous Flipper, Experimental Debugger, and Hermes debugger (Chrome) frontends.

It is not possible to set up React Native DevTools with any older versions of React Native.

- **Chrome Browser DevTools — unsupported**
  - Connecting to React Native via `chrome://inspect` is no longer supported. Features may not work correctly, as the latest versions of Chrome DevTools (which are built to match the latest browser capabilities and APIs) have not been tested, and this frontend lacks our customisations. Instead, we ship a supported version with React Native DevTools.
- **Visual Studio Code — unsupported** (pre-existing)
  - Third party extensions such as [Expo Tools](https://github.com/expo/vscode-expo) and [Radon IDE](https://ide.swmansion.com/) may have improved compatibility, but are not directly supported by the React team.

</details>

<details>
<summary>**💡 Feedback & FAQs**</summary>

We want the tooling you use to debug React across all platforms to be reliable, familiar, simple, and cohesive. All the features described on this page are built with these principles in mind, and we also want to offer more capabilities in future.

We are actively iterating on the future of React Native DevTools, and have created a centralized [GitHub discussion](https://github.com/react-native-community/discussions-and-proposals/discussions/819) to keep track of issues, frequently asked questions, and feedback.

</details>

## 核心功能

React Native DevTools 基於 Chrome DevTools 前端架構。若您有網頁開發背景，對其功能應不陌生。建議先瀏覽[Chrome DevTools 文件](https://developer.chrome.com/docs/devtools)，內含完整指南與影音資源。

### 主控台

![React Native DevTools 原始碼視圖中的系列日誌與裝置並列顯示](/docs/assets/debugging-rndt-console.jpg)

主控台面板可讓您檢視/篩選訊息、執行 JavaScript、檢查物件屬性等。

[主控台功能參考 | Chrome DevTools](https://developer.chrome.com/docs/devtools/console/reference)

#### 實用技巧

- 若應用日誌過多，可使用篩選框或調整顯示的日誌層級
- 透過[即時運算式](https://developer.chrome.com/docs/devtools/console/live-expressions)持續觀察數值變化
- 使用[保留日誌](https://developer.chrome.com/docs/devtools/console/reference#persist)功能在重新載入時保持訊息
- 快捷鍵 <kbd>Ctrl</kbd> + <kbd>L</kbd> 可清除主控台視圖

### 原始碼與中斷點

![React Native DevTools 原始碼視圖中暫停的中斷點與裝置並列顯示](/docs/assets/debugging-rndt-sources-paused-with-device.jpg)

原始碼面板可檢視應用源文件並設置中斷點。中斷點能讓程式暫停在指定程式碼行，便於檢查即時狀態並逐步執行代碼。

[使用中斷點暫停程式碼 | Chrome DevTools](https://developer.chrome.com/docs/devtools/javascript/breakpoints)

:::tip

#### 簡易指南

中斷點是除錯工具的核心功能！

1. 透過側邊欄或快捷鍵 <kbd>Cmd ⌘</kbd>+<kbd>P</kbd> / <kbd>Ctrl</kbd>+<kbd>P</kbd> 導航至源文件
2. 點擊程式碼行號旁的行號欄位設置中斷點
3. 暫停時使用右上角導航控制[逐步執行代碼](https://developer.chrome.com/docs/devtools/javascript/reference#stepping)

:::

#### 實用技巧

- 程式暫停時會顯示「除錯器中暫停」疊層，點擊即可繼續
- 中斷點時注意右側面板，可檢查當前作用域、呼叫堆疊及設置監控運算式
- 使用 `debugger;` 語句快速設置中斷點（透過快速刷新立即生效於裝置）
- 中斷點有多種類型！例如[條件式中斷點與日誌點](https://developer.chrome.com/docs/devtools/javascript/breakpoints#overview)

### 記憶體

![在記憶體面板檢查堆快照](/docs/assets/debugging-rndt-memory.jpg)

記憶體面板可拍攝堆快照，監控 JavaScript 代碼的記憶體使用狀況。

[記錄堆快照 | Chrome DevTools](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots)

#### 實用技巧

- 使用 <kbd>Cmd ⌘</kbd>+<kbd>F</kbd> / <kbd>Ctrl</kbd>+<kbd>F</kbd> 篩選堆中的特定物件。
- 建立[分配時間軸報告](https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler)可將記憶體使用情況以圖表形式呈現，有助於識別可能的記憶體洩漏。

## React DevTools 功能

在整合的「元件」和「分析器」面板中，您會找到[React DevTools](https://react.dev/learn/react-developer-tools)瀏覽器擴充功能的所有功能。這些功能在 React Native DevTools 中無縫運作。

### React 元件

![使用 React 元件面板選取並定位元素](/docs/assets/debugging-rndt-react-components.gif)

React 元件面板可讓您檢查並更新渲染的 React 元件樹。

- 在 DevTools 中懸停或選取元素，即可在裝置上高亮顯示該元素。
- 若要定位 DevTools 中的元素，請點擊左上角的「選取元素」按鈕，然後點擊應用程式中的任何元素。

#### 實用技巧

- 可透過右側面板查看並修改執行時元件的 props 和 state。
- 使用 [React Compiler](https://react.dev/learn/react-compiler) 優化的元件會標註「Memo ✨」徽章。

:::tip

#### 專業技巧：高亮重新渲染

重新渲染可能是 React 應用程式效能問題的主要因素。DevTools 可在元件重新渲染時高亮顯示。

- 啟用方式：點擊檢視設定圖標（`⚙︎`）並勾選「元件渲染時高亮更新」。

![「高亮更新」設定位置，旁邊是即時渲染疊加層的錄製畫面](/docs/assets/debugging-rndt-highlight-renders.gif)

:::

### React 分析器

![以火焰圖形式呈現的分析記錄](/docs/assets/debugging-rndt-react-profiler.jpg)

React 分析器面板可讓您記錄效能分析，以了解元件渲染和 React 提交的時間。

更多資訊，請參閱[2018 年原始指南](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html#reading-performance-data)（請注意部分內容可能已過時）。

## 重新連接 DevTools

有時 DevTools 可能會與目標裝置斷開連接。這可能發生在以下情況：

- 應用程式關閉。
- 應用程式重新建置（安裝了新的原生建置版本）。
- 應用程式在原生端崩潰。
- 開發伺服器（Metro）停止運作。
- 實體裝置斷開連接。

斷開連接時，會顯示「偵錯連接已關閉」的對話框。

![裝置斷開連接時顯示的重新連接對話框](/docs/assets/debugging-reconnect-menu.jpg)

此時您可以選擇：

- **關閉**：點擊關閉圖標（`×`）或點擊對話框外部，返回斷開連接前的最後狀態。
- **重新連接**：解決斷開連接的原因後，選擇「重新連接 DevTools」。