---
id: virtualizedlist
title: VirtualizedList
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

這是更便利的 [`<FlatList>`](flatlist.md) 和 [`<SectionList>`](sectionlist.md) 元件的基礎實現，這些元件也有更完善的文件說明。通常只有在需要比 [`FlatList`](flatlist.md) 提供更高靈活性的情況下才應使用此元件，例如需搭配不可變數據而非普通陣列時。

虛擬化技術透過維護一個有限的可見區域渲染窗口，並將窗口外的項目替換為適當大小的空白空間，大幅改善了大型列表的記憶體消耗與效能表現。該窗口會根據滾動行為動態調整，遠離可見區域的項目會以低優先級（在執行中的互動之後）漸進式渲染，而靠近可見區域的項目則以高優先級渲染，以最小化出現空白空間的可能性。

## 範例

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=VirtualizedListExample&ext=js
import React from 'react';
import {View, VirtualizedList, StyleSheet, Text, StatusBar} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const getItem = (_data, index) => ({
  id: Math.random().toString(12).substring(0),
  title: `Item ${index + 1}`,
});

const getItemCount = _data => 50;

const Item = ({title}) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container} edges={['top']}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight,
  },
  item: {
    backgroundColor: '#f9c2ff',
    height: 150,
    justifyContent: 'center',
    marginVertical: 8,
    marginHorizontal: 16,
    padding: 20,
  },
  title: {
    fontSize: 32,
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=VirtualizedListExample&ext=tsx
import React from 'react';
import {View, VirtualizedList, StyleSheet, Text, StatusBar} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

type ItemData = {
  id: string;
  title: string;
};

const getItem = (_data: unknown, index: number): ItemData => ({
  id: Math.random().toString(12).substring(0),
  title: `Item ${index + 1}`,
});

const getItemCount = (_data: unknown) => 50;

type ItemProps = {
  title: string;
};

const Item = ({title}: ItemProps) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container} edges={['top']}>
      <VirtualizedList
        initialNumToRender={4}
        renderItem={({item}) => <Item title={item.title} />}
        keyExtractor={item => item.id}
        getItemCount={getItemCount}
        getItem={getItem}
      />
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight,
  },
  item: {
    backgroundColor: '#f9c2ff',
    height: 150,
    justifyContent: 'center',
    marginVertical: 8,
    marginHorizontal: 16,
    padding: 20,
  },
  title: {
    fontSize: 32,
  },
});

export default App;
```

</TabItem>
</Tabs>

---

需注意事項：

- 當內容滾出渲染窗口時，內部狀態不會保留。請確保所有數據都儲存在項目數據或外部狀態庫（如 Flux、Redux 或 Relay）中。
- 此為 `PureComponent`，意味著若 `props` 進行淺層比較後相等則不會觸發重新渲染。請確認 `renderItem` 函數所依賴的所有內容都通過 prop 傳遞（例如 `extraData`），且這些 prop 在更新後不應保持 `===` 相等，否則變更時 UI 可能不會更新。這包括 `data` prop 和父元件狀態。
- 為限制記憶體使用並實現平滑滾動，內容會以異步方式在屏幕外渲染。這可能導致滾動速度超過填充速率而暫時看到空白內容。此為可根據應用需求調整的權衡取捨，我們也在持續優化此機制。
- 預設情況下，列表會尋找每個項目的 `key` prop 作為 React 的 key。您也可透過自定義 `keyExtractor` prop 來提供 key。

---

# 參考文檔

## 屬性

### [ScrollView 屬性](scrollview.md#props)

繼承 [ScrollView 屬性](scrollview.md#props)。

---

### `data`

傳遞給 `getItem` 和 `getItemCount` 以獲取項目的不透明數據類型。

| Type |
| ---- |
| any  |

---

### <div class="label required basic">必填</div> **`getItem`**

```tsx
(data: any, index: number) => any;
```

用於從任何類型的數據塊中提取項目的通用訪問器。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">必填</div> **`getItemCount`**

```tsx
(data: any) => number;
```

決定數據塊中包含多少個項目。

| Type     |
| -------- |
| function |

---

### <div class="label required basic">必填</div> **`renderItem`**

```tsx
(info: any) => ?React.Element<any>
```

從 `data` 中取得項目並將其渲染到列表中

| Type     |
| -------- |
| function |

---

### `CellRendererComponent`

CellRendererComponent 允許自定義由 `renderItem`/`ListItemComponent` 渲染的單元格在放入底層 ScrollView 時的包裝方式。此元件必須接受事件處理程序以通知 VirtualizedList 單元格內的變更。

| Type                                     |
| ---------------------------------------- |
| `React.ComponentType<CellRendererProps>` |

---

### `ItemSeparatorComponent`

在每個項目之間渲染，但不會出現在頂部或底部。預設情況下會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供的 `separators.highlight`/`unhighlight` 會更新 `highlighted` 屬性，但您也可以透過 `separators.updateProps` 添加自訂屬性。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListItemComponent`

