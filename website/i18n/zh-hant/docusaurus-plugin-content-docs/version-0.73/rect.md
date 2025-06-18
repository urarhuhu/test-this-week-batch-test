---
id: rect
title: Rect Object Type
---

`Rect` 接受數值型像素值來描述矩形區域的擴展範圍。這些值會與原始區域的大小相加以進行擴展。

## 範例

```js
{
    bottom: 20,
    left: null,
    right: undefined,
    top: 50
}
```

## 鍵與值

### `bottom`

| Type                        | Required |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

### `left`

| Type                        | Required |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

### `right`

| Type                        | Required |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

### `top`

| Type                        | Required |
| --------------------------- | -------- |
| number, `null`, `undefined` | No       |

## 使用於

- [`Image`](image)
- [`Pressable`](pressable)
- [`Text`](text)
- [`TouchableWithoutFeedback`](touchablewithoutfeedback)