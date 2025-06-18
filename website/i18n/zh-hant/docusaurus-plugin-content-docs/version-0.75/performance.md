---
id: performance
title: Performance Overview
---

相較於基於WebView的工具，使用React Native的一個重要原因是為了實現每秒60幀的流暢度，並為應用程式提供原生外觀與操作體驗。在可行情況下，我們希望React Native能自動處理優化工作，讓開發者能專注於應用邏輯而無需擔心效能問題。但仍有某些領域尚未達到此理想狀態，或是React Native（如同直接編寫原生代碼）無法自動判斷最佳優化方案的情況，此時便需要手動介入調整。我們預設會提供如絲般順滑的UI效能表現，但偶爾仍可能出現無法達標的狀況。

本指南旨在傳授基礎知識，協助您[診斷效能問題](profiling.md)，並探討[常見問題根源與對應解決方案](performance.md#common-sources-of-performance-problems)。

## 關於影格必須瞭解的事

老一輩將電影稱為「活動影像」有其道理：影片中流暢的動態效果其實是透過快速切換靜態畫面產生的視覺幻覺。我們稱這些靜態畫面為「影格」。每秒顯示的影格數（FPS）直接影響影片（或使用者介面）的流暢度與真實感。iOS裝置以每秒60幀的速率刷新，這意味著您與UI系統僅有約16.67毫秒來完成生成該影格所需的所有運算工作。若無法在此時間內完成運算，就會發生「掉幀」現象，導致UI出現卡頓。

需要特別說明的是，當您在應用中開啟[開發者選單](debugging.md#accessing-the-dev-menu)並啟用「顯示效能監測器」時，會注意到系統實際顯示兩種不同的幀率。

![](/docs/assets/PerfUtil.png)

### JS幀率（JavaScript執行緒）

多數React Native應用的業務邏輯都運行在JavaScript執行緒上。這裡是React應用核心所在，負責API呼叫、觸控事件處理等工作。對原生視圖的更新會批次處理，並在事件循環每次迭代結束時（理想情況下於影格截止期限前）發送至原生端。若JavaScript執行緒在某影格期間無響應，該影格即被視為掉幀。例如，當您在複雜應用的根元件呼叫`this.setState`，觸發計算密集型子樹重新渲染時，可能耗費200毫秒導致連續12幀丟失，此時所有由JavaScript控制的動畫都會出現凍結現象。任何超過100毫秒的延遲都會被使用者明顯感知。

此現象常見於`Navigator`轉場過程：當推送新路由時，JavaScript執行緒需渲染場景所需的所有元件，才能向原生端發送正確指令來建立對應視圖。此過程常會耗費數幀時間，造成[卡頓](https://jankfree.org/)，因為轉場動畫由JavaScript執行緒控制。有時元件在`componentDidMount`中執行額外工作，可能導致轉場過程中出現二次卡頓。

另一個典型範例是觸控回應：若JavaScript執行緒持續進行跨越多幀的運算，您可能會注意到`TouchableOpacity`等元件對觸控的反應延遲。這是因為JavaScript執行緒繁忙無法及時處理從主執行緒傳送的原始觸控事件，導致`TouchableOpacity`無法即時調降原生視圖的透明度。

### UI幀率（主執行緒）

許多開發者注意到`NavigatorIOS`的預設效能表現優於`Navigator`，其關鍵原因在於前者的轉場動畫完全在主執行緒執行，因此不會受JavaScript執行緒掉幀影響。

同樣地，當 JavaScript 執行緒被鎖定時，您仍能順暢地在 `ScrollView` 中上下滾動，因為 `ScrollView` 運作於主執行緒上。滾動事件會被派發到 JS 執行緒，但滾動行為的發生並不依賴這些事件的接收。

## 常見效能問題來源

### 處於開發模式 (`dev=true`)

在開發模式下，JavaScript 執行緒的效能會大幅下降。這是不可避免的：為了提供良好的警告和錯誤訊息，運行時需要執行更多工作。請務必在[發行版本](running-on-device.md#building-your-app-for-production)中測試效能。

### 使用 `console.log` 語句

在運行打包後的應用時，這些語句會嚴重阻塞 JavaScript 執行緒。這包括來自除錯庫（如 [redux-logger](https://github.com/evgenyrodionov/redux-logger)）的呼叫，請確保在打包前移除它們。您也可以使用這個 [babel 插件](https://babeljs.io/docs/plugins/transform-remove-console/)來移除所有 `console.*` 呼叫。首先需透過 `npm i babel-plugin-transform-remove-console --save-dev` 安裝，然後編輯專案目錄下的 `.babelrc` 檔案如下：

```json
{
  "env": {
    "production": {
      "plugins": ["transform-remove-console"]
    }
  }
}
```

這將自動移除專案發行（生產）版本中的所有 `console.*` 呼叫。

即使專案中沒有直接使用 `console.*` 呼叫，仍建議使用此插件。第三方函式庫也可能呼叫它們。

### `ListView` 初始渲染過慢或大型列表滾動效能不佳

請改用新的 [`FlatList`](flatlist.md) 或 [`SectionList`](sectionlist.md) 元件。除了簡化 API 外，新列表元件還具有顯著的效能提升，主要特點是無論行數多少，記憶體使用量幾乎保持恆定。

如果您的 [`FlatList`](flatlist.md) 渲染速度慢，請確保實作了 [`getItemLayout`](flatlist.md#getitemlayout)，透過跳過渲染項目的測量來優化渲染速度。

### 重新渲染幾乎未變化的視圖時 JS FPS 驟降

若使用 ListView，必須提供 `rowHasChanged` 函式，透過快速判斷行是否需要重新渲染來減少大量工作。若使用不可變資料結構，僅需進行參考相等性檢查即可。

同樣地，您可以實作 `shouldComponentUpdate` 並明確指定元件需要重新渲染的條件。如果是純元件（渲染函式的返回值完全依賴於 props 和 state），可以利用 PureComponent 自動處理。再次強調，不可變資料結構有助於保持高效——若需對大型物件列表進行深度比較，直接重新渲染整個元件可能更快，且程式碼量更少。

### 因 JavaScript 執行緒同時處理大量工作導致 JS FPS 下降

「導航器轉場卡頓」是最常見的表現形式，但其他情況也可能發生。使用 InteractionManager 是個好方法，但若延遲動畫期間的工作對使用者體驗影響過大，則可考慮改用 LayoutAnimation。

目前 Animated API 會在 JavaScript 執行緒上按需計算每個關鍵影格（除非您[設定 `useNativeDriver: true`](/blog/2017/02/14/using-native-driver-for-animated#how-do-i-use-this-in-my-app)），而 LayoutAnimation 則利用 Core Animation，不受 JS 執行緒和主執行緒掉幀影響。

我曾將此技術用於模態視窗的動畫（從頂部滑下並淡入半透明遮罩），同時初始化多個網路請求、渲染模態內容並更新開啟模態的原始視圖。詳見動畫指南以獲取更多 LayoutAnimation 的使用資訊。

注意事項：

- LayoutAnimation 僅適用於一次性觸發的動畫（「靜態」動畫）——如果需要中斷動畫，則必須使用 `Animated`。

### 在螢幕上移動視圖（滾動、平移、旋轉）導致 UI 執行緒 FPS 下降

這種情況尤其發生於當文字帶有透明背景並疊加在圖片上方時，或任何其他需要在每一幀重新繪製視圖的 alpha 合成場景。您會發現啟用 `shouldRasterizeIOS` 或 `renderToHardwareTextureAndroid` 能顯著改善此問題。

請注意不要過度使用這些屬性，否則記憶體用量可能會暴增。使用這些屬性時，請務必分析效能和記憶體用量。如果不再需要移動視圖，請關閉此屬性。

### 調整圖片大小動畫導致 UI 執行緒 FPS 下降

在 iOS 上，每次調整 Image 元件的寬度或高度時，都會從原始圖片重新裁剪和縮放。這對大型圖片尤其耗費資源。建議改用 `transform: [{scale}]` 樣式屬性來實現尺寸動畫。例如，點擊圖片後將其放大至全螢幕的場景就適合使用此方法。

### 我的 TouchableX 視圖反應遲鈍

有時，如果我們在調整觸控回應元件的透明度或高亮效果的同一幀中執行動作，這些視覺變化會等到 `onPress` 函數執行完畢後才會顯現。若 `onPress` 中的 `setState` 觸發大量運算導致掉幀，就可能發生這種情況。解決方法是將 `onPress` 處理程序內的所有動作包裹在 `requestAnimationFrame` 中：

```tsx
handleOnPress() {
  requestAnimationFrame(() => {
    this.doExpensiveAction();
  });
}
```

### 導航器轉場動畫卡頓

如前所述，`Navigator` 的動畫由 JavaScript 執行緒控制。以「從右側推入」的場景轉場為例：每一幀中，新場景從右側（假設初始 x 偏移為 320）向左移動，最終停在 x 偏移為 0 的位置。在此過程中，JavaScript 執行緒需要將新的 x 偏移值傳送給主執行緒。如果 JavaScript 執行緒被阻塞，動畫就會因無法更新而出現卡頓。

解決方案之一是將基於 JavaScript 的動畫卸載到主執行緒處理。沿用上述例子，我們可以在轉場開始時預先計算新場景的所有 x 偏移值，並將其發送給主執行緒以優化方式執行。這樣釋放 JavaScript 執行緒後，即使渲染場景時掉幾幀也無妨——流暢的轉場動畫會讓您根本注意不到這些微小卡頓。

新推出的 [React Navigation](navigation.md) 庫正是為解決此問題而設計。它採用原生元件和 [`Animated`](animated.md) 庫來實現由原生執行緒運行的 60 FPS 流暢動畫。