---
id: flatlist
title: FlatList
---

一個高效能的介面，用於渲染基礎的平面列表，支援最便利的功能：

- 完全跨平台。
- 可選的水平模式。
- 可配置的可見性回調。
- 支援標頭。
- 支援頁尾。
- 支援分隔線。
- 下拉刷新。
- 滾動載入。
- 支援 ScrollToIndex。
- 支援多列。

如果需要分區塊支援，請使用 [`<SectionList>`](sectionlist.md)。

## 範例

```SnackPlayer name=flatlist-simple
import React from 'react';
import { SafeAreaView, View, FlatList, StyleSheet, Text, StatusBar } from 'react-native';

const DATA = [
  {
    id: 'bd7acbea-c1b1-46c2-aed5-3ad53abb28ba',
    title: 'First Item',
  },
  {
    id: '3ac68afc-c605-48d3-a4f8-fbd91aa97f63',
    title: 'Second Item',
  },
  {
    id: '58694a0f-3da1-471f-bd96-145571e29d72',
    title: 'Third Item',
  },
];

const Item = ({ title }) => (
  <View style={styles.item}>
    <Text style={styles.title}>{title}</Text>
  </View>
);

const App = () => {
  const renderItem = ({ item }) => (
    <Item title={item.title} />
  );

  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        data={DATA}
        renderItem={renderItem}
        keyExtractor={item => item.id}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight || 0,
  },
  item: {
    backgroundColor: '#f9c2ff',
    padding: 20,
    marginVertical: 8,
    marginHorizontal: 16,
  },
  title: {
    fontSize: 32,
  },
});

export default App;
```