每個資料項目都會使用此元素進行渲染。可以是 React 元件類別或渲染函式。

| Type                |
| ------------------- |
| component, function |

---

### `ListFooterComponent`

在所有項目的底部渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponentStyle`

用於 `ListFooterComponent` 內部 View 的樣式設定。

| Type          | Required |
| ------------- | -------- |
| ViewStyleProp | No       |

---

### `ListHeaderComponent`

在所有項目的頂部渲染。可以是 React 元件（例如 `SomeComponent`）或 React 元素（例如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListHeaderComponentStyle`

用於 `ListHeaderComponent` 內部 View 的樣式設定。

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `debug`

`debug` 會開啟額外的日誌記錄和視覺疊加層，以協助除錯使用和實作問題，但會顯著影響效能。

| Type    |
| ------- |
| boolean |

---

### `disableVirtualization`

> **已棄用。** 虛擬化提供了顯著的效能和記憶體優化，但會完全卸載渲染視窗之外的 React 實例。您應該僅在除錯時需要停用此功能。

| Type    |
| ------- |
| boolean |

---

### `extraData`

一個標記屬性，用於通知列表重新渲染（因為它實作了 `PureComponent`）。如果您的 `renderItem`、Header、Footer 等函式依賴於 `data` 屬性之外的任何內容，請將其放在這裡並視為不可變。

| Type |
| ---- |
| any  |

---

### `getItemLayout`

```tsx
(
  data: any,
  index: number,
) => {length: number, offset: number, index: number}
```

| Type     |
| -------- |
| function |

---

### `horizontal`

如果為 `true`，則將項目水平並排渲染，而不是垂直堆疊。

| Type    |
| ------- |
| boolean |

---

### `initialNumToRender`

初始批次中要渲染的項目數量。這應該足以填滿螢幕，但不要太多。請注意，這些項目永遠不會作為視窗化渲染的一部分被卸載，以提高滾動到頂部操作的感知效能。

| Type   | Default |
| ------ | ------- |
| number | `10`    |

---

### `initialScrollIndex`

不從頂部的第一個項目開始，而是從 `initialScrollIndex` 指定的位置開始。這會停用「滾動至頂部」的優化功能（該功能會始終渲染前 `initialNumToRender` 個項目），並立即從此初始索引開始渲染項目。需要實作 `getItemLayout` 方法。

| Type   |
| ------ |
| number |

---

### `inverted`

反轉滾動方向。使用 `-1` 的縮放變換實現。

| Type    |
| ------- |
| boolean |

---

### `keyExtractor`

```tsx
(item: any, index: number) => string;
```

用於為指定索引的項目提取唯一鍵值。該鍵值用於緩存及作為 React 的 key 來追蹤項目重新排序。預設提取器會檢查 `item.key`，其次是 `item.id`，最後回退到使用索引（與 React 的行為一致）。

| Type     |
| -------- |
| function |

---

### `maxToRenderPerBatch`

每次增量渲染批次中要渲染的最大項目數。一次渲染越多，填充率越好，但響應性可能受影響，因為渲染內容可能會干擾按鈕點擊或其他互動操作。

| Type   |
| ------ |
| number |

---

### `onEndReached`

當滾動位置接近列表邏輯末尾的 `onEndReachedThreshold` 範圍內時，會觸發一次此回調。

| Type                                        |
| ------------------------------------------- |
| `(info: {distanceFromEnd: number}) => void` |

---

### `onEndReachedThreshold`

