---
id: profile-hermes
title: Profiling with Hermes
---

您可以使用 [Hermes](https://github.com/facebook/hermes) 來視覺化 React Native 應用程式中 JavaScript 的效能表現。Hermes 是一個輕量級的 JavaScript 引擎，專為運行 React Native 優化（您可以在[這裡閱讀更多關於與 React Native 搭配使用的資訊](hermes)）。Hermes 有助於提升應用程式效能，並提供了分析其所運行 JavaScript 效能的方法。

在本節中，您將學習如何對運行於 Hermes 上的 React Native 應用程式進行效能分析，以及如何使用 [Chrome DevTools 的 Performance 分頁](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference) 來視覺化分析結果。

:::caution
開始之前，請確保您已[在應用程式中啟用 Hermes](hermes)！
:::

請按照以下指示開始進行效能分析：

1. [記錄 Hermes 取樣分析檔](profile-hermes.md#record-a-hermes-sampling-profile)
2. [從 CLI 執行指令](profile-hermes.md#execute-command-from-cli)
3. [在 Chrome DevTools 中開啟下載的分析檔](profile-hermes.md#open-the-downloaded-profile-on-chrome-devtools)

## 記錄 Hermes 取樣分析檔

要從開發者選單記錄取樣分析器：

1. 導航至您正在運行的 Metro 伺服器終端。
2. 按下 `d` 開啟**開發者選單**。
3. 選擇**啟用取樣分析器**。
4. 在應用程式中執行您的 JavaScript（例如按鈕操作等）。
5. 再次按下 `d` 開啟**開發者選單**。
6. 選擇**停用取樣分析器**以停止記錄並儲存取樣分析器。

一個提示訊息會顯示取樣分析器儲存的位置，通常位於 `/data/user/0/com.appName/cache/*.cpuprofile`。

<img src="/docs/assets/HermesProfileSaved.png" height="465" width="250" alt="Toast Notification of Profile saving" />

## 從 CLI 執行指令

您可以使用 [React Native CLI](https://github.com/react-native-community/cli) 將 Hermes 追蹤分析檔轉換為 Chrome 追蹤分析檔，然後使用以下指令將其拉取到本地機器：

```sh
npx react-native profile-hermes [destinationDir]
```

### 啟用原始碼對應

:::info
您可以在[除錯發行版本](debugging-release-builds.md)頁面閱讀關於原始碼對應的更多資訊。
:::

### 常見錯誤

#### `adb: no devices/emulators found` 或 `adb: device offline`

- **原因** CLI 無法存取您用來運行應用程式的裝置或模擬器（透過 adb）。
- **解決方法** 確保您的 Android 裝置/模擬器已連接並正在運行。該指令僅在能夠存取 adb 時有效。

#### `There is no file in the cache/ directory`

- **原因** CLI 在您應用程式的 **cache/** 目錄中找不到任何 **.cpuprofile** 檔案。您可能忘記從裝置記錄分析檔。
- **解決方法** 按照[指示](profile-hermes.md#record-a-hermes-sampling-profile)從裝置啟用/停用分析器。

#### `Error: your_profile_name.cpuprofile is an empty file`

- **原因** 分析檔為空，可能是因為 Hermes 未正確運行。
- **解決方法** 確保您的應用程式運行於最新版本的 Hermes 上。

## 在 Chrome DevTools 中開啟下載的分析檔

要在 Chrome DevTools 中開啟分析檔：

1. 開啟 Chrome DevTools。
2. 選擇 **Performance** 分頁。
3. 右鍵點擊並選擇 **Load profile...**。

<img src="/docs/assets/openChromeProfile.png" alt="Loading a performance profile on Chrome DevTools" />

## Hermes 分析轉換器是如何工作的？

Hermes 取樣分析檔案的格式為 `JSON 物件格式`，而 Google DevTools 支援的格式則是 `JSON 陣列格式`。（更多關於格式的資訊可參考 [Trace Event 格式文件](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview)）

```ts
export interface HermesCPUProfile {
  traceEvents: SharedEventProperties[];
  samples: HermesSample[];
  stackFrames: {[key in string]: HermesStackFrame};
}
```

Hermes 分析檔案的大部分資訊都編碼在 `samples` 和 `stackFrames` 屬性中。每個取樣點都是特定時間戳下的函式呼叫堆疊快照，每個取樣點都有一個 `sf` 屬性，對應到一個函式呼叫。

```ts
export interface HermesSample {
  cpu: string;
  name: string;
  ts: string;
  pid: number;
  tid: string;
  weight: string;
  /**
   * Will refer to an element in the stackFrames object of the Hermes Profile
   */
  sf: number;
  stackFrameData?: HermesStackFrame;
}
```

關於函式呼叫的資訊可在 `stackFrames` 中找到，其中包含鍵值對，鍵是 `sf` 編號，對應的物件則提供該函式的所有相關資訊，包括其父函式的 `sf` 編號。這種父子關係可向上追溯，以找出特定時間戳下所有執行中函式的資訊。

```ts
export interface HermesStackFrame {
  line: string;
  column: string;
  funcLine: string;
  funcColumn: string;
  name: string;
  category: string;
  /**
   * A parent function may or may not exist
   */
  parent?: number;
}
```

此時，您應先定義幾個術語：

1. 節點（Nodes）：`stackFrames` 中對應 `sf` 編號的物件
2. 活動節點（Active Nodes）：當前時間戳下正在執行的節點。若某節點的 `sf` 編號存在於函式呼叫堆疊中，則該節點被歸類為執行中。此呼叫堆疊可從取樣點的 `sf` 編號開始，向上追溯父 `sf` 編號來取得

結合 `samples` 和 `stackFrames`，即可在對應時間戳下生成所有開始與結束事件，其中：

1. 開始節點/事件（Start Nodes/Events）：在前一個取樣點的呼叫堆疊中不存在，但在當前取樣點中出現的節點
2. 結束節點/事件（End Nodes/Events）：在前一個取樣點的呼叫堆疊中存在，但在當前取樣點中消失的節點

<img src="/docs/assets/CallStackDemo.jpg" height="800" width="600" alt="CallStack Terms Explained" />

現在您已擁有所有函式資訊（包括其開始與結束時間戳），可據此建構函式呼叫的 `火焰圖（flamechart）`。

`hermes-profile-transformer` 能將任何使用 Hermes 生成的分析檔案轉換為可直接在 Chrome DevTools 中顯示的格式。更多相關資訊請參考 [ `@react-native-community/hermes-profile-transformer` ](https://github.com/react-native-community/hermes-profile-transformer)