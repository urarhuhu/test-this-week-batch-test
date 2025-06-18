---
id: settings
title: Settings
---

`Settings` 作為 [`NSUserDefaults`](https://developer.apple.com/documentation/foundation/nsuserdefaults) 的封裝層，這是一個僅在 iOS 上可用的持久化鍵值儲存系統。

## 範例

```SnackPlayer name=Settings%20Example&supportedPlatforms=ios
import React, {useState} from 'react';
import {Button, Settings, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [data, setData] = useState(() => Settings.get('data'));

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text>Stored value:</Text>
        <Text style={styles.value}>{data}</Text>
        <Button
          onPress={() => {
            Settings.set({data: 'React'});
            setData(Settings.get('data'));
          }}
          title="Store 'React'"
        />
        <Button
          onPress={() => {
            Settings.set({data: 'Native'});
            setData(Settings.get('data'));
          }}
          title="Store 'Native'"
        />
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  value: {
    fontSize: 24,
    marginVertical: 12,
  },
});

export default App;
```

---

# 參考文獻

## 方法

### `clearWatch()`

```tsx
static clearWatch(watchId: number);
```

`watchId` 是當初訂閱時由 `watchKeys()` 回傳的數字識別碼。

---

### `get()`

```tsx
static get(key: string): any;
```

從 `NSUserDefaults` 中取得指定 `key` 的當前值。

---

### `set()`

```tsx
static set(settings: Record<string, any>);
```

在 `NSUserDefaults` 中設定一個或多個值。

---

### `watchKeys()`

```tsx
static watchKeys(keys: string | array<string>, callback: () => void): number;
```

訂閱以接收當 `keys` 參數指定的任何鍵值在 `NSUserDefaults` 中被更改時的通知。回傳一個 `watchId` 數字，可用於 `clearWatch()` 取消訂閱。

> **注意：** `watchKeys()` 的設計會忽略內部 `set()` 呼叫，僅在 React Native 程式碼外部觸發變更時才會執行回調函數。