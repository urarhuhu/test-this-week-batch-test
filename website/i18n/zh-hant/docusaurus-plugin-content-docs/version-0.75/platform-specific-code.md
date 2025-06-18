---
id: platform-specific-code
title: Platform-Specific Code
---

當開發跨平台應用程式時，你會希望盡可能重複使用程式碼。某些情況下可能需要針對不同平台撰寫不同程式碼，例如你可能需要為 Android 和 iOS 分別實作不同的視覺元件。

React Native 提供兩種方式來組織程式碼並依平台進行區分：

- 使用 [`Platform` 模組](platform-specific-code.md#platform-module)
- 使用 [平台特定副檔名](platform-specific-code.md#platform-specific-extensions)

某些元件可能具有僅在單一平台有效的屬性。所有這類屬性都會標註 `@platform` 並在官方網站上顯示小徽章。

## Platform 模組

React Native 提供了一個能檢測應用程式運行平台的模組。你可以利用此檢測邏輯來實作平台特定程式碼。當只有元件的小部分需要區分平台時，適合使用此方式。

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  height: Platform.OS === 'ios' ? 200 : 100,
});
```

`Platform.OS` 在 iOS 上會回傳 `ios`，在 Android 上則會回傳 `android`。

另提供 `Platform.select` 方法，該方法接收一個鍵值可為 `'ios' | 'android' | 'native' | 'default'` 的物件，並回傳最符合當前運行平台的值。具體來說，在手機上運行時會優先採用 `ios` 和 `android` 鍵值；若未指定，則會使用 `native` 鍵值，最後才是 `default` 鍵值。

```tsx
import {Platform, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  container: {
    flex: 1,
    ...Platform.select({
      ios: {
        backgroundColor: 'red',
      },
      android: {
        backgroundColor: 'green',
      },
      default: {
        // other platforms, web for example
        backgroundColor: 'blue',
      },
    }),
  },
});
```

這會讓容器在所有平台上都具有 `flex: 1` 樣式，在 iOS 上顯示紅色背景，Android 上顯示綠色背景，其他平台則顯示藍色背景。

由於該方法接受 `any` 型別值，你也可以用它來回傳平台特定元件，如下所示：

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

### 檢測 Android 版本 <div class="label android" title="此章節與 Android 平台相關">Android</div>

在 Android 上，`Platform` 模組還可用於檢測應用程式運行的 Android 平台版本：

```tsx
import {Platform} from 'react-native';

if (Platform.Version === 25) {
  console.log('Running on Nougat!');
}
```

**注意**：`Version` 對應的是 Android API 版本號而非作業系統版本號。版本對照表請參考 [Android 版本歷史](https://en.wikipedia.org/wiki/Android_version_history#Overview)。

### 檢測 iOS 版本 <div class="label ios" title="此章節與 iOS 平台相關">iOS</div>

在 iOS 上，`Version` 是透過 `-[UIDevice systemVersion]` 取得的字串，表示當前作業系統版本（例如 "10.3"）。若要檢測 iOS 的主要版本號：

```tsx
import {Platform} from 'react-native';

const majorVersionIOS = parseInt(Platform.Version, 10);
if (majorVersionIOS <= 9) {
  console.log('Work around a change in behavior');
}
```

## 平台特定副檔名

當平台特定程式碼較為複雜時，建議將程式碼拆分至獨立檔案。React Native 會自動識別具有 `.ios.` 或 `.android.` 副檔名的檔案，並在需要時載入對應平台的檔案。

例如，假設你的專案中有以下檔案：

```shell
BigButton.ios.js
BigButton.android.js
```

接著你可以這樣引入元件：

```tsx
import BigButton from './BigButton';
```

React Native 會根據運行平台自動選擇正確的檔案。

## 原生特定副檔名（例如與 NodeJS 和 Web 共用程式碼）

當模組需要在 NodeJS/Web 與 React Native 之間共用且無 Android/iOS 差異時，可使用 `.native.js` 副檔名。這對於 React Native 與 ReactJS 共用程式碼的專案特別有用。

舉例來說，假設您的專案中有以下檔案：

```shell
Container.js # picked up by webpack, Rollup or any other Web bundler
Container.native.js # picked up by the React Native bundler for both Android and iOS (Metro)
```

您仍可不使用 `.native` 副檔名進行導入，如下所示：

```tsx
import Container from './Container';
```

**專業建議：** 請設定您的網頁打包工具忽略 `.native.js` 副檔名，以避免在生產環境打包時包含未使用的程式碼，從而減少最終打包體積。