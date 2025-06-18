---
id: animatedvalue
title: Animated.Value
---

驅動動畫的標準值。一個 `Animated.Value` 可以同步驅動多個屬性，但一次只能由一種機制驅動。使用新機制（例如啟動新動畫或調用 `setValue`）將停止先前的所有機制。

通常在類組件中初始化為 `useAnimatedValue(0);` 或 `new Animated.Value(0);`。

---

# 參考文檔

## 方法

### `setValue()`

```tsx
setValue(value: number);
```

直接設定數值。這會停止該數值上運行的所有動畫，並更新所有綁定的屬性。

**參數：**

| Name  | Type   | Required | Description |
| ----- | ------ | -------- | ----------- |
| value | number | Yes      | Value       |

---

### `setOffset()`

```tsx
setOffset(offset: number);
```

設定一個偏移量，該偏移量會附加在通過 `setValue`、動畫或 `Animated.event` 設定的數值之上。適用於補償平移手勢的起始點等情況。

**參數：**

| Name   | Type   | Required | Description  |
| ------ | ------ | -------- | ------------ |
| offset | number | Yes      | Offset value |

---

### `flattenOffset()`

```tsx
flattenOffset();
```

將偏移量合併到基礎數值中，並將偏移量重置為零。數值的最終輸出保持不變。

---

### `extractOffset()`

```tsx
extractOffset();
```

將偏移量設為基礎數值，並將基礎數值重置為零。數值的最終輸出保持不變。

---

### `addListener()`

```tsx
addListener(callback: (state: {value: number}) => void): string;
```

為數值添加異步監聽器，以便觀察動畫的更新。這很有用，因為無法同步讀取數值（可能由原生驅動）。

返回一個字符串作為監聽器的標識符。

**參數：**

| Name     | Type     | Required | Description                                                                                 |
| -------- | -------- | -------- | ------------------------------------------------------------------------------------------- |
| callback | function | Yes      | The callback function which will receive an object with a `value` key set to the new value. |

---

### `removeListener()`

```tsx
removeListener(id: string);
```

取消註冊監聽器。`id` 參數需與先前 `addListener()` 返回的標識符匹配。

**參數：**

| Name | Type   | Required | Description                        |
| ---- | ------ | -------- | ---------------------------------- |
| id   | string | Yes      | Id for the listener being removed. |

---

### `removeAllListeners()`

```tsx
removeAllListeners();
```

移除所有已註冊的監聽器。

---

### `stopAnimation()`

```tsx
stopAnimation(callback?: (value: number) => void);
```

停止任何正在運行的動畫或追蹤。`callback` 會在停止動畫後以最終數值調用，便於將狀態更新至與動畫位置匹配的佈局。

**參數：**

| Name     | Type     | Required | Description                                   |
| -------- | -------- | -------- | --------------------------------------------- |
| callback | function | No       | A function that will receive the final value. |

---

### `resetAnimation()`

```tsx
resetAnimation(callback?: (value: number) => void);
```

停止任何動畫並將數值重置為原始狀態。

**參數：**

| Name     | Type     | Required | Description                                      |
| -------- | -------- | -------- | ------------------------------------------------ |
| callback | function | No       | A function that will receive the original value. |

---

### `interpolate()`

```tsx
interpolate(config: InterpolationConfigType);
```

在更新屬性前對數值進行插值，例如將 0-1 映射到 0-10。

參見 `AnimatedInterpolation.js`

**參數：**

| Name   | Type   | Required | Description |
| ------ | ------ | -------- | ----------- |
| config | object | Yes      | See below.  |

`config` 物件由以下鍵值組成：

- `inputRange`: 數字陣列
- `outputRange`: 數字或字串陣列
- `easing` (選填): 接收輸入數字並回傳數字的函式
- `extrapolate` (選填): 字串，例如 'extend'、'identity' 或 'clamp'
- `extrapolateLeft` (選填): 字串，例如 'extend'、'identity' 或 'clamp'
- `extrapolateRight` (選填): 字串，例如 'extend'、'identity' 或 'clamp'

---

### `animate()`

```tsx
animate(animation, callback);
```

通常僅在內部使用，但可被自訂動畫類別調用。

**參數：**

| Name      | Type      | Required | Description         |
| --------- | --------- | -------- | ------------------- |
| animation | Animation | Yes      | See `Animation.js`. |
| callback  | function  | Yes      | Callback function.  |