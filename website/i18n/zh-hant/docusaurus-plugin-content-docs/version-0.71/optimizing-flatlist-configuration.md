---
id: optimizing-flatlist-configuration
title: Optimizing Flatlist Configuration
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 術語

- **VirtualizedList:** `FlatList` 的底層組件（React Native 對 [`虛擬列表`](https://bvaughn.github.io/react-virtualized/#/components/List) 概念的實現。）

- **記憶體消耗:** 列表相關資訊佔用的記憶體量，過高可能導致應用程式崩潰。

- **響應性:** 應用程式對互動的反應能力。例如低響應性表現為觸控元件後延遲響應，而非預期的立即反應。

- **空白區域:** 當 `VirtualizedList` 無法快速渲染項目時，列表可能出現未渲染元件形成的空白區域。

- **視窗區域:** 實際渲染為像素的可見內容區域。

- **渲染窗口:** 應掛載項目的區域範圍，通常遠大於視窗區域。

## 屬性

以下是有助提升 `FlatList` 效能的屬性列表：

### removeClippedSubviews

| Type    | Default |
| ------- | ------- |
| Boolean | False   |

設為 `true` 時，視窗區域外的視圖會從原生視圖層級中分離。

**優點:** 通過排除視窗區域外的視圖，減少主執行緒處理時間，從而降低掉幀風險。

**缺點:** 需注意此實現可能存在錯誤（尤其在 iOS 平台），例如內容缺失（常見於複雜變換或絕對定位場景）。另請注意由於視圖僅被分離而非釋放，記憶體節省效果有限。

### maxToRenderPerBatch

| Type   | Default |
| ------ | ------- |
| Number | 10      |

此為可透過 `FlatList` 傳遞的 `VirtualizedList` 屬性，控制每批次渲染的項目數量（即每次滾動時渲染的新項目塊大小）。

**優點:** 較大數值可減少滾動時的視覺空白區域（提升填充率）。

**缺點:** 每批次處理更多項目會導致 JavaScript 執行時間延長，可能阻塞點擊等事件處理，影響響應性。

### updateCellsBatchingPeriod

| Type   | Default |
| ------ | ------- |
| Number | 50      |

`maxToRenderPerBatch` 控制每批次渲染量，而 `updateCellsBatchingPeriod` 則設定批次渲染間的毫秒延遲（控制窗口化項目的渲染頻率）。

**優點:** 結合此屬性與 `maxToRenderPerBatch` 可實現靈活配置，例如低頻次大批量渲染或高頻次小批量渲染。

**缺點:** 低頻次批次可能導致空白區域，高頻次批次可能引發響應性問題。

### initialNumToRender

| Type   | Default |
| ------ | ------- |
| Number | 10      |

初始渲染的項目數量。

**優點:** 精確定義覆蓋各裝置螢幕的項目數，可大幅提升初始渲染效能。

**缺點:** 過低的 `initialNumToRender` 可能導致空白區域，特別是初始渲染時無法覆蓋整個視窗區域的情況。

### windowSize

| Type   | Default |
| ------ | ------- |
| Number | 21      |

此數值以視窗高度為單位（1代表等同視窗高度）。預設值為21（上方10視窗、下方10視窗及中間1視窗）。

**優點:** 較大的 `windowSize` 可降低滾動時出現空白區域的機率；較小的 `windowSize` 則會減少同時掛載的項目數，節省記憶體。

**缺點：** 較大的 `windowSize` 會消耗更多記憶體。較小的 `windowSize` 則會增加出現空白區域的機率。

## 列表項目

以下是一些關於列表項目元件的建議。它們是列表的核心，因此需要保持高效運作。

### 使用基礎元件

元件越複雜，渲染速度就越慢。盡量避免在列表項目中加入過多邏輯或嵌套結構。如果該列表項目元件在應用中重複使用頻繁，建議為大型列表專門建立一個元件，並盡可能減少邏輯和嵌套層級。

### 使用輕量元件

元件越重，渲染速度越慢。避免使用高解析度圖片（在列表項目中使用裁剪版或縮圖，尺寸越小越好）。與設計團隊溝通，盡量減少列表中的特效、互動和資訊量，將這些內容保留在項目詳情頁面中展示。

### 使用 shouldComponentUpdate

為元件實作更新驗證機制。React 的 `PureComponent` 透過淺比較實作了 [`shouldComponentUpdate`](https://reactjs.org/docs/react-component.html#shouldcomponentupdate)。這在列表情境中成本較高，因為需要檢查所有 props。若要追求位元級效能優化，應為列表項目元件制定最嚴格的更新規則，僅檢查可能變動的 props。如果列表結構非常簡單，甚至可以直接使用

```tsx
shouldComponentUpdate() {
  return false
}
```

### 使用快取優化圖片

可選用社群套件（例如 [@DylanVann](https://github.com/DylanVann) 開發的 [react-native-fast-image](https://github.com/DylanVann/react-native-fast-image)）來提升圖片效能。列表中的每張圖片都是獨立的 `new Image()` 實例，越快觸發 `loaded` 鉤子，JavaScript 執行緒就能越快釋放。

### 使用 getItemLayout

若所有列表項目元件高度相同（水平列表則為寬度），提供 [getItemLayout](flatlist#getitemlayout) prop 可免除 `FlatList` 進行非同步佈局計算的需求，這是非常理想的優化技巧。

若元件尺寸動態變化且極需效能提升，可考慮與設計團隊討論是否調整設計方案以優化表現。

### 使用 keyExtractor 或 key

可為 `FlatList` 元件設定 [`keyExtractor`](flatlist#keyextractor)。此 prop 用於快取機制，同時作為 React `key` 來追蹤項目重新排序。

也可直接在項目元件中使用 `key` prop。

### 避免在 renderItem 使用匿名函式

將 `renderItem` 函式移至 render 函式外部，避免每次呼叫 render 時重新建立函式實例。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="classical">

```tsx
renderItem = ({item}) => (<View key={item.key}><Text>{item.title}</Text></View>);

render(){
  // ...

  <FlatList
    data={items}
    renderItem={this.renderItem}
  />

  // ...
}

```

</TabItem>
<TabItem value="functional">

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

</TabItem>
</Tabs>