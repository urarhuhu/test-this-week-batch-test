---
id: touchablenativefeedback
title: TouchableNativeFeedback
---

> 若您需要更全面且具未來兼容性的觸控輸入處理方案，請參閱 [Pressable](pressable.md) API。

用於讓視圖正確響應觸控操作的封裝元件（僅限Android平台）。在Android上，此元件使用原生狀態繪製物件來顯示觸控反饋。

目前僅支援單一View實例作為子節點，其實作方式是將該View替換為另一個設定了附加屬性的RCTView節點實例。

可透過`background`屬性自訂原生反饋觸控元件的背景繪製物件。

## 範例

```SnackPlayer name=TouchableNativeFeedback%20Android%20Component%20Example&supportedPlatforms=android
import React, {useState} from 'react';
import {
  Text,
  View,
  StyleSheet,
  TouchableNativeFeedback,
  StatusBar,
} from 'react-native';

const App = () => {
  const [rippleColor, setRippleColor] = useState(randomHexColor());
  const [rippleOverflow, setRippleOverflow] = useState(false);
  return (
    <View style={styles.container}>
      <TouchableNativeFeedback
        onPress={() => {
          setRippleColor(randomHexColor());
          setRippleOverflow(!rippleOverflow);
        }}
        background={TouchableNativeFeedback.Ripple(
          rippleColor,
          rippleOverflow,
        )}>
        <View style={styles.touchable}>
          <Text style={styles.text}>TouchableNativeFeedback</Text>
        </View>
      </TouchableNativeFeedback>
    </View>
  );
};

const randomHexColor = () => {
  return '#000000'.replace(/0/g, function () {
    return Math.round(Math.random() * 16).toString(16);
  });
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingTop: StatusBar.currentHeight,
    backgroundColor: '#ecf0f1',
    padding: 8,
  },
  touchable: {flex: 0.5, borderColor: 'black', borderWidth: 1},
  text: {alignSelf: 'center'},
});

export default App;
```

---

# 參考文獻

## 屬性

### [TouchableWithoutFeedback 屬性](touchablewithoutfeedback.md#props)

繼承 [TouchableWithoutFeedback 屬性](touchablewithoutfeedback.md#props)。

---

### `background`

決定用於顯示反饋的背景繪製物件類型。接收包含`type`屬性的物件，並根據`type`攜帶額外資料。建議使用靜態方法生成該字典。

| Type               |
| ------------------ |
| backgroundPropType |

---

### `useForeground`

設為true可將漣漪效果添加到視圖前景而非背景。當子視圖已有自身背景或顯示圖片等情況下，此屬性可避免漣漪效果被遮蓋。

需先檢查TouchableNativeFeedback.canUseNativeForeground()，因該功能僅在Android 6.0及以上版本可用。若在舊版嘗試使用，將觸發警告並回退至背景模式。

| Type |
| ---- |
| bool |

---

### `hasTVPreferredFocus` <div class="label android">Android</div>

電視首選焦點（參見View元件的文件說明）。

| Type |
| ---- |
| bool |

---

### `nextFocusDown` <div class="label android">Android</div>

電視向下焦點導航（參見View元件的文件說明）。

| Type   |
| ------ |
| number |

---

### `nextFocusForward` <div class="label android">Android</div>

電視向前焦點導航（參見View元件的文件說明）。

| Type   |
| ------ |
| number |

---

### `nextFocusLeft` <div class="label android">Android</div>

電視向左焦點導航（參見View元件的文件說明）。

| Type   |
| ------ |
| number |

---

### `nextFocusRight` <div class="label android">Android</div>

電視向右焦點導航（參見View元件的文件說明）。

| Type   |
| ------ |
| number |

---

### `nextFocusUp` <div class="label android">Android</div>

電視向上焦點導航（參見View元件的文件說明）。

| Type   |
| ------ |
| number |

## 方法

### `SelectableBackground()`

```tsx
static SelectableBackground(
  rippleRadius: number | null,
): ThemeAttributeBackgroundPropType;
```

建立代表Android主題預設可選元素背景的物件（`?android:attr/selectableItemBackground`）。`rippleRadius`參數控制漣漪效果半徑。

---

### `SelectableBackgroundBorderless()`

```tsx
static SelectableBackgroundBorderless(
  rippleRadius: number | null,
): ThemeAttributeBackgroundPropType;
```

建立一個物件，用於表示Android主題中無邊框可選元素的預設背景（`?android:attr/selectableItemBackgroundBorderless`）。需Android API等級21以上版本支援。`rippleRadius`參數可控制漣漪效果的半徑。

---

### `Ripple()`

```tsx
static Ripple(
  color: ColorValue,
  borderless: boolean,
  rippleRadius?: number | null,
): RippleBackgroundPropType;
```

建立一個物件，表示具有指定顏色（以字串形式）的漣漪繪製效果。若`borderless`屬性值為true，漣漪效果將渲染在視圖邊界之外（參見原生動作列按鈕的行為範例）。此背景類型需Android API等級21以上版本支援。

**參數：**

| Name         | Type    | Required | Description                                 |
| ------------ | ------- | -------- | ------------------------------------------- |
| color        | string  | Yes      | The ripple color                            |
| borderless   | boolean | Yes      | If the ripple can render outside its bounds |
| rippleRadius | ?number | No       | controls the radius of the ripple effect    |

---

### `canUseNativeForeground()`

```tsx
static canUseNativeForeground(): boolean;
```