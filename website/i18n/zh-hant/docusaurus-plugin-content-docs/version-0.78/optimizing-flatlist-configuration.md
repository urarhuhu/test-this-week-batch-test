---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

## 術語

- **VirtualizedList:** `FlatList` 的底層組件（React Native 對 [`虛擬列表`](https://bvaughn.github.io/react-virtualized/#/components/List) 概念的實現）

- **記憶體消耗:** 列表相關資訊佔用的記憶體量，過高可能導致應用程式崩潰

- **響應速度:** 應用程式對互動的反應能力。例如低響應速度表現為觸控組件後需延遲才會反應，而非預期的立即響應

- **空白區域:** 當 `VirtualizedList` 無法快速渲染項目時，列表會出現未渲染組件形成的空白區塊

- **視窗區域:** 實際被渲染為像素的可見內容區域

- **緩衝窗口:** 應掛載項目的區域範圍，通常遠大於視窗區域

## 屬性參數

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，視窗區域外的視圖會從原生視圖層級中分離

**優點:** 通過排除視窗區域外的視圖，減少主執行緒處理時間，從而降低掉幀風險

**缺點:** 需注意此實現可能存在錯誤（尤其在iOS平台），例如內容缺失（常見於使用複雜變換或絕對定位時）。另請注意這不會顯著節省記憶體，因為視圖僅被分離而非釋放

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

可透過 `FlatList` 傳遞的 `VirtualizedList` 屬性，控制每批次渲染的項目數量（即每次滾動時渲染的下一個項目區塊）

**優點:** 設置較大數值可減少滾動時的視覺空白區域（提升填充率）

**缺點:** 每批次更多項目意味著JavaScript執行時間更長，可能阻塞點擊等事件處理，影響響應速度

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

`maxToRenderPerBatch` 控制每批次渲染量，而此屬性設定批次渲染間的毫秒延遲（控制組件渲染窗口項目的頻率）

**優點:** 與 `maxToRenderPerBatch` 配合可實現靈活策略，例如「低頻次大批量」或「高頻次小批量」渲染

**缺點:** 低頻次批次可能導致空白區域，高頻次批次可能引發響應遲緩

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量

**優點:** 精確定義覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能

**缺點:** 設置過低的 `initialNumToRender` 可能導致空白區域，特別是初始渲染時無法完整覆蓋視窗

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（預設值21，表示上方10視窗+下方10視窗+中間1視窗）

**優點:** 較大的 `windowSize` 可降低滾動空白率；較小的 `windowSize` 能減少同時掛載的項目數以節省記憶體

**缺點:** 較大的 `windowSize` 會增加記憶體消耗；較小的 `windowSize` 則提高出現空白區域的機率

## 列表項目

以下是一些關於列表項元件的技巧。它們是列表的核心，因此需要保持高效。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項中放入過多邏輯和嵌套結構。如果這個列表項元件在應用中被大量重複使用，可以專門為大型列表創建一個元件，並盡可能減少邏輯和嵌套層次。

### 使用輕量級元件

元件越重，渲染速度越慢。避免使用過大的圖片（在列表項中使用裁剪版或縮略圖，尺寸越小越好）。與設計團隊溝通，在列表中盡量減少效果、互動和資訊的展示，可以將這些內容放在項目的詳細頁面中。

### 使用 `memo()`

`React.memo()` 會創建一個記憶化的元件，只有在傳遞給元件的屬性發生變化時才會重新渲染。我們可以使用這個函數來優化 FlatList 中的元件。

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

在這個例子中，我們確定 MyListItem 只有在標題變化時才需要重新渲染。我們將比較函數作為第二個參數傳遞給 React.memo()，這樣元件就只會在指定的屬性變化時重新渲染。如果比較函數返回 true，元件將不會重新渲染。

### 使用緩存的優化圖片

你可以使用社區套件（例如來自 [@dream-sports-labs](https://github.com/dream-sports-labs) 的 [@d11/react-native-fast-image](https://github.com/dream-sports-labs/react-native-fast-image)）來獲得更高效的圖片。列表中的每一張圖片都是一個 `new Image()` 實例。圖片越快達到 `loaded` 鉤子，JavaScript 線程就能越快釋放。

### 使用 getItemLayout

如果所有列表項元件的高度相同（或寬度相同，對於水平列表），提供 [getItemLayout](flatlist#getitemlayout) 屬性可以讓 `FlatList` 不需要管理異步佈局計算。這是一個非常理想的優化技術。

如果元件尺寸是動態的且你確實需要性能，可以考慮與設計團隊溝通，看看是否可以重新設計以提高性能。

### 使用 keyExtractor 或 key

你可以為 `FlatList` 元件設置 [`keyExtractor`](flatlist#keyextractor) 屬性。這個屬性用於緩存和作為 React `key` 來追蹤項目的重新排序。

你也可以在項目元件中使用 `key` 屬性。

### 避免在 renderItem 中使用匿名函數

對於函數式元件，將 `renderItem` 函數移到返回的 JSX 之外。同時，確保它被包裹在 `useCallback` 鉤子中，以防止每次渲染時重新創建。

對於類元件，將 `renderItem` 函數移到 render 函數之外，這樣它就不會在每次調用 render 函數時重新創建。

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