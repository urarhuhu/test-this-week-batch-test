---
id: actionsheetios
title: ActionSheetIOS
---

顯示 iOS 原生的 [操作選單](https://developer.apple.com/design/human-interface-guidelines/ios/views/action-sheets/) 元件。

## 範例

```SnackPlayer name=ActionSheetIOS%20Example&supportedPlatforms=ios
import React, {useState} from 'react';
import {ActionSheetIOS, Button, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [result, setResult] = useState('🔮');

  const onPress = () =>
    ActionSheetIOS.showActionSheetWithOptions(
      {
        options: ['Cancel', 'Generate number', 'Reset'],
        destructiveButtonIndex: 2,
        cancelButtonIndex: 0,
        userInterfaceStyle: 'dark',
      },
      buttonIndex => {
        if (buttonIndex === 0) {
          // cancel action
        } else if (buttonIndex === 1) {
          setResult(String(Math.floor(Math.random() * 100) + 1));
        } else if (buttonIndex === 2) {
          setResult('🔮');
        }
      },
    );

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text style={styles.result}>{result}</Text>
        <Button onPress={onPress} title="Show Action Sheet" />
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
  },
  result: {
    fontSize: 64,
    textAlign: 'center',
  },
});

export default App;
```

# 參考文獻

## 方法

### `showActionSheetWithOptions()`

```tsx
static showActionSheetWithOptions: (
  options: ActionSheetIOSOptions,
  callback: (buttonIndex: number) => void,
);
```

顯示 iOS 操作選單。`options` 物件必須包含以下一或多項：

- `options` (字串陣列) - 按鈕標題列表 (必填)
- `cancelButtonIndex` (整數) - `options` 中取消按鈕的索引
- `cancelButtonTintColor` (字串) - 用於變更取消按鈕文字顏色的 [顏色](colors)
- `destructiveButtonIndex` (整數或整數陣列) - `options` 中破壞性按鈕的索引
- `title` (字串) - 顯示在操作選單上方的標題
- `message` (字串) - 顯示在標題下方的訊息
- `anchor` (數字) - 操作選單應錨定的節點 (用於 iPad)
- `tintColor` (字串) - 用於非破壞性按鈕標題的 [顏色](colors)
- `disabledButtonIndices` (數字陣列) - 應禁用的按鈕索引列表
- `userInterfaceStyle` (字串) - 操作選單使用的介面樣式，可設為 `light` 或 `dark`，否則將使用預設系統樣式

'callback' 函式接受一個參數，即所選項目的從零開始的索引。

最小範例：

```tsx
ActionSheetIOS.showActionSheetWithOptions(
  {
    options: ['Cancel', 'Remove'],
    destructiveButtonIndex: 1,
    cancelButtonIndex: 0,
  },
  buttonIndex => {
    if (buttonIndex === 1) {
      /* destructive action */
    }
  },
);
```

---

### `dismissActionSheet()`

```tsx
static dismissActionSheet();
```

關閉最上層的 iOS 操作選單，如果沒有操作選單則顯示警告。

---

### `showShareActionSheetWithOptions()`

```tsx
static showShareActionSheetWithOptions: (
  options: ShareActionSheetIOSOptions,
  failureCallback: (error: Error) => void,
  successCallback: (success: boolean, method: string) => void,
);
```

顯示 iOS 分享選單。`options` 物件應包含 `message` 和 `url` 其中一項或兩項，並可額外包含 `subject` 或 `excludedActivityTypes`：

- `url` (字串) - 要分享的 URL
- `message` (字串) - 要分享的訊息
- `subject` (字串) - 訊息的主題
- `excludedActivityTypes` (陣列) - 要從操作選單中排除的活動類型

> **注意：** 如果 `url` 指向本地文件或是 base64 編碼的 uri，則會直接載入並分享該文件。透過這種方式，您可以分享圖片、影片、PDF 文件等。如果 `url` 指向遠端文件或地址，則必須符合 [RFC 2396](https://www.ietf.org/rfc/rfc2396.txt) 中描述的 URL 格式。例如，沒有正確協議 (HTTP/HTTPS) 的網址將無法分享。

'failureCallback' 函式接受一個參數，即錯誤物件。此物件上唯一定義的屬性是選用的 `stack` 屬性，類型為 `string`。

'successCallback' 函式接受兩個參數：

- 表示成功或失敗的布林值
- 在成功情況下，表示分享方式的字串