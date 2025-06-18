---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，選擇React Native的一個重要原因是為了實現每秒60幀的流暢度以及應用程式的原生外觀與操作體驗。在可能的情況下，我們希望React Native能自動處理好這些細節，讓開發者能專注於應用邏輯而非效能優化。然而在某些領域，我們尚未完全實現這個目標；還有一些情況（如同直接編寫原生代碼時）React Native無法自動為您做出最佳優化決策，此時就需要手動介入。我們盡力預設提供如絲般順滑的UI效能，但有時這並不可行。

本指南旨在傳授一些基礎知識，幫助您[排查效能問題](profiling.md)，同時討論[常見問題來源及其解決方案](performance.md#common-sources-of-performance-problems)。

## 關於幀的必備知識

老一輩人將電影稱為「活動影像」有其道理：影片中流暢的動態效果其實是通過以固定速度快速切換靜態畫面營造的錯覺。我們稱這些靜態畫面為「幀」。每秒顯示的幀數直接影響影片（或使用者介面）的流暢度與真實感。iOS設備以每秒60幀運作，這意味著您與UI系統只有約16.67毫秒來完成生成單一靜態畫面（幀）所需的所有工作。若無法在規定時間內完成幀生成，就會發生「掉幀」，導致UI出現卡頓。

需要特別說明的是，當您在應用中開啟開發者選單並啟用「顯示效能監測器」時，會注意到系統顯示兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

大多數React Native應用的業務邏輯都運行在JavaScript執行緒上。這裡是React應用運作的核心區域：API呼叫、觸控事件處理等都在此進行。對原生視圖的更新會被批量處理，並在事件循環的每次迭代結束時（理想情況下在幀截止時間前）發送到原生端。若JavaScript執行緒在某一幀期間無響應，就會被視為掉幀。例如，當您在複雜應用的根組件上調用`this.setState`，導致需要重新渲染計算密集的子組件樹時，這個過程可能耗時200毫秒，造成12幀丟失。此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

這種情況常見於`Navigator`轉場時：當推送新路由時，JavaScript執行緒需要渲染場景所需的所有組件，才能向原生端發送正確指令來創建底層視圖。這個過程通常會耗費數幀時間，由於轉場由JavaScript執行緒控制，會導致[卡頓](http://jankfree.org/)。有時組件在`componentDidMount`中執行額外工作，可能造成轉場過程中的二次卡頓。

另一個典型例子是觸控響應：若JavaScript執行緒持續多幀時間處理任務，您可能會注意到`TouchableOpacity`等組件的響應延遲。這是因為JavaScript執行緒繁忙，無法處理從主線程傳來的原始觸控事件，導致`TouchableOpacity`無法即時響應觸控並通知原生視圖調整透明度。

### UI幀率（主執行緒）

許多開發者注意到`NavigatorIOS`的預設效能優於`Navigator」，其關鍵原因在於轉場動畫完全在主執行緒執行，因此不會受JavaScript執行緒掉幀的影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍可以順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 是運行在主執行緒上的。滾動事件會被派發到 JS 執行緒，但滾動行為的發生並不依賴於這些事件的接收。

## 常見的效能問題來源

### 在開發模式下運行 (`dev=true`)

在開發模式下運行時，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息（例如驗證 propTypes 和其他各種斷言），運行時需要執行更多工作。請務必在[發佈版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用程式時，這些語句可能會導致 JavaScript 執行緒出現嚴重的瓶頸。這包括來自除錯函式庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，因此請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需要安裝它：`npm i babel-plugin-transform-remove-console --save-dev`，然後編輯專案目錄下的 `.babelrc` 文件如下：

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

即使專案中沒有使用 `console.*` 呼叫，也建議使用此插件。第三方函式庫也可能會呼叫它們。

### `ListView` 初始渲染過慢或大型列表的滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新的列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持不變。

如果您的 [`FlatList`](flatlist.md) 渲染速度較慢，請確保已實作 [`getItemLayout`](flatlist.md#getitemlayout)，通過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎不變的視圖時，JS FPS 驟降

如果您使用 ListView，必須提供一個 `rowHasChanged` 函式，該函式可以通過快速判斷行是否需要重新渲染來減少大量工作。如果使用不可變數據結構，這只需要進行引用相等性檢查。

同樣地，您可以實作 `shouldComponentUpdate` 並指定您希望元件重新渲染的具體條件。如果您編寫純元件（渲染函式的返回值完全依賴於 props 和 state），可以利用 PureComponent 來為您完成這項工作。再次強調，不可變數據結構有助於保持高效——如果必須對大型物件列表進行深度比較，重新渲染整個元件可能會更快，且無疑需要更少的程式碼。

### 由於在 JavaScript 執行緒上同時執行大量工作導致 JS 執行緒 FPS 下降

「導航器過渡緩慢」是這種情況最常見的表現，但也有其他時候會發生這種情況。使用 InteractionManager 可能是一個好方法，但如果延遲動畫期間的工作對使用者體驗的代價太高，則可以考慮使用 LayoutAnimation。

除非您[設置 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)，否則 Animated API 目前會在 JavaScript 執行緒上按需計算每個關鍵影格，而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒影格丟失的影響。

我曾使用這種方法的一個案例是：在初始化並可能接收多個網路請求的回應、渲染模態框的內容以及更新模態框打開的視圖時，動畫顯示模態框（從頂部滑下並淡入半透明覆蓋層）。有關如何使用 LayoutAnimation 的更多資訊，請參閱動畫指南。

注意事項：

- LayoutAnimation 僅適用於「一發不可收拾」的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生在文字帶有透明背景並疊加在圖片上，或任何需要在每一幀重新繪製視圖的 alpha 合成場景。啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 可以顯著改善此問題。

注意不要過度使用這些屬性，否則記憶體使用量可能會暴增。在使用這些屬性時，請務必分析效能和記憶體使用情況。如果不再移動視圖，請關閉此屬性。

### 動畫調整圖片大小導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對於大圖來說非常耗資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如，點擊圖片後將其放大至全螢幕時可以使用此方法。

### 我的 TouchableX 視圖反應遲鈍

有時，如果我們在調整觸控回應元件的透明度或高亮效果的同一幀中執行動作，這些效果可能要到 `onPress` 函數返回後才會顯示。如果 `onPress` 中的 `setState` 導致大量工作並丟失幾幀，就可能發生這種情況。解決方法是將 `onPress` 處理程序中的任何動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器過渡動畫緩慢

如前所述，`Navigator` 動畫由 JavaScript 執行緒控制。以「從右側推入」的場景過渡為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）移動到左側，最終停在 x 偏移為 0 的位置。在此過渡期間，JavaScript 執行緒需要將新的 x 偏移發送到主執行緒。如果 JavaScript 執行緒被鎖定，則無法完成此操作，導致該幀沒有更新，動畫就會卡頓。

解決方法之一是將基於 JavaScript 的動畫卸載到主執行緒。如果採用這種方法，我們可以在開始過渡時計算新場景的所有 x 偏移值，並將其發送到主執行緒以優化方式執行。這樣 JavaScript 執行緒就無需承擔此責任，即使在渲染場景時丟失幾幀也無關緊要——你可能根本不會注意到，因為漂亮的過渡效果會吸引你的注意力。

解決此問題是新版 [React Navigation](navigation.md) 庫的主要目標之一。React Navigation 中的視圖使用原生元件和 [`Animated`](animated.md) 庫，以在主執行緒上實現 60 FPS 的流暢動畫。