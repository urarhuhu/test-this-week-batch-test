---
id: profiling
title: Profiling
---

使用內建的效能分析工具來獲取 JavaScript 執行緒與主執行緒工作細節的並列資訊。可透過 Debug 選單中的 Perf Monitor 來存取此功能。

在 iOS 上，Instruments 是不可或缺的工具；而在 Android 上，你應該學習使用 [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace)。

但首先，[**請確保開發者模式已關閉！**](performance.md#running-in-development-mode-devtrue) 你應該在應用程式日誌中看到 `__DEV__ === false, development-level warning are OFF, performance optimizations are ON`。

另一種分析 JavaScript 效能的方式是在除錯時使用 Chrome 的效能分析工具。由於程式碼是在 Chrome 中執行，這不會提供精確的結果，但能讓你大致了解可能的瓶頸所在。在 Chrome 的 `Performance` 分頁下執行分析工具。`User Timing` 下方會出現火焰圖。若要查看表格形式的詳細資訊，點擊下方的 `Bottom Up` 分頁，然後從左上角選單中選擇 `DedicatedWorker Thread`。

## 使用 `systrace` 分析 Android UI 效能

Android 支援超過一萬種不同的手機，並且通用於支援軟體渲染：框架架構與需要跨多種硬體目標的通用性，意味著相對於 iOS，你能獲得的免費資源較少。但有時，仍有可以改進的地方——而且很多時候根本不是原生程式碼的問題！

除錯這種卡頓的第一步，是回答一個基本問題：在每個 16 毫秒的幀中，你的時間都花在哪裡。為此，我們將使用一個標準的 Android 分析工具，稱為 `systrace`。

`systrace` 是一個標準的基於標記的 Android 分析工具（安裝 Android platform-tools 套件時會一併安裝）。被分析的程式碼區塊會被開始/結束標記包圍，然後以彩色圖表形式視覺化。Android SDK 和 React Native 框架都提供了可視覺化的標準標記。

### 1. 收集追蹤資料

首先，透過 USB 將表現出你想要調查的卡頓現象的裝置連接到電腦，並讓它進入你想要分析的導航/動畫之前的那一刻。執行 `systrace` 如下：

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

快速解析此命令：

- `time` 是追蹤資料收集的時間長度，單位為秒
- `sched`、`gfx` 和 `view` 是我們關注的 Android SDK 標記（標記集合）：`sched` 提供關於手機每個核心上正在執行的資訊，`gfx` 提供圖形資訊如幀邊界，`view` 提供關於測量、佈局和繪製過程的資訊
- `-a <your_package_name>` 啟用應用程式特定的標記，特別是內建於 React Native 框架的標記。`your_package_name` 可以在你應用的 `AndroidManifest.xml` 中找到，看起來像 `com.example.app`

一旦追蹤開始收集，執行你關注的動畫或互動。在追蹤結束時，systrace 會提供一個追蹤資料的連結，你可以在瀏覽器中開啟。

### 2. 閱讀追蹤資料

在瀏覽器（最好是 Chrome）中開啟追蹤資料後，你應該會看到類似以下的內容：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用 WASD 鍵來平移和縮放。
:::

如果你的追蹤 .html 檔案無法正確開啟，請檢查瀏覽器控制台是否有以下內容：

![ObjectObserveError](/docs/assets/ObjectObserveError.png)

由於 `Object.observe` 在最近的瀏覽器中已被棄用，你可能需要透過 Google Chrome Tracing 工具來開啟檔案。你可以這樣做：

- 在 Chrome 中開啟分頁 `chrome://tracing`
- 選擇載入
- 選擇上一個命令生成的 html 檔案。

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的這個核取方塊，以高亮標示16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

你應該會看到如上截圖中的斑馬條紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示垂直同步時會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到你的進程

滾動直到看到你的套件名稱（部分）。在這個例子中，我正在分析`com.facebook.adsmanager`，由於核心中執行緒名稱長度的限制，它顯示為`book.adsmanager`。

在左側，你會看到一組執行緒，對應右側的時間軸行。我們關注的幾個執行緒包括：UI執行緒（顯示你的套件名稱或UI Thread）、`mqt_js`和`mqt_native_modules`。如果你在Android 5+上執行，我們還需要關注Render Thread。

- **UI執行緒。** 這是標準Android測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是book.adsmanager）或UI Thread。你在這個執行緒上看到的事件應該與`Choreographer`、`traversals`和`DispatchUI`有關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒。** 這是JavaScript執行的位置。執行緒名稱會是`mqt_js`或`<...>`，取決於你裝置上的核心是否配合。如果沒有名稱，可以尋找`JSCall`、`Bridge.executeJSCall`等來識別它：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（例如`UIManager`）執行的位置。執行緒名稱會是`mqt_native_modules`或`<...>`。在後者的情況下，可以尋找`NativeCall`、`callJavaModuleMethod`和`onBatchComplete`來識別它：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：Render執行緒。** 如果你使用Android L（5.0）及以上版本，你的應用中還會有一個Render執行緒。這個執行緒生成用於繪製UI的實際OpenGL命令。執行緒名稱會是`RenderThread`或`<...>`。在後者的情況下，可以尋找`DrawFrame`和`queueBuffer`來識別它：

  ![Render執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題所在

流暢的動畫應該看起來像這樣：

![流暢的動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的UI工作需要在16毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用是以60 FPS運行的。

如果你注意到卡頓，可能會看到這樣的情況：

![JS導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎一直在執行，並且跨越了幀邊界！這個應用沒有以60 FPS渲染。在這種情況下，**問題出在JS**。

你可能還會看到這樣的情況：

![UI導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI和Render執行緒的工作跨越了幀邊界。我們試圖在每一幀上渲染的UI需要太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你將獲得一些非常有用的信息來指導你的下一步行動。

## 解決 JavaScript 問題

若您確認問題出在 JavaScript 端，請從執行的特定 JS 程式碼中尋找線索。在上述情境中，我們看到每幀多次呼叫 `RCTEventEmitter`。以下是從上方追蹤記錄中放大檢視的 JS 執行緒：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不合理。為何它被如此頻繁地呼叫？這些真的是不同的事件嗎？問題答案通常取決於您的產品程式碼。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題出在原生的 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 在 GPU 上需要過多運算資源，或
2. 您在動畫/互動過程中動態建立新 UI（例如在滾動時載入新內容）。

### GPU 負載過高

第一種情境中，您會看到 UI 執行緒和/或渲染執行緒的追蹤記錄呈現以下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意跨越幀邊界的 `DrawFrame` 長時間執行。這表示 GPU 正在等待清空前一幀的指令緩衝區。

緩解方法包括：

- 對複雜但靜態的動畫/變形內容（如 `Navigator` 的滑動/透明度動畫）使用 `renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的 `needsOffscreenAlphaCompositing`，此選項在多數情況下會大幅增加 GPU 的每幀負載。

### 在 UI 執行緒建立新視圖

第二種情境中，追蹤記錄會呈現類似以下狀態：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

注意 JS 執行緒先進行運算，接著原生模組執行緒處理工作，最後 UI 執行緒執行昂貴的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化新建的 UI 結構，否則目前沒有快速解決方案。React Native 團隊正在基礎架構層面開發解決方案，未來將允許在非主執行緒建立與配置新 UI，確保互動過程保持流暢。