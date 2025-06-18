---
id: debugging
title: Debugging
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 存取應用內開發者選單

您可以透過搖動裝置，或在 iOS 模擬器的「硬體」選單中選擇「搖動手勢」來開啟開發者選單。在 iOS 模擬器中執行應用時，也可使用鍵盤快捷鍵 `⌘D`；在 macOS 的 Android 模擬器上則使用 `⌘M`，Windows 和 Linux 上則為 `Ctrl+M`。另外針對 Android 裝置，可執行指令 `adb shell input keyevent 82` 來開啟開發者選單（82 為選單鍵的代碼）。

![](/docs/assets/DeveloperMenu.png)

> 開發者選單在正式（生產）版本中會被停用。

## 啟用快速重新整理

快速重新整理是 React Native 的一項功能，可讓您幾乎即時獲得對 React 元件變更的反饋。在除錯時，啟用[快速重新整理](fast-refresh.md)會有所幫助。快速重新整理預設為啟用狀態，您可以在 React Native 開發者選單中切換「啟用快速重新整理」。啟用後，大多數編輯應能在一兩秒內看到效果。

## 啟用鍵盤快捷鍵

React Native 在 iOS 模擬器中支援一些鍵盤快捷鍵，如下所述。要在 macOS 上啟用這些快捷鍵，請在模擬器應用程式中開啟 I/O 選單，選擇「鍵盤」，並確保「連接硬體鍵盤」已勾選。

## LogBox

開發版本中的錯誤和警告會顯示在應用內的 LogBox 中。

> LogBox 在正式（生產）版本中會自動停用。

### 主控台錯誤與警告

主控台錯誤和警告會以帶有紅色或黃色標記的螢幕通知形式顯示，分別代表主控台中錯誤或警告的數量。要查看主控台錯誤或警告，請點擊通知以查看該日誌的完整畫面資訊，並可翻頁查看主控台中的所有日誌。

這些通知可以透過 `LogBox.ignoreAllLogs()` 來隱藏。這在進行產品演示等場合非常有用。此外，也可以透過 `LogBox.ignoreLogs()` 針對特定日誌隱藏通知。這對於無法修復的第三方依賴中的嘈雜警告特別有用。

> 僅在最後手段時忽略日誌，並建立任務來修復任何被忽略的日誌。

```jsx
import {LogBox} from 'react-native';

// Ignore log notification by message:
LogBox.ignoreLogs(['Warning: ...']);

// Ignore all log notifications:
LogBox.ignoreAllLogs();
```

### 未處理的錯誤

未處理的 JavaScript 錯誤（例如 `undefined is not a function`）會自動開啟全螢幕的 LogBox 錯誤畫面，顯示錯誤來源。這些錯誤可以關閉或最小化，以便您能查看錯誤發生時應用程式的狀態，但應始終予以解決。

### 語法錯誤

當發生語法錯誤時，全螢幕的 LogBox 錯誤畫面會自動開啟，顯示堆疊追蹤和語法錯誤的位置。此錯誤無法關閉，因為它代表無效的 JavaScript 執行，必須在繼續應用程式前予以修正。要關閉這些錯誤，請修正語法錯誤，然後儲存以自動關閉（快速重新整理啟用時）或按 cmd+r 重新載入（快速重新整理停用時）。

## Chrome 開發者工具

