---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

## 術語

- **VirtualizedList:** `FlatList` 的底層組件（React Native 對 [`虛擬列表`](https://bvaughn.github.io/react-virtualized/#/components/List) 概念的實現。）

- **記憶體消耗:** 列表相關資訊在記憶體中的佔用量，過高可能導致應用程式崩潰。

- **響應性:** 應用程式對互動的反應能力。例如低響應性表現為觸控元件後延遲響應，而非預期的立即反應。

- **空白區域:** 當 `VirtualizedList` 無法快速渲染項目時，列表會出現未渲染元件形成的空白區塊。

- **視窗區域:** 實際渲染為像素的可見內容區域。

- **渲染窗口:** 應掛載項目的區域範圍，通常遠大於視窗區域。

## 屬性

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，視窗區域外的視圖會從原生視圖層級中分離。

**優點:** 透過排除視窗區域外的視圖，減少主執行緒處理時間，從而降低掉幀風險。

**缺點:** 此實作可能存在錯誤（尤其在 iOS 上），例如內容缺失（常見於使用變形或絕對定位的複雜情境）。需注意此設定不會顯著節省記憶體，因視圖僅被分離而非釋放。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

可透過 `FlatList` 傳遞的 `VirtualizedList` 屬性，控制每次滾動時渲染的項目批次數量。

**優點:** 較大的數值能減少滾動時的視覺空白（提升填充率）。

**缺點:** 每批次處理更多項目會導致 JavaScript 執行時間延長，可能阻塞點擊等事件處理，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

此屬性與 `maxToRenderPerBatch` 搭配使用，設定批次渲染間的毫秒延遲（控制窗口項目的渲染頻率）。

**優點:** 結合兩屬性可靈活配置，例如「較少批次處理更多項目」或「較多批次處理較少項目」。

**缺點:** 低頻批次可能導致空白區域，高頻批次可能引發響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 過低的設定可能導致空白區域，尤其在初始渲染時無法覆蓋整個視窗的情況下。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（預設值 21 表示上方 10 單位、下方 10 單位及中間 1 單位）。

**優點:** 較大的 `windowSize` 可降低滾動空白機率，較小的值則減少同時掛載項目以節省記憶體。

**缺點:** 較大值增加記憶體消耗，較小值提高出現空白區域的風險。

## 列表項目

以下是關於列表項元件的一些建議。它們是列表的核心，因此需要保持高效運作。

### 使用基礎元件

元件結構越複雜，渲染速度就越慢。請盡量避免在列表項中放置過多邏輯或嵌套層次。若該列表項元件需在應用中大量重複使用，建議為大型列表專門設計一個元件，並盡可能簡化其邏輯與嵌套結構。

### 使用輕量級元件

元件負載越重，渲染速度越慢。避免使用高解析度圖片（列表項應使用裁剪版或縮略圖，尺寸盡可能小）。與設計團隊溝通，在列表中盡量減少特效、互動元素和資訊量，可將這些內容保留在項目詳情頁呈現。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化元件，僅當傳入的 props 發生變化時才會重新渲染。我們可利用此函數來優化 FlatList 中的元件效能。

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

此範例中，我們設定 MyListItem 僅在 title 變更時重新渲染。透過向 React.memo() 傳入比較函數作為第二參數，可指定僅在特定 prop 變化時觸發重新渲染。若比較函數返回 true，則元件不會重新渲染。

### 使用快取優化圖片

可選用社群套件（例如 [@DylanVann](https://github.com/DylanVann) 開發的 [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image)）來提升圖片效能。列表中每個圖片都是獨立的 `new Image()` 實例，越快觸發 `loaded` 鉤子，JavaScript 線程就能越快釋放。

### 使用 getItemLayout

若所有列表項元件具有相同高度（水平列表則為寬度），提供 [getItemLayout](flatlist#getitemlayout) prop 可免除 FlatList 進行非同步佈局計算的需求，這是一項極具價值的優化技術。

若元件尺寸動態變化且亟需效能提升，可考慮與設計團隊討論是否調整設計方案以改善效能表現。

### 使用 keyExtractor 或 key

可為 FlatList 元件設置 [`keyExtractor`](flatlist#keyextractor) prop。此 prop 用於快取機制，並作為 React 的 `key` 來追蹤項目重新排序。

也可直接在項目元件中使用 `key` prop。

### 避免在 renderItem 使用匿名函數

對於函數式元件，應將 `renderItem` 函數移至返回的 JSX 外部，並用 `useCallback` 鉤子包裹以避免每次渲染時重新創建。

對於類別元件，應將 `renderItem` 函數移至 render 函數外部，防止每次呼叫 render 時重新創建。

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