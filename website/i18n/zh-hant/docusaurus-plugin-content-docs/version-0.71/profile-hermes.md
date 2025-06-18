---
id: profile-hermes
title: Profiling with Hermes
---

您可以使用 [Hermes](https://github.com/facebook/hermes) 來視覺化 React Native 應用程式中 JavaScript 的效能表現。Hermes 是一個輕量級的 JavaScript 引擎，專為運行 React Native 優化（您可以在[這裡閱讀更多關於與 React Native 搭配使用的資訊](hermes)）。Hermes 有助於提升應用程式效能，同時也提供了分析其所運行 JavaScript 效能的方法。

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

### 啟用原始碼映射

:::info
您可以在[原始碼映射](sourcemaps.md)頁面閱讀更多關於原始碼映射的資訊。
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

Hermes 取樣效能分析檔採用的是 `JSON 物件格式`，而 Google DevTools 支援的則是 `JSON 陣列格式`。（更多關於格式的資訊可參考 [Trace Event 格式文件](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview)）

```ts
export interface HermesCPUProfile {
  traceEvents: SharedEventProperties[];
  samples: HermesSample[];
  stackFrames: {[key in string]: HermesStackFrame};
}
```

Hermes 效能分析檔的主要資訊編碼在 `samples` 和 `stackFrames` 屬性中。每個取樣點都是特定時間戳下的函式呼叫堆疊快照，每個取樣點的 `sf` 屬性對應到一個函式呼叫。

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

函式呼叫的詳細資訊儲存在 `stackFrames` 中，其內容為鍵值對結構——鍵是 `sf` 編號，對應的物件則提供該函式的所有相關資訊，包含其父函式的 `sf` 編號。透過這種父子關係向上追溯，就能找出特定時間戳下所有執行中函式的資訊。

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

此時需要明確定義幾個術語：

1. 節點（Nodes）：`stackFrames` 中對應 `sf` 編號的物件
2. 活躍節點（Active Nodes）：特定時間戳下正在執行的節點。若某節點的 `sf` 編號存在於函式呼叫堆疊中（可透過樣本的 `sf` 編號向上追溯父級 `sf` 取得），即被歸類為執行中

結合 `samples` 和 `stackFrames` 即可在對應時間戳生成所有開始與結束事件：

1. 開始節點/事件：存在於當前樣本呼叫堆疊，但不存在於前一樣本中的節點
2. 結束節點/事件：存在於前一樣本呼叫堆疊，但不存在於當前樣本中的節點

<img src="/docs/assets/CallStackDemo.jpg" height="800" width="600" alt="CallStack Terms Explained" />

至此即可建構出函式呼叫的 `火焰圖（flamechart）`，因為已掌握所有函式資訊（包含開始與結束時間戳）。

`hermes-profile-transformer` 能將任何 Hermes 生成的效能分析檔轉換為 Chrome DevTools 可直接顯示的格式。更多資訊請參考 [ `@react-native-community/hermes-profile-transformer` ](https://github.com/react-native-community/hermes-profile-transformer)