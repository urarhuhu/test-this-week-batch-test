---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個強力理由在於能實現至少60幀/秒的流暢度，並為應用提供原生外觀與操作體驗。我們盡可能讓React Native自動處理效能優化，使開發者能專注於應用邏輯。但某些情況下，框架尚未達到全自動優化階段，或類似直接編寫原生代碼時，無法自動判斷最佳優化方案。此時便需手動介入。我們預設追求絲滑般的UI表現，但偶爾仍可能出現未達標的狀況。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源與對應解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀率的必備知識

老一輩將電影稱為「活動影像」有其道理：流暢動態效果實為靜態畫面高速切換產生的視覺幻象。每一幅靜態畫面稱為「幀」。每秒顯示幀數（FPS）直接影響動畫（或UI）的流暢度與真實感。iOS設備以60FPS為基準，這意味著您與UI系統僅有16.67毫秒來完成生成單幀所需的所有運算。若無法在此時限內完成，就會「掉幀」導致界面卡頓。

需特別說明的是：開啟應用中的[開發者選單](debugging.md#opening-the-dev-menu)並切換「顯示效能監測器」時，您會觀察到兩個獨立幀率指標。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript線程）

多數React Native應用的業務邏輯運行於JavaScript線程。這裡承載著React組件樹、API調用、觸控事件處理等核心邏輯。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下於幀截止期限前）發送至原生端。若JavaScript線程在單幀周期內無響應，即判定為掉幀。例如：當您在複雜應用的根組件調用`this.setState`，觸發高計算量的子樹重渲染時，可能耗時200ms導致丟失12幀——此期間所有由JavaScript控制的動畫都將凍結。任何超過100ms的延遲都會被用戶明顯感知。

此現象常見於`Navigator`轉場：推送新路由時，JavaScript線程需渲染場景所需所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常耗費數幀時間，造成[卡頓](https://jankfree.org/)（因轉場動畫受控於JavaScript線程）。有時組件在`componentDidMount`執行額外工作，可能導致轉場過程中二次卡頓。

另一典型案例是觸控響應：若JavaScript線程持續進行跨越多幀的運算，您可能注意到`TouchableOpacity`等組件的響應延遲。這是因為JavaScript線程繁忙無法處理從主線程傳遞的原始觸控事件，致使`TouchableOpacity`無法即時調節原生視圖的透明度。

### UI幀率（主線程）

許多開發者注意到`NavigatorIOS`的預設性能優於`Navigator`，關鍵原因在於其轉場動畫完全在主線程執行，因此不受JavaScript線程掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以流暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運行在主執行緒上。滾動事件會被分派到 JS 執行緒，但滾動行為的發生並不依賴於這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[正式版建置](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在打包應用中，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

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

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具有顯著的效能優化，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。如果是純元件（渲染函式的返回值完全依賴於 props 和 state），可以利用 PureComponent 自動處理。不可變資料結構再次派上用場——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼更簡潔。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場動畫卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是解決方案之一，但若延遲動畫期間的工作會嚴重影響使用者體驗，則可考慮改用 LayoutAnimation。

目前 Animated API 預設在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 基於 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾用此技術實作模態動畫（從頂部滑下並淡入半透明遮罩），同時初始化多個網路請求、渲染模態內容並更新開啟模態的原始視圖。詳見動畫指南以了解 LayoutAnimation 的使用方法。

注意事項：

- LayoutAnimation 僅適用於「一發不可收拾」的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生在文字帶有透明背景並疊加在圖片上，或任何需要每幀重新繪製視圖的 alpha 合成情境。你會發現啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 能顯著改善此問題。

注意不要過度使用這些屬性，否則記憶體用量可能會暴增。使用這些屬性時，請務必分析效能和記憶體用量。如果不再移動視圖，請關閉此屬性。

### 調整圖片大小動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對大型圖片尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片放大至全螢幕的情境。

### 我的 TouchableX 視圖反應遲鈍

有時，如果我們在調整觸控回應元件的透明度或高亮效果的同一幀中執行動作，這些視覺效果會等到 `onPress` 函數返回後才會顯示。若 `onPress` 觸發的 `setState` 導致大量運算並掉幀，就可能發生此狀況。解決方法是將 `onPress` 處理程序內的動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）向左移動，最終停在 x 偏移為 0 的位置。在此過程中，JavaScript 執行緒需每幀傳送新偏移值給主線程。若 JavaScript 執行緒卡住，動畫就會掉幀。

解決方案之一是將 JavaScript 動畫卸載到主線程執行。以上述案例為例，我們可以在轉場開始時預先計算所有 x 偏移值，並交由主線程優化執行。釋放 JavaScript 執行緒後，即使渲染場景時掉幾幀也無妨——漂亮的轉場動畫會讓你忽略這些微小卡頓。

[React Navigation](navigation.md) 新庫的主要目標正是解決此問題。它採用原生元件和 [`Animated`](animated.md) 庫實現至少 60 FPS 的原生線程動畫。