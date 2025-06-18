---
id: switch
title: Switch
---

渲染一個布林值輸入元件。

這是一個受控元件，需要透過`onValueChange`回調函數來更新`value`屬性值以反映使用者操作。若未更新`value`屬性，元件將持續顯示初始設定的`value`值而非使用者操作後的預期結果。

## 範例

```SnackPlayer name=Switch&supportedPlatforms=android,ios
import React, {useState} from 'react';
import {Switch, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [isEnabled, setIsEnabled] = useState(false);
  const toggleSwitch = () => setIsEnabled(previousState => !previousState);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Switch
          trackColor={{false: '#767577', true: '#81b0ff'}}
          thumbColor={isEnabled ? '#f5dd4b' : '#f4f3f4'}
          ios_backgroundColor="#3e3e3e"
          onValueChange={toggleSwitch}
          value={isEnabled}
        />
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

---

# 參考文件

## 屬性

### [View 屬性](view.md#props)

繼承 [View 元件所有屬性](view.md#props)。

---

### `disabled`

設為`true`時，使用者將無法切換開關狀態。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `ios_backgroundColor` <div class="label ios">iOS</div>

iOS專用，可自訂背景色。此背景色會在開關值為`false`或開關被禁用時顯示（此時開關呈半透明狀）。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `onChange`

當使用者嘗試變更開關值時觸發。接收變更事件作為參數。若只需獲取新值，請改用`onValueChange`。

| Type     |
| -------- |
| function |

---

### `onValueChange`

當使用者嘗試變更開關值時觸發。接收新值作為參數。若需獲取事件物件，請改用`onChange`。

| Type     |
| -------- |
| function |

---

### `thumbColor`

開關滑塊的顏色。在iOS平台設定此屬性會使滑塊失去陰影效果。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `trackColor`

自訂開關軌道的顏色。

_iOS專屬行為_：當開關值為`false`時，軌道會向邊框收縮。若要調整收縮後露出的背景色，請使用[`ios_backgroundColor`](switch.md#ios_backgroundColor)屬性。

| Type                                                         |
| ------------------------------------------------------------ |
| `md object: {false: [color](colors), true: [color](colors)}` |

---

### `value`

開關的當前值。設為`true`時開關將呈開啟狀態，預設值為`false`。

| Type |
| ---- |
| bool |