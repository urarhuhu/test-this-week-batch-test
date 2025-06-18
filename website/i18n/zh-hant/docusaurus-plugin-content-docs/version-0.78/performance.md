---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，選擇React Native的一個重要理由在於它能實現至少每秒60幀的流暢度，並為應用提供原生般的視覺與操作體驗。在理想情況下，React Native會自動處理效能優化，讓開發者能專注於應用邏輯而無需擔心效能問題。然而，目前仍有部分領域尚未達到此理想狀態，也有些情況（如同直接編寫原生代碼時）React Native無法自動判斷最佳優化方式。此時便需要手動介入調整。我們致力於預設提供絲滑般的UI效能，但某些情況下可能仍無法完全實現。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果，實際上是透過快速連續顯示靜態畫面所營造的視覺幻象。我們稱這些靜態畫面為「幀」。每秒顯示的幀數直接影響影片（或使用者介面）的流暢度與真實感。iOS裝置的畫面更新率至少為60幀/秒，這意味著您與UI系統最多只有16.67毫秒的時間來完成生成該幀所需的所有運算工作。若無法在此時間內完成運算，就會發生「掉幀」，導致介面出現卡頓。

需要特別說明的是，當您在應用中開啟[開發者選單](debugging.md#opening-the-dev-menu)並啟用「顯示效能監測器」時，會注意到畫面上顯示兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

多數React Native應用的業務邏輯都在JavaScript執行緒上運行——這裡是React應用核心所在，包含API呼叫、觸控事件處理等操作。對原生視圖的更新會被批量處理，並在事件循環的每個迭代週期結束時（若一切順利）趕在幀截止期限前發送至原生端。若JavaScript執行緒在某幀期間無響應，該幀即被視為掉幀。例如，當您在複雜應用的根組件上呼叫`this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗費200毫秒並造成12幀丟失。此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常見於`Navigator`轉場過程：當推送新路由時，JavaScript執行緒需渲染場景所需的所有組件，才能向原生端發送正確指令來建立對應視圖。此過程常會耗費數幀時間，造成[卡頓現象](https://jankfree.org/)，因為轉場動畫由JavaScript執行緒控制。有時組件在`componentDidMount`中執行額外工作，可能導致轉場過程中出現二次卡頓。

另一個典型例子是觸控響應：若JavaScript執行緒持續多幀處於忙碌狀態，您可能會注意到`TouchableOpacity`等組件的觸控反饋出現延遲。這是因為JavaScript執行緒無法及時處理從主執行緒傳來的原始觸控事件，導致`TouchableOpacity`無法即時調整原生視圖的透明度。

### UI幀率（主執行緒）

許多開發者注意到`NavigatorIOS`的預設效能優於`Navigator`，其關鍵原因在於前者的轉場動畫完全在主執行緒上執行，因此不會受JavaScript執行緒掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被分派到 JS 執行緒，但滾動行為的發生並不依賴於這些事件的接收。

## 常見的效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需要透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案正式版（生產環境）中的所有 `console.*` 呼叫。

It is recommended to use the plugin even if no `console.*` calls are made in your project. A third party library could also call them.

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新的列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時 JS FPS 驟降

如果使用 ListView，您必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。如果使用不可變資料結構，這只需要進行參考相等性檢查。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。如果編寫純元件（渲染函式的返回值完全依賴於 props 和 state），可以利用 PureComponent 來實現這一點。再次強調，不可變資料結構有助於保持高效——如果需要對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 由於同時在 JavaScript 執行緒上執行大量工作導致 JS 執行緒 FPS 下降

「導航器轉場過慢」是最常見的表現形式，但也可能在其他情況下發生。使用 InteractionManager 是一個不錯的解決方案，但如果延遲動畫期間的工作對使用者體驗影響太大，則可以考慮使用 LayoutAnimation。

目前，Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格，除非您[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀的影響。

我曾使用此方法的一個案例是：在初始化並可能接收多個網路請求回應、渲染模態框內容以及更新開啟模態框的視圖時，同時以動畫形式顯示模態框（從頂部滑下並淡入半透明遮罩）。有關如何使用 LayoutAnimation 的更多資訊，請參閱動畫指南。

注意事項：

- LayoutAnimation only works for fire-and-forget animations ("static" animations) -- if it must be interruptible, you will need to use `Animated`.

### Moving a view on the screen (scrolling, translating, rotating) drops UI thread FPS

This is especially true when you have text with a transparent background positioned on top of an image, or any other situation where alpha compositing would be required to re-draw the view on each frame. You will find that enabling `shouldRasterizeIOS` or `renderToHardwareTextureAndroid` can help with this significantly.

Be careful not to overuse this or your memory usage could go through the roof. Profile your performance and memory usage when using these props. If you don't plan to move a view anymore, turn this property off.

### Animating the size of an image drops UI thread FPS

On iOS, each time you adjust the width or height of an Image component it is re-cropped and scaled from the original image. This can be very expensive, especially for large images. Instead, use the `transform: [{scale}]` style property to animate the size. An example of when you might do this is when you tap an image and zoom it in to full screen.

### My TouchableX view isn't very responsive

Sometimes, if we do an action in the same frame that we are adjusting the opacity or highlight of a component that is responding to a touch, we won't see that effect until after the `onPress` function has returned. If `onPress` does a `setState` that results in a lot of work and a few frames dropped, this may occur. A solution to this is to wrap any action inside of your `onPress` handler in `requestAnimationFrame`:

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### Slow navigator transitions

As mentioned above, `Navigator` animations are controlled by the JavaScript thread. Imagine the "push from right" scene transition: each frame, the new scene is moved from the right to left, starting offscreen (let's say at an x-offset of 320) and ultimately settling when the scene sits at an x-offset of 0. Each frame during this transition, the JavaScript thread needs to send a new x-offset to the main thread. If the JavaScript thread is locked up, it cannot do this and so no update occurs on that frame and the animation stutters.

One solution to this is to allow for JavaScript-based animations to be offloaded to the main thread. If we were to do the same thing as in the above example with this approach, we might calculate a list of all x-offsets for the new scene when we are starting the transition and send them to the main thread to execute in an optimized way. Now that the JavaScript thread is freed of this responsibility, it's not a big deal if it drops a few frames while rendering the scene -- you probably won't even notice because you will be too distracted by the pretty transition.

Solving this is one of the main goals behind the new [React Navigation](navigation.md) library. The views in React Navigation use native components and the [`Animated`](animated.md) library to deliver at least 60 FPS animations that are run on the native thread.