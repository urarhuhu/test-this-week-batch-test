---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的執行效能、資源使用情況和行為模式，來識別潛在瓶頸或效率問題的過程。善用效能分析工具能確保您的應用在不同裝置和環境下流暢運作。

在iOS平台，Instruments是不可或缺的工具；而Android平台則建議掌握[Android Studio Profiler](profiling.md#profiling-android-ui-performance-with-system-tracing)的使用技巧。

但首先請務必[關閉開發者模式！](performance.md#running-in-development-mode-devtrue)您應在應用日誌中看到`__DEV__ === false, 開發層級警告已關閉，效能優化已啟用`的提示。

## 使用系統追蹤分析Android UI效能

Android需支援上萬種手機型號並採用通用軟體渲染架構，這種框架設計與硬體兼容性需求意味著相較iOS平台，開發者需投入更多優化工作。但許多時候，效能問題的根源其實並非原生代碼！

診斷畫面卡頓的第一步，是釐清每個16毫秒幀周期內的耗時分佈。我們將使用[Android Studio內建的System Tracing分析工具](https://developer.android.com/studio/profile)。

### 1. 收集追蹤數據

首先透過USB連接出現卡頓問題的裝置，在Android Studio中開啟專案的`android`資料夾，於右上角面板選擇裝置後，[以可分析模式執行專案](https://developer.android.com/studio/profile#build-and-run)。

當應用以可分析模式在裝置運行後，導航至欲分析的動畫/交互操作前，點擊Android Studio分析器面板中的[「Capture System Activities」按鈕](https://developer.android.com/studio/profile#start-profiling)開始追蹤。

追蹤啟動後執行目標操作，完成後點擊「停止錄製」。您可[直接在Android Studio檢視追蹤數據](https://developer.android.com/studio/profile/jank-detection)，或於「Past Recordings」面板選擇記錄檔後點擊「Export recording」，透過[Perfetto](https://perfetto.dev/)等工具分析。

### 2. 解讀追蹤數據

在Android Studio或Perfetto中開啟追蹤檔案後，您將看到類似下圖的界面：

![範例](/docs/assets/SystraceExample.png)

:::note[操作提示]
使用WASD鍵進行平移和縮放。
:::

不同工具的界面可能略有差異，但以下分析原則通用。

:::info[啟用垂直同步標記]
勾選畫面右上角的核取方塊以標記16毫秒幀邊界：

![啟用垂直同步標記](/docs/assets/SystraceHighlightVSync.png)

如截圖所示應出現斑馬條紋。若未顯示，建議更換測試裝置：三星設備曾有標記顯示問題，Nexus系列通常較可靠。
:::

### 3. 定位進程

滾動直至看見您的套件名稱片段。本例分析的`com.facebook.adsmanager`因系統核心限制顯示為`book.adsmanager`。

左側線程列表對應右側時間軸，我們需重點關注：標註套件名或「UI Thread」的UI線程、`mqt_js`和`mqt_native_modules`。若運行於Android 5+系統，還需注意Render Thread。

- **UI 執行緒。** 這是標準 Android 測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是 book.adsmanager）或 UI Thread。你在這個執行緒上看到的事件應該類似以下內容，並與 `Choreographer`、`traversals` 和 `DispatchUI` 相關：

  ![UI 執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS 執行緒。** 這是執行 JavaScript 的地方。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你設備的內核是否配合。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等來識別它：

  ![JS 執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是執行原生模組呼叫（例如 `UIManager`）的地方。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。在後者的情況下，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 來識別它：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 如果你使用的是 Android L（5.0）及以上版本，你的應用中還會有一個渲染執行緒。這個執行緒生成用於繪製 UI 的實際 OpenGL 指令。執行緒名稱會是 `RenderThread` 或 `<...>`。在後者的情況下，可以透過尋找 `DrawFrame` 和 `queueBuffer` 來識別它：

  ![渲染執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該看起來像以下這樣：

![流暢的動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的 UI 工作需要在 16 毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用是以 60 FPS 運行的。

然而，如果你注意到卡頓，可能會看到類似以下的情況：

![JS 導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意 JS 執行緒幾乎一直在執行，並且跨越幀邊界！這個應用沒有以 60 FPS 渲染。在這種情況下，**問題出在 JS 上**。

你也可能會看到類似以下的情況：

![UI 導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI 和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀渲染的 UI 需要完成太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你將獲得一些非常有用的信息來指導你的下一步。

## 解決 JavaScript 問題

如果你確定是 JS 問題，請查看你執行的具體 JS 代碼以尋找線索。在上面的情境中，我們看到 `RCTEventEmitter` 在每一幀被多次調用。以下是從上述追蹤中放大的 JS 執行緒：

![過多的 JS](/docs/assets/SystraceBadJS2.png)

這看起來不太對。為什麼它被調用得如此頻繁？它們真的是不同的事件嗎？這些問題的答案可能取決於你的產品代碼。很多時候，你會需要查看 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。

## 解決原生 UI 問題

如果你確定是原生 UI 問題，通常有兩種情況：

1. 你試圖在每一幀繪製的 UI 在 GPU 上涉及太多工作，或者
2. 你在動畫/交互過程中構建了新的 UI（例如在滾動時加載新內容）。

### GPU 負載過高

在第一種情境中，您會看到追蹤記錄顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意跨越影格邊界的 `DrawFrame` 長時間執行。這表示 GPU 正在等待排空前一影格的指令緩衝區。

緩解方法包括：

- 對複雜靜態內容（如 `Navigator` 的滑動/透明度動畫）使用 `renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的 `needsOffscreenAlphaCompositing`，此功能在多數情況下會大幅增加 GPU 每影格負載

### 在 UI 執行緒創建新視圖

第二種情境的追蹤記錄會呈現如下模式：

![創建視圖](/docs/assets/SystraceBadCreateUI.png)

可觀察到 JS 執行緒先進行運算，接著原生模組執行緒處理任務，最後 UI 執行緒執行昂貴的遍歷操作

除非能延遲至互動結束後創建新 UI，或簡化 UI 結構，否則無快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，將允許在非主執行緒創建設定新 UI，確保互動流暢度

### 定位原生 CPU 熱點

若問題疑似源自原生端，可使用 [CPU 熱點分析器](https://developer.android.com/studio/profile/record-java-kotlin-methods)進一步診斷。開啟 Android Studio Profiler 面板並選擇「Find CPU Hotspots (Java/Kotlin Method Recording)」

:::info[選擇 Java/Kotlin 記錄]

務必選擇「Find CPU Hotspots **(Java/Kotlin Recording)**」而非「Find CPU Hotspots (Callstack Sample)」。兩者圖示相似但功能不同
:::

執行互動後點擊「Stop recording」。記錄過程會消耗資源，建議縮短互動時間。產生的追蹤記錄可在 Android Studio 分析，或匯出至 [Firefox Profiler](https://profiler.firefox.com/) 等線上工具

與系統追蹤不同，CPU 熱點分析速度較慢且無法提供精確測量，但能顯示原生方法呼叫狀況及各影格時間佔比分布