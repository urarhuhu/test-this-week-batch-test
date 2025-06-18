---
id: progressbarandroid
title: '🚧 ProgressBarAndroid'
---

> **已棄用。** 請改用[社群套件](https://reactnative.directory/?search=progressbar)中的替代方案。

僅限Android的React元件，用於表示應用程式正在載入或有活動進行中。

### 範例

```SnackPlayer name=ProgressBarAndroid&supportedPlatforms=android
import React from 'react';
import {View, StyleSheet, ProgressBarAndroid, Text} from 'react-native';

const App = () => {
  return (
    <View style={styles.container}>
      <View style={styles.example}>
        <Text>Circle Progress Indicator</Text>
        <ProgressBarAndroid />
      </View>
      <View style={styles.example}>
        <Text>Horizontal Progress Indicator</Text>
        <ProgressBarAndroid styleAttr="Horizontal" />
      </View>
      <View style={styles.example}>
        <Text>Colored Progress Indicator</Text>
        <ProgressBarAndroid styleAttr="Horizontal" color="#2196F3" />
      </View>
      <View style={styles.example}>
        <Text>Fixed Progress Value</Text>
        <ProgressBarAndroid
          styleAttr="Horizontal"
          indeterminate={false}
          progress={0.5}
        />
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  example: {
    marginVertical: 24,
  },
});

export default App;
```

---

# 參考文獻

## 屬性

繼承自[View屬性](view.md#props)。

### `animating`

是否顯示進度條（預設為true顯示，false隱藏）。

| Type | Required |
| ---- | -------- |
| bool | No       |

---

### `color`

進度條的顏色。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `indeterminate`

是否顯示不確定進度。注意：僅當styleAttr為Horizontal時可設為false，且需搭配`progress`數值使用。

| Type              | Required |
| ----------------- | -------- |
| indeterminateType | No       |

---

### `progress`

進度值（範圍0到1之間）。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `styleAttr`

進度條的樣式，可選值為：

- Horizontal（水平）
- Normal（預設）
- Small（小型）
- Large（大型）
- Inverse（反色）
- SmallInverse（小型反色）
- LargeInverse（大型反色）

| Type                                                                                      | Required |
| ----------------------------------------------------------------------------------------- | -------- |
| enum('Horizontal', 'Normal', 'Small', 'Large', 'Inverse', 'SmallInverse', 'LargeInverse') | No       |

---

### `testID`

用於在端到端測試中定位此視圖。

| Type   | Required |
| ------ | -------- |
| string | No       |