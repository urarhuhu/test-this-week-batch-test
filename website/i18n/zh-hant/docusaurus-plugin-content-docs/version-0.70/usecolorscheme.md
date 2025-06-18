---
id: usecolorscheme
title: useColorScheme
---

```jsx
import {useColorScheme} from 'react-native';
```

`useColorScheme` React Hook 會從 [`Appearance`](appearance) 模組提供並訂閱色彩方案的更新。回傳值表示使用者當前偏好的色彩方案。該值可能會在之後更新，可能是透過使用者直接操作（例如在裝置設定中選擇主題），或是根據排程（例如跟隨日夜循環的淺色與深色主題）。

### 支援的色彩方案

- `"light"`：使用者偏好淺色主題。
- `"dark"`：使用者偏好深色主題。
- `null`：使用者尚未指定偏好的色彩主題。

> **注意：** 目前由於技術限制，當啟用 Chrome 偵錯工具時，此 Hook 將_總是_回傳 `"light"`。

---

## 範例

```SnackPlayer
import React from 'react';
import { Text, StyleSheet, useColorScheme, View } from 'react-native';

const App = () => {
  const colorScheme = useColorScheme();
  return (
    <View style={styles.container}>
      <Text>useColorScheme(): {colorScheme}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
});

export default App;
```

你可以在 [`AppearanceExample.js`](https://github.com/facebook/react-native/blob/0.70-stable/packages/rn-tester/js/examples/Appearance/AppearanceExample.js) 找到一個完整範例，展示如何搭配 React context 使用此 Hook，為你的應用程式加入淺色與深色主題的支援。