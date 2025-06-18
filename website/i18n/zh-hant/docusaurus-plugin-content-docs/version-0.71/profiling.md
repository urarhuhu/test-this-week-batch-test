---
id: profiling
title: Profiling
---

使用內建分析器獲取 JavaScript 執行緒與主執行緒工作的詳細並列資訊。透過 Debug 選單選擇「效能監測器」即可開啟。

在 iOS 上，Instruments 是不可或缺的工具；而在 Android 平台，你應該學習使用 [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)。

但首先，[**務必關閉開發者模式！**](performance.md#running-in-development-mode-devtrue) 你應該在應用程式日誌中看到 `__DEV__ === false, development-level warning are OFF, performance optimizations are ON` 的提示。

另一種分析 JavaScript 效能的方式是在除錯時使用 Chrome 分析器。由於程式碼是在 Chrome 中執行，這不會提供精確結果，但能幫助定位潛在瓶頸。在 Chrome 的「Performance」分頁下執行分析器，「User Timing」區會顯示火焰圖。若要查看表格化詳細數據，點擊下方的「Bottom Up」分頁，然後從左上選單選擇「DedicatedWorker Thread」。

## 使用 `systrace` 分析 Android UI 效能

Android 支援上萬種裝置且採用通用軟體渲染架構：這種框架設計與硬體兼容性需求，意味著相較 iOS 你能獲得的原生優化較少。但有時仍有改進空間——許多時候問題甚至與原生程式碼無關！

偵測卡頓的第一步是釐清每個 16ms 幀的時間消耗分佈。我們將使用 Android 標準分析工具 `systrace`。

`systrace` 是 Android 標準的基於標記的分析工具（安裝 Android platform-tools 套件時會一併安裝）。被分析的程式區塊會被開始/結束標記包圍，並以彩色圖表視覺化呈現。Android SDK 和 React Native 框架都提供可視化的標準標記。

### 1. 收集追蹤數據

首先透過 USB 連接出現卡頓的裝置至電腦，並將裝置狀態調整至即將進行要分析的導航/動畫前。執行以下 `systrace` 指令：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

指令快速解析：

- `time` 是追蹤收集時間長度（秒）
- `sched`、`gfx` 和 `view` 是我們關注的 Android SDK 標記群組：`sched` 提供手機各核心的執行資訊，`gfx` 提供圖形相關數據如幀邊界，`view` 提供測量、佈局和繪製階段的資訊
- `-a <your_package_name>` 啟用應用專用標記，特別是 React Native 框架內建的標記。`your_package_name` 可在應用的 `AndroidManifest.xml` 中找到，格式如 `com.example.app`

追蹤開始後，執行你要分析的動畫或互動操作。追蹤結束時，systrace 會提供可在瀏覽器開啟的追蹤檔案連結。

### 2. 解讀追蹤數據

在瀏覽器（建議 Chrome）開啟追蹤檔案後，你會看到類似下圖的內容：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用 WASD 鍵平移與縮放視圖。
:::

若追蹤 .html 檔案無法正確開啟，請檢查瀏覽器控制台是否出現以下錯誤：

![ObjectObserveError](/docs/assets/ObjectObserveError.png)

由於 `Object.observe` 已在新版瀏覽器中棄用，你可能需要透過 Google Chrome Tracing 工具開啟檔案。操作步驟：

- 在 Chrome 開啟分頁 `chrome://tracing`
- 點擊「載入」按鈕
- 選擇先前指令生成的 html 檔案

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的此核取方塊以高亮顯示16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應該會看到如上截圖中的斑馬條紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示垂直同步時可能會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到您的進程

滾動直到看見您的套件名稱（部分）。在此案例中，我分析的對象是`com.facebook.adsmanager`，由於核心執行緒名稱長度限制，顯示為`book.adsmanager`。

左側會顯示一組執行緒，對應右側時間軸的行數。我們關注以下幾個執行緒：UI執行緒（顯示您的套件名稱或「UI Thread」）、`mqt_js`和`mqt_native_modules`。若在Android 5+上執行，還需關注Render Thread。

- **UI執行緒**：標準Android測量/佈局/繪製的執行位置。右側執行緒名稱會是您的套件名稱（如我的案例為book.adsmanager）或「UI Thread」。此執行緒上的事件應類似以下內容，並與`Choreographer`、`traversals`和`DispatchUI`相關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒**：JavaScript執行的位置。執行緒名稱會是`mqt_js`或`<...>`（取決於裝置核心的配合度）。若無名稱，可透過尋找`JSCall`、`Bridge.executeJSCall`等來識別：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒**：原生模組呼叫（如`UIManager`）執行的位置。執行緒名稱會是`mqt_native_modules`或`<...>`。若無名稱，可透過尋找`NativeCall`、`callJavaModuleMethod`和`onBatchComplete`來識別：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：Render執行緒**：若使用Android L（5.0）及以上版本，您的應用中還會有一個Render執行緒。此執行緒產生用於繪製UI的實際OpenGL指令。執行緒名稱會是`RenderThread`或`<...>`。若無名稱，可透過尋找`DrawFrame`和`queueBuffer`來識別：

  ![Render執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應如下所示：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色變化代表一幀——請記住，要顯示一幀，所有UI工作必須在16毫秒內完成。注意沒有任何執行緒的工作接近幀邊界。如此渲染的應用能以60 FPS運行。

若發現卡頓，可能會看到如下情況：

![JS導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎全程執行，且跨越幀邊界！此應用未以60 FPS渲染。**問題出在JS**。

您也可能看到以下情況：

![UI導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

此案例中，UI和Render執行緒的工作跨越了幀邊界。每幀嘗試渲染的UI需要過多工作。**問題出於渲染的原生視圖**。

此時，您將獲得關鍵資訊以決定後續步驟。

## 解決 JavaScript 問題

若您確認問題出在 JavaScript 層面，請從執行的具體 JS 程式碼中尋找線索。在上述情境中，我們觀察到每幀都會多次呼叫 `RCTEventEmitter`。以下是從前述追蹤記錄中放大檢視 JS 執行緒的畫面：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不正常。為何呼叫如此頻繁？這些是否為不同事件？答案通常取決於您的產品程式碼。多數情況下，您需要檢視 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題源自原生 UI，通常有兩種情境：

1. 每幀繪製的 UI 對 GPU 造成過大負載，或
2. 在動畫/互動過程中動態建立新 UI（例如滾動時載入新內容）。

### GPU 負載過高

第一種情境下，追蹤記錄中的 UI 執行緒和/或渲染執行緒會呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意 `DrawFrame` 耗時過長且跨越幀邊界，這表示 GPU 正在排空前一幀的命令緩衝區。

緩解方法包括：

- 對複雜靜態內容（如 `Navigator` 的滑動/透明度動畫）使用 `renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的 `needsOffscreenAlphaCompositing`，此功能在多數情況下會大幅增加 GPU 每幀負載。

若仍無法解決，可透過 [Tracer for OpenGL ES](http://www.androiddocs.com/tools/help/gltracer.html) 深入分析 GPU 運作細節。

### 在 UI 執行緒建立新視圖

第二種情境的追蹤記錄會呈現以下模式：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

可見 JS 執行緒先進行運算，接著原生模組執行緒處理任務，最後 UI 執行緒執行昂貴的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化 UI 結構，否則無快速解決方案。React Native 團隊正開發基礎架構層級的解決方案，將允許在非主執行緒建立與配置新 UI，確保互動流暢度。