---
id: view-style-props
title: View Style Props
---

### 範例

```SnackPlayer name=ViewStyleProps
import React from 'react';
import {View, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
      <View style={styles.top} />
      <View style={styles.middle} />
      <View style={styles.bottom} />
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-between',
    padding: 20,
    margin: 10,
  },
  top: {
    flex: 0.3,
    backgroundColor: 'grey',
    borderWidth: 5,
    borderTopLeftRadius: 20,
    borderTopRightRadius: 20,
  },
  middle: {
    flex: 0.3,
    backgroundColor: 'beige',
    borderWidth: 5,
  },
  bottom: {
    flex: 0.3,
    backgroundColor: 'pink',
    borderWidth: 5,
    borderBottomLeftRadius: 20,
    borderBottomRightRadius: 20,
  },
});

export default App;
```

# 參考文獻

## 屬性

### `backfaceVisibility`

| Type                          |
| ----------------------------- |
| enum(`'visible'`, `'hidden'`) |

---

### `backgroundColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderBottomEndRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomLeftRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomRightRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomStartRadius`

| Type   |
| ------ |
| number |

---

### `borderStartEndRadius`

| Type   |
| ------ |
| number |

---

### `borderStartStartRadius`

| Type   |
| ------ |
| number |

---

### `borderEndEndRadius`

| Type   |
| ------ |
| number |

---

### `borderEndStartRadius`

| Type   |
| ------ |
| number |

---

### `borderBottomWidth`

| Type   |
| ------ |
| number |

---

### `borderColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderCurve` <div class="label ios">iOS</div>

在 iOS 13+ 上，可以變更邊框的角落曲線。

| Type                               |
| ---------------------------------- |
| enum(`'circular'`, `'continuous'`) |

---

### `borderEndColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderLeftWidth`

| Type   |
| ------ |
| number |

---

### `borderRadius`

如果圓角邊框不可見，請嘗試同時套用 `overflow: 'hidden'`。

| Type   |
| ------ |
| number |

---

### `borderRightColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderRightWidth`

| Type   |
| ------ |
| number |

---

### `borderStartColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderStyle`

| Type                                    |
| --------------------------------------- |
| enum(`'solid'`, `'dotted'`, `'dashed'`) |

---

### `borderTopColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `borderTopEndRadius`

| Type   |
| ------ |
| number |

---

### `borderTopLeftRadius`

| Type   |
| ------ |
| number |

---

### `borderTopRightRadius`

| Type   |
| ------ |
| number |

---

### `borderTopStartRadius`

| Type   |
| ------ |
| number |

---

### `borderTopWidth`

| Type   |
| ------ |
| number |

---

### `borderWidth`

| Type   |
| ------ |
| number |

### `boxShadow`

:::note
`boxShadow` 僅在[新架構](/architecture/landing-page)中可用。外陰影僅支援 **Android 9+**。內陰影僅支援 **Android 10+**。
:::

為元素添加陰影效果，可控制陰影的位置、顏色、大小和模糊程度。此陰影可根據是否設置為_內陰影_，出現在元素邊框框的外部或內部。這是符合規範的[同名網頁樣式屬性](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow)實現。詳細參數說明請參閱[BoxShadowValue](./boxshadowvalue)文檔。

這些陰影可以組合使用，因此單個`boxShadow`可由多個不同的陰影組成。

`boxShadow`接受模仿[網頁語法](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow#syntax)的字串，或[BoxShadowValue](./boxshadowvalue)物件的陣列。

| Type |
| --------------------------- |
| array of BoxShadowValue ojects \| string |

### `cursor` <div class="label ios">iOS</div>

在 iOS 17+ 上，設置為`pointer`可在指針（如 iOS 上的觸控板或觸控筆，或 visionOS 上的用戶視線）懸停在視圖上時啟用懸停效果。

| Type                        |
| --------------------------- |
| enum(`'auto'`, `'pointer'`) |

---

### `elevation` <div class="label android">Android</div>

使用 Android 底層的[高程 API](https://developer.android.com/training/material/shadows-clipping.html#Elevation)設置視圖的高程。這會為項目添加投影並影響重疊視圖的 z 軸順序。僅支援 Android 5.0+，對早期版本無效。

| Type   |
| ------ |
| number |

---

### `filter`

:::note
`filter` 僅在[新架構](/architecture/landing-page)中可用
:::

為`View`添加圖形濾鏡。此濾鏡由任意數量的_濾鏡函數_組成，每個函數代表對`View`圖形合成的某種原子性改變。所有有效濾鏡函數的完整列表如下所述。`filter`將應用於`View`的後代元素以及`View`本身。`filter`隱含`overflow: hidden`，因此後代元素將被裁剪以適應`View`的邊界。

以下濾鏡函數在所有平台上均可用：

- `brightness`: 改變`View`的亮度。接受非負數或百分比。
- `opacity`: 改變`View`的不透明度（alpha值）。接受非負數或百分比。

:::note
由於性能和規範合規性問題，iOS 上僅支援這兩種濾鏡函數。計劃探索使用 SwiftUI 替代 UIKit 實現的潛在解決方案。
:::

<div class="label basic android">Android</div>

以下濾鏡函數僅在 Android 上可用：

- `blur`: 使用[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)對`View`進行模糊處理，指定長度代表模糊演算法中使用的半徑。任何非負的DIP值均有效（不接受百分比）。數值越大，模糊效果越明顯。
- `contrast`: 調整`View`的對比度。接受非負數值或百分比。
- `dropShadow`: 在`View`的alpha遮罩周圍添加陰影（僅`View`中非零alpha像素會投射陰影）。可選參數包括代表陰影顏色的色值，以及2或3個長度值。若指定2個長度，則解釋為`offsetX`和`offsetY`，分別表示陰影在X和Y軸上的位移。若提供第3個長度，則解釋為高斯模糊的標準差——數值越大陰影越模糊。詳細參數說明請參閱[DropShadowValue](./dropshadowvalue.md)。
- `grayscale`: 將`View`轉換為[灰階](https://en.wikipedia.org/wiki/Grayscale)，轉換程度由指定數值控制。接受非負數值或百分比，其中`1`或`100%`表示完全灰階。
- `hueRotate`: 調整`View`的[色調](https://en.wikipedia.org/wiki/Hue)。此函數參數定義色輪旋轉角度（例如`360deg`將無效果），單位可為`deg`或`rad`。
- `invert`: 反轉`View`的顏色。接受非負數值或百分比，其中`1`或`100%`表示完全反轉。
- `sepia`: 將`View`轉換為[棕褐色調](<https://en.wikipedia.org/wiki/Sepia_(color)>)。接受非負數值或百分比，其中`1`或`100%`表示完全轉換。
- `saturate`: 調整`View`的[飽和度](https://en.wikipedia.org/wiki/Colorfulness)。接受非負數值或百分比。

:::note
`blur`和`dropShadow`僅支援 **Android 12+** 版本
:::

`filter`可接受由上述濾鏡函數組成的物件陣列，或模仿[網頁語法](https://developer.mozilla.org/en-US/docs/Web/CSS/filter#syntax)的字串。

| Type |
| ------ |
| array of objects: `{brightness: number\|string}`, `{opacity: number\|string}`, `{blur: number\|string}`, `{contrast: number\|string}`, `{dropShadow: DropShadowValue\|string}`, `{grayscale: number\|string}`, `{hueRotate: number\|string}`, `{invert: number\|string}`, `{sepia: number\|string}`, `{saturate: number\|string}` or string|

---

### `opacity`

| Type   |
| ------ |
| number |

### `pointerEvents`

控制`View`是否能成為觸控事件的目標。

- `'auto'`: View可成為觸控事件的目標。
- `'none'`: View永遠不會成為觸控事件的目標。
- `'box-none'`: View本身不會成為觸控事件目標，但其子視圖可以。
- `'box-only'`: View可成為觸控事件目標，但其子視圖不能。

| Type                                                  |
| ----------------------------------------------------- |
| enum(`'auto'`, `'box-none'`, `'box-only'`, `'none'` ) |