---
id: imagepickerios
title: '🚧 ImagePickerIOS'
---

> **已棄用。** 請改用 [社群套件](https://reactnative.directory/?search=image+picker) 中的其中一個方案。

---

# 參考文獻

## 方法

### `canRecordVideos()`

```jsx
static canRecordVideos(callback)
```

---

### `canUseCamera()`

```jsx
static canUseCamera(callback)
```

---

### `openCameraDialog()`

```jsx
static openCameraDialog(config, successCallback, cancelCallback)
```

**參數：**

| Name            | Type     | Required | Description |
| --------------- | -------- | -------- | ----------- |
| config          | object   | No       | See below.  |
| successCallback | function | No       | See below.  |
| cancelCallback  | function | No       | See below.  |

`config` 是一個包含以下屬性的物件：

- `videoMode` : 可選的布林值，預設為 false。

`successCallback` 是一個可選的回呼函式，當選擇對話框成功開啟時會被調用。它將包含以下資料：

- `[字串, 數字, 數字]`

`cancelCallback` 是一個可選的回呼函式，當相機對話框被取消時會被調用。

---

### `openSelectDialog()`

```jsx
static openSelectDialog(config, successCallback, cancelCallback)
```

**參數：**

| Name            | Type     | Required | Description |
| --------------- | -------- | -------- | ----------- |
| config          | object   | No       | See below.  |
| successCallback | function | No       | See below.  |
| cancelCallback  | function | No       | See below.  |

`config` 是一個包含以下屬性的物件：

- `showImages` : 可選的布林值，預設為 false。
- `showVideos`: 可選的布林值，預設為 false。

`successCallback` 是一個可選的回呼函式，當選擇對話框成功開啟時會被調用。它將包含以下資料：

- `[字串, 數字, 數字]`

`cancelCallback` 是一個可選的回呼函式，當選擇對話框被取消時會被調用。