---
id: drawerlayoutandroid
title: DrawerLayoutAndroid
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

此 React 元件封裝了平台專用的 `DrawerLayout`（僅限 Android）。抽屜（通常用於導航）透過 `renderNavigationView` 渲染，其子元件則作為主視圖（放置內容之處）。導航視圖初始狀態不會顯示在螢幕上，但可從 `drawerPosition` 屬性指定的視窗側邊拉出，其寬度可由 `drawerWidth` 屬性設定。

## 範例

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=DrawerLayoutAndroid%20Component%20Example&supportedPlatforms=android&ext=js
import React, {useRef, useState} from 'react';
import {Button, DrawerLayoutAndroid, Text, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  const drawer = useRef(null);
  const [drawerPosition, setDrawerPosition] = useState('left');
  const changeDrawerPosition = () => {
    if (drawerPosition === 'left') {
      setDrawerPosition('right');
    } else {
      setDrawerPosition('left');
    }
  };

  const navigationView = () => (
    <SafeAreaView style={[styles.container, styles.navigationContainer]}>
      <Text style={styles.paragraph}>I'm in the Drawer!</Text>
      <Button
        title="Close drawer"
        onPress={() => drawer.current.closeDrawer()}
      />
    </SafeAreaView>
  );

  return (
    <SafeAreaProvider>
      <DrawerLayoutAndroid
        ref={drawer}
        drawerWidth={300}
        drawerPosition={drawerPosition}
        renderNavigationView={navigationView}>
        <SafeAreaView style={styles.container}>
          <Text style={styles.paragraph}>Drawer on the {drawerPosition}!</Text>
          <Button
            title="Change Drawer Position"
            onPress={() => changeDrawerPosition()}
          />
          <Text style={styles.paragraph}>
            Swipe from the side or press button below to see it!
          </Text>
          <Button
            title="Open drawer"
            onPress={() => drawer.current.openDrawer()}
          />
        </SafeAreaView>
      </DrawerLayoutAndroid>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingHorizontal: 16,
  },
  navigationContainer: {
    backgroundColor: '#ecf0f1',
  },
  paragraph: {
    padding: 16,
    fontSize: 15,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=DrawerLayoutAndroid%20Component%20Example&supportedPlatforms=android&ext=tsx
import React, {useRef, useState} from 'react';
import {
  Button,
  DrawerLayoutAndroid,
  Text,
  StyleSheet,
  View,
} from 'react-native';

const App = () => {
  const drawer = useRef<DrawerLayoutAndroid>(null);
  const [drawerPosition, setDrawerPosition] = useState<'left' | 'right'>(
    'left',
  );
  const changeDrawerPosition = () => {
    if (drawerPosition === 'left') {
      setDrawerPosition('right');
    } else {
      setDrawerPosition('left');
    }
  };

  const navigationView = () => (
    <View style={[styles.container, styles.navigationContainer]}>
      <Text style={styles.paragraph}>I'm in the Drawer!</Text>
      <Button
        title="Close drawer"
        onPress={() => drawer.current?.closeDrawer()}
      />
    </View>
  );

  return (
    <DrawerLayoutAndroid
      ref={drawer}
      drawerWidth={300}
      drawerPosition={drawerPosition}
      renderNavigationView={navigationView}>
      <View style={styles.container}>
        <Text style={styles.paragraph}>Drawer on the {drawerPosition}!</Text>
        <Button
          title="Change Drawer Position"
          onPress={() => changeDrawerPosition()}
        />
        <Text style={styles.paragraph}>
          Swipe from the side or press button below to see it!
        </Text>
        <Button
          title="Open drawer"
          onPress={() => drawer.current?.openDrawer()}
        />
      </View>
    </DrawerLayoutAndroid>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 16,
  },
  navigationContainer: {
    backgroundColor: '#ecf0f1',
  },
  paragraph: {
    padding: 16,
    fontSize: 15,
    textAlign: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

---

# 參考文獻

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view.md#props)。

---

### `drawerBackgroundColor`

指定抽屜的背景顏色。預設值為 `white`。若需設定抽屜透明度，請使用 rgba。範例：

```tsx
return (
  <DrawerLayoutAndroid drawerBackgroundColor="rgba(0,0,0,0.5)" />
);
```

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

---

### `drawerLockMode`

指定抽屜的鎖定模式。抽屜可鎖定於三種狀態：

- 未鎖定（預設值），抽屜會回應觸控手勢（開啟/關閉）。
- 鎖定關閉，抽屜將保持關閉且不回應手勢。
- 鎖定開啟，抽屜將保持開啟且不回應手勢。抽屜仍可透過程式控制開啟/關閉（`openDrawer`/`closeDrawer`）。

| Type                                             | Required |
| ------------------------------------------------ | -------- |
| enum('unlocked', 'locked-closed', 'locked-open') | No       |

---

### `drawerPosition`

指定抽屜從螢幕哪一側滑入。預設值為 `left`。

| Type                  | Required |
| --------------------- | -------- |
| enum('left', 'right') | No       |

---

### `drawerWidth`

指定抽屜的寬度，更精確地說是指可從視窗邊緣拉入的視圖寬度。

| Type   | Required |
| ------ | -------- |
| number | No       |

---

### `keyboardDismissMode`

決定鍵盤是否因拖曳動作而關閉。

- 'none'（預設值），拖曳時不會關閉鍵盤。
- 'on-drag'，開始拖曳時即關閉鍵盤。

| Type                    | Required |
| ----------------------- | -------- |
| enum('none', 'on-drag') | No       |

---

### `onDrawerClose`

當導航視圖關閉時呼叫的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerOpen`

當導航視圖開啟時呼叫的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerSlide`

當與導航視圖發生互動時呼叫的函式。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `onDrawerStateChanged`

當抽屜狀態變更時呼叫的函式。抽屜可能處於三種狀態：

- idle（閒置），表示當前無任何與導航視圖的互動
- dragging（拖曳中），表示正在與導航視圖互動
- settling（沉降中），表示已完成與導航視圖的互動，且導航視圖正在完成其關閉或開啟的動畫

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `renderNavigationView`

將渲染於螢幕側邊並可拉入的導航視圖。

| Type     | Required |
| -------- | -------- |
| function | Yes      |

---

### `statusBarBackgroundColor`

讓抽屜佔據整個螢幕並繪製狀態列背景，使其能夠在狀態列上方開啟。此屬性僅在 API 21+ 以上版本有效。

| Type               | Required |
| ------------------ | -------- |
| [color](colors.md) | No       |

## 方法

### `closeDrawer()`

```tsx
closeDrawer();
```

關閉抽屜。

---

### `openDrawer()`

```tsx
openDrawer();
```

開啟抽屜。