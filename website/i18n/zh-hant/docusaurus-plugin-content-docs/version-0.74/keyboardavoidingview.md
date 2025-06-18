---
id: keyboardavoidingview
title: KeyboardAvoidingView
---

此元件會根據鍵盤高度自動調整其高度、位置或底部內距，確保虛擬鍵盤顯示時仍保持可見。

## 範例

```SnackPlayer name=KeyboardAvoidingView&supportedPlatforms=android,ios
import React from 'react';
import {
  View,
  KeyboardAvoidingView,
  TextInput,
  StyleSheet,
  Text,
  Platform,
  TouchableWithoutFeedback,
  Button,
  Keyboard,
} from 'react-native';

const KeyboardAvoidingComponent = () => {
  return (
    <KeyboardAvoidingView
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={styles.container}>
      <TouchableWithoutFeedback onPress={Keyboard.dismiss}>
        <View style={styles.inner}>
          <Text style={styles.header}>Header</Text>
          <TextInput placeholder="Username" style={styles.textInput} />
          <View style={styles.btnContainer}>
            <Button title="Submit" onPress={() => null} />
          </View>
        </View>
      </TouchableWithoutFeedback>
    </KeyboardAvoidingView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  inner: {
    padding: 24,
    flex: 1,
    justifyContent: 'space-around',
  },
  header: {
    fontSize: 36,
    marginBottom: 48,
  },
  textInput: {
    height: 40,
    borderColor: '#000000',
    borderBottomWidth: 1,
    marginBottom: 36,
  },
  btnContainer: {
    backgroundColor: 'white',
    marginTop: 12,
  },
});

export default KeyboardAvoidingComponent;
```

---

# 參考文件

## 屬性

### [View 元件屬性](view.md#props)

繼承 [View 元件屬性](view.md#props)。

---

### `behavior`

指定如何因應鍵盤的出現。

> Android 和 iOS 對此屬性的處理方式不同。建議在兩個平台都設定 `behavior` 屬性。

| Type                                        |
| ------------------------------------------- |
| enum(`'height'`, `'position'`, `'padding'`) |

---

### `contentContainerStyle`

當 behavior 設為 `'position'` 時，內容容器 (View) 的樣式。

| Type                              |
| --------------------------------- |
| [View Style](view-style-props.md) |

---

### `enabled`

啟用或停用 KeyboardAvoidingView。

| Type    | Default |
| ------- | ------- |
| boolean | `true`  |

---

### `keyboardVerticalOffset`

此數值代表使用者螢幕頂部與 React Native 視圖之間的距離，在某些使用情境中可能非零。

| Type   | Default |
| ------ | ------- |
| number | `0`     |