---
id: profiling
title: Profiling
---

效能分析是透過監測應用程式的效能、資源使用情況和行為來識別潛在瓶頸或低效問題的過程。建議使用效能分析工具來確保應用程式在不同裝置和條件下都能流暢運作。

對於iOS，Instruments是一個極具價值的工具；而在Android上，您應該學習使用[Android Studio Profiler](profiling.md#profiling-android-ui-performance-with-system-tracing)。

但首先，[**請確保開發者模式已關閉！**](performance.md#running-in-development-mode-devtrue)您應該在應用程式日誌中看到`__DEV__ === false, development-level warning are OFF, performance optimizations are ON`。

## 使用系統追蹤分析Android UI效能

Android支援超過1萬種不同的手機型號，並通用於軟體渲染：框架架構和需要兼容多種硬體目標的特性，意味著相對於iOS，您能獲得的免費優化較少。但有時仍有改進空間——很多時候問題甚至不在原生代碼！

偵測卡頓的第一步是回答一個基本問題：在每個16毫秒的幀期間，時間都花在哪裡？為此，我們將使用[Android Studio內建的系統追蹤分析工具](https://developer.android.com/studio/profile)。

### 1. 收集追蹤數據

首先，透過USB將出現卡頓問題的裝置連接到電腦。在Android Studio中打開專案的`android`資料夾，在右上角面板選擇您的裝置，然後[以可分析模式運行專案](https://developer.android.com/studio/profile#build-and-run)。

當應用程式以可分析模式構建並在裝置上運行後，將應用程式導航到您想分析的動畫或互動前一刻，然後在Android Studio分析器面板中啟動["捕獲系統活動"任務](https://developer.android.com/studio/profile#start-profiling)。

開始收集追蹤數據後，執行您關注的動畫或互動操作。接著點擊"停止錄製"。您現在可以直接[在Android Studio中檢查追蹤數據](https://developer.android.com/studio/profile/jank-detection)。或者，您可以在"過往錄製"面板中選擇它，點擊"導出錄製"，然後在[Perfetto](https://perfetto.dev/)等工具中打開。

### 2. 解讀追蹤數據

在Android Studio或Perfetto中打開追蹤數據後，您應該會看到類似以下的內容：

![範例](/docs/assets/SystraceExample.png)

:::note[提示]
使用WASD鍵來平移和縮放。
:::

具體界面可能有所不同，但以下說明適用於您使用的任何工具。

:::info[啟用VSync高亮顯示]
勾選畫面右上角的這個核取方塊以高亮顯示16毫秒幀邊界：

![啟用VSync高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應該會看到如上截圖中的斑馬紋。如果沒有，請嘗試在其他裝置上進行分析：已知三星裝置在顯示vsync時可能會有問題，而Nexus系列通常較為可靠。
:::

### 3. 找到您的進程

滾動直到看到您套件名稱的部分。在這個例子中，我正在分析`com.facebook.adsmanager`，由於內核中執行緒名稱長度限制，它顯示為`book.adsmanager`。

左側您會看到一組執行緒，對應右側時間軸上的行。我們關注幾個關鍵執行緒：UI執行緒（顯示您的套件名稱或UI Thread）、`mqt_js`和`mqt_native_modules`。如果您在Android 5+上運行，我們還需要關注Render Thread。

- **UI 執行緒。**這是標準 Android 測量/佈局/繪製發生的地方。右側的執行緒名稱會是你的套件名稱（在我的例子中是 book.adsmanager）或 UI Thread。你在這個執行緒上看到的事件應該類似這樣，並與 `Choreographer`、`traversals` 和 `DispatchUI` 有關：

  ![UI Thread Example](/docs/assets/SystraceUIThreadExample.png)

- **JS 執行緒。**這是執行 JavaScript 的地方。執行緒名稱會是 `mqt_js` 或 `<...>`，取決於你設備上的核心是否配合。如果沒有名稱，可以透過尋找 `JSCall`、`Bridge.executeJSCall` 等來識別它：

  ![JS Thread Example](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。**這是執行原生模組呼叫（例如 `UIManager`）的地方。執行緒名稱會是 `mqt_native_modules` 或 `<...>`。在後者的情況下，可以透過尋找 `NativeCall`、`callJavaModuleMethod` 和 `onBatchComplete` 來識別它：

  ![Native Modules Thread Example](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：渲染執行緒。**如果你使用的是 Android L（5.0）及以上版本，你的應用程式中還會有一個渲染執行緒。這個執行緒生成用於繪製 UI 的實際 OpenGL 命令。執行緒名稱會是 `RenderThread` 或 `<...>`。在後者的情況下，可以透過尋找 `DrawFrame` 和 `queueBuffer` 來識別它：

  ![Render Thread Example](/docs/assets/SystraceRenderThreadExample.png)

## 識別問題根源

流暢的動畫應該看起來像這樣：

![Smooth Animation](/docs/assets/SystraceWellBehaved.png)

每個顏色的變化代表一幀——記住，為了顯示一幀，我們所有的 UI 工作需要在 16 毫秒的週期內完成。注意沒有執行緒的工作接近幀邊界。這樣渲染的應用程式是以 60 FPS 運行的。

如果你注意到卡頓，可能會看到類似這樣的情況：

![Choppy Animation from JS](/docs/assets/SystraceBadJS.png)

注意 JS 執行緒幾乎一直在執行，並且跨越幀邊界！這個應用程式沒有以 60 FPS 渲染。在這種情況下，**問題出在 JS**。

你也可能會看到這樣的情況：

![Choppy Animation from UI](/docs/assets/SystraceBadUI.png)

在這種情況下，UI 和渲染執行緒的工作跨越了幀邊界。我們試圖在每一幀渲染的 UI 需要完成太多工作。在這種情況下，**問題出於正在渲染的原生視圖**。

此時，你已經掌握了一些非常有用的資訊來指導下一步行動。

## 解決 JavaScript 問題

如果你識別出 JS 問題，請在執行的特定 JS 中尋找線索。在上面的情境中，我們看到 `RCTEventEmitter` 每幀被呼叫多次。這是從上述追蹤中放大的 JS 執行緒：

![Too much JS](/docs/assets/SystraceBadJS2.png)

這看起來不太對。為什麼它被呼叫得這麼頻繁？它們真的是不同的事件嗎？這些問題的答案可能取決於你的產品代碼。很多時候，你可能需要查看 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。

## 解決原生 UI 問題

如果你識別出原生 UI 問題，通常有兩種情況：

1. 你試圖在每一幀繪製的 UI 在 GPU 上涉及太多工作，或者
2. 你在動畫/互動期間構建了新的 UI（例如在滾動時載入新內容）。

### GPU負載過高

在第一種情境中，您會看到追蹤記錄顯示UI執行緒和/或渲染執行緒呈現以下狀態：

![超載的GPU](/docs/assets/SystraceBadUI.png)

注意跨越影格邊界的`DrawFrame`長時間執行。這表示GPU正在等待排空前一影格的指令緩衝區。

緩解方法包括：

- 對複雜靜態內容的動畫/變形（如`Navigator`滑動/透明度動畫）使用`renderToHardwareTextureAndroid`
- 確認**未**啟用預設關閉的`needsOffscreenAlphaCompositing`，該功能在多數情況下會大幅增加GPU每影格負載

### 在UI執行緒創建新視圖

第二種情境的追蹤記錄會呈現：

![創建視圖](/docs/assets/SystraceBadCreateUI.png)

注意JS執行緒先進行運算，接著原生模組執行緒處理任務，最後UI執行緒執行昂貴的遍歷操作

除非能延遲至互動結束後創建新UI，或簡化UI結構，否則無快速解決方案。React Native團隊正在開發基礎架構級解決方案，將允許在非主執行緒創建與配置新UI，保持互動流暢性

### 定位原生CPU熱點

若問題疑似源自原生端，可使用[CPU熱點分析器](https://developer.android.com/studio/profile/record-java-kotlin-methods)獲取詳細資訊。開啟Android Studio分析器面板選擇「尋找CPU熱點（Java/Kotlin方法記錄）」

:::info[選擇Java/Kotlin記錄]

務必選擇「尋找CPU熱點**(Java/Kotlin記錄)**」而非「尋找CPU熱點（呼叫堆疊取樣）」。兩者圖示相似但功能不同
:::

執行互動後點擊「停止記錄」。記錄過程資源消耗大，請保持互動簡短。結果追蹤可在Android Studio檢視，或匯出至[Firefox Profiler](https://profiler.firefox.com/)等線上工具

與系統追蹤不同，CPU熱點分析速度較慢，無法提供精確測量值。但能顯示原生方法呼叫情況，以及各影格時間消耗比例分佈