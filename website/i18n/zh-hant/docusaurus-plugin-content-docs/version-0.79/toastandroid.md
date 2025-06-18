---
id: toastandroid
title: ToastAndroid
---

React Native 的 ToastAndroid API 將 Android 平台的 ToastAndroid 模組作為 JS 模組公開。它提供了 `show(message, duration)` 方法，該方法接受以下參數：

- _message_ 一個包含要顯示訊息的字串
- _duration_ 訊息顯示的持續時間——可以是 `ToastAndroid.SHORT` 或 `ToastAndroid.LONG`

或者，您可以使用 `showWithGravity(message, duration, gravity)` 來指定訊息在螢幕佈局中的顯示位置。可以是 `ToastAndroid.TOP`、`ToastAndroid.BOTTOM` 或 `ToastAndroid.CENTER`。

`showWithGravityAndOffset(message, duration, gravity, xOffset, yOffset)` 方法增加了以像素為單位指定偏移量的功能。

> 從 Android 11（API 級別 30）開始，設置重力對文字訊息沒有影響。閱讀相關變更[這裡](https://developer.android.com/about/versions/11/behavior-changes-11#text-toast-api-changes)。

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

# 參考

## 方法

### `show()`

```tsx
static show(message: string, duration: number);
```

---

### `showWithGravity()`

此屬性僅適用於 Android API 29 及以下版本。對於更高 Android API 的類似功能，請考慮使用 snackbar 或通知。

```tsx
static showWithGravity(message: string, duration: number, gravity: number);
```

---

### `showWithGravityAndOffset()`

此屬性僅適用於 Android API 29 及以下版本。對於更高 Android API 的類似功能，請考慮使用 snackbar 或通知。

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

表示訊息在螢幕上的顯示時間。

```tsx
static SHORT: number;
```

---

### `LONG`

表示訊息在螢幕上的顯示時間。

```tsx
static LONG: number;
```

---

### `TOP`

表示訊息在螢幕上的位置。

```tsx
static TOP: number;
```

---

### `BOTTOM`

表示訊息在螢幕上的位置。

```tsx
static BOTTOM: number;
```

---

### `CENTER`

表示訊息在螢幕上的位置。

```tsx
static CENTER: number;
```