---
id: toastandroid
title: ToastAndroid
---

React Native 的 ToastAndroid API 將 Android 平台的 ToastAndroid 模組以 JS 模組形式公開。它提供方法 `show(message, duration)`，接收以下參數：

- _message_ 要顯示的提示文字字串
- _duration_ 提示顯示的持續時間——可選 `ToastAndroid.SHORT` 或 `ToastAndroid.LONG`

您也可以使用 `showWithGravity(message, duration, gravity)` 來指定提示在畫面中的顯示位置。可選值為 `ToastAndroid.TOP`、`ToastAndroid.BOTTOM` 或 `ToastAndroid.CENTER`。

方法 `showWithGravityAndOffset(message, duration, gravity, xOffset, yOffset)` 則額外支援以像素為單位的偏移量設定。

> 從 Android 11（API 級別 30）開始，設定重力參數對文字提示不再有效。相關變更請參閱[此處](https://developer.android.com/about/versions/11/behavior-changes-11#text-toast-api-changes)。

```SnackPlayer name=Toast%20Android%20API%20Example&supportedPlatforms=android
import React from 'react';
import {StyleSheet, ToastAndroid, Button, StatusBar} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const showToast = () => {
    ToastAndroid.show('A pikachu appeared nearby !', ToastAndroid.SHORT);
  };

  const showToastWithGravity = () => {
    ToastAndroid.showWithGravity(
      'All Your Base Are Belong To Us',
      ToastAndroid.SHORT,
      ToastAndroid.CENTER,
    );
  };

  const showToastWithGravityAndOffset = () => {
    ToastAndroid.showWithGravityAndOffset(
      'A wild toast appeared!',
      ToastAndroid.LONG,
      ToastAndroid.BOTTOM,
      25,
      50,
    );
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Button title="Toggle Toast" onPress={() => showToast()} />
        <Button
          title="Toggle Toast With Gravity"
          onPress={() => showToastWithGravity()}
        />
        <Button
          title="Toggle Toast With Gravity & Offset"
          onPress={() => showToastWithGravityAndOffset()}
        />
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingTop: StatusBar.currentHeight,
    backgroundColor: '#888888',
    padding: 8,
  },
});

export default App;
```

---

# 參考文件

## 方法

### `show()`

```tsx
static show(message: string, duration: number);
```

---

### `showWithGravity()`

此屬性僅在 Android API 29 及以下版本有效。如需在更高版本實現類似功能，建議使用 snackbar 或 notification。

```tsx
static showWithGravity(message: string, duration: number, gravity: number);
```

---

### `showWithGravityAndOffset()`

此屬性僅在 Android API 29 及以下版本有效。如需在更高版本實現類似功能，建議使用 snackbar 或 notification。

```tsx
static showWithGravityAndOffset(
  message: string,
  duration: number,
  gravity: number,
  xOffset: number,
  yOffset: number,
);
```

## 屬性

### `SHORT`

表示提示在畫面上的顯示持續時間。

```tsx
static SHORT: number;
```

---

### `LONG`

表示提示在畫面上的顯示持續時間。

```tsx
static LONG: number;
```

---

### `TOP`

表示提示在畫面上的顯示位置。

```tsx
static TOP: number;
```

---

### `BOTTOM`

表示提示在畫面上的顯示位置。

```tsx
static BOTTOM: number;
```

---

### `CENTER`

表示提示在畫面上的顯示位置。

```tsx
static CENTER: number;
```