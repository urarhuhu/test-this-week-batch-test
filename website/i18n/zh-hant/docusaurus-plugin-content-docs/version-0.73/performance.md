---
id: performance
title: Performance Overview
---

相較於基於 WebView 的工具，選擇使用 React Native 的一個重要原因是為了實現每秒 60 幀的流暢度，並為應用程式提供原生外觀與操作體驗。在可行情況下，我們希望 React Native 能自動處理優化工作，讓開發者能專注於應用功能而無需擔心效能問題。然而，目前仍有部分領域尚未達到此理想狀態，還有一些情況是 React Native（如同直接編寫原生代碼）無法自動為您決定最佳優化方案。此時便需要手動介入調整。我們致力於預設提供絲滑般的 UI 效能表現，但某些情況下可能仍無法完全實現。

本指南旨在傳授基礎知識，協助您[排查效能問題](profiling.md)，同時探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果其實是透過快速切換靜態畫面產生的視覺錯覺。我們將每張靜態畫面稱為「幀」。每秒顯示的幀數（FPS）直接影響影片（或使用者介面）的流暢度與真實感。iOS 裝置以每秒 60 幀的速率刷新，這意味著您與 UI 系統只有約 16.67 毫秒的時間來完成生成單一幀所需的所有工作。若無法在此時間內完成幀生成，就會發生「掉幀」現象，導致介面出現卡頓。

補充說明：當您在應用中開啟[開發者選單](debugging.md#accessing-the-dev-menu)並啟用「顯示效能監測器」時，會注意到系統實際上追蹤兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JavaScript 線程幀率

多數 React Native 應用的業務邏輯都運行於 JavaScript 線程。這裡是 React 組件運作的核心場域：API 呼叫、觸控事件處理等操作都在此執行。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下於幀截止期限前）發送至原生端。若 JavaScript 線程在單幀周期內無響應，就會被判定為掉幀。例如當您在複雜應用的根組件調用 `this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗費 200 毫秒而導致 12 幀丟失——此期間所有由 JavaScript 控制的動畫都將呈現凍結狀態。任何超過 100 毫秒的延遲都會被使用者明顯感知。

這種情況常見於 `Navigator` 轉場動畫：當推送新路由時，JavaScript 線程需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常會耗費數幀時間，由於轉場由 JavaScript 線程控制，就會導致[卡頓現象](https://jankfree.org/)。有時組件在 `componentDidMount` 中執行額外工作，可能造成轉場過程中出現二次停頓。

另一個典型範例是觸控響應：若 JavaScript 線程持續進行跨越多幀的繁重任務，您可能會觀察到 `TouchableOpacity` 等組件出現響應延遲。這是因為 JavaScript 線程處於忙碌狀態，無法及時處理從主線程傳送的原始觸控事件，導致 `TouchableOpacity` 無法根據觸控事件通知原生視圖調整透明度。

### 主線程幀率（UI 線程）

許多開發者注意到 `NavigatorIOS` 的預設效能表現優於 `Navigator`，其關鍵原因在於前者的轉場動畫完全在主線程執行，因此不會受 JavaScript 線程掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發至 JS 執行緒，但滾動行為的發生並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供完善的警告和錯誤訊息（例如驗證 propTypes 和其他斷言），運行時需執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在執行打包後的應用程式時，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [Babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。需先透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

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

即使專案中沒有使用 `console.*` 呼叫，仍建議使用此插件。第三方函式庫也可能呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還有顯著的效能優化，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout) 以跳過渲染項目的測量步驟，從而優化渲染速度。

### 重新渲染幾乎未變化的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件重新渲染的條件。如果是純元件（渲染函式的返回值完全依賴於 props 和 state），可透過 PureComponent 自動處理。再次強調，不可變資料結構能保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼更簡潔。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場過慢」是最常見的表現，但其他情況也可能發生。使用 InteractionManager 是解決方案之一，但若延遲動畫期間的工作會嚴重影響使用者體驗，則可考慮改用 LayoutAnimation。

目前 Animated API 預設在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於以下場景：在初始化多個網路請求、渲染模態框內容、更新模態框開啟來源視圖的同時，以動畫形式呈現模態框（從頂部滑下並淡入半透明遮罩）。詳見動畫指南以瞭解如何使用 LayoutAnimation。

注意事項：

- LayoutAnimation 僅適用於「一次性動畫」（靜態動畫）——若需中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

當文字具有透明背景並疊加在圖片上，或任何需要每幀重新繪製視圖的 alpha 合成情境時，此問題尤其明顯。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 屬性可顯著改善效能。

請謹慎使用這些屬性，避免記憶體用量暴增。使用時務必分析效能與記憶體消耗。若視圖不再需要移動，請關閉此屬性。

### 調整圖片尺寸動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬高時，系統會從原始圖片重新裁剪和縮放。這對大圖尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片放大至全螢幕的情境。

### TouchableX 元件響應遲鈍

若在調整觸控回應元件的不透明度或高亮效果的同一幀中執行動作，可能需等待 `onPress` 函數完成後才會看到視覺回饋。當 `onPress` 觸發的 `setState` 導致大量運算而掉幀時，此現象更明顯。解決方案是將 `onPress` 處理程序內的動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀中，新場景需從螢幕外（假設 x 偏移 320）移動至 x 偏移 0 的定位點。若 JavaScript 執行緒被鎖定，將無法發送新偏移值給主執行緒，導致動畫掉幀。

解決方案之一是將 JavaScript 動畫卸載至主執行緒。以上述案例為例，我們可在轉場開始時預先計算所有 x 偏移值序列，交由主執行緒以優化方式執行。如此釋放 JavaScript 執行緒後，即使渲染場景時掉幀，流暢的轉場動畫仍能維持良好使用者體驗。

[React Navigation](navigation.md) 新函式庫的主要目標正是解決此問題。其採用原生元件與 [`Animated`](animated.md) 函式庫，透過原生執行緒實現 60 FPS 的流暢動畫。