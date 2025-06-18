---
id: shadow-props
title: Shadow Props
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Shadow%20Props&supportedPlatforms=ios&ext=js&dependencies=@react-native-community/slider
import React, {useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';
import Slider from '@react-native-community/slider';

const ShadowPropSlider = ({label, value, ...props}) => {
  return (
    <>
      <Text>
        {label} ({value.toFixed(2)})
      </Text>
      <Slider step={1} value={value} {...props} />
    </>
  );
};

const App = () => {
  const [shadowOffsetWidth, setShadowOffsetWidth] = useState(0);
  const [shadowOffsetHeight, setShadowOffsetHeight] = useState(0);
  const [shadowRadius, setShadowRadius] = useState(0);
  const [shadowOpacity, setShadowOpacity] = useState(0.1);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View
          style={[
            styles.square,
            {
              shadowOffset: {
                width: shadowOffsetWidth,
                height: -shadowOffsetHeight,
              },
              shadowOpacity,
              shadowRadius,
            },
          ]}
        />
        <View style={styles.controls}>
          <ShadowPropSlider
            label="shadowOffset - X"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetWidth}
            onValueChange={setShadowOffsetWidth}
          />
          <ShadowPropSlider
            label="shadowOffset - Y"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetHeight}
            onValueChange={setShadowOffsetHeight}
          />
          <ShadowPropSlider
            label="shadowRadius"
            minimumValue={0}
            maximumValue={100}
            value={shadowRadius}
            onValueChange={setShadowRadius}
          />
          <ShadowPropSlider
            label="shadowOpacity"
            minimumValue={0}
            maximumValue={1}
            step={0.05}
            value={shadowOpacity}
            onValueChange={val => setShadowOpacity(val)}
          />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  square: {
    alignSelf: 'center',
    backgroundColor: 'white',
    borderRadius: 4,
    height: 150,
    shadowColor: 'black',
    width: 150,
  },
  controls: {
    paddingHorizontal: 12,
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Shadow%20Props&supportedPlatforms=ios&ext=tsx&dependencies=@react-native-community/slider
import React, {useState} from 'react';
import {Text, View, StyleSheet} from 'react-native';
import Slider, {SliderProps} from '@react-native-community/slider';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

type ShadowPropSliderProps = SliderProps & {
  label: string;
};

const ShadowPropSlider = ({label, value, ...props}: ShadowPropSliderProps) => {
  return (
    <>
      <Text>
        {label} ({value?.toFixed(2)})
      </Text>
      <Slider step={1} value={value} {...props} />
    </>
  );
};

const App = () => {
  const [shadowOffsetWidth, setShadowOffsetWidth] = useState(0);
  const [shadowOffsetHeight, setShadowOffsetHeight] = useState(0);
  const [shadowRadius, setShadowRadius] = useState(0);
  const [shadowOpacity, setShadowOpacity] = useState(0.1);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <View
          style={[
            styles.square,
            {
              shadowOffset: {
                width: shadowOffsetWidth,
                height: -shadowOffsetHeight,
              },
              shadowOpacity,
              shadowRadius,
            },
          ]}
        />
        <View style={styles.controls}>
          <ShadowPropSlider
            label="shadowOffset - X"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetWidth}
            onValueChange={setShadowOffsetWidth}
          />
          <ShadowPropSlider
            label="shadowOffset - Y"
            minimumValue={-50}
            maximumValue={50}
            value={shadowOffsetHeight}
            onValueChange={setShadowOffsetHeight}
          />
          <ShadowPropSlider
            label="shadowRadius"
            minimumValue={0}
            maximumValue={100}
            value={shadowRadius}
            onValueChange={setShadowRadius}
          />
          <ShadowPropSlider
            label="shadowOpacity"
            minimumValue={0}
            maximumValue={1}
            step={0.05}
            value={shadowOpacity}
            onValueChange={val => setShadowOpacity(val)}
          />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  square: {
    alignSelf: 'center',
    backgroundColor: 'white',
    borderRadius: 4,
    height: 150,
    shadowColor: 'black',
    width: 150,
  },
  controls: {
    paddingHorizontal: 12,
  },
});

export default App;
```

</TabItem>
</Tabs>

---

# 參考文檔

React Native 提供三組陰影 API：

- `boxShadow`：一個 View 樣式屬性，完全符合 [同名網頁樣式屬性](https://developer.mozilla.org/en-US/docs/Web/CSS/box-shadow) 的規範。
- `dropShadow`：作為 [`filter`](./view-style-props#filter) View 樣式屬性的一部分提供的特定濾鏡函數。
- 多種 `shadow` 屬性（`shadowColor`、`shadowOffset`、`shadowOpacity`、`shadowRadius`）：這些直接對應平台級 API 暴露的原生屬性。

`dropShadow` 與 `boxShadow` 的主要差異如下：

- `dropShadow` 是 `filter` 的一部分，而 `boxShadow` 是獨立的樣式屬性。
- `dropShadow` 是 alpha 遮罩，因此只有具有正 alpha 值的像素才會「投射」陰影。`boxShadow` 則會圍繞元素的邊框盒投射陰影（除非設置為內陰影）。
- `dropShadow` 僅在 Android 上可用，`boxShadow` 在 iOS 和 Android 上均可使用。
- `dropShadow` 無法像 `boxShadow` 那樣設置為內陰影。
- `dropShadow` 沒有 `boxShadow` 的 `spreadDistance` 參數。

總體而言，`boxShadow` 和 `dropShadow` 的功能比 `shadow` 屬性更強大。但 `shadow` 屬性直接映射到原生平台 API，若只需基礎陰影效果，建議優先使用這些屬性。需注意只有 `shadowColor` 在 Android 和 iOS 上均有效，其他 `shadow` 屬性僅在 iOS 上可用。

## 屬性

### `boxShadow`

詳見 [View 樣式屬性](./view-style-props#boxshadow) 文檔。

### `dropShadow` <div class="label android">Android</div>

詳見 [View 樣式屬性](./view-style-props#filter) 文檔。

### `shadowColor`

設置陰影顏色。

此屬性僅在 Android API 28 及以上版本生效。若需在較低 Android API 實現類似效果，請使用 [`elevation` 屬性](view-style-props#elevation-android)。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `shadowOffset` <div class="label ios">iOS</div>

設置陰影偏移量。

| Type                                     |
| ---------------------------------------- |
| object: `{width: number,height: number}` |

---

### `shadowOpacity` <div class="label ios">iOS</div>

設置陰影透明度（會與顏色的 alpha 分量相乘）。

| Type   |
| ------ |
| number |

---

### `shadowRadius` <div class="label ios">iOS</div>

設置陰影模糊半徑。

| Type   |
| ------ |
| number |