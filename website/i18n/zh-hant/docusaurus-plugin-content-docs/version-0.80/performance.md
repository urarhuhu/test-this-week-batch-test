---
id: performance
title: Performance Overview
---

選擇使用 React Native 而非基於 WebView 的工具，其關鍵原因在於能實現至少 60 FPS 的流暢度，並為應用程式提供原生般的視覺與操作體驗。在理想情況下，React Native 會自動處理效能優化，讓開發者能專注於應用邏輯而無需擔心效能問題。然而，目前仍有部分領域尚未達到此理想狀態，或是像直接編寫原生程式碼一樣，React Native 無法自動為您選擇最佳優化方案。此時便需要手動介入調整。我們雖預設追求絲滑般的 UI 效能，但某些情況下仍可能無法完美實現。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源與對應解決方案](performance.md#common-sources-of-performance-problems)。

## 關於影格的基本概念

老一輩將電影稱為「活動影像」有其道理：影片中流暢的動態效果，實質上是透過快速連續播放靜態畫面所營造的視覺幻覺。我們稱這些靜態畫面為「影格」。每秒顯示的影格數（FPS）直接影響影片（或使用者介面）的流暢度與真實感。iOS 和 Android 裝置的畫面更新率至少為 60 FPS，這意味著您與 UI 系統最多只有 16.67 毫秒的時間來完成生成單一影格所需的所有運算。若無法在此時間內完成工作，就會發生「掉幀」，導致介面出現卡頓。

需特別說明的是，當您在應用中開啟[開發者選單](debugging.md#opening-the-dev-menu)並切換「顯示效能監控」時，會注意到系統實際顯示兩種不同的幀率。

![效能監控截圖](/docs/assets/PerfUtil.png)

### JS 影格率（JavaScript 執行緒）

多數 React Native 應用的商業邏輯都執行於 JavaScript 執行緒上，這裡是 React 應用運作的核心：處理 API 呼叫、觸控事件等。對原生視圖的更新會批次處理，並在事件循環的每次迭代結束時（理想情況下於影格截止期限前）傳送至原生端。若 JavaScript 執行緒在某影格期間無回應，該影格即被視為掉幀。例如，當您在複雜應用的根元件設定新狀態，導致需要重新渲染計算密集的子元件樹時，可能耗費 200 毫秒並造成 12 幀丟失。此時所有由 JavaScript 控制的動畫都會出現凍結現象。若掉幀過多，使用者將明顯感受到卡頓。

以觸控回應為例：若 JavaScript 執行緒持續進行跨越多幀的繁重運算，您可能會注意到 `TouchableOpacity` 等元件對觸控的反應延遲。這是因為 JavaScript 執行緒忙碌而無法及時處理從主執行緒傳送的原始觸控事件，導致 `TouchableOpacity` 無法即時調控原生視圖的透明度變化。

### UI 影格率（主執行緒）

您可能已觀察到，原生堆疊導覽器（如 React Navigation 提供的 [@react-navigation/native-stack](https://reactnavigation.org/docs/native-stack-navigator)）的預設效能優於基於 JavaScript 的導覽器。這是因為其轉場動畫直接在原生主 UI 執行緒執行，故不會受 JavaScript 執行緒掉幀影響。

同理，當 JavaScript 執行緒阻塞時，您仍能流暢滾動 `ScrollView`——因其運作於主執行緒。雖然滾動事件會被派發至 JS 執行緒，但滾動行為本身無需等待 JS 執行緒回應。

## 常見效能問題根源

### 處於開發模式（`dev=true`）

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[正式版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

當運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。你也可以使用這個 [Babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需要安裝它：`npm i babel-plugin-transform-remove-console --save-dev`，然後編輯專案目錄下的 `.babelrc` 文件如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案正式（生產）版本中的所有 `console.*` 呼叫。

即使專案中沒有使用 `console.*` 呼叫，也建議使用此插件。第三方函式庫也可能會呼叫它們。

### `FlatList` 渲染過慢或大型列表的滾動效能不佳

如果你的 [`FlatList`](flatlist.md) 渲染速度慢，請確保你已實作 [`getItemLayout`](flatlist.md#getitemlayout)，通過跳過渲染項目的測量來優化渲染速度。

還有其他針對效能優化的第三方列表函式庫，包括 [FlashList](https://github.com/shopify/flash-list) 和 [Legend List](https://github.com/legendapp/legend-list)。

### 由於同時在 JavaScript 執行緒上執行大量工作導致 JS 執行幀率下降

「導航器過渡動畫緩慢」是最常見的表現，但也有其他情況可能發生。使用 [`InteractionManager`](interactionmanager.md) 可能是一個好方法，但如果延遲動畫期間的工作對使用者體驗影響太大，則可以考慮使用 [`LayoutAnimation`](layoutanimation.md)。

[`Animated API`](animated.md) 目前會在 JavaScript 執行緒上按需計算每個關鍵幀，除非你[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，而 [`LayoutAnimation`](layoutanimation.md) 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀的影響。

一個使用場景是在顯示模態視窗時（從頂部滑下並淡入半透明遮罩），同時初始化並可能接收多個網路請求的回應、渲染模態視窗的內容，以及更新模態視窗開啟的來源視圖。詳見[動畫指南](animations.md)以獲取更多關於如何使用 `LayoutAnimation` 的資訊。

**注意事項：**

- `LayoutAnimation` 僅適用於「一勞永逸」的動畫（「靜態」動畫）——如果需要中斷動畫，則需要使用 [`Animated`](animated.md)。

### 移動畫面中的視圖（滾動、平移、旋轉）導致 UI 執行緒幀率下降

這種情況在 Android 上尤其明顯，當你有文字帶有透明背景並位於圖片上方，或任何其他需要在每一幀重新繪製視圖的 alpha 合成情況時。你會發現啟用 `renderToHardwareTextureAndroid` 可以顯著改善此問題。在 iOS 上，`shouldRasterizeIOS` 預設已啟用。

注意不要過度使用此功能，否則記憶體使用量可能會暴增。在使用這些屬性時，請分析你的效能和記憶體使用情況。如果你不再移動視圖，請關閉此屬性。

### 動畫調整圖片大小導致 UI 執行緒幀率下降

在 iOS 上，每次調整 [`Image` 元件](image.md) 的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對於大型圖片來說可能非常耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫效果。一個典型的使用場景是點擊圖片後將其放大至全螢幕。

### 我的 TouchableX 元件觸控反應遲鈍

有時，如果我們在調整元件透明度或觸控高亮效果的同一幀中執行操作，可能要到 `onPress` 函數返回後才會看到視覺反饋。這種情況通常發生在 `onPress` 觸發了導致重度重新渲染的狀態更新，進而導致數幀畫面丟失。解決方法是將 `onPress` 處理程序內的操作包裹在 `requestAnimationFrame` 中：

```tsx
function handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```