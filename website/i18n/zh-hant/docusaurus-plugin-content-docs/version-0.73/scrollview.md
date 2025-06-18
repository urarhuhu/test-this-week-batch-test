---
id: scrollview
title: ScrollView
---

該元件封裝了平台原生的 ScrollView，同時整合了觸控鎖定「響應者」系統。

請注意，ScrollView 必須具有固定高度才能正常運作，因為它將無限高度的子元件封裝在有限容器中（透過滾動互動）。要限制 ScrollView 的高度，可以直接設定視圖高度（不建議），或確保所有父視圖都具有固定高度。忘記沿視圖堆疊向下傳遞 `{flex: 1}` 可能導致錯誤，此時元素檢查器能快速除錯。

目前尚不支援其他內含的響應者阻止此滾動視圖成為主響應者。

`<ScrollView>` 與 [`<FlatList>`](flatlist.md) 該如何選擇？

`ScrollView` 會一次性渲染所有 React 子元件，但這會帶來效能負擔。

假設您需要顯示一個非常長的項目列表，可能橫跨多個螢幕。一次性為所有內容建立 JS 元件和原生視圖（其中許多可能根本不會顯示），將導致渲染速度變慢和記憶體使用量增加。

這時 `FlatList` 就派上用場。`FlatList` 會惰性渲染項目（僅在即將顯示時才渲染），並移除滾動出螢幕外的項目以節省記憶體和處理時間。

若您需要渲染項目間的分隔線、多欄佈局、無限滾動載入，或任何其他開箱即用的功能，`FlatList` 也十分便利。

## 範例

```SnackPlayer name=ScrollView
import React from 'react';
import {
  StyleSheet,
  Text,
  SafeAreaView,
  ScrollView,
  StatusBar,
} from 'react-native';

const App = () => {
  return (
    <SafeAreaView style={styles.container}>
      <ScrollView style={styles.scrollView}>
        <Text style={styles.text}>
          Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do
          eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
          minim veniam, quis nostrud exercitation ullamco laboris nisi ut
          aliquip ex ea commodo consequat. Duis aute irure dolor in
          reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla
          pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
          culpa qui officia deserunt mollit anim id est laborum.
        </Text>
      </ScrollView>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: StatusBar.currentHeight,
  },
  scrollView: {
    backgroundColor: 'pink',
    marginHorizontal: 20,
  },
  text: {
    fontSize: 42,
  },
});

export default App;
```

---

# 參考文獻

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view#props)。

---

### `StickyHeaderComponent`

用於渲染黏性標頭的 React 元件，需與 `stickyHeaderIndices` 搭配使用。若您的黏性標頭使用自訂變形（例如希望列表具有可動畫顯示/隱藏的標頭），則可能需要設定此元件。若未提供元件，將使用預設的 [`ScrollViewStickyHeader`](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/Components/ScrollView/ScrollViewStickyHeader.js) 元件。

| Type               |
| ------------------ |
| component, element |

---

### `alwaysBounceHorizontal` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，滾動視圖在水平滾動到底部時仍會彈跳。

| Type | Default                                               |
| ---- | ----------------------------------------------------- |
| bool | `true` when `horizontal={true}`<hr/>`false` otherwise |

---

### `alwaysBounceVertical` <div class="label ios">iOS</div>

設為 true 時，即使內容小於滾動視圖本身，滾動視圖在垂直滾動到底部時仍會彈跳。

| Type | Default                                             |
| ---- | --------------------------------------------------- |
| bool | `false` when `vertical={true}`<hr/>`true` otherwise |

---

### `automaticallyAdjustContentInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整位於導覽列或標籤列/工具列後方的滾動視圖的內容插邊距。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `automaticallyAdjustKeyboardInsets` <div class="label ios">iOS</div>

控制當鍵盤大小變更時，ScrollView 是否應自動調整其 `contentInset` 和 `scrollViewInsets`。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `automaticallyAdjustsScrollIndicatorInsets` <div class="label ios">iOS</div>