要在 Chrome 中除錯 JavaScript 程式碼，請從開發者選單中選擇「遠端除錯 JS」。這將在 [http://localhost:8081/debugger-ui](http://localhost:8081/debugger-ui) 開啟一個新分頁。

從 Chrome 選單中選擇「工具 → 開發者工具」以開啟[開發者工具](https://developer.chrome.com/devtools)。您也可以使用鍵盤快捷鍵（macOS 上為 `⌘⌥I`，Windows 上為 `Ctrl` `Shift` `I`）存取開發者工具。您可能還想啟用[在捕獲例外時暫停](http://stackoverflow.com/questions/2233339/javascript-is-there-a-way-to-get-chrome-to-break-on-all-errors/17324511#17324511)以獲得更好的除錯體驗。

> 注意：在 Android 上，若偵錯工具與裝置間的時間不同步，可能會導致動畫效果、事件行為等運作異常或結果不準確。請在偵錯機器上執行 ``adb shell "date `date +%m%d%H%M%Y.%S%3N`"`` 指令校正時間（實體裝置需 root 權限）。

> 注意：React Developer Tools Chrome 擴充功能無法與 React Native 搭配使用，但可改用其獨立版本。詳見[此章節](debugging.md#react-developer-tools)說明。

### 使用自訂 JavaScript 偵錯工具

若要改用自訂 JavaScript 偵錯工具取代 Chrome 開發者工具，請將 `REACT_DEBUGGER` 環境變數設為啟動該偵錯工具的命令。接著從開發者選單選擇「遠端偵錯 JS」即可開始偵錯。

偵錯工具會收到以空格分隔的專案根目錄清單。例如若設定 `REACT_DEBUGGER="node /path/to/launchDebugger.js --port 2345 --type ReactNative"`，則會執行 `node /path/to/launchDebugger.js --port 2345 --type ReactNative /path/to/reactNative/app` 來啟動偵錯工具。

> 此方式執行的自訂偵錯命令應為短生命週期程序，且輸出不得超過 200 KB。

## Safari 開發者工具

無需啟用「遠端偵錯 JS」，即可使用 Safari 偵錯 iOS 版應用程式。

- 實體裝置設定：`設定 → Safari → 進階 → 確保「網頁檢查器」已啟用`（模擬器不需此步驟）
- Mac 端設定：於 Safari `偏好設定 → 進階 → 勾選「在選單列顯示開發選單」`
- 選取應用程式的 JSContext：`開發 → 模擬器（或其他裝置） → JSContext`
- Safari 的網頁檢查器將開啟，內含主控台與偵錯工具

原始碼對應(sourcemap)可能預設未啟用，可參照[此指南](http://blog.nparashuram.com/2019/10/debugging-react-native-ios-apps-with.html)或[影片](https://www.youtube.com/watch?v=GrGqIIz51k4)啟用，並在原始碼正確位置設定中斷點。

但每次重新載入應用程式（透過即時重新載入或手動重新載入）時，都會建立新的 JSContext。勾選「自動顯示 JSContext 的網頁檢查器」可避免手動選取最新 JSContext。

## React 開發者工具

可使用 [React 開發者工具獨立版本](https://github.com/facebook/react/tree/main/packages/react-devtools)來偵錯 React 元件層級結構。請先全域安裝 `react-devtools` 套件：

> 注意：`react-devtools` 第 4 版需搭配 `react-native` 0.62 以上版本才能正常運作。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install -g react-devtools
```

</TabItem>
<TabItem value="yarn">

```shell
yarn global add react-devtools
```

</TabItem>
</Tabs>

在終端機執行 `react-devtools` 以啟動獨立版開發工具應用程式：

```shell
react-devtools
```

![React 開發者工具](/docs/assets/ReactDevTools.png)

應能在數秒內連接到模擬器。

:::info
若連接模擬器發生問題（特別是 Android 12），可嘗試在新終端機執行 `adb reverse tcp:8097 tcp:8097`。
:::

:::info
若不想全域安裝，可將 `react-devtools` 加入專案依賴。透過 `npm install --save-dev react-devtools` 安裝套件，並在 `package.json` 的 `scripts` 區段加入 `"react-devtools": "react-devtools"`，最後在專案目錄執行 `npm run react-devtools` 即可開啟開發工具。
:::

### 與 React Native 檢查器整合

開啟應用內開發者選單並選擇「切換檢查器」。這將顯示一個疊加層，讓您可以點擊任何 UI 元素並查看相關資訊：

![React Native 檢查器](/docs/assets/Inspector.gif)

然而，當 `react-devtools` 運行時，檢查器會進入折疊模式，並改以 DevTools 作為主要介面。在此模式下，點擊模擬器中的元素將在 DevTools 中顯示相關元件：

![React DevTools 檢查器整合](/docs/assets/ReactDevToolsInspector.gif)

您可以在同一選單中選擇「切換檢查器」以退出此模式。

### 檢查元件實例

在 Chrome 中調試 JavaScript 時，您可以在瀏覽器控制台中檢查 React 元件的 props 和 state。

首先，按照在 Chrome 中調試的指示開啟 Chrome 控制台。

確保 Chrome 控制台左上角的下拉選單顯示 `debuggerWorker.js`。**此步驟至關重要。**

接著在 React DevTools 中選擇一個 React 元件。頂部的搜尋框可幫助您按名稱查找元件。一旦選中，該元件將在 Chrome 控制台中以 `$r` 的形式提供，讓您可以檢查其 props、state 和實例屬性。

![React DevTools Chrome 控制台整合](/docs/assets/ReactDevToolsDollarR.gif)

## 效能監控

您可以透過在開發者選單中選擇「效能監控」來啟用效能疊加層，以協助調試效能問題。

<hr style={{marginTop: 25, marginBottom: 25}} />

## 調試應用程式狀態

[Reactotron](https://github.com/infinitered/reactotron) 是一個開源桌面應用程式，可讓您檢查 Redux 或 MobX-State-Tree 的應用程式狀態，查看自訂日誌，執行自訂命令（如重置狀態），儲存和恢復狀態快照，以及其他有助於調試 React Native 應用的功能。

您可以在 [README](https://github.com/infinitered/reactotron) 中查看安裝說明。如果您使用 Expo，這裡有一篇文章詳細介紹了[如何在 Expo 上安裝](https://shift.infinite.red/start-using-reactotron-in-your-expo-project-today-in-3-easy-steps-a03d11032a7a)。

# 原生調試

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3><p>
    The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/workflow/prebuild/" target="_blank">prebuild</a> to use this API.</p>
</div>

## 存取控制台日誌

您可以在應用程式運行時，於終端機中使用以下命令來顯示 iOS 或 Android 應用的控制台日誌：

```shell
npx react-native log-ios
npx react-native log-android
```

您也可以透過 iOS 模擬器中的 `Debug → Open System Log...` 或在 Android 裝置或模擬器上運行應用時，於終端機中執行 `adb logcat *:S ReactNative:V ReactNativeJS:V` 來存取這些日誌。

> 如果您使用 Create React Native App 或 Expo CLI，控制台日誌已顯示在與打包工具相同的終端輸出中。

## 在裝置上使用 Chrome 開發者工具調試

> 如果您使用 Create React Native App 或 Expo CLI，此功能已為您配置好。

在 iOS 裝置上，開啟檔案 [`RCTWebSocketExecutor.mm`](https://github.com/facebook/react-native/blob/0.70-stable/React/CoreModules/RCTWebSocketExecutor.mm) 並將 "localhost" 更改為您電腦的 IP 位址，然後在開發者選單中選擇「遠端調試 JS」。

在透過 USB 連接的 Android 5.0+ 裝置上，您可以使用 [`adb` 命令列工具](http://developer.android.com/tools/help/adb.html) 來設置從裝置到電腦的端口轉發：

`adb reverse tcp:8081 tcp:8081`

或者，從開發者選單中選擇「開發設定」，然後將「裝置的除錯伺服器主機」設定更新為與您電腦的 IP 位址相符。

> 如果遇到任何問題，可能是您的某個 Chrome 擴充功能與除錯工具產生了非預期的互動。嘗試停用所有擴充功能，然後逐一重新啟用，直到找出有問題的擴充功能為止。

## 除錯原生程式碼

當處理原生程式碼時，例如編寫原生模組時，您可以從 Android Studio 或 Xcode 啟動應用程式，並利用原生除錯功能（設置斷點等），就像在建置標準原生應用程式時一樣。