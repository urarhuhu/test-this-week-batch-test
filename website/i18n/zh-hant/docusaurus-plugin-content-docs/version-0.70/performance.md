---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個重要理由在於能實現每秒60幀的流暢度，並賦予應用程式原生般的視覺體驗。React Native會盡可能自動處理優化，讓您專注於應用開發而非效能調校，但在某些領域我們尚未完善，或是像直接編寫原生代碼一樣無法自動判斷最佳優化方式，此時仍需手動介入。我們致力於預設提供絲滑般的UI效能，但有時仍難以完全實現。

本指南旨在傳授基礎知識，協助您[排查效能問題](profiling.md)，並探討[常見問題根源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果，實際上是透過穩定速度快速切換靜態畫面所營造的幻覺。我們稱這些靜態畫面為「幀」。每秒顯示的幀數直接影響影片（或使用者介面）的流暢度與真實感。iOS裝置以每秒60幀運作，這意味著您與UI系統僅有約16.67毫秒來完成生成該幀所需的所有工作。若無法在此時間內完成，就會發生「掉幀」，導致介面出現卡頓。

進一步觀察時，請開啟應用程式的開發者選單並啟用「顯示效能監測器」，您會注意到系統顯示兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

多數React Native應用的業務邏輯運行於JavaScript執行緒，這裡處理React組件生命週期、API呼叫、觸控事件等。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下於幀截止期限前）發送至原生端。若JavaScript執行緒在某幀期間無響應，即視為掉幀。例如，當您在複雜應用的根組件呼叫`this.setState`，導致需要重新渲染計算密集的子組件樹時，可能耗時200毫秒並造成12幀丟失，此時JavaScript控制的動畫會明顯凍結。任何超過100毫秒的延遲都會被使用者感知。

這種情況常見於`Navigator`轉場期間：當推送新路由時，JavaScript執行緒需渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。此過程常耗費數幀時間，導致[卡頓現象](http://jankfree.org/)，因為轉場動畫由JavaScript執行緒控制。有時組件在`componentDidMount`中執行額外工作，可能造成轉場過程中的二次卡頓。

另一個典型例子是觸控回應：若JavaScript執行緒持續處理跨越多幀的任務，您可能會注意到`TouchableOpacity`等組件的回應延遲。這是因為JavaScript執行緒忙碌而無法及時處理從主執行緒傳來的原始觸控事件，導致`TouchableOpacity`無法即時調整原生視圖的透明度。

### UI幀率（主執行緒）

許多人注意到`NavigatorIOS`的預設效能優於`Navigator`，關鍵原因在於其轉場動畫完全在主執行緒執行，因此不會受JavaScript執行緒掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發至 JS 執行緒，但滾動動作的執行無需等待這些事件被接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下執行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他斷言），運行時需執行更多工作。請務必在[發佈版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在執行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案發佈（生產）版本中的所有 `console.*` 呼叫。

即使專案中沒有使用 `console.*` 呼叫，仍建議使用此插件。第三方函式庫也可能會呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新的列表元件還具有顯著的效能提升，主要特點是無論有多少行數，記憶體使用量幾乎保持不變。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保已實作 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖時，JS FPS 驟降

若使用 ListView，您必須提供 `rowHasChanged` 函式，該函式能快速判斷某行是否需要重新渲染，從而減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需重新渲染的條件。若撰寫純元件（即渲染函式的返回值完全依賴於 props 和 state），可透過 PureComponent 自動處理此邏輯。不可變資料結構再次發揮作用——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導覽器轉場動畫卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是可行的解決方案，但若延遲動畫期間的工作對使用者體驗影響過大，則可考慮改用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格（除非[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於以下場景：在初始化多個網路請求、渲染模態框內容、更新模態框開啟來源視圖的同時，以動畫形式呈現模態框（從頂部滑下並淡入半透明遮罩）。詳見動畫指南以瞭解如何使用 LayoutAnimation。

注意事項：

- LayoutAnimation 僅適用於一次性觸發的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生於當文字帶有透明背景並疊加在圖片上方時，或任何其他需要在每一幀重新繪製視圖的 alpha 合成場景。您會發現啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 能顯著改善此問題。

請謹慎使用這些屬性，否則記憶體用量可能暴增。使用這些屬性時，請務必分析效能和記憶體用量。如果不再需要移動視圖，請關閉此屬性。

### 動畫調整圖片大小導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對大型圖片尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如點擊圖片放大至全螢幕的場景。

### TouchableX 視圖反應遲鈍

有時，若我們在調整回應觸控的元件透明度或高亮效果的同一幀中執行動作，這些視覺變化會等到 `onPress` 函數執行完畢後才會顯現。如果 `onPress` 觸發的 `setState` 導致大量運算並丟失數幀，就可能發生此情況。解決方法是將 `onPress` 處理程序內的所有動作包裹在 `requestAnimationFrame` 中：

```jsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫卡頓

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）向左移動，最終停駐在 x 偏移為 0 的位置。在此過程中，JavaScript 執行緒需將每一幀的新 x 偏移值傳送給主執行緒。若 JavaScript 執行緒被阻塞，動畫就會因無法更新而出現卡頓。

解決方案之一是將基於 JavaScript 的動畫卸載到主執行緒。以前述場景為例，我們可以在轉場開始時預先計算所有 x 偏移值並提交給主執行緒優化執行。如此釋放 JavaScript 執行緒後，即便渲染場景時丟失幾幀也無妨——流暢的轉場動畫會讓您忽略這些微小卡頓。

[React Navigation](navigation.md) 新庫的主要目標正是解決此問題。它採用原生元件和 [`Animated`](animated.md) 庫實現 60 FPS 的原生執行緒動畫。