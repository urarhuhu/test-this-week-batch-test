---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個重要原因是為了達到每秒60幀的流暢度，並為應用程式提供原生的外觀與操作體驗。在可行情況下，我們希望React Native能自動處理優化工作，讓開發者能專注於應用功能而無需擔心效能問題。但現階段仍有部分領域尚未實現自動優化，也有些情況（如同直接編寫原生代碼時）React Native無法自動判斷最佳優化方式，此時就需要手動介入調整。我們致力於預設提供絲滑般的UI效能，但某些情況下可能仍無法完全達成。

本指南旨在傳授基礎知識，幫助您[排查效能問題](profiling.md)，並探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果其實是通過快速連續顯示靜態畫面產生的錯覺。我們稱這些靜態畫面為「幀」。每秒顯示的幀數（FPS）直接影響影片（或使用者介面）的流暢度與真實感。iOS設備以每秒60幀的速率刷新，這意味著您與UI系統只有約16.67毫秒的時間來完成生成單幀畫面所需的所有工作。若無法在此時間內完成幀渲染，就會發生「掉幀」，導致介面出現卡頓。

需要特別說明的是：當您在應用中開啟[開發者選單](debugging.md#accessing-the-dev-menu)並啟用「顯示效能監測器」時，會注意到系統實際上追蹤兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JavaScript幀率（JS線程）

多數React Native應用的業務邏輯都在JavaScript線程上執行——這裡運行著您的React組件、處理API調用、觸控事件等。對原生視圖的更新會被批量處理，並在事件循環的每次迭代結束時（理想情況下在幀截止時間前）發送到原生端。若JavaScript線程在單幀周期內無響應，就會被視為掉幀。例如：當您在複雜應用的根組件調用`this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗費200毫秒並導致丟失12幀，此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常見於`Navigator`轉場時：當推送新路由時，JavaScript線程需要渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程通常會耗費數幀時間，造成[卡頓現象](https://jankfree.org/)，因為轉場動畫由JavaScript線程控制。有時組件在`componentDidMount`中執行額外工作，可能導致轉場過程中出現二次卡頓。

另一個典型例子是觸控響應：若JavaScript線程持續多幀處於忙碌狀態，您可能會注意到`TouchableOpacity`等組件的觸控反饋出現延遲。這是因為JavaScript線程無法及時處理從主線程傳遞過來的原始觸控事件，導致`TouchableOpacity`無法立即響應觸控並通知原生視圖調整透明度。

### UI幀率（主線程）

許多開發者注意到`NavigatorIOS`的預設效能優於`Navigator」，其關鍵原因在於前者的轉場動畫完全在主線程執行，因此不會受JavaScript線程掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被派發到 JS 執行緒，但滾動行為的發生並不依賴於這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他斷言），運行時需要執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

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

即使專案中沒有直接使用 `console.*` 呼叫，仍建議使用此插件，因為第三方函式庫也可能呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具有顯著的效能優化，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖導致 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。如果是純元件（渲染函式的返回值完全依賴於 props 和 state），可以利用 PureComponent 自動處理。再次強調，不可變資料結構能保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼更簡潔。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場動畫卡頓」是最常見的表現，但其他情況也可能發生。使用 InteractionManager 是較好的解決方案，但若延遲動畫期間的工作會嚴重影響使用者體驗，則可考慮改用 LayoutAnimation。

目前 Animated API 預設會在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 直接利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾用此技術處理以下情境：在初始化多個網路請求、渲染模態框內容、更新模態框來源視圖的同時，播放模態框動畫（從頂部滑入並淡入半透明遮罩）。詳見動畫指南以了解如何使用 LayoutAnimation。

注意事項：

- LayoutAnimation 僅適用於「一次性觸發即忘」的動畫（「靜態」動畫）——若需中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

當文字具有透明背景並疊加在圖片上方時尤為明顯，或任何需要每幀重新繪製視圖的 alpha 合成情境。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可顯著改善此問題。

請謹慎使用以避免記憶體暴增。使用這些屬性時務必分析效能與記憶體用量。若視圖不再需要移動，請關閉此屬性。

### 調整圖片尺寸動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬高時，系統會從原始圖片重新裁剪和縮放。這對大圖尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片放大至全螢幕的情境。

### TouchableX 視圖響應遲鈍

若在調整觸控回應元件的不透明度或高亮效果的同一幀中執行操作，可能需等待 `onPress` 函數完成後才會看到視覺變化。若 `onPress` 觸發的 `setState` 導致大量運算而掉幀，可將操作包裹在 `requestAnimationFrame` 中解決：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導覽器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀需將新場景從 x 軸偏移 320（螢幕外）移動至 0。若 JavaScript 執行緒阻塞無法傳送新偏移值，動畫就會卡頓。

解決方案是將 JavaScript 動畫卸載到主執行緒執行。例如預先計算所有偏移值並交由主執行緒優化執行，此時 JavaScript 執行緒即使掉幀也不影響動畫流暢度。

[React Navigation](navigation.md) 庫正是為此設計，其使用原生元件與 [`Animated`](animated.md) 庫實現原生執行緒的 60 FPS 動畫。