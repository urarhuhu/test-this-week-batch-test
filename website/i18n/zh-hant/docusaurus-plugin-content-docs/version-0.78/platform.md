---
id: platform
title: Platform
---

## 範例

```SnackPlayer name=Platform%20API%20Example&supportedPlatforms=ios,android
import React from 'react';
import {Platform, StyleSheet, Text, ScrollView} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  return (
    <SafeAreaProvider>
      <SafeAreaView>
        <ScrollView contentContainerStyle={styles.container}>
          <Text>OS</Text>
          <Text style={styles.value}>{Platform.OS}</Text>
          <Text>OS Version</Text>
          <Text style={styles.value}>{Platform.Version}</Text>
          <Text>isTV</Text>
          <Text style={styles.value}>{Platform.isTV.toString()}</Text>
          {Platform.OS === 'ios' && (
            <>
              <Text>isPad</Text>
              <Text style={styles.value}>{Platform.isPad.toString()}</Text>
            </>
          )}
          <Text>Constants</Text>
          <Text style={styles.value}>
            {JSON.stringify(Platform.constants, null, 2)}
          </Text>
        </ScrollView>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  value: {
    fontWeight: '600',
    padding: 4,
    marginBottom: 8,
  },
});

export default App;
```

---

# 參考文獻

## 屬性

### `constants`

```tsx
static constants: PlatformConstants;
```

返回一個包含所有與平台相關的通用及特定常數的物件。

**屬性：**

| <div className="widerColumn">Name</div>                   | Type    | Optional | Description                                                                                                                                                                                       |
| --------------------------------------------------------- | ------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| isTesting                                                 | boolean | No       |                                                                                                                                                                                                   |
| reactNativeVersion                                        | object  | No       | Information about React Native version. Keys are `major`, `minor`, `patch` with optional `prerelease` and values are `number`s.                                                                   |
| Version <div className="label android">Android</div>      | number  | No       | OS version constant specific to Android.                                                                                                                                                          |
| Release <div className="label android">Android</div>      | string  | No       |                                                                                                                                                                                                   |
| Serial <div className="label android">Android</div>       | string  | No       | Hardware serial number of an Android device.                                                                                                                                                      |
| Fingerprint <div className="label android">Android</div>  | string  | No       | A string that uniquely identifies the build.                                                                                                                                                      |
| Model <div className="label android">Android</div>        | string  | No       | The end-user-visible name for the Android device.                                                                                                                                                 |
| Brand <div className="label android">Android</div>        | string  | No       | The consumer-visible brand with which the product/hardware will be associated.                                                                                                                    |
| Manufacturer <div className="label android">Android</div> | string  | No       | The manufacturer of the Android device.                                                                                                                                                           |
| ServerHost <div className="label android">Android</div>   | string  | Yes      |                                                                                                                                                                                                   |
| uiMode <div className="label android">Android</div>       | string  | No       | Possible values are: `'car'`, `'desk'`, `'normal'`,`'tv'`, `'watch'` and `'unknown'`. Read more about [Android ModeType](https://developer.android.com/reference/android/app/UiModeManager.html). |
| forceTouchAvailable <div className="label ios">iOS</div>  | boolean | No       | Indicate the availability of 3D Touch on a device.                                                                                                                                                |
| interfaceIdiom <div className="label ios">iOS</div>       | string  | No       | The interface type for the device. Read more about [UIUserInterfaceIdiom](https://developer.apple.com/documentation/uikit/uiuserinterfaceidiom).                                                  |
| osVersion <div className="label ios">iOS</div>            | string  | No       | OS version constant specific to iOS.                                                                                                                                                              |
| systemName <div className="label ios">iOS</div>           | string  | No       | OS name constant specific to iOS.                                                                                                                                                                 |

---

### `isPad` <div class="label ios">iOS</div>

```tsx
static isPad: boolean;
```

返回一個布林值，表示裝置是否為 iPad。

| Type    |
| ------- |
| boolean |

---

### `isTV`

```tsx
static isTV: boolean;
```

返回一個布林值，表示裝置是否為電視。

| Type    |
| ------- |
| boolean |

---

### `isVision`

```tsx
static isVision: boolean;
```

返回一個布林值，表示裝置是否為 Apple Vision。_若您使用 [Apple Vision Pro (專為 iPad 設計)](https://developer.apple.com/documentation/visionos/checking-whether-your-app-is-compatible-with-visionos)，`isVision` 將為 `false` 但 `isPad` 會是 `true`_

| Type    |
| ------- |
| boolean |

---

### `isTesting`

```tsx
static isTesting: boolean;
```

返回一個布林值，表示應用程式是否在開發者模式下運行且測試標記已設定。

| Type    |
| ------- |
| boolean |

---

### `OS`

```tsx
static OS: 'android' | 'ios';
```

返回代表當前作業系統的字串值。

| Type                       |
| -------------------------- |
| enum(`'android'`, `'ios'`) |

---

### `Version`

```tsx
static Version: 'number' | 'string';
```

返回作業系統的版本。

| Type                                                                                                 |
| ---------------------------------------------------------------------------------------------------- |
| number <div className="label android">Android</div><hr />string <div className="label ios">iOS</div> |

## 方法

### `select()`

```tsx
static select(config: Record<string, T>): T;
```

返回最適合當前運行平台的值。

#### 參數：

| Name   | Type   | Required | Description                   |
| ------ | ------ | -------- | ----------------------------- |
| config | object | Yes      | See config description below. |

`select` 方法會返回最適合當前運行平台的值。也就是說，如果您在手機上運行，`android` 和 `ios` 鍵將優先。如果未指定這些鍵，則會使用 `native` 鍵，然後是 `default` 鍵。

`config` 參數是一個具有以下鍵的物件：

- `android`
- `ios`
- `macos` (僅適用於使用 [react-native-macos](https://github.com/microsoft/react-native-macos) 時)
- `native` (適用於除 `web` 外的所有平台的後備選項)
- `web` (僅適用於使用 [react-native-web](https://github.com/necolas/react-native-web) 時)
- `windows` (僅適用於使用 [react-native-windows](https://github.com/microsoft/react-native-windows) 時)
- `default` (適用於所有平台的後備選項)

**使用範例：**

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      android: {
        backgroundColor: 'green',
      },
      ios: {
        backgroundColor: 'red',
      },
      default: {
        // other platforms, web for example
        backgroundColor: 'blue',
      },
    }),
  },
});
```

這將導致容器在所有平台上具有 `flex: 1`，在 Android 上具有綠色背景色，在 iOS 上具有紅色背景色，在其他平台上具有藍色背景色。

由於相應平台鍵的值可以是 `any` 類型，[`select`](platform.md#select) 方法也可用於返回平台特定的組件，如下所示：

```tsx
const Component = Platform.select({
  ios: () => require('ComponentIOS'),
  android: () => require('ComponentAndroid'),
})();

<Component />;
```

```tsx
const Component = Platform.select({
  native: () => require('ComponentForNative'),
  default: () => require('ComponentForWeb'),
})();

<Component />;
```