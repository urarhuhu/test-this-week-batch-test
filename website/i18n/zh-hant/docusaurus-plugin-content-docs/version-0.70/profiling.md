---
id: profiling
title: Profiling
---

Use the built-in profiler to get detailed information about work done in the JavaScript thread and main thread side-by-side. Access it by selecting Perf Monitor from the Debug menu.

For iOS, Instruments is an invaluable tool, and on Android you should learn to use [`systrace`](profiling.md#profiling-android-ui-performance-with-systrace).

But first, [**make sure that Development Mode is OFF!**](performance.md#running-in-development-mode-devtrue) You should see `__DEV__ === false, development-level warning are OFF, performance optimizations are ON` in your application logs.

Another way to profile JavaScript is to use the Chrome profiler while debugging. This won't give you accurate results as the code is running in Chrome but will give you a general idea of where bottlenecks might be. Run the profiler under Chrome's `Performance` tab. A flame graph will appear under `User Timing`. To view more details in tabular format, click at the `Bottom Up` tab below and then select `DedicatedWorker Thread` at the top left menu.

## Profiling Android UI Performance with `systrace`

Android supports 10k+ different phones and is generalized to support software rendering: the framework architecture and need to generalize across many hardware targets unfortunately means you get less for free relative to iOS. But sometimes, there are things you can improve -- and many times it's not native code's fault at all!

The first step for debugging this jank is to answer the fundamental question of where your time is being spent during each 16ms frame. For that, we'll be using a standard Android profiling tool called `systrace`.

`systrace` is a standard Android marker-based profiling tool (and is installed when you install the Android platform-tools package). Profiled code blocks are surrounded by start/end markers which are then visualized in a colorful chart format. Both the Android SDK and React Native framework provide standard markers that you can visualize.

### 1. Collecting a trace

First, connect a device that exhibits the stuttering you want to investigate to your computer via USB and get it to the point right before the navigation/animation you want to profile. Run `systrace` as follows:

```shell
$ <path_to_android_sdk>/platform-tools/systrace/systrace.py --time=10 -o trace.html sched gfx view -a <your_package_name>
```

A quick breakdown of this command:

- `time` is the length of time the trace will be collected in seconds
- `sched`, `gfx`, and `view` are the android SDK tags (collections of markers) we care about: `sched` gives you information about what's running on each core of your phone, `gfx` gives you graphics info such as frame boundaries, and `view` gives you information about measure, layout, and draw passes
- `-a <your_package_name>` enables app-specific markers, specifically the ones built into the React Native framework. `your_package_name` can be found in the `AndroidManifest.xml` of your app and looks like `com.example.app`

Once the trace starts collecting, perform the animation or interaction you care about. At the end of the trace, systrace will give you a link to the trace which you can open in your browser.

### 2. Reading the trace

After opening the trace in your browser (preferably Chrome), you should see something like this:

![Example](/docs/assets/SystraceExample.png)

:::note[Hint]
Use the WASD keys to strafe and zoom.
:::

If your trace .html file isn't opening correctly, check your browser console for the following:

![ObjectObserveError](/docs/assets/ObjectObserveError.png)

Since `Object.observe` was deprecated in recent browsers, you may have to open the file from the Google Chrome Tracing tool. You can do so by:

- Opening tab in chrome `chrome://tracing`
- Selecting load
- Selecting the html file generated from the previous command.

:::info[啟用垂直同步高亮顯示]
勾選畫面右上角的此選項以突顯16毫秒的幀邊界：

![啟用垂直同步高亮顯示](/docs/assets/SystraceHighlightVSync.png)

您應會看到如上截圖中的斑馬條紋。若未顯示，請嘗試在其他裝置上分析：三星設備已知有顯示垂直同步的問題，而Nexus系列通常較可靠。
:::

### 3. 找到您的進程

滾動直到看見您的套件名稱（部分）。此例中，我分析的`com.facebook.adsmanager`會顯示為`book.adsmanager`，因核心執行緒名稱長度限制所致。

左側會顯示一組執行緒，對應右側的時間軸行。我們關注以下幾個執行緒：UI執行緒（顯示您的套件名稱或「UI Thread」）、`mqt_js`和`mqt_native_modules`。若在Android 5+上運行，還需關注Render Thread。

- **UI執行緒。** 此處進行標準Android測量/佈局/繪製。右側執行緒名稱會是您的套件名稱（本例為book.adsmanager）或「UI Thread」。此執行緒上的事件應類似以下內容，與`Choreographer`、`traversals`和`DispatchUI`相關：

  ![UI執行緒範例](/docs/assets/SystraceUIThreadExample.png)

- **JS執行緒。** 此處執行JavaScript。執行緒名稱會是`mqt_js`或`<...>`，取決於裝置核心的配合度。若無名稱，可透過尋找`JSCall`、`Bridge.executeJSCall`等來識別：

  ![JS執行緒範例](/docs/assets/SystraceJSThreadExample.png)

- **原生模組執行緒。** 此處執行原生模組呼叫（如`UIManager`）。執行緒名稱會是`mqt_native_modules`或`<...>`。若為後者，可透過尋找`NativeCall`、`callJavaModuleMethod`和`onBatchComplete`來識別：

  ![原生模組執行緒範例](/docs/assets/SystraceNativeModulesThreadExample.png)

- **額外：Render執行緒。** 若使用Android L（5.0）及以上版本，應用中還會有一個Render執行緒。此執行緒產生用於繪製UI的實際OpenGL指令。執行緒名稱會是`RenderThread`或`<...>`。若為後者，可透過尋找`DrawFrame`和`queueBuffer`來識別：

  ![Render執行緒範例](/docs/assets/SystraceRenderThreadExample.png)

## 找出問題根源

流暢的動畫應如下所示：

![流暢動畫](/docs/assets/SystraceWellBehaved.png)

每個顏色變化代表一幀——請記住，要顯示一幀，所有UI工作需在16毫秒內完成。注意沒有執行緒的工作接近幀邊界。如此渲染的應用能以60 FPS運行。

若發現卡頓，可能會看到如下情況：

![JS導致的卡頓動畫](/docs/assets/SystraceBadJS.png)

注意JS執行緒幾乎全程執行，且跨越幀邊界！此應用未以60 FPS渲染。此情況下，**問題出在JS**。

您也可能看到以下情況：

![原生UI導致的卡頓動畫](/docs/assets/SystraceBadUI.png)

此情況下，UI和Render執行緒的工作跨越了幀邊界。每幀嘗試渲染的UI需過多工作。此時，**問題出於渲染的原生視圖**。

至此，您將獲得有助於後續步驟的關鍵資訊。

## 解決 JavaScript 問題

若確認問題出在 JavaScript 端，請檢查執行的具體 JS 程式碼尋找線索。在上述情境中，我們看到每幀多次呼叫 `RCTEventEmitter`。以下是從追蹤記錄中放大檢視 JS 執行緒的畫面：

![過多的 JS 執行](/docs/assets/SystraceBadJS2.png)

這顯然不正常。為何呼叫如此頻繁？這些是不同的事件嗎？答案通常取決於產品程式碼。多數情況下，您需要檢查 [shouldComponentUpdate](https://reactjs.org/docs/react-component.html#shouldcomponentupdate) 的實作。

## 解決原生 UI 問題

若問題出在原生 UI 端，通常有兩種情境：

1. 每幀繪製的 UI 在 GPU 上負載過重，或
2. 在動畫/互動過程中建構新 UI（例如滾動時載入新內容）。

### GPU 負載過重

第一種情境下，追蹤記錄會顯示 UI 執行緒和/或渲染執行緒呈現如下狀態：

![GPU 超載](/docs/assets/SystraceBadUI.png)

注意跨越幀邊界的長時間 `DrawFrame` 操作，這表示等待 GPU 清空前一幀命令緩衝區的時間。

緩解方法包括：

- 對複雜靜態內容（如 `Navigator` 滑動/透明度動畫）使用 `renderToHardwareTextureAndroid`
- 確認未啟用預設關閉的 `needsOffscreenAlphaCompositing`，此選項在多數情況下會大幅增加 GPU 每幀負載。

若仍無法解決，可透過 [Tracer for OpenGL ES](http://www.androiddocs.com/tools/help/gltracer.html) 深入分析 GPU 實際運作情況。

### 在 UI 執行緒建立新視圖

第二種情境的追蹤記錄通常如下：

![建立視圖](/docs/assets/SystraceBadCreateUI.png)

可見 JS 執行緒先運算一段時間，接著原生模組執行緒處理任務，最後 UI 執行緒進行高成本的遍歷操作。

除非能延遲建立新 UI 至互動結束後，或簡化 UI 結構，否則沒有快速解決方案。React Native 團隊正在開發基礎架構層級的解決方案，將允許在非主執行緒建立與配置新 UI，確保互動過程保持流暢。