列表尾部邊緣距離內容末尾多遠時（以列表可見長度為單位）會觸發 `onEndReached` 回調。例如，值為 0.5 表示當內容末尾處於列表可見長度的一半範圍內時觸發。

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onRefresh`

```tsx
() => void;
```

若提供此屬性，將添加標準的 `RefreshControl` 組件以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

| Type     |
| -------- |
| function |

---

### `onScrollToIndexFailed`

```tsx
(info: {
  index: number,
  highestMeasuredFrameIndex: number,
  averageItemLength: number,
}) => void;
```

用於處理滾動至尚未測量索引時的失敗情況。建議操作是自行計算偏移量並調用 `scrollTo`，或先滾動到最遠位置後等待更多項目渲染完成再重試。

| Type     |
| -------- |
| function |

---

### `onStartReached`

當滾動位置接近列表邏輯開頭的 `onStartReachedThreshold` 範圍內時，會觸發一次此回調。

| Type                                          |
| --------------------------------------------- |
| `(info: {distanceFromStart: number}) => void` |

---

### `onStartReachedThreshold`

列表頭部邊緣距離內容開頭多遠時（以列表可見長度為單位）會觸發 `onStartReached` 回調。例如，值為 0.5 表示當內容開頭處於列表可見長度的一半範圍內時觸發。

| Type   | Default |
| ------ | ------- |
| number | `2`     |

---

### `onViewableItemsChanged`

當行的可見性發生變化時觸發（由 `viewabilityConfig` 屬性定義的可見性條件）。

| Type                                                                                                  |
| ----------------------------------------------------------------------------------------------------- |
| `md (callback: {changed: [ViewToken](viewtoken)[], viewableItems: [ViewToken](viewtoken)[]}) => void` |

---

### `persistentScrollbar`

| Type |
| ---- |
| bool |

---

### `progressViewOffset`

當需要讓載入指示器正確顯示時，可設置此偏移量。

| Type   |
| ------ |
| number |

---

### `refreshControl`

自定義的刷新控制元件。設置後會覆蓋內建預設的 `<RefreshControl>` 元件，同時忽略 `onRefresh` 和 `refreshing` 屬性。僅適用於垂直方向的 VirtualizedList。

| Type    |
| ------- |
| element |

---

### `refreshing`

在等待刷新數據時，將此屬性設為 `true`。

| Type    |
| ------- |
| boolean |

---

### `removeClippedSubviews`

此屬性可提升大型列表的滾動效能。

> 注意：某些情況下可能出現錯誤（內容缺失）—— 使用時需自行承擔風險。

| Type    |
| ------- |
| boolean |

---

### `renderScrollComponent`

```tsx
(props: object) => element;
```

渲染自定義滾動元件，例如使用不同樣式的 `RefreshControl`。

| Type     |
| -------- |
| function |

---

### `viewabilityConfig`

參見 `ViewabilityHelper.js` 的 flow 類型與進一步文檔說明。

| Type              |
| ----------------- |
| ViewabilityConfig |

---

### `viewabilityConfigCallbackPairs`

`ViewabilityConfig` 與 `onViewableItemsChanged` 的配對列表。當對應 `ViewabilityConfig` 的條件滿足時，會觸發特定的 `onViewableItemsChanged` 回調。詳見 `ViewabilityHelper.js` 的 flow 類型與文檔。

| Type                                   |
| -------------------------------------- |
| array of ViewabilityConfigCallbackPair |

---

### `updateCellsBatchingPeriod`

低優先級項目渲染批次間的時間間隔（例如渲染遠離螢幕可視區域的項目）。與 `maxToRenderPerBatch` 類似，需在填充率與響應速度之間權衡。

| Type   |
| ------ |
| number |

---

### `windowSize`

決定在可視區域外渲染的最大項目數量（以可視區域長度為單位）。例如列表填滿螢幕時，預設值 `windowSize={21}` 會渲染可視區域加上視窗上方10屏與下方10屏的內容。減少此數值可降低記憶體消耗並可能提升效能，但快速滾動時可能短暫出現未渲染內容的空白區域。

| Type   |
| ------ |
| number |

## 方法

### `flashScrollIndicators()`

```tsx
flashScrollIndicators();
```

---

### `getScrollableNode()`

```tsx
getScrollableNode(): any;
```

---

### `getScrollRef()`

```tsx
getScrollRef():
  | React.ElementRef<typeof ScrollView>
  | React.ElementRef<typeof View>
  | null;
```

---

### `getScrollResponder()`

```tsx
getScrollResponder () => ScrollResponderMixin | null;
```

提供底層滾動響應器的操作句柄。注意 `this._scrollRef` 可能非 `ScrollView` 實例，需確認其是否實現 `getScrollResponder` 方法後再調用。

---

### `scrollToEnd()`

```tsx
scrollToEnd(params?: {animated?: boolean});
```

滾動至內容末端。若未設置 `getItemLayout` 屬性可能出現卡頓。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵值包括：

- `'animated'` (布林值) - 列表滾動時是否執行動畫。預設為 `true`。

---

### `scrollToIndex()`

```tsx
scrollToIndex(params: {
  index: number;
  animated?: boolean;
  viewOffset?: number;
  viewPosition?: number;
});
```

有效的 `params` 包含：

- 'index' (數字)。必填。
- 'animated' (布林值)。選填。
- 'viewOffset' (數字)。選填。
- 'viewPosition' (數字)。選填。

---

### `scrollToItem()`

```tsx
scrollToItem(params: {
  item: ItemT;
  animated?: boolean;
  viewOffset?: number;
  viewPosition?: number;
);
```

有效的 `params` 包含：

- 'item' (項目)。必填。
- 'animated' (布林值)。選填。
- 'viewOffset' (數字)。選填。
- 'viewPosition' (數字)。選填。

---

### `scrollToOffset()`

```tsx
scrollToOffset(params: {
  offset: number;
  animated?: boolean;
});
```

滾動至列表中的特定內容像素偏移位置。

參數 `offset` 需指定要滾動到的偏移量。若 `horizontal` 為 true，則偏移量為 x 值；其他情況下偏移量為 y 值。

參數 `animated` (預設為 `true`) 定義列表滾動時是否執行動畫。