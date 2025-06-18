---
id: react-native-devtools
title: React Native DevTools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native DevTools 是我們專為 React Native 打造的現代化除錯體驗。從零開始構建，旨在比過往的除錯方法更深度整合、更準確可靠。

![React Native DevTools 開啟至「歡迎」面板](/docs/assets/debugging-rndt-welcome.jpg)

React Native DevTools 專注於 React 應用層面的除錯，並非用來取代原生工具。若需檢查 React Native 底層平台（例如開發原生模組時），請使用 Android Studio 和 Xcode 的除錯工具（參閱[原生程式碼除錯](/docs/debugging-native-code)）。

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

React Native DevTools 基於 Chrome DevTools 前端架構。若有網頁開發背景，您會對其功能感到熟悉。建議先瀏覽[ Chrome DevTools 文件](https://developer.chrome.com/docs/devtools)，內含完整指南與影音資源。

### 主控台

![React Native DevTools 原始碼檢視中的日誌序列與裝置並列](/docs/assets/debugging-rndt-console.jpg)

主控台面板可檢視/篩選訊息、執行 JavaScript、檢查物件屬性等。

[主控台功能參考 | Chrome DevTools](https://developer.chrome.com/docs/devtools/console/reference)

#### 實用技巧

- 若應用日誌量龐大，請使用篩選框或調整顯示的日誌層級
- 透過[即時運算式](https://developer.chrome.com/docs/devtools/console/live-expressions)追蹤數值變化
- 使用[保留日誌](https://developer.chrome.com/docs/devtools/console/reference#persist)功能在重新載入時保持訊息
- 快捷鍵 <kbd>Ctrl</kbd> + <kbd>L</kbd> 可清除主控台視窗

### 原始碼與中斷點

![React Native DevTools 原始碼檢視中暫停的中斷點與裝置並列](/docs/assets/debugging-rndt-sources-paused-with-device.jpg)

原始碼面板可檢視應用原始檔並設置中斷點。中斷點能指定程式暫停執行的程式碼行，讓您檢查即時狀態並逐步執行程式碼。

[使用中斷點暫停程式碼 | Chrome DevTools](https://developer.chrome.com/docs/devtools/javascript/breakpoints)

:::tip

#### 簡易指南

中斷點是除錯工具的核心！

1. 透過側邊欄或快捷鍵 <kbd>Cmd ⌘</kbd>+<kbd>P</kbd> / <kbd>Ctrl</kbd>+<kbd>P</kbd> 導覽至原始檔
2. 點擊程式碼行號旁的行號欄位添加中斷點
3. 暫停時使用右上角導覽控制[逐步執行程式碼](https://developer.chrome.com/docs/devtools/javascript/reference#stepping)

:::

#### 實用技巧

- 應用暫停時會顯示「除錯器中暫停」疊加層，點擊即可繼續
- 中斷點狀態時注意右側面板，可檢查當前作用域、呼叫堆疊及設置監控運算式
- 使用 `debugger;` 陳述句快速從編輯器設置中斷點，透過快速重新整理立即生效於裝置
- 中斷點有多種類型！例如[條件式中斷點與日誌點](https://developer.chrome.com/docs/devtools/javascript/breakpoints#overview)

### 記憶體

![在記憶體面板檢查堆積快照](/docs/assets/debugging-rndt-memory.jpg)

記憶體面板可擷取堆積快照，檢視 JavaScript 程式碼隨時間變化的記憶體使用狀況。

[記錄堆積快照 | Chrome DevTools](https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots)

#### 實用技巧

- 使用 <kbd>Cmd ⌘</kbd>+<kbd>F</kbd> / <kbd>Ctrl</kbd>+<kbd>F</kbd> 在堆積中篩選特定物件。
- 生成[記憶體分配時間軸報告](https://developer.chrome.com/docs/devtools/memory-problems/allocation-profiler)有助於以圖表形式觀察記憶體使用隨時間變化，識別可能的記憶體洩漏。

## React DevTools 功能

在整合的「元件」與「效能分析器」面板中，您可找到[React DevTools](https://react.dev/learn/react-developer-tools)瀏覽器擴充功能的所有特性，這些功能在 React Native DevTools 中能無縫運作。

### React 元件

![使用 React 元件面板選取與定位元素](/docs/assets/debugging-rndt-react-components.gif)

React 元件面板讓您能檢查並更新渲染出的 React 元件樹。

- 在 DevTools 中懸停或選取元素，裝置上的對應元件會高亮顯示。
- 若要在 DevTools 中定位元件，點擊左上角的「選取元素」按鈕，然後點擊應用中的任意元件。

#### 實用技巧

- 透過右側面板可即時查看並修改元件的 props 與 state。
- 使用 [React Compiler](https://react.dev/learn/react-compiler) 優化的元件會標註「Memo ✨」徽章。

:::tip

#### 專業技巧：高亮重新渲染

元件重新渲染可能是 React 應用效能問題的主因，DevTools 可即時高亮顯示重新渲染的元件。

- 啟用方式：點擊視圖設定圖標（`⚙︎`）並勾選「元件渲染時高亮更新」。

![「高亮更新」設定位置與即時渲染高亮的錄製畫面](/docs/assets/debugging-rndt-highlight-renders.gif)

:::

### React 效能分析器

![以火焰圖形式呈現的效能分析結果](/docs/assets/debugging-rndt-react-profiler.jpg)

React 效能分析器面板讓您錄製效能分析資料，以了解元件渲染與 React 提交階段的時間分佈。

詳情請參閱[2018年原始指南](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html#reading-performance-data)（請注意部分內容可能已過時）。

## 重新連接 DevTools

DevTools 有時會與目標裝置斷開連接，可能原因包括：

- 應用程式關閉。
- 應用程式重新建置（安裝了新的原生版本）。
- 應用程式在原生層面崩潰。
- 開發伺服器（Metro）停止運行。
- 實體裝置斷開連接。

斷開連接時，會顯示「偵錯連線已關閉」的對話框。

![裝置斷線時顯示的重新連接對話框](/docs/assets/debugging-reconnect-menu.jpg)

此時您可以選擇：

- **忽略**：點擊關閉圖標（`×`）或對話框外部，返回斷線前的 DevTools 介面狀態。
- **重新連接**：在解決斷線原因後，點選「重新連接 DevTools」。