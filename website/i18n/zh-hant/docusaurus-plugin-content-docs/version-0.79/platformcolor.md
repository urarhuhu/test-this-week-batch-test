---
id: platformcolor
title: PlatformColor
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

```js
PlatformColor(color1, [color2, ...colorN]);
```

您可以使用 `PlatformColor` 函數，透過傳入對應平台原生顏色的字串值來存取目標平台的原生色彩。將字串傳遞給 `PlatformColor` 函數後，若該平台存在此顏色，函數將返回對應的原生色彩，您可將其應用於應用程式的任何部分。

若您向 `PlatformColor` 函數傳遞多個字串值，函數會將第一個值視為預設值，其餘值作為備用選項。

```js
PlatformColor('bogusName', 'linkColor');
```

由於原生色彩可能受到主題或高對比度設定的影響，此平台特定邏輯也會在您的元件內部自動適應調整。

### 支援的色彩類型

完整支援的系統色彩類型清單請參閱：

- Android：
  - [R.attr](https://developer.android.com/reference/android/R.attr) - 需使用 `?attr` 前綴
  - [R.color](https://developer.android.com/reference/android/R.color) - 需使用 `@android:color` 前綴
- iOS（Objective-C 與 Swift 標記法）：
  - [UIColor 標準色彩](https://developer.apple.com/documentation/uikit/uicolor/standard_colors)
  - [UIColor UI 元素色彩](https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors)

#### 開發者注意事項

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["web"])}>

<TabItem value="web">

> If you’re familiar with design systems, another way of thinking about this is that `PlatformColor` lets you tap into the local design system's color tokens so your app can blend right in!

</TabItem>
</Tabs>

## 範例

```SnackPlayer name=PlatformColor%20Example&supportedPlatforms=android,ios
import React from 'react';
import {Platform, PlatformColor, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
      <Text style={styles.label}>I am a special label color!</Text>
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  label: {
    padding: 16,
    fontWeight: '800',
    ...Platform.select({
      ios: {
        color: PlatformColor('label'),
        backgroundColor: PlatformColor('systemTealColor'),
      },
      android: {
        color: PlatformColor('?android:attr/textColor'),
        backgroundColor: PlatformColor('@android:color/holo_blue_bright'),
      },
      default: {color: 'black'},
    }),
  },
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

傳遞給 `PlatformColor` 函數的字串值必須完全匹配應用程式運行時所在原生平台的對應字串。為避免運行時錯誤，應透過 `Platform.OS === '平台名稱'` 或 `Platform.select()` 進行平台檢查後再呼叫此函數，如上例所示。

> **注意：** 您可在 [PlatformColorExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/PlatformColor/PlatformColorExample.js) 找到完整範例，展示 `PlatformColor` 的正確使用方式。