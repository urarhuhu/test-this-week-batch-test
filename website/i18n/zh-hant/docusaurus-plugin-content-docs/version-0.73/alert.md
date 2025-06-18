---
id: alert
title: Alert
---

顯示一個圓形的載入指示器。

觸發一個帶有指定標題和訊息的警示對話框。

可選擇性提供按鈕列表。點擊任何按鈕將觸發相應的 onPress 回調並關閉警示框。預設情況下，僅會顯示一個「確定」按鈕。

## 範例

```SnackPlayer name=Alert%20Example&supportedPlatforms=ios,android
import React from 'react';
import {View, StyleSheet, Button, Alert} from 'react-native';

const App = () => {
  const createTwoButtonAlert = () =>
    Alert.alert('Alert Title', 'My Alert Msg', [
      {
        text: 'Cancel',
        onPress: () => console.log('Cancel Pressed'),
        style: 'cancel',
      },
      {text: 'OK', onPress: () => console.log('OK Pressed')},
    ]);

  const createThreeButtonAlert = () =>
    Alert.alert('Alert Title', 'My Alert Msg', [
      {
        text: 'Ask me later',
        onPress: () => console.log('Ask me later pressed'),
      },
      {
        text: 'Cancel',
        onPress: () => console.log('Cancel Pressed'),
        style: 'cancel',
      },
      {text: 'OK', onPress: () => console.log('OK Pressed')},
    ]);

  return (
    <View style={styles.container}>
      <Button title={'2-Button Alert'} onPress={createTwoButtonAlert} />
      <Button title={'3-Button Alert'} onPress={createThreeButtonAlert} />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-around',
    alignItems: 'center',
  },
});

export default App;
```

## iOS

在 iOS 上，您可以指定任意數量的按鈕。每個按鈕可選擇性地設定樣式或強調顯示，可用選項由 [AlertButtonStyle](#alertbuttonstyle-ios) 枚舉和 [AlertButton](alert#alertbutton) 上的 `isPreferred` 欄位表示。

## Android

在 Android 上最多可指定三個按鈕。Android 有中性、負面和正面按鈕的概念：

- 如果指定一個按鈕，它將是「正面」按鈕（例如「確定」）
- 兩個按鈕表示「負面」、「正面」（例如「取消」、「確定」）
- 三個按鈕表示「中性」、「負面」、「正面」（例如「稍後」、「取消」、「確定」）

在 Android 上，可通過點擊警示框外部來關閉警示。預設情況下此功能是關閉的，可通過提供可選的 [AlertOptions](alert#alertoptions) 參數並將 cancelable 屬性設為 `true`（即 `{cancelable: true}`）來啟用。

取消事件可通過在 `options` 參數中提供 `onDismiss` 回調屬性來處理。

### 範例 <div class="label android">Android</div>

```SnackPlayer name=Alert%20Android%20Dissmissable%20Example&supportedPlatforms=android
import React from 'react';
import {View, StyleSheet, Button, Alert} from 'react-native';

const showAlert = () =>
  Alert.alert(
    'Alert Title',
    'My Alert Msg',
    [
      {
        text: 'Cancel',
        onPress: () => Alert.alert('Cancel Pressed'),
        style: 'cancel',
      },
    ],
    {
      cancelable: true,
      onDismiss: () =>
        Alert.alert(
          'This alert was dismissed by tapping outside of the alert dialog.',
        ),
    },
  );

const App = () => (
  <View style={styles.container}>
    <Button title="Show alert" onPress={showAlert} />
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

---

# 參考

## 方法

### `alert()`

```tsx
static alert (
  title: string,
  message?: string,
  buttons?: AlertButton[],
  options?: AlertOptions,
);
```

**參數：**

| Name                                                   | Type                               | Description                                                             |
| ------------------------------------------------------ | ---------------------------------- | ----------------------------------------------------------------------- |
| title <div class="label basic required">Required</div> | string                             | The dialog's title. Passing `null` or empty string will hide the title. |
| message                                                | string                             | An optional message that appears below the dialog's title.              |
| buttons                                                | [AlertButton](alert#alertbutton)[] | An optional array containing buttons configuration.                     |
| options                                                | [AlertOptions](alert#alertoptions) | An optional Alert configuration.                                        |

---

### `prompt()` <div class="label ios">iOS</div>

```tsx
static prompt: (
  title: string,
  message?: string,
  callbackOrButtons?: ((text: string) => void) | AlertButton[],
  type?: AlertType,
  defaultValue?: string,
  keyboardType?: string,
);
```

建立並顯示一個以警示框形式輸入文字的提示。

**參數：**

| Name                                                   | Type                                            | Description                                                                                                                                                                                           |
| ------------------------------------------------------ | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| title <div class="label basic required">Required</div> | string                                          | The dialog's title.                                                                                                                                                                                   |
| message                                                | string                                          | An optional message that appears above the text input.                                                                                                                                                |
| callbackOrButtons                                      | function<hr/>[AlertButton](alert#alertButton)[] | If passed a function, it will be called with the prompt's value<br/>`(text: string) => void`, when the user taps 'OK'.<hr/>If passed an array, buttons will be configured based on the array content. |
| type                                                   | [AlertType](alert#alerttype-ios)                | This configures the text input.                                                                                                                                                                       |
| defaultValue                                           | string                                          | The default text in text input.                                                                                                                                                                       |
| keyboardType                                           | string                                          | The keyboard type of first text field (if exists). One of TextInput [keyboardTypes](textinput#keyboardtype).                                                                                          |
| options                                                | [AlertOptions](alert#alertoptions)              | An optional Alert configuration.                                                                                                                                                                      |

---

## 類型定義

### AlertButtonStyle <div class="label ios">iOS</div>

iOS 警示按鈕樣式。

| Type |
| ---- |
| enum |

**常數：**

| Value           | Description               |
| --------------- | ------------------------- |
| `'default'`     | Default button style.     |
| `'cancel'`      | Cancel button style.      |
| `'destructive'` | Destructive button style. |

---

### AlertType <div class="label ios">iOS</div>

iOS 警示類型。

| Type |
| ---- |
| enum |

**常數：**

| Value              | Description                  |
| ------------------ | ---------------------------- |
| `'default'`        | Default alert with no inputs |
| `'plain-text'`     | Plain text input alert       |
| `'secure-text'`    | Secure text input alert      |
| `'login-password'` | Login and password alert     |

---

### AlertButton

描述警示框中按鈕配置的物件。

| Type             |
| ---------------- |
| array of objects |

**物件屬性：**

| Name                                         | Type                                           | Description                                                                    |
| -------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------ |
| text                                         | string                                         | Button label.                                                                  |
| onPress                                      | function                                       | Callback function when button is pressed.                                      |
| style <div class="label ios">iOS</div>       | [AlertButtonStyle](alert#alertbuttonstyle-ios) | Button style, on Android this property will be ignored.                        |
| isPreferred <div class="label ios">iOS</div> | boolean                                        | Whether button should be emphasized, on Android this property will be ignored. |

---

### AlertOptions

| Type   |
| ------ |
| object |

**屬性：**

| Name                                                | Type     | Description                                                                                                               |
| --------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------- |
| cancelable <div class="label android">Android</div> | boolean  | Defines if alert can be dismissed by tapping outside of the alert box.                                                    |
| userInterfaceStyle <div class="label ios">iOS</div> | string   | The interface style used for the alert, can be set to `light` or `dark`, otherwise the default system style will be used. |
| onDismiss <div class="label android">Android</div>  | function | Callback function fired when alert has been dismissed.                                                                    |