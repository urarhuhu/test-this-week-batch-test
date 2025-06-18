---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，選擇React Native的一個重要原因是它能實現至少每秒60幀的流暢度，並為應用提供原生外觀與操作體驗。在理想情況下，我們希望React Native能自動處理優化工作，讓開發者只需專注於應用邏輯而無需擔心效能問題。但目前仍有部分領域尚未達到此目標，還有一些情況（如同直接編寫原生代碼時）React Native無法自動判斷最佳優化方案。此時便需要手動介入調整。我們致力於預設提供絲滑流暢的UI效能，但某些特殊場景可能仍無法完全實現。

本指南旨在傳授基礎知識，協助您[排查效能問題](profiling.md)，並探討[常見問題根源與建議解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果其實是通過快速連續播放靜態畫面產生的錯覺。我們稱這些靜態畫面為「幀」。每秒顯示的幀數直接影響影片（或使用者介面）的流暢度與真實感。iOS設備的畫面更新率至少為60FPS，這意味著您與UI系統最多只有16.67毫秒來完成生成單一靜態畫面（幀）所需的所有工作。若無法在此時間內完成幀生成，就會發生「掉幀」，導致介面出現卡頓。

補充說明：在應用中開啟[開發者選單](debugging.md#opening-the-dev-menu)並切換「顯示效能監測器」時，您會注意到系統實際顯示兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

多數React Native應用的業務邏輯都運行在JavaScript執行緒上。這裡是React應用核心所在，包含API調用、觸控事件處理等操作。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下在幀截止時間前）發送至原生端。若JavaScript執行緒在某幀期間無響應，就會被視為掉幀。例如，當您在複雜應用的根組件調用`this.setState`導致需要重新渲染計算密集的子組件樹時，可能耗費200毫秒並造成12幀丟失。此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常見於`Navigator`轉場時：當推送新路由時，JavaScript執行緒需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常會耗費數幀時間，由於轉場動畫由JavaScript執行緒控制，會導致[界面卡頓](https://jankfree.org/)。有時組件在`componentDidMount`中執行額外工作，可能造成轉場過程中再次出現停頓。

另一個典型例子是觸控響應：若JavaScript執行緒持續進行跨越多幀的繁重工作，您可能會注意到`TouchableOpacity`等組件的響應延遲。這是因為JavaScript執行緒處於忙碌狀態，無法及時處理從主執行緒傳來的原始觸控事件，導致`TouchableOpacity`無法根據觸控事件通知原生視圖調整透明度。

### UI幀率（主執行緒）

許多開發者注意到`NavigatorIOS`的預設效能優於`Navigator`，其關鍵原因在於前者的轉場動畫完全在主執行緒執行，因此不會受JavaScript執行緒掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被分派到 JS 執行緒，但滾動行為的發生並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供完善的警告和錯誤訊息，運行時需要執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在執行打包後的應用時，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，請務必在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。需先透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

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

即使專案中沒有直接使用 `console.*` 呼叫，仍建議使用此插件，因為第三方函式庫可能呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確認是否實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，可實作 `shouldComponentUpdate` 並明確指定觸發元件重新渲染的條件。如果是純元件（渲染函式的返回值完全依賴 props 和 state），可透過 PureComponent 自動處理。再次強調，不可變資料結構能加速此過程——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼更簡潔。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是可行的解決方案，但若延遲動畫期間的工作會嚴重影響使用者體驗，則可考慮改用 LayoutAnimation。

目前 Animated API 預設會在 JavaScript 執行緒上逐幀計算關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 直接利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於模態視窗的動畫（從頂部滑下並淡入半透明遮罩），同時初始化多個網路請求、渲染模態內容並更新開啟模態的原始視圖。詳見動畫指南以了解 LayoutAnimation 的使用方式。

注意事項：

- LayoutAnimation 僅適用於「一次性觸發即忘」的動畫（「靜態」動畫）——若需支援中斷操作，則必須改用 `Animated`。

### 移動畫面上的視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

當文字帶有透明背景並疊加於圖片上方，或任何需要每幀重新繪製視圖的 alpha 合成情境時，此問題尤其明顯。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可大幅改善效能。

需謹慎避免過度使用，否則記憶體用量可能暴增。使用這些屬性時，請務必分析效能與記憶體消耗。若視圖不再需要移動，應立即關閉此屬性。

### 調整圖片尺寸動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬高時，系統會從原始圖片重新裁剪與縮放。對於大圖而言效能消耗極大，建議改用 `transform: [{scale}]` 樣式屬性實現尺寸動畫。典型應用場景如點擊圖片後放大至全螢幕。

### TouchableX 元件響應遲鈍

若在調整觸控回應元件透明度或高亮效果的同一幀中執行操作，可能需等待 `onPress` 函數完成後才會看到視覺反饋。當 `onPress` 觸發的 `setState` 導致大量運算而掉幀時，此現象更明顯。解決方案是將 `onPress` 處理程序內的操作包裹於 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「右側推入」場景轉場為例：每一幀需將新場景從 x 軸偏移 320（初始位於螢幕外）移動至 x 軸 0 的位置。若 JavaScript 執行緒阻塞，則無法發送新偏移值至主執行緒，導致動畫幀缺失。

解決方案之一是將基於 JavaScript 的動畫卸載至主執行緒。以前述場景為例，我們可在轉場開始時預先計算所有 x 軸偏移值序列，並交由主執行緒優化執行。如此釋放 JavaScript 執行緒後，即使渲染場景時偶爾掉幀，流暢的轉場動畫仍能維持良好體驗。

[React Navigation](navigation.md) 新庫的核心目標正是解決此問題。其採用原生元件與 [`Animated`](animated.md) 庫實現原生執行緒驅動的 60 FPS 動畫。