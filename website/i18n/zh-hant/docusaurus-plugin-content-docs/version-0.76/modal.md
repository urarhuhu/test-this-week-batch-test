---
id: modal
title: Modal
---

Modal 元件是用於在當前視圖上方呈現內容的基本方式。

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

# 參考文獻

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
- `none` 無動畫出現

| Type                                | Default |
| ----------------------------------- | ------- |
| enum(`'none'`, `'slide'`, `'fade'`) | `none`  |

---

### `hardwareAccelerated` <div class="label android">Android</div>

`hardwareAccelerated` 屬性控制是否強制底層視窗使用硬體加速。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `onDismiss` <div class="label ios">iOS</div>

`onDismiss` 屬性允許傳遞一個函數，該函數將在模態框被關閉後調用。

| Type     |
| -------- |
| function |

---

### `onOrientationChange` <div class="label ios">iOS</div>

`onOrientationChange` 回調函數在模態框顯示期間方向改變時被調用。提供的方向僅為 'portrait' 或 'landscape'。此回調也會在初始渲染時調用，無論當前方向為何。

| Type     |
| -------- |
| function |

---

### `onRequestClose`

`onRequestClose` 回調函數在使用者點擊 Android 的硬體返回按鈕或 Apple TV 的選單按鈕時被調用。由於此屬性是必需的，請注意只要模態框處於開啟狀態，`BackHandler` 事件將不會被觸發。
在 iOS 上，當 `presentationStyle` 為 `pageSheet` 或 `formSheet` 時，此回調會在通過拖動手勢關閉模態框時被調用。

| Type                                                                                                                                                                                           |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| function <div className="label basic required">Required</div><div className="label android">Android</div><div className="label tv">TV</div><hr />function <div className="label ios">iOS</div> |

---

### `onShow`

`onShow` 屬性允許傳遞一個函數，該函數將在模態框顯示後被調用。

| Type     |
| -------- |
| function |

---

### `presentationStyle` <div class="label ios">iOS</div>

`presentationStyle` 屬性控制模態框的顯示方式（通常在較大設備如 iPad 或加大版 iPhone 上）。詳情請參閱 https://developer.apple.com/reference/uikit/uimodalpresentationstyle。

可能的值：

- `fullScreen` 完全覆蓋螢幕
- `pageSheet` 居中覆蓋豎向寬度視圖（僅在較大設備上）
- `formSheet` 居中覆蓋窄寬度視圖（僅在較大設備上）
- `overFullScreen` 完全覆蓋螢幕，但允許透明

| Type                                                                   | Default                                                                             |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| enum(`'fullScreen'`, `'pageSheet'`, `'formSheet'`, `'overFullScreen'`) | `fullScreen` if `transparent={false}`<hr />`overFullScreen` if `transparent={true}` |

---

### `statusBarTranslucent` <div class="label android">Android</div>

`statusBarTranslucent` 屬性決定模態框是否應該延伸至系統狀態欄下方。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `supportedOrientations` <div class="label ios">iOS</div>

`supportedOrientations` 屬性允許模態視窗旋轉至指定的任何方向。在 iOS 上，模態視窗仍會受到應用程式 Info.plist 中 UISupportedInterfaceOrientations 欄位所指定方向的限制。

> 當使用 `pageSheet` 或 `formSheet` 的 `presentationStyle` 時，iOS 將忽略此屬性。

| Type                                                                                                           | Default        |
| -------------------------------------------------------------------------------------------------------------- | -------------- |
| array of enums(`'portrait'`, `'portrait-upside-down'`, `'landscape'`, `'landscape-left'`, `'landscape-right'`) | `['portrait']` |

---

### `transparent`

`transparent` 屬性決定模態視窗是否會填滿整個視圖。設為 `true` 時，模態視窗將在透明背景上渲染。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `visible`

`visible` 屬性決定模態視窗是否可見。

| Type | Default |
| ---- | ------- |
| bool | `true`  |