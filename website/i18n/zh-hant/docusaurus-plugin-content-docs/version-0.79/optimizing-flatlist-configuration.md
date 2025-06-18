---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

## 術語

- **VirtualizedList:** `FlatList` 底層的元件（React Native 對 [`虛擬列表`](https://bvaughn.github.io/react-virtualized/#/components/List) 概念的實現。）

- **記憶體消耗:** 應用程式記憶體中儲存的列表資訊量，過高可能導致應用程式崩潰。

- **響應速度:** 應用程式對互動的反應能力。例如低響應速度表現為觸控元件後延遲回應，而非預期的立即反應。

- **空白區域:** 當 `VirtualizedList` 無法快速渲染項目時，列表可能出現未渲染元件形成的空白區塊。

- **視窗區域:** 實際渲染為像素的可見內容區域。

- **渲染窗口:** 應掛載項目的區域範圍，通常遠大於視窗區域。

## 屬性參數

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，視窗區域外的子視圖會從原生視圖層級中分離。

**優點:** 透過排除視窗區域外的視圖，減少主執行緒處理時間，從而降低掉幀風險。

**缺點:** 需注意此實現可能存在錯誤（尤其在iOS平台），例如內容缺失（常見於使用複雜變換或絕對定位時）。另請注意此設定不會顯著節省記憶體，因視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

此為可透過 `FlatList` 傳遞的 `VirtualizedList` 屬性，控制每批次渲染的項目數量（每次滾動時渲染的新項目區塊大小）。

**優點:** 設定較大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目會導致JavaScript執行時間延長，可能阻塞點擊等事件處理，影響響應速度。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

`maxToRenderPerBatch` 控制每批次渲染量，而此屬性設定批次渲染間的毫秒延遲（決定元件渲染窗口項目的頻率）。

**優點:** 與 `maxToRenderPerBatch` 搭配可靈活調控，例如實現「高頻次少項目」或「低頻次多項目」渲染策略。

**缺點:** 低頻批次可能導致空白區域，高頻批次可能引發響應遲緩。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確設定覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 過低的設定可能導致初始渲染時出現空白區域（特別是無法完整覆蓋視窗的情況）。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（預設值21，表示上方10單位+下方10單位+中間1單位）。

**優點:** 較大值可降低滾動空白率；較小值則減少同時掛載項目數以節省記憶體。

**缺點:** 較大值增加記憶體消耗，較小值提高出現空白區域的機率。

## 列表項目

以下是關於列表項元件的一些建議。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度越慢。盡量避免在列表項中放入過多邏輯和嵌套結構。如果這個列表項元件在應用中被大量重複使用，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套層次。

### 使用輕量級元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版或縮略圖，盡可能小）。與設計團隊溝通，在列表中盡量減少效果、互動和資訊量，將這些內容放在項目的詳細頁面中展示。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化的元件，只有在傳遞給元件的 props 發生變化時才會重新渲染。我們可以使用這個函數來優化 FlatList 中的元件。

```tsx
import React, {memo} from 'react';
import {View, Text} from 'react-native';

const MyListItem = memo(
  ({title}: {title: string}) => (
    <View>
      <Text>{title}</Text>
    </View>
  ),
  (prevProps, nextProps) => {
    return prevProps.title === nextProps.title;
  },
);

export default MyListItem;
```

在這個例子中，我們確定 MyListItem 應該只在標題變化時重新渲染。我們將比較函數作為第二個參數傳遞給 React.memo()，這樣元件就只會在指定的 prop 變化時重新渲染。如果比較函數返回 true，元件將不會重新渲染。

### 使用緩存的優化圖片

你可以使用社區套件（例如來自 [@DylanVann](https://github.com/DylanVann) 的 [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image)）來獲得更高效的圖片。列表中的每個圖片都是一個 `new Image()` 實例。它越快達到 `loaded` 鉤子，JavaScript 線程就能越快釋放。

### 使用 getItemLayout

如果所有列表項元件的高度相同（對於水平列表則是寬度相同），提供 [getItemLayout](flatlist#getitemlayout) prop 可以讓 `FlatList` 不需要管理異步佈局計算。這是一個非常理想的優化技術。

如果元件有動態大小且你真的需要性能，可以考慮詢問設計團隊是否能重新設計以提高性能。

### 使用 keyExtractor 或 key

你可以為 `FlatList` 元件設置 [`keyExtractor`](flatlist#keyextractor)。這個 prop 用於緩存和作為 React `key` 來追蹤項目的重新排序。

你也可以在項目元件中使用 `key` prop。

### 避免在 renderItem 中使用匿名函數

對於函數式元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被包裹在 `useCallback` 鉤子中，以防止每次渲染時重新創建。

對於類元件，將 `renderItem` 函數移到 render 函數之外，這樣它就不會在每次調用 render 函數時重新創建自己。

```tsx
const renderItem = useCallback(({item}) => (
   <View key={item.key}>
      <Text>{item.title}</Text>
   </View>
 ), []);

return (
  // ...

  <FlatList data={items} renderItem={renderItem} />;
  // ...
);
```