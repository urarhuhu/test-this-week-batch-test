---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的效能、資源使用情況和行為來識別潛在瓶頸或低效問題的過程。使用效能分析工具能確保您的應用在不同裝置和條件下都能流暢運行。

在iOS上，Instruments是不可或缺的工具；而在Android平台，您應該學習使用[Android Studio Profiler](profiling.md#profiling-android-ui-performance-with-system-tracing)。

但首先，[**請確保開發者模式已關閉！**](performance.md#running-in-development-mode-devtrue) 您應在應用日誌中看到`__DEV__ === false, 開發級警告已關閉, 效能優化已啟用`的提示。

## 使用系統追蹤分析Android UI效能

Android支援上萬種不同手機型號，並通用於軟體渲染架構：這種框架設計和對多樣硬體的兼容性，意味著相較iOS您需要更多自主優化。但有些時候問題並非源自原生代碼！

偵測卡頓的第一步是確認每16毫秒幀的時間消耗分佈。我們將使用[Android Studio內建的System Tracing分析工具](https://developer.android.com/studio/profile)。

### 1. 收集追蹤數據

首先透過USB連接出現卡頓的裝置到電腦。在Android Studio中開啟專案的`android`資料夾，於右上角面板選擇裝置，並[以可分析模式運行專案](https://developer.android.com/studio/profile#build-and-run)。

當應用以可分析模式建置並運行於裝置時，導航至欲分析的動畫前階段，在Android Studio Profiler面板啟動[「Capture System Activities」任務](https://developer.android.com/studio/profile#start-profiling)。

開始記錄後執行目標動畫操作，點擊「Stop recording」結束。[可直接在Android Studio檢視追蹤數據](https://developer.android.com/studio/profile/jank-detection)，或於「Past Recordings」面板選擇記錄點擊「Export recording」後用[Perfetto](https://perfetto.dev/)等工具開啟。

### 2. 解讀追蹤數據

在Android Studio或Perfetto中開啟追蹤文件後，您將看到類似以下畫面：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用WASD鍵進行平移和縮放。
:::

介面可能略有差異，但下列說明適用於任何分析工具。

:::info[啟用垂直同步高亮]
勾選畫面右上角此核取方塊以標記16毫秒幀邊界：

![啟用垂直同步高亮](/docs/assets/SystraceHighlightVSync.png)

您應看到如上圖的斑馬條紋。若未顯示，請嘗試在其他裝置分析：三星裝置可能無法正確顯示垂直同步標記，Nexus系列通常較可靠。
:::

### 3. 定位您的進程

滾動直至看見您的套件名稱（部分）。本例中分析的是`com.facebook.adsmanager`，因核心執行緒名稱限制顯示為`book.adsmanager`。

左側面板顯示的執行緒列表對應右側時間軸。我們主要關注：UI執行緒（顯示您的套件名或UI Thread）、`mqt_js`和`mqt_native_modules`。若運行於Android 5+系統，還需關注Render Thread。

- **UI 執行緒。** 這是標準 Android 測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的案例中是 book.adsmanager）或 UI Thread。你在這個執行緒上看到的事件應該類似這樣，並與 `Choreographer`、`traversals` 和 `DispatchUI` 相關：

  ![UI 執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS 執行緒。** 這是 JavaScript 執行的位置。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你裝置上的核心是否配合。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等來識別它：

  ![JS 執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 這是原生模組呼叫（例如 `UIManager`）執行的位置。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。在後者的情況下，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 等來識別它：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。** 如果你使用的是 Android L（5.0）及以上版本，你的應用程式中還會有一個渲染執行緒。這個執行緒生成用於繪製 UI 的實際 OpenGL 指令。執行緒名稱會是 `RenderThread` 或 `<...>`。在後者的情況下，可以透過尋找 `DrawFrame` 和 `queueBuffer` 等來識別它：

  ![渲染執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該看起來像這樣：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的 UI 工作需要在 16 毫秒的週期內完成。注意沒有任何執行緒的工作接近幀邊界。這樣渲染的應用程式是以 60 FPS 運行的。

如果你注意到卡頓，可能會看到類似這樣的情況：

![JS 導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意 JS 執行緒幾乎一直在執行，並且跨越幀邊界！這個應用程式沒有以 60 FPS 渲染。在這種情況下，**問題出在 JS**。

你也可能會看到這樣的情況：

![UI 導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

在這種情況下，UI 和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀渲染的 UI 需要太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你已經掌握了一些非常有用的資訊來指導下一步行動。

## 解決 JavaScript 問題

如果你確認是 JS 問題，請在執行的特定 JS 中尋找線索。在上面的情境中，我們看到 `RCTEventEmitter` 每幀被呼叫多次。這是從上述追蹤中放大的 JS 執行緒：

![過多 JS](/docs/assets/SystraceBadJS2.png)

這看起來不太對。為什麼它被呼叫得如此頻繁？它們實際上是不同的事件嗎？這些問題的答案可能取決於你的產品代碼。很多時候，你可能需要查看 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。

## 解決原生 UI 問題

如果你確認是原生 UI 問題，通常有兩種情況：

1. 你試圖在每一幀繪製的 UI 在 GPU 上涉及太多工作，或者
2. 你在動畫/互動期間構建新的 UI（例如在滾動時載入新內容）。

### GPU工作負載過高

在第一種情境中，您會看到追蹤記錄中UI執行緒和/或渲染執行緒呈現如下狀態：

![GPU超載](/docs/assets/SystraceBadUI.png)

注意跨越影格邊界的`DrawFrame`長時間執行。這表示GPU正在等待清空前一影格的指令緩衝區。

緩解方法包括：

- 對複雜靜態內容的動畫/變形（如`Navigator`滑動/透明度動畫）使用`renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的`needsOffscreenAlphaCompositing`，該功能在多數情況下會大幅增加GPU每影格負載

### 在UI執行緒創建新視圖

第二種情境的追蹤記錄會呈現如下模式：

![創建視圖](/docs/assets/SystraceBadCreateUI.png)

注意JS執行緒先進行運算，接著原生模組執行緒處理任務，最後UI執行緒執行昂貴的遍歷操作

除非能延遲建立新UI至互動結束後，或簡化新建UI的複雜度，否則無快速解決方案。React Native團隊正在開發基礎架構級解決方案，將允許在非主執行緒創建與配置新UI，確保互動流暢性

### 定位原生CPU熱點

若問題源自原生端，可使用[CPU熱點分析器](https://developer.android.com/studio/profile/record-java-kotlin-methods)獲取詳細資訊。開啟Android Studio分析器面板，選擇「記錄Java/Kotlin方法」

:::info[選擇Java/Kotlin記錄]

務必選擇「查找CPU熱點**(Java/Kotlin記錄)**」而非「查找CPU熱點(呼叫堆疊取樣)」。兩者圖標相似但功能不同
:::

執行互動後點擊「停止記錄」。記錄過程資源消耗大，請保持互動簡短。結果追蹤可在Android Studio檢視，或匯出至[Firefox Profiler](https://profiler.firefox.com/)等線上工具分析

與系統追蹤不同，CPU熱點分析速度較慢且無法提供精確測量，但能顯示原生方法呼叫情況及各影格時間消耗比例分佈