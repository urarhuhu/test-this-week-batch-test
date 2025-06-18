---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的效能、資源使用狀況與行為模式，來識別潛在瓶頸或效率問題的過程。善用分析工具能確保您的應用程式在不同裝置與條件下流暢運作。

在iOS平台，Instruments是不可或缺的工具；而在Android平台，您應該學習使用[`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)。

但首先，[**請務必關閉開發者模式！**](performance.md#running-in-development-mode-devtrue) 您應在應用程式日誌中看到 `__DEV__ === false, development-level warning are OFF, performance optimizations are ON` 的提示。

## 使用 `systrace` 分析 Android UI 效能

Android 支援上萬種手機型號並採用通用軟體渲染架構：這種框架設計與硬體兼容性需求，意味著相較於 iOS 平台，您需要自行優化的部分更多。但許多時候效能問題的根源其實並非原生程式碼！

偵測畫面卡頓的第一步，是釐清每個16毫秒幀周期中的時間消耗分佈。我們將使用 Android 標準分析工具 `systrace` 來達成此目的。

`systrace` 是 Android 標準的基於標記的分析工具（隨 Android platform-tools 套件安裝）。被分析的程式區塊會被起始/結束標記包圍，並以彩色圖表形式可視化。Android SDK 與 React Native 框架皆提供標準標記供您分析。

### 1. 收集追蹤資料

首先透過USB連接會出現卡頓現象的裝置，並將應用程式導航至您想分析的動畫觸發前狀態。執行下列 `systrace` 指令：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

指令參數說明：

- `time` 代表追蹤收集時間長度（秒）
- `sched`、`gfx` 和 `view` 是我們需要監測的Android SDK標記群：`sched` 提供手機各核心的執行緒資訊，`gfx` 包含圖形相關數據如幀邊界，`view` 則記錄測量、佈局與繪製階段的資訊
- `-a <your_package_name>` 啟用應用專用標記（特別是React Native框架內建標記）。套件名稱可於 `AndroidManifest.xml` 中找到，格式如 `com.example.app`

開始收集追蹤資料後，立即執行您想分析的動畫或互動操作。追蹤結束時，systrace 會提供瀏覽器可開啟的追蹤檔案連結。

### 2. 解讀追蹤結果

在瀏覽器（建議Chrome）中開啟追蹤檔案後，您會看到類似以下畫面：

![範例](/docs/assets/SystraceExample.png)

:::note[操作提示]
使用 WASD 鍵進行平移與縮放。
:::

若追蹤.html檔案無法正常開啟，請檢查瀏覽器控制台是否出現以下錯誤：

![ObjectObserve錯誤](/docs/assets/ObjectObserveError.png)

由於現代瀏覽器已棄用 `Object.observe` 功能，您可能需要透過Chrome Tracing工具開啟檔案：

- 在Chrome地址欄輸入 `chrome://tracing`
- 點擊「載入」按鈕
- 選擇先前指令產生的html檔案

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的這個核取方塊以高亮標示16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應該會看到如上截圖中的斑馬條紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示垂直同步時可能會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到您的進程

滾動直到看見您的套件名稱（部分）。在此案例中，我正在分析`com.facebook.adsmanager`，由於核心執行緒名稱長度限制，它顯示為`book.adsmanager`。

左側會顯示一組執行緒，對應右側時間軸的行。我們關注以下幾個執行緒：UI執行緒（顯示您的套件名稱或「UI Thread」）、`mqt_js`和`mqt_native_modules`。若在Android 5+上執行，還需關注Render Thread。

- **UI執行緒。** 這是Android標準的測量/佈局/繪製發生處。右側執行緒名稱會是您的套件名稱（本例為book.adsmanager）或「UI Thread」。此執行緒上的事件應類似以下內容，與`Choreographer`、`traversals`和`DispatchUI`相關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒。** 這是JavaScript執行處。執行緒名稱會是`mqt_js`或`<...>`（視裝置核心配合度而定）。若無名稱，可透過尋找`JSCall`、`Bridge.executeJSCall`等來識別：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（如`UIManager`）執行處。執行緒名稱會是`mqt_native_modules`或`<...>`。若為後者，可透過尋找`NativeCall`、`callJavaModuleMethod`和`onBatchComplete`來識別：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 若使用Android L（5.0）及以上版本，您的應用還會有一個渲染執行緒。此執行緒產生用於繪製UI的實際OpenGL指令。執行緒名稱會是`RenderThread`或`<...>`。若為後者，可透過尋找`DrawFrame`和`queueBuffer`來識別：

  ![渲染執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應如下所示：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色變化代表一幀——請記住，要顯示一幀，所有UI工作都需在16毫秒週期內完成。注意沒有執行緒的工作接近幀邊界。如此渲染的應用能以60 FPS運行。

若發現卡頓，可能會看到如下情況：

![JS導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎全程工作，且跨越幀邊界！此應用未以60 FPS渲染。**問題出在JS**。

您也可能看到如下情況：

![UI導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

此案例中，UI和渲染執行緒的工作跨越了幀邊界。每幀要渲染的UI需要過多工作量。**問題出於正在渲染的原生視圖**。

此時，您已獲得寶貴資訊來決定後續步驟。

## 解決 JavaScript 問題

若您確認問題出在 JavaScript 端，請從執行的具體 JS 程式碼中尋找線索。在上述情境中，我們觀察到每幀畫面多次呼叫 `RCTEventEmitter`。以下是從前述追蹤記錄中放大檢視的 JS 執行緒：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不正常。為何呼叫如此頻繁？這些是否為不同事件？答案通常取決於您的產品程式碼。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題根源在原生 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 在 GPU 端負載過高，或
2. 在動畫/互動過程中動態建立新 UI（例如滾動時載入新內容）。

### GPU 負載過高

第一種情境下，您會看到 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意跨越幀邊界的 `DrawFrame` 長時間執行。這表示 GPU 正在處理前一幀的指令緩衝區。

緩解方法包括：

- 對複雜靜態內容（如 `Navigator` 的滑動/透明度動畫）使用 `renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的 `needsOffscreenAlphaCompositing`，此選項在多數情況下會大幅增加 GPU 每幀負載。

### 在 UI 執行緒建立新視圖

第二種情境的追蹤記錄會呈現如下模式：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

注意 JS 執行緒先進行運算，接著原生模組執行緒處理任務，最後 UI 執行緒執行高成本的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化新建 UI 的複雜度，否則暫無快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，將允許在非主執行緒建立與配置新 UI，確保互動流暢度。