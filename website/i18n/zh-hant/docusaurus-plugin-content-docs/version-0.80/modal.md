---
id: modal
title: Modal
---

Modal 元件是將內容顯示在封裝視圖上方的基礎方式。

## 範例

```SnackPlayer name=Modal&supportedPlatforms=android,ios
import React, {useState} from 'react';
import {Alert, Modal, StyleSheet, Text, Pressable, View} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const [modalVisible, setModalVisible] = useState(false);
  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.centeredView}>
        <Modal
          animationType="slide"
          transparent={true}
          visible={modalVisible}
          onRequestClose={() => {
            Alert.alert('Modal has been closed.');
            setModalVisible(!modalVisible);
          }}>
          <View style={styles.centeredView}>
            <View style={styles.modalView}>
              <Text style={styles.modalText}>Hello World!</Text>
              <Pressable
                style={[styles.button, styles.buttonClose]}
                onPress={() => setModalVisible(!modalVisible)}>
                <Text style={styles.textStyle}>Hide Modal</Text>
              </Pressable>
            </View>
          </View>
        </Modal>
        <Pressable
          style={[styles.button, styles.buttonOpen]}
          onPress={() => setModalVisible(true)}>
          <Text style={styles.textStyle}>Show Modal</Text>
        </Pressable>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  centeredView: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  modalView: {
    margin: 20,
    backgroundColor: 'white',
    borderRadius: 20,
    padding: 35,
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: {
      width: 0,
      height: 2,
    },
    shadowOpacity: 0.25,
    shadowRadius: 4,
    elevation: 5,
  },
  button: {
    borderRadius: 20,
    padding: 10,
    elevation: 2,
  },
  buttonOpen: {
    backgroundColor: '#F194FF',
  },
  buttonClose: {
    backgroundColor: '#2196F3',
  },
  textStyle: {
    color: 'white',
    fontWeight: 'bold',
    textAlign: 'center',
  },
  modalText: {
    marginBottom: 15,
    textAlign: 'center',
  },
});

export default App;
```

---

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### `animated`

> **已棄用。** 請改用 [`animationType`](modal.md#animationtype) 屬性。

---

### `animationType`

`animationType` 屬性控制模態框的動畫效果。

可能的值：

- `slide` 從底部滑入
- `fade` 淡入視圖
- `none` 無動畫顯示

| Type                                | Default |
| ----------------------------------- | ------- |
| enum(`'none'`, `'slide'`, `'fade'`) | `none`  |

---

### `backdropColor`

模態框的背景顏色（或模態框容器的背景色）。若未提供且 `transparent` 為 `false`，則預設為 `white`。若 `transparent` 為 `true`，則此屬性會被忽略。

| Type            | Default |
| --------------- | ------- |
| [color](colors) | white   |

---

### `hardwareAccelerated` <div class="label android">Android</div>

`hardwareAccelerated` 屬性控制是否強制底層視窗使用硬體加速。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `navigationBarTranslucent` <div class="label android">Android</div>

`navigationBarTranslucent` 屬性決定模態框是否應延伸至系統導覽列下方。但需同時將 `statusBarTranslucent` 設為 `true` 才能使導覽列透明。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onDismiss` <div class="label ios">iOS</div>

`onDismiss` 屬性允許傳遞一個函式，該函式會在模態框關閉後被呼叫。

| Type     |
| -------- |
| function |

---

### `onOrientationChange` <div class="label ios">iOS</div>

`onOrientationChange` 回調會在模態框顯示期間發生螢幕方向變更時被呼叫。提供的方向僅為 'portrait' 或 'landscape'。此回調也會在初始渲染時呼叫，無論當前方向為何。

| Type     |
| -------- |
| function |

---

### `onRequestClose`

當用戶在 Android 上點擊硬體返回按鈕或在 Apple TV 上點擊選單按鈕時，會呼叫 `onRequestClose` 回調。由於此為必要屬性，請注意只要模態框處於開啟狀態，`BackHandler` 事件將不會被觸發。
在 iOS 上，當使用拖動手勢關閉模態框且 `presentationStyle` 為 `pageSheet` 或 `formSheet` 時，會呼叫此回調。

| Type                                                                                                                                                                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| function <div className="label basic required">Required</div><div className="label android">Android</div><div className="label tv">TV</div><hr />function <div className="label ios">iOS</div> |

---

### `onShow`

`onShow` 屬性允許傳遞一個函式，該函式會在模態框顯示後被呼叫。

| Type     |
| -------- |
| function |

---

### `presentationStyle` <div class="label ios">iOS</div>

`presentationStyle` 屬性控制模態框的顯示方式（通常在較大裝置如 iPad 或加大版 iPhone 上）。詳情請參閱 https://developer.apple.com/reference/uikit/uimodalpresentationstyle。

可能的值：

- `fullScreen` 完全覆蓋整個螢幕
- `pageSheet` 以直向寬度居中顯示（僅適用於較大裝置）
- `formSheet` 以窄幅寬度居中顯示（僅適用於較大裝置）
- `overFullScreen` 完全覆蓋螢幕但允許透明背景

| Type                                                                   | Default                                                                             |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| enum(`'fullScreen'`, `'pageSheet'`, `'formSheet'`, `'overFullScreen'`) | `fullScreen` if `transparent={false}`<hr />`overFullScreen` if `transparent={true}` |

---

### `statusBarTranslucent` <div class="label android">Android</div>

`statusBarTranslucent` 屬性決定模態視窗是否應延伸至系統狀態欄下方。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `supportedOrientations` <div class="label ios">iOS</div>

`supportedOrientations` 屬性允許模態視窗旋轉至指定的任何方向。在 iOS 上，模態視窗仍受應用程式 Info.plist 中 UISupportedInterfaceOrientations 欄位的限制。

> 當使用 `pageSheet` 或 `formSheet` 的 `presentationStyle` 時，此屬性將被 iOS 忽略。

| Type                                                                                                           | Default        |
| -------------------------------------------------------------------------------------------------------------- | -------------- |
| array of enums(`'portrait'`, `'portrait-upside-down'`, `'landscape'`, `'landscape-left'`, `'landscape-right'`) | `['portrait']` |

---

### `transparent`

`transparent` 屬性決定模態視窗是否填滿整個視圖。設為 `true` 時，模態視窗將渲染於透明背景之上。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `visible`

`visible` 屬性控制模態視窗的可見狀態。

| Type | Default |
| ---- | ------- |
| bool | `true`  |