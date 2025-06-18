---
id: refreshcontrol
title: RefreshControl
---

此元件用於 ScrollView 或 ListView 內部，以實現下拉刷新功能。當 ScrollView 處於 `scrollY: 0` 位置時，向下滑動會觸發 `onRefresh` 事件。

## 範例

```SnackPlayer name=RefreshControl&supportedPlatforms=ios,android
import React from 'react';
import {RefreshControl, ScrollView, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [refreshing, setRefreshing] = React.useState(false);

  const onRefresh = React.useCallback(() => {
    setRefreshing(true);
    setTimeout(() => {
      setRefreshing(false);
    }, 2000);
  }, []);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <ScrollView
          contentContainerStyle={styles.scrollView}
          refreshControl={
            <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
          }>
          <Text>Pull down to see RefreshControl indicator</Text>
        </ScrollView>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  scrollView: {
    flex: 1,
    backgroundColor: 'pink',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

> 注意：`refreshing` 是一個受控屬性，因此必須在 `onRefresh` 函式中將其設為 `true`，否則刷新指示器會立即停止。

---

# 參考文檔

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### <div class="label required basic">必填</div>**`refreshing`**

是否顯示正在刷新的指示狀態。

| Type    |
| ------- |
| boolean |

---

### `colors` <div class="label android">Android</div>

用於繪製刷新指示器的顏色（至少需指定一種）。

| Type                         |
| ---------------------------- |
| array of [colors](colors.md) |

---

### `enabled` <div class="label android">Android</div>

是否啟用下拉刷新功能。

| Type    | Default |
| ------- | ------- |
| boolean | `true`  |

---

### `onRefresh`

當視圖開始刷新時呼叫。

| Type     |
| -------- |
| function |

---

### `progressBackgroundColor` <div class="label android">Android</div>

刷新指示器的背景顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `progressViewOffset`

進度視圖的頂部偏移量。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `size` <div class="label android">Android</div>

刷新指示器的大小。

| Type                         | Default     |
| ---------------------------- | ----------- |
| enum(`'default'`, `'large'`) | `'default'` |

---

### `tintColor` <div class="label ios">iOS</div>

刷新指示器的顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `title` <div class="label ios">iOS</div>

顯示在刷新指示器下方的標題文字。

| Type   |
| ------ |
| string |

---

### `titleColor` <div class="label ios">iOS</div>

刷新指示器標題的文字顏色。

| Type               |
| ------------------ |
| [color](colors.md) |