控制 iOS 是否應自動調整滾動指示器的內邊距。詳見 Apple 的[屬性說明文件](https://developer.apple.com/documentation/uikit/uiscrollview/3198043-automaticallyadjustsscrollindica)。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `bounces` <div class="label ios">iOS</div>

當為 `true` 時，若內容超出滾動方向軸線的範圍，滾動視圖在到達內容邊緣時會彈跳。設為 `false` 時，即使 `alwaysBounce*` 屬性為 `true` 也會禁用所有彈跳效果。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `bouncesZoom` <div class="label ios">iOS</div>

當為 `true` 時，手勢操作可暫時突破縮放的最小/最大值限制，並在手勢結束時動畫回歸到限制值；否則縮放比例絕不會超出限制範圍。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `canCancelContentTouches` <div class="label ios">iOS</div>

設為 `false` 時，一旦開始追蹤觸控，即使移動觸控點也不會觸發拖曳操作。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `centerContent` <div class="label ios">iOS</div>

當為 `true` 時，若內容小於滾動視圖邊界會自動居中；若內容大於滾動視圖則此屬性無效。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `contentContainerStyle`

這些樣式將套用至包裹所有子視圖的滾動視圖內容容器。範例：

```
return (
  <ScrollView contentContainerStyle={styles.contentContainer}>
  </ScrollView>
);
...
const styles = StyleSheet.create({
  contentContainer: {
    paddingVertical: 20
  }
});
```

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `contentInset` <div class="label ios">iOS</div>

滾動視圖內容與其邊緣之間的內縮距離。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `contentInsetAdjustmentBehavior` <div class="label ios">iOS</div>

此屬性指定如何利用安全區域邊距來調整滾動視圖的內容區域。僅限 iOS 11 及以上版本。

| Type                                                           | Default   |
| -------------------------------------------------------------- | --------- |
| enum(`'automatic'`, `'scrollableAxes'`, `'never'`, `'always'`) | `'never'` |

---

### `contentOffset`

用於手動設定起始滾動偏移量。

| Type  | Default        |
| ----- | -------------- |
| Point | `{x: 0, y: 0}` |

---

### `decelerationRate`

此浮點數決定用戶抬起手指後滾動視圖的減速快慢。也可使用字串快捷參數 `"normal"` 和 `"fast"`，分別對應 iOS 底層設置的 `UIScrollViewDecelerationRateNormal` 和 `UIScrollViewDecelerationRateFast`。

- `'normal'`：iOS 為 0.998，Android 為 0.985
- `'fast'`：iOS 為 0.99，Android 為 0.9

| Type                               | Default    |
| ---------------------------------- | ---------- |
| enum(`'fast'`, `'normal'`), number | `'normal'` |

---

### `directionalLockEnabled` <div class="label ios">iOS</div>

設為 `true` 時，拖曳過程中 ScrollView 會嘗試鎖定僅允許垂直或水平方向的滾動。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableIntervalMomentum`

當設為 true 時，無論手勢速度多快，滾動視圖都會在釋放時的下一個索引位置停止（相對於釋放時的滾動位置）。這可用於實現分頁效果，尤其當頁面寬度小於水平 ScrollView 的寬度或頁面高度小於垂直 ScrollView 的高度時。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `disableScrollViewPanResponder`

當設為 true 時，ScrollView 上預設的 JS 手勢響應器會被停用，ScrollView 內部的觸控事件將完全由其子元件控制。此屬性特別適用於啟用了 `snapToInterval` 的情況，因為該模式不符合常規觸控行為模式。若未啟用 `snapToInterval`，請勿在常規 ScrollView 使用情境中啟用此屬性，否則可能導致滾動時出現非預期的觸控行為。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `endFillColor` <div class="label android">Android</div>

當 ScrollView 的內容區域小於其可視區域時，此屬性可指定填充剩餘空間的顏色，避免設置背景色造成不必要的過度繪製。這屬於進階優化手段，一般情境無需使用。

| Type            |
| --------------- |
| [color](colors) |

---

### `fadingEdgeLength` <div class="label android">Android</div>

為滾動內容邊緣添加漸隱效果。

若值大於 `0`，系統會根據當前滾動方向與位置顯示漸隱邊緣，提示用戶尚有更多內容可供瀏覽。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `horizontal`

當設為 `true` 時，滾動視圖的子元件會以水平排列方式佈局，而非預設的垂直排列。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `indicatorStyle` <div class="label ios">iOS</div>

滾動指示器的樣式。

- `'default'` 等同於 `black`
- `'black'` 黑色滾動指示器，適合淺色背景
- `'white'` 白色滾動指示器，適合深色背景

| Type                                    | Default     |
| --------------------------------------- | ----------- |
| enum(`'default'`, `'black'`, `'white'`) | `'default'` |

---

### `invertStickyHeaders`

控制黏性標頭應固定在 ScrollView 底部而非頂部，通常與反向滾動的 ScrollView 搭配使用。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `keyboardDismissMode`

決定拖曳行為是否觸發鍵盤收起。

- `'none'` 拖曳時不收起鍵盤
- `'on-drag'` 開始拖曳時立即收起鍵盤

**僅限 iOS**

- `'interactive'` 鍵盤會隨拖曳手勢互動式收起，向上拖曳可取消收起動作。Android 不支援此模式，行為將與 `'none'` 相同

| Type                                                                                                                                                            | Default  |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| enum(`'none'`, `'on-drag'`) <div className="label android">Android</div><hr />enum(`'none'`, `'on-drag'`, `'interactive'`) <div className="label ios">iOS</div> | `'none'` |

---

### `keyboardShouldPersistTaps`

決定點擊後鍵盤是否保持顯示狀態。

- `'never'` 當鍵盤開啟時，點擊聚焦的文字輸入框外部會關閉鍵盤。此時子元件不會接收到點擊事件。
- `'always'` 鍵盤不會自動關閉，且滾動視圖不會攔截點擊事件，但滾動視圖的子元件可以捕獲點擊。
- `'handled'` 當點擊事件被滾動視圖的子元件（或被上層元件捕獲）處理時，鍵盤不會自動關閉。
- `false` **_已棄用_**，請改用 `'never'`
- `true` **_已棄用_**，請改用 `'always'`

| Type                                                      | Default   |
| --------------------------------------------------------- | --------- |
| enum(`'always'`, `'never'`, `'handled'`, `false`, `true`) | `'never'` |

---

### `maintainVisibleContentPosition`

設定此屬性後，滾動視圖會調整滾動位置，使當前可見且位於或超過 `minIndexForVisible` 的第一個子元件保持位置不變。這適用於雙向載入內容的列表（例如聊天串），新訊息加入時可能導致滾動位置跳動。常見值為 0，但也可設為 1 以跳過載入指示器或其他不應固定位置的內容。

可選參數 `autoscrollToTopThreshold` 可在調整位置後，若使用者原本位於頂部閾值範圍內，則自動將內容滾動至頂部。這同樣適用於聊天類應用，希望新訊息滾動到位，但若使用者已向上滾動一段距離，則避免造成干擾性大幅滾動。

注意事項 1：啟用此功能時重新排序滾動視圖中的元件可能導致跳動和卡頓。此問題可修復，但目前無相關計劃。現階段請勿對使用此功能的滾動視圖或列表重新排序內容。

注意事項 2：此功能透過原生代碼中的 `contentOffset` 和 `frame.origin` 計算可見性。遮擋、變形等複雜因素不會被納入內容是否「可見」的判斷。

| Type                                                                     |
| ------------------------------------------------------------------------ |
| object: `{minIndexForVisible: number, autoscrollToTopThreshold: number}` |

---

### `maximumZoomScale` <div class="label ios">iOS</div>

允許的最大縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

### `minimumZoomScale` <div class="label ios">iOS</div>

允許的最小縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

### `nestedScrollEnabled` <div class="label android">Android</div>

為 Android API 等級 21+ 啟用巢狀滾動功能。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onContentSizeChange`

當 ScrollView 的可滾動內容視圖尺寸變化時呼叫。

處理函式會接收兩個參數：內容寬度與內容高度 `(contentWidth, contentHeight)`。

其實作方式是透過附加到 ScrollView 渲染的內容容器上的 onLayout 處理器完成。

| Type     |
| -------- |
| function |

---

### `onMomentumScrollBegin`

當動量滾動開始時呼叫（即 ScrollView 開始滑行時的滾動）。

| Type     |
| -------- |
| function |

---

### `onMomentumScrollEnd`

當動量滾動結束時呼叫（即 ScrollView 滑行至停止時的滾動）。

| Type     |
| -------- |
| function |

---

### `onScroll`

滾動期間每幀最多觸發一次。可透過 `scrollEventThrottle` 屬性控制事件觸發頻率。事件格式如下（所有值均為數字）：

```js
{
  nativeEvent: {
    contentInset: {bottom, left, right, top},
    contentOffset: {x, y},
    contentSize: {height, width},
    layoutMeasurement: {height, width},
    zoomScale
  }
}
```

| Type     |
| -------- |
| function |

---

### `onScrollBeginDrag`

當用戶開始拖動滾動視圖時調用。

| Type     |
| -------- |
| function |

---

### `onScrollEndDrag`

當用戶停止拖動滾動視圖且視圖停止或開始滑動時調用。

| Type     |
| -------- |
| function |

---

### `onScrollToTop` <div class="label ios">iOS</div>

當滾動視圖因點擊狀態列而滾動至頂部時觸發。

| Type     |
| -------- |
| function |

---

### `overScrollMode` <div class="label android">Android</div>

用於覆寫預設的過度滾動模式值。

可能的值：

- `'auto'` - 僅當內容足夠大時才允許用戶過度滾動此視圖。
- `'always'` - 始終允許用戶過度滾動此視圖。
- `'never'` - 禁止用戶過度滾動此視圖。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'always'`, `'never'`) | `'auto'` |

---

### `pagingEnabled`

當設為 true 時，滾動視圖會在滾動時停在視圖大小的整數倍位置。這可用於實現水平分頁效果。

> 注意：Android 不支援垂直分頁。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `persistentScrollbar` <div class="label android">Android</div>

使滾動條在不使用時保持可見（不變透明）。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `pinchGestureEnabled` <div class="label ios">iOS</div>

當設為 true 時，允許使用捏合手勢來縮放 ScrollView。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `refreshControl`

用於實現下拉刷新功能的 RefreshControl 組件。僅適用於垂直 ScrollView（必須將 `horizontal` 屬性設為 `false`）。

參見 [RefreshControl](refreshcontrol)。

| Type    |
| ------- |
| element |

---

### `removeClippedSubviews`

實驗性功能：當設為 true 時，會將不可見的子視圖（其 `overflow` 值為 `hidden`）從原生父視圖中移除，可提升長列表的滾動性能。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `scrollEnabled`

當設為 false 時，禁止通過觸摸交互滾動視圖。

注意：仍可通過調用 `scrollTo` 方法滾動視圖。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `scrollEventThrottle`

控制滾動事件觸發頻率（單位：毫秒）。較低的值可提高追蹤滾動位置的精度，但可能因橋接數據量過大而影響性能。由於 JS 運行循環與屏幕刷新率同步，1-16 之間的設置值實際效果無差異。若不需要精確追蹤位置，可設較高值以減少橋接數據量。預設值為 `0`，表示每次滾動視圖時僅發送一次事件。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `scrollIndicatorInsets` <div class="label ios">iOS</div>

滾動指示器與滾動視圖邊緣的內嵌距離。通常應設為與 `contentInset` 相同的值。

| Type                                                                 | Default                                  |
| -------------------------------------------------------------------- | ---------------------------------------- |
| object: `{top: number, left: number, bottom: number, right: number}` | `{top: 0, left: 0, bottom: 0, right: 0}` |

---

### `scrollPerfTag` <div class="label android">Android</div>

用於記錄此滾動視圖性能的標籤。會強制啟用動量事件（參見 sendMomentumEvents）。預設情況下此功能無效，需實作自訂原生 FpsListener 才能發揮作用。

| Type   |
| ------ |
| string |

---

### `scrollToOverflowEnabled` <div class="label ios">iOS</div>

設為 `true` 時，可通過程式控制將滾動視圖滾動超出其內容尺寸。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `scrollsToTop` <div class="label ios">iOS</div>

設為 `true` 時，點擊狀態列會使滾動視圖滾動至頂部。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsHorizontalScrollIndicator`

設為 `true` 時，顯示水平滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `showsVerticalScrollIndicator`

設為 `true` 時，顯示垂直滾動指示器。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToAlignment`

當設定 `snapToInterval` 時，`snapToAlignment` 會定義對齊滾動視圖的吸附關係。

可能的值：

- `'start'` 將吸附對齊左側（水平）或頂部（垂直）。
- `'center'` 將吸附對齊中心。
- `'end'` 將吸附對齊右側（水平）或底部（垂直）。

| Type                                 | Default   |
| ------------------------------------ | --------- |
| enum(`'start'`, `'center'`, `'end'`) | `'start'` |

---

### `snapToEnd`

需與 `snapToOffsets` 搭配使用。預設情況下，列表末端會計為吸附偏移點。設為 `false` 可禁用此行為，允許列表在末端與最後一個 `snapToOffsets` 偏移點間自由滾動。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `snapToInterval`

設定後，滾動視圖會在 `snapToInterval` 值的倍數處停止。可用於分頁瀏覽尺寸小於滾動視圖的子項目。通常與 `snapToAlignment` 和 `decelerationRate="fast"` 組合使用。會覆蓋較不靈活的 `pagingEnabled` 屬性。

| Type   |
| ------ |
| number |

---

### `snapToOffsets`

設定後，滾動視圖會在指定偏移點停止。可用於分頁瀏覽尺寸各異且小於滾動視圖的子項目。通常與 `decelerationRate="fast"` 組合使用。會覆蓋較不靈活的 `pagingEnabled` 和 `snapToInterval` 屬性。

| Type            |
| --------------- |
| array of number |

---

### `snapToStart`

需與 `snapToOffsets` 搭配使用。預設情況下，列表起始處會計為吸附偏移點。設為 `false` 可禁用此行為，允許列表在起始處與第一個 `snapToOffsets` 偏移點間自由滾動。

| Type | Default |
| ---- | ------- |
| bool | `true`  |

---

### `stickyHeaderHiddenOnScroll`

設為 `true` 時，當向下滾動列表時，黏性標頭會隱藏；向上滾動時則會固定在列表頂部。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `stickyHeaderIndices`

一個子元素索引陣列，決定哪些子元素在滾動時會固定在螢幕頂部。例如，傳入 `stickyHeaderIndices={[0]}` 會使第一個子元素固定在滾動視圖頂部。也可使用 [x,y,z] 格式讓多個項目在頂部時保持黏性。此屬性不支援與 `horizontal={true}` 同時使用。

| Type            |
| --------------- |
| array of number |

---

### `zoomScale` <div class="label ios">iOS</div>

滾動視圖內容的當前縮放比例。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

## 方法

### `flashScrollIndicators()`

```tsx
flashScrollIndicators();
```

短暫顯示滾動指示器。

---

### `scrollTo()`

```tsx
scrollTo(
  options?: {x?: number, y?: number, animated?: boolean} | number,
  deprecatedX?: number,
  deprecatedAnimated?: boolean,
);
```

滾動到指定的 x、y 偏移位置，可選擇立即執行或平滑動畫效果。

**範例：**

`scrollTo({x: 0, y: 0, animated: true})`

> 注意：此函數簽名的特殊形式是由於歷史原因，它同時接受獨立參數作為選項物件的替代方案。由於參數順序（y 在 x 之前）的模糊性，此用法已被棄用，不應繼續使用。

---

### `scrollToEnd()`

```tsx
scrollToEnd(options?: {animated?: boolean});
```

如果是垂直 ScrollView 則滾動到底部；如果是水平 ScrollView 則滾動到右側。

使用 `scrollToEnd({animated: true})` 可啟用平滑動畫滾動，`scrollToEnd({animated: false})` 則立即滾動。若未傳入選項，`animated` 預設為 `true`。