要渲染多列，請使用 [`numColumns`](flatlist.md#numcolumns) 屬性。使用這種方法而不是 `flexWrap` 佈局可以避免與項目高度邏輯發生衝突。

下方是更複雜、可選取的範例。

- 通過將 `extraData={selectedId}` 傳遞給 `FlatList`，我們確保 `FlatList` 本身會在狀態變化時重新渲染。如果不設置這個屬性，`FlatList` 不會知道需要重新渲染任何項目，因為它是一個 `PureComponent`，且屬性比較不會顯示任何變化。
- `keyExtractor` 告訴列表使用 `id` 作為 React 鍵，而不是預設的 `key` 屬性。

```SnackPlayer name=flatlist-selectable
import React, { useState } from "react";
import { FlatList, SafeAreaView, StatusBar, StyleSheet, Text, TouchableOpacity } from "react-native";

const DATA = [
  {
    id: "bd7acbea-c1b1-46c2-aed5-3ad53abb28ba",
    title: "First Item",
  },
  {
    id: "3ac68afc-c605-48d3-a4f8-fbd91aa97f63",
    title: "Second Item",
  },
  {
    id: "58694a0f-3da1-471f-bd96-145571e29d72",
    title: "Third Item",
  },
];

const Item = ({ item, onPress, backgroundColor, textColor }) => (
  <TouchableOpacity onPress={onPress} style={[styles.item, backgroundColor]}>
    <Text style={[styles.title, textColor]}>{item.title}</Text>
  </TouchableOpacity>
);

const App = () => {
  const [selectedId, setSelectedId] = useState(null);

  const renderItem = ({ item }) => {
    const backgroundColor = item.id === selectedId ? "#6e3b6e" : "#f9c2ff";
    const color = item.id === selectedId ? 'white' : 'black';

    return (
      <Item
        item={item}
        onPress={() => setSelectedId(item.id)}
        backgroundColor={{ backgroundColor }}
        textColor={{ color }}
      />
    );
  };

  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        data={DATA}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        extraData={selectedId}
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    marginTop: StatusBar.currentHeight || 0,
  },
  item: {
    padding: 20,
    marginVertical: 8,
    marginHorizontal: 16,
  },
  title: {
    fontSize: 32,
  },
});

export default App;
```

這是 [`<VirtualizedList>`](virtualizedlist.md) 的便利包裝，因此繼承了其屬性（以及 [`<ScrollView>`](scrollview.md) 的屬性），這些屬性未在此明確列出，同時有以下注意事項：

- 當內容滾動出渲染窗口時，內部狀態不會保留。確保所有數據都捕獲在項目數據或外部存儲（如 Flux、Redux 或 Relay）中。
- 這是一個 `PureComponent`，意味著如果 `props` 保持淺層相等，它不會重新渲染。確保 `renderItem` 函數依賴的所有內容都作為屬性傳遞（例如 `extraData`），且在更新後不會 `===`，否則您的 UI 可能不會在變化時更新。這包括 `data` 屬性和父組件狀態。
- 為了限制內存並實現平滑滾動，內容會異步渲染到屏幕外。這意味著可能會滾動得比填充速率更快，並暫時看到空白內容。這是一個可以根據每個應用程序的需求進行調整的權衡，我們正在幕後努力改進它。
- 預設情況下，列表會查找每個項目上的 `key` 屬性並將其用作 React 鍵。或者，您可以提供自定義的 `keyExtractor` 屬性。

---

# 參考

## 屬性

### [VirtualizedList 屬性](virtualizedlist.md#props)

繼承 [VirtualizedList 屬性](virtualizedlist.md#props)。

---

### <div class="label required basic">必填</div> **`renderItem`**

```jsx
renderItem({item, index, separators});
```

從 `data` 中取得一個項目並將其渲染到列表中。

提供額外的元數據，如 `index`（如果需要），以及更通用的 `separators.updateProps` 函數，讓您可以設置任何屬性來更改前導分隔線或尾隨分隔線的渲染，以防更常見的 `highlight` 和 `unhighlight`（設置 `highlighted: boolean` 屬性）不足以滿足您的使用情況。

| Type     |
| -------- |
| function |

- `item` (Object)：從 `data` 中渲染的項目。
- `index` (number)：對應於 `data` 陣列中此項目的索引。
- `separators` (Object)
  - `highlight` (Function)
  - `unhighlight` (Function)
  - `updateProps` (Function)
    - `select` (enum('leading', 'trailing'))
    - `newProps` (Object)

使用範例：

```jsx
<FlatList
  ItemSeparatorComponent={
    Platform.OS !== 'android' &&
    (({highlighted}) => (
      <View
        style={[style.separator, highlighted && {marginLeft: 0}]}
      />
    ))
  }
  data={[{title: 'Title Text', key: 'item1'}]}
  renderItem={({item, index, separators}) => (
    <TouchableHighlight
      key={item.key}
      onPress={() => this._onPress(item)}
      onShowUnderlay={separators.highlight}
      onHideUnderlay={separators.unhighlight}>
      <View style={{backgroundColor: 'white'}}>
        <Text>{item.title}</Text>
      </View>
    </TouchableHighlight>
  )}
/>
```

---

### <div class="label required basic">Required</div> **`data`**

為簡化操作，data 是一個普通陣列。若需使用其他資料結構（如不可變列表），請直接使用底層的 [`VirtualizedList`](virtualizedlist.md)。

| Type  |
| ----- |
| array |

---

### `ItemSeparatorComponent`

在每個項目之間渲染（但不在頂部或底部）。預設會提供 `highlighted` 和 `leadingItem` 屬性。`renderItem` 提供的 `separators.highlight`/`unhighlight` 可更新 `highlighted` 屬性，也可透過 `separators.updateProps` 添加自訂屬性。可為 React 元件（如 `SomeComponent`）或 React 元素（如 `<SomeComponent />`）。

| Type                         |
| ---------------------------- |
| component, function, element |

---

### `ListEmptyComponent`

當列表為空時渲染。可為 React 元件（如 `SomeComponent`）或 React 元素（如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponent`

在所有項目底部渲染。可為 React 元件（如 `SomeComponent`）或 React 元素（如 `<SomeComponent />`）。

| Type               |
| ------------------ |
| component, element |

---

### `ListFooterComponentStyle`

用於 `ListFooterComponent` 內部 View 的樣式設定。

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `ListHeaderComponent`

在所有項目頂部渲染。可為 React 元件（如 `SomeComponent`）或 React 元素（如 `<SomeComponent />`）。

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

### `columnWrapperStyle`

當 `numColumns > 1` 時，用於生成多項目行的選用自訂樣式。

| Type                           |
| ------------------------------ |
| [View Style](view-style-props) |

---

### `extraData`

由於列表實作了 `PureComponent`，此標記屬性用於通知列表重新渲染。若 `renderItem`、Header、Footer 等函式依賴於 `data` 屬性外的任何內容，請將其置於此並視為不可變資料。

| Type |
| ---- |
| any  |

---

### `getItemLayout`

```jsx
(data, index) => {length: number, offset: number, index: number}
```

`getItemLayout` 是選用的效能優化方法，若已知項目尺寸（高度或寬度），可跳過動態內容的測量。對於固定尺寸項目（例如以下情況）特別有效：

```jsx
  getItemLayout={(data, index) => (
    {length: ITEM_HEIGHT, offset: ITEM_HEIGHT * index, index}
  )}
```

添加 `getItemLayout` 能大幅提升數百個項目列表的效能。若指定了 `ItemSeparatorComponent`，請記得在偏移計算中包含分隔線長度（高度或寬度）。

| Type     |
| -------- |
| function |

---

### `horizontal`

設為 `true` 時，項目會水平並排渲染而非垂直堆疊。

| Type    |
| ------- |
| boolean |

---

### `initialNumToRender`

初始批次中要渲染的項目數量。這個數量應該足以填滿螢幕，但不需要過多。請注意，這些項目永遠不會因為視窗化渲染而被卸載，以提升滾動至頂部的感知效能。

| Type   | Default |
| ------ | ------- |
| number | `10`    |

---

### `initialScrollIndex`

不從第一個項目開始，而是從 `initialScrollIndex` 指定的位置開始。這會停用「滾動至頂部」的優化（該優化會保持前 `initialNumToRender` 個項目始終渲染），並立即渲染從此初始索引開始的項目。需要實作 `getItemLayout`。

| Type   |
| ------ |
| number |

---

### `inverted`

反轉滾動方向。使用 `-1` 的比例變換。

| Type    |
| ------- |
| boolean |

---

### `keyExtractor`

```jsx
(item: object, index: number) => string;
```

用於為指定索引的項目提取唯一鍵。該鍵用於快取和作為 React 的鍵來追蹤項目重新排序。預設提取器會檢查 `item.key`，然後是 `item.id`，最後回退到使用索引（與 React 的行為相同）。

| Type     |
| -------- |
| function |

---

### `numColumns`

多列只能在 `horizontal={false}` 時渲染，並且會以類似 `flexWrap` 佈局的鋸齒狀排列。所有項目應具有相同高度——不支援磚石佈局。

| Type   |
| ------ |
| number |

---

### `onEndReached`

```jsx
(info: {distanceFromEnd: number}) => void
```

當滾動位置接近渲染內容的 `onEndReachedThreshold` 範圍內時，會呼叫一次。

| Type     |
| -------- |
| function |

---

### `onEndReachedThreshold`

列表底部邊緣距離內容末尾多遠（以列表可見長度為單位）時觸發 `onEndReached` 回調。例如，值為 0.5 表示當內容末尾在列表可見長度的一半範圍內時觸發 `onEndReached`。

| Type   |
| ------ |
| number |

---

### `onRefresh`

```jsx
() => void
```

如果提供，將添加標準的 RefreshControl 以實現「下拉刷新」功能。請確保同時正確設置 `refreshing` 屬性。

| Type     |
| -------- |
| function |

---

### `onViewableItemsChanged`

當行的可見性發生變化時呼叫，由 `viewabilityConfig` 屬性定義。

| Type                                                                                                  |
| ----------------------------------------------------------------------------------------------------- |
| `md (callback: {changed: [ViewToken](viewtoken)[], viewableItems: [ViewToken](viewtoken)[]} => void;` |

---

### `progressViewOffset`

當需要偏移量以正確顯示載入指示器時設置此屬性。

| Type   |
| ------ |
| number |

---

### `refreshing`

在等待刷新數據時將其設置為 true。

| Type    |
| ------- |
| boolean |

---

### `removeClippedSubviews`

這可能會提升大型列表的滾動效能。在 Android 上預設值為 `true`。

> 注意：在某些情況下可能會出現錯誤（內容缺失）——使用時需自行承擔風險。

| Type    |
| ------- |
| boolean |

---

### `viewabilityConfig`

請參閱 [`ViewabilityHelper.js`](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Lists/ViewabilityHelper.js) 以獲取流程類型和進一步文檔。

| Type              |
| ----------------- |
| ViewabilityConfig |

`viewabilityConfig` 接受一個類型為 `ViewabilityConfig` 的物件，該物件具有以下屬性：

| Property                         | Type    |
| -------------------------------- | ------- |
| minimumViewTime                  | number  |
| viewAreaCoveragePercentThreshold | number  |
| itemVisiblePercentThreshold      | number  |
| waitForInteraction               | boolean |

至少需要設定 `viewAreaCoveragePercentThreshold` 或 `itemVisiblePercentThreshold` 其中之一。這需要在 `constructor` 中完成，以避免以下錯誤（[參考](https://github.com/facebook/react-native/issues/17408)）：

```
  Error: Changing viewabilityConfig on the fly is not supported
```

```jsx
constructor (props) {
  super(props)

  this.viewabilityConfig = {
      waitForInteraction: true,
      viewAreaCoveragePercentThreshold: 95
  }
}
```

```jsx
<FlatList
    viewabilityConfig={this.viewabilityConfig}
  ...
```

#### minimumViewTime

項目必須在物理上可見的最短時間（以毫秒為單位），才會觸發可見性回調。數值越高，表示快速滾動內容而不停止時，不會將內容標記為可見。

#### viewAreaCoveragePercentThreshold

視窗必須被覆蓋的百分比（0-100），部分被遮擋的項目才會被視為「可見」。完全可見的項目始終被視為可見。值為 0 表示視窗中的單個像素即可使項目可見，值為 100 表示項目必須完全可見或覆蓋整個視窗才能被視為可見。

#### itemVisiblePercentThreshold

與 `viewAreaCoveragePercentThreshold` 類似，但考慮的是項目本身的可見百分比，而非其覆蓋視窗區域的比例。

#### waitForInteraction

在用戶滾動或在渲染後調用 `recordInteraction` 之前，不會將任何內容視為可見。

---

### `viewabilityConfigCallbackPairs`

`ViewabilityConfig`/`onViewableItemsChanged` 配對的列表。當對應的 `ViewabilityConfig` 條件滿足時，將調用特定的 `onViewableItemsChanged`。請參閱 `ViewabilityHelper.js` 以獲取流程類型和進一步文檔。

| Type                                   |
| -------------------------------------- |
| array of ViewabilityConfigCallbackPair |

## 方法

### `flashScrollIndicators()`

```jsx
flashScrollIndicators();
```

短暫顯示滾動指示器。

---

### `getNativeScrollRef()`

```jsx
getNativeScrollRef();
```

提供對底層滾動元件的引用。

---

### `getScrollResponder()`

```jsx
getScrollResponder();
```

提供對底層滾動響應器的引用。

---

### `getScrollableNode()`

```jsx
getScrollableNode();
```

提供對底層滾動節點的引用。

---

### `recordInteraction()`

```jsx
recordInteraction();
```

通知列表發生了交互，這應觸發可見性計算，例如當 `waitForInteractions` 為 true 且用戶尚未滾動時。通常由點擊項目或導航操作調用。

---

### `scrollToEnd()`

```jsx
scrollToEnd(params);
```

滾動到內容的末尾。若未設定 `getItemLayout` 屬性，可能會出現卡頓。

**參數：**

| Name   | Type   |
| ------ | ------ |
| params | object |

有效的 `params` 鍵為：

- 'animated' (boolean) - 滾動時是否執行動畫。預設為 `true`。

---

### `scrollToIndex()`

```jsx
scrollToIndex(params);
```

滾動至指定索引的項目，使其位於可視區域內：`viewPosition` 為 0 時置頂，1 時置底，0.5 時居中對齊。

> 注意：若未指定 `getItemLayout` 屬性，則無法滾動至渲染視窗外的位置。

**參數：**

| Name                                                        | Type   |
| ----------------------------------------------------------- | ------ |
| params <div className="label basic required">Required</div> | object |

有效的 `params` 鍵值為：

- 'animated' (布林值) - 滾動時是否啟用動畫效果。預設為 `true`。
- 'index' (數字) - 要滾動至的索引值。必填。
- 'viewOffset' (數字) - 最終目標位置的固定像素偏移量。
- 'viewPosition' (數字) - 值為 `0` 時將指定索引項目置頂，`1` 時置底，`0.5` 時居中。

---

### `scrollToItem()`

```jsx
scrollToItem(params);
```

需線性掃描數據集，建議優先使用 `scrollToIndex`。

> 注意：若未指定 `getItemLayout` 屬性，則無法滾動至渲染視窗外的位置。

**參數：**

| Name                                                        | Type   |
| ----------------------------------------------------------- | ------ |
| params <div className="label basic required">Required</div> | object |

有效的 `params` 鍵值為：

- 'animated' (布林值) - 滾動時是否啟用動畫效果。預設為 `true`。
- 'item' (物件) - 要滾動至的項目。必填。
- 'viewPosition' (數字)

---

### `scrollToOffset()`

```jsx
scrollToOffset(params);
```

滾動至列表中的特定內容像素偏移位置。

**參數：**

| Name                                                        | Type   |
| ----------------------------------------------------------- | ------ |
| params <div className="label basic required">Required</div> | object |

有效的 `params` 鍵值為：

- 'offset' (數字) - 要滾動至的偏移量。若 `horizontal` 為 true，偏移量為 x 值，否則為 y 值。必填。
- 'animated' (布林值) - 滾動時是否啟用動畫效果。預設為 `true`。