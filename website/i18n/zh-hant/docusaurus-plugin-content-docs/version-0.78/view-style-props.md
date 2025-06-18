---
id: view-style-props
title: View Style Props
---

### Example

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

# Reference

## Props

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

On iOS 13+, it is possible to change the corner curve of borders.

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

If the rounded border is not visible, try applying `overflow: 'hidden'` as well.

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
`boxShadow` is only available on the [New Architecture](/architecture/landing-page). Outset shadows are only supported on **Android 9+**. Inset shadows are only supported on **Android 10+**.
:::

Adds a shadow effect to an element, with the ability to control the position, color, size, and blurriness of the shadow. This shadow either appears around the outside or inside of the border box of the element, depending on whether or not the shadow is _inset_. This is a spec-compliant implementation of the [web style prop of the same name](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow). Read more about all the arguments available in the [BoxShadowValue](./boxshadowvalue) documentation.

These shadows can be composed together so that a single `boxShadow` can be comprised of multiple different shadows.

`boxShadow` takes either a string which mimics the [web syntax](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow#syntax) or an array of [BoxShadowValue](./boxshadowvalue) objects.

| Type |
| --------------------------- |
| array of BoxShadowValue ojects \| string |

### `cursor` <div class="label ios">iOS</div>

On iOS 17+, Setting to `pointer` allows hover effects when a pointer (such as a trackpad or stylus on iOS, or the users' gaze on visionOS) is over the view.

| Type                        |
| --------------------------- |
| enum(`'auto'`, `'pointer'`) |

---

### `elevation` <div class="label android">Android</div>

Sets the elevation of a view, using Android's underlying [elevation API](https://developer.android.com/training/material/shadows-clipping.html#Elevation). This adds a drop shadow to the item and affects z-order for overlapping views. Only supported on Android 5.0+, has no effect on earlier versions.

| Type   |
| ------ |
| number |

---

### `filter`

:::note
`filter` is only available on the [New Architecture](/architecture/landing-page)
:::

Adds a graphical filter to the `View`. This filter is comprised of any number of _filter functions_, which each represent some atomic change to the graphical composition of the `View`. The complete list of valid filter functions is defined below. `filter` will apply to descendants of the `View` as well as the `View` itself. `filter` implies `overflow: hidden`, so descendants will be clipped to fit the bounds of the `View`.

The following filter functions work across all platforms:

- `brightness`: Changes the brightness of the `View`. Takes a non-negative number or percentage.
- `opacity`: Changes the opacity, or alpha, of the `View`. Takes a non-negative number or percentage.

:::note
Due to issues with performance and spec compliance, these are the only two filter functions available on iOS. There are plans to explore some potential workarounds using SwiftUI instead of UIKit for this implementation.
:::

<div class="label basic android">Android</div>

The following filter functions work on Android only:

- `blur`: 使用[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)對`View`進行模糊處理，指定數值代表模糊演算法中使用的半徑。任何非負的DIP值均有效（不接受百分比）。數值越大，模糊效果越明顯。
- `contrast`: 調整`View`的對比度。接受非負數值或百分比。
- `dropShadow`: 在`View`的alpha遮罩周圍添加陰影（僅`View`中非零alpha像素會投射陰影）。可選參數包括代表陰影顏色的顏色值，以及2或3個長度值。若指定2個長度值，則分別解釋為`offsetX`和`offsetY`，表示陰影在X和Y軸上的位移。若提供第3個長度值，則解釋為高斯模糊的標準差——數值越大陰影越模糊。詳見[DropShadowValue](./dropshadowvalue.md)說明。
- `grayscale`: 將`View`轉換為[灰階](https://en.wikipedia.org/wiki/Grayscale)，轉換程度由指定數值控制。接受非負數值或百分比，`1`或`100%`表示完全灰階。
- `hueRotate`: 調整`View`的[色調](https://en.wikipedia.org/wiki/Hue)。參數定義色輪旋轉角度（例如`360deg`將無效果），單位可為`deg`或`rad`。
- `invert`: 反轉`View`的顏色。接受非負數值或百分比，`1`或`100%`表示完全反轉。
- `sepia`: 將`View`轉換為[深褐色調](https://en.wikipedia.org/wiki/Sepia_(color))。接受非負數值或百分比，`1`或`100%`表示完全轉換。
- `saturate`: 調整`View`的[飽和度](https://en.wikipedia.org/wiki/Colorfulness)。接受非負數值或百分比。

:::note
`blur`和`dropShadow`僅支援 **Android 12+**
:::

`filter`可接受由上述濾鏡函式組成的物件陣列，或模仿[網頁語法](https://developer.mozilla.org/en-US/docs/Web/CSS/filter#syntax)的字串。

| Type |
| ------ |
| array of objects: `{brightness: number\|string}`, `{opacity: number\|string}`, `{blur: number\|string}`, `{contrast: number\|string}`, `{dropShadow: DropShadowValue\|string}`, `{grayscale: number\|string}`, `{hueRotate: number\|string}`, `{invert: number\|string}`, `{sepia: number\|string}`, `{saturate: number\|string}` or string|

---

### `opacity`

| Type   |
| ------ |
| number |

---

### `outlineColor`

:::note
`outlineColor`僅在[新架構](/architecture/landing-page)中可用
:::

設定元素外框的顏色。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-color)。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `outlineOffset`

:::note
`outlineOffset`僅在[新架構](/architecture/landing-page)中可用
:::

設定外框與元素邊界之間的間距。不影響佈局。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-offset)。

| Type   |
| ------ |
| number |

---

### `outlineStyle`

:::note
`outlineStyle`僅在[新架構](/architecture/landing-page)中可用
:::

設定元素外框的樣式。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-style)。

| Type                                    |
| --------------------------------------- |
| enum(`'solid'`, `'dotted'`, `'dashed'`) |

---

### `outlineWidth`

:::note
`outlineWidth`僅在[新架構](/architecture/landing-page)中可用
:::

設定元素外圍繪製的輪廓線寬度（位於邊框之外）。不影響佈局。詳見[網頁文件](https://developer.mozilla.org/en-US/docs/Web/CSS/outline-width)。

| Type   |
| ------ |
| number |

---

### `pointerEvents`

控制`View`是否能成為觸控事件的目標。

- `'auto'`: 該View可成為觸控事件的目標。
- `'none'`: 該View永遠不會成為觸控事件的目標。
- `'box-none'`: 該View本身不會成為觸控事件目標，但其子視圖可以。
- `'box-only'`: 該View可成為觸控事件目標，但其子視圖不可。

| Type                                                  |
| ----------------------------------------------------- |
| enum(`'auto'`, `'box-none'`, `'box-only'`, `'none'` ) |