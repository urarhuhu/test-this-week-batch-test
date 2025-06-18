---
id: platformcolor
title: PlatformColor
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

```js
PlatformColor(color1, [color2, ...colorN]);
```

您可以使用 `PlatformColor` 函數，通過提供對應的原生顏色字符串值來存取目標平台上的原生顏色。將字符串傳遞給 `PlatformColor` 函數，如果該顏色在該平台上存在，它將返回相應的原生顏色，您可以將其應用於應用程式的任何部分。

如果您向 `PlatformColor` 函數傳遞多個字符串值，它會將第一個值視為默認值，其餘的作為備用。

```js
PlatformColor('bogusName', 'linkColor');
```

由於原生顏色可能對主題和/或高對比度敏感，這種平台特定的邏輯也會在您的組件內部轉換。

### 支援的顏色

有關支援的系統顏色類型的完整列表，請參閱：

- Android：
  - [R.attr](https://developer.android.com/reference/android/R.attr) - `?attr` 前綴
  - [R.color](https://developer.android.com/reference/android/R.color) - `@android:color` 前綴
- iOS（Objective-C 和 Swift 表示法）：
  - [UIColor Standard Colors](https://developer.apple.com/documentation/uikit/uicolor/standard_colors)
  - [UIColor UI Element Colors](https://developer.apple.com/documentation/uikit/uicolor/ui_element_colors)

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

提供給 `PlatformColor` 函數的字符串值必須與應用程式運行的原生平台上的字符串完全匹配。為了避免運行時錯誤，該函數應通過 `Platform.OS === 'platform'` 或 `Platform.select()` 進行平台檢查，如上例所示。

> **注意：** 您可以在 [PlatformColorExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/PlatformColor/PlatformColorExample.js) 中找到一個完整的範例，展示了 `PlatformColor` 的正確使用方式。