---
id: dropshadowvalue
title: DropShadowValue Object Type
---

`DropShadowValue` 物件由 [`filter`](./view-style-props.md#filter) 樣式屬性用於 `dropShadow` 函數。它由 2 或 3 個長度值和一個可選的顏色組成。這些值共同定義了陰影的顏色、位置和模糊程度。

## 範例

```js
{
  offsetX: 10,
  offsetY: -3,
  standardDeviation: '15px',
  color: 'blue',
}
```

## 鍵值與數值

### `offsetX`

X 軸上的偏移量。可以是正值或負值。正值表示向右偏移，負值表示向左偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `offsetY`

Y 軸上的偏移量。可以是正值或負值。正值表示向上偏移，負值表示向下偏移。

| Type             | Optional |
| ---------------- | -------- |
| number \| string | No       |

### `standardDeviation`

代表用於[高斯模糊](https://en.wikipedia.org/wiki/Gaussian_blur)演算法的標準差。數值越大，陰影越模糊。僅允許非負值，預設值為 0。

| Type            | Optional |
| --------------- | -------- |
| numer \| string | Yes      |

### `color`

陰影的顏色。預設為 `black`（黑色）。

| Type                 | Optional |
| -------------------- | -------- |
| [color](./colors.md) | Yes      |

## 使用場景

- [`filter`](./view-style-props.md#filter)