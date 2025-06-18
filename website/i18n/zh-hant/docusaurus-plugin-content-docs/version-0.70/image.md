---
id: image
title: Image
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

一個用於顯示不同類型圖片的 React 元件，包括網路圖片、靜態資源、臨時本地圖片以及來自本地磁碟的圖片（例如相機膠卷）。

此範例展示如何從本地儲存空間載入並顯示圖片，同時也包含從網路載入圖片，甚至使用 `'data:'` URI 方案提供的圖片數據。

> 注意：對於網路圖片和數據圖片，您需要手動指定圖片的尺寸！

## 範例

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Function%20Component%20Example

import React from 'react';
import { View, Image, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: 50,
  },
  tinyLogo: {
    width: 50,
    height: 50,
  },
  logo: {
    width: 66,
    height: 58,
  },
});

const DisplayAnImage = () => {
  return (
    <View style={styles.container}>
      <Image
        style={styles.tinyLogo}
        source={require('@expo/snack-static/react-native-logo.png')}
      />
      <Image
        style={styles.tinyLogo}
        source={{
          uri: 'https://reactnative.dev/img/tiny_logo.png',
        }}
      />
      <Image
        style={styles.logo}
        source={{
          uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg==',
        }}
      />
    </View>
  );
}

export default DisplayAnImage;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Class%20Component%20Example

import React, { Component } from 'react';
import { AppRegistry, View, Image, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: 50,
  },
  tinyLogo: {
    width: 50,
    height: 50,
  },
  logo: {
    width: 66,
    height: 58,
  },
});

class DisplayAnImage extends Component {
  render() {
    return (
      <View style={styles.container}>
        <Image
          style={styles.tinyLogo}
          source={require('@expo/snack-static/react-native-logo.png')}
        />
        <Image
          style={styles.tinyLogo}
          source={{uri: 'https://reactnative.dev/img/tiny_logo.png'}}
        />
        <Image
          style={styles.logo}
          source={{uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg=='}}
        />
      </View>
    );
  }
}

export default DisplayAnImage;
```

</TabItem>
</Tabs>

您也可以為圖片添加 `style`：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Function%20Component%20Example

import React from 'react';
import { View, Image, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    paddingTop: 50,
  },
  stretch: {
    width: 50,
    height: 200,
    resizeMode: 'stretch',
  },
});

const DisplayAnImageWithStyle = () => {
  return (
    <View style={styles.container}>
      <Image
        style={styles.stretch}
        source={require('@expo/snack-static/react-native-logo.png')}
      />
    </View>
  );
}

export default DisplayAnImageWithStyle;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Class%20Component%20Example

import React, { Component } from 'react';
import { View, Image, StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  stretch: {
    width: 50,
    height: 200,
    resizeMode: 'stretch'
  }
});

class DisplayAnImageWithStyle extends Component {
  render() {
    return (
      <View>
        <Image
          style={styles.stretch}
          source={require('@expo/snack-static/react-native-logo.png')}
        />
      </View>
    );
  }
}

export default DisplayAnImageWithStyle;
```

</TabItem>
</Tabs>

## Android 上的 GIF 和 WebP 支援

在自行建置原生程式碼時，Android 預設不支援 GIF 和 WebP 格式。

您需要根據應用需求，在 `android/app/build.gradle` 中添加一些可選模組。

```groovy
dependencies {
  // If your app supports Android versions before Ice Cream Sandwich (API level 14)
  implementation 'com.facebook.fresco:animated-base-support:1.3.0'

  // For animated GIF support
  implementation 'com.facebook.fresco:animated-gif:2.5.0'

  // For WebP support, including animated WebP
  implementation 'com.facebook.fresco:animated-webp:2.5.0'
  implementation 'com.facebook.fresco:webpsupport:2.5.0'

  // For WebP support, without animations
  implementation 'com.facebook.fresco:webpsupport:2.5.0'
}
```

> 注意：上述列出的版本可能未及時更新。請查閱主倉庫中的 [`ReactAndroid/gradle.properties`](https://github.com/facebook/react-native/blob/0.70-stable/ReactAndroid/gradle.properties) 以查看特定標記版本中使用的 fresco 版本。

---

# 參考文獻

## 屬性

### [View 屬性](view.md#props)

繼承自 [View 屬性](view#props)。

---

### `accessible`

當設為 true 時，表示該圖片是一個無障礙元素。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `accessibilityLabel`

當使用者與圖片互動時，螢幕閱讀器會朗讀的文字。

| Type   |
| ------ |
| string |

---

### `blurRadius`

blurRadius：添加到圖片的模糊濾鏡的模糊半徑。

| Type   |
| ------ |
| number |

> 提示：在 iOS 上，您需要將 `blurRadius` 設為大於 `5` 的值。

---

### `capInsets` <div class="label ios">iOS</div>

當圖片被調整大小時，由 `capInsets` 指定的尺寸的角落將保持固定大小，但圖片的中間內容和邊框將被拉伸。這對於創建可調整大小的圓角按鈕、陰影和其他可調整大小的資源非常有用。更多資訊請參閱 [Apple 官方文檔](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/index.html#//apple_ref/occ/instm/UIImage/resizableImageWithCapInsets)。

| Type         |
| ------------ |
| [Rect](rect) |

---

### `defaultSource`

在載入圖片來源時顯示的靜態圖片。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

> **注意：** 在 Android 的除錯版本中，defaultSource 屬性會被忽略。

---

### `fadeDuration` <div class="label android">Android</div>

淡入淡出動畫的持續時間（以毫秒為單位）。

| Type   | Default |
| ------ | ------- |
| number | `300`   |

---

### `loadingIndicatorSource`

與 `source` 類似，此屬性表示用於渲染圖片載入指示器的資源。載入指示器會在圖片準備好顯示之前顯示，通常是在圖片下載完成後。

| Type                                                  |
| ----------------------------------------------------- |
| [ImageSource](image#imagesource) (`uri` only), number |

---

### `onError`

在載入錯誤時觸發。

| Type                                   |
| -------------------------------------- |
| (`{ nativeEvent: { error } }`) => void |

---

### `onLayout`

在元件掛載或佈局變更時觸發。

| Type                                                    |
| ------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)} => void` |

---

### `onLoad`

在載入成功完成時觸發。

**範例:** `onLoad={({nativeEvent: {source: {width, height}}}) => setImageRealSize({width, height})}`

| Type                                                                |
| ------------------------------------------------------------------- |
| `md ({nativeEvent: [ImageLoadEvent](image#imageloadevent)} => void` |

---

### `onLoadEnd`

在載入成功或失敗時觸發。

| Type       |
| ---------- |
| () => void |

---

### `onLoadStart`

在載入開始時觸發。

**範例:** `onLoadStart={() => this.setState({loading: true})}`

| Type       |
| ---------- |
| () => void |

---

### `onPartialLoad` <div class="label ios">iOS</div>

當圖像部分載入完成時觸發。「部分載入」的定義取決於載入器，通常用於漸進式JPEG載入。

| Type       |
| ---------- |
| () => void |

---

### `onProgress`

在下載進度更新時觸發。

| Type                                           |
| ---------------------------------------------- |
| (`{ nativeEvent: { loaded, total } }`) => void |

---

### `progressiveRenderingEnabled` <div class="label android">Android</div>

設為 `true` 時啟用漸進式JPEG串流 - 詳見 https://frescolib.org/docs/progressive-jpegs。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `resizeMethod` <div class="label android">Android</div>

當圖像尺寸與圖像視圖尺寸不符時，決定調整圖像大小的機制。預設為 `auto`。

- `auto`: 根據啟發式方法在 `resize` 和 `scale` 之間選擇。

- `resize`: 一種軟體操作，在解碼前改變記憶體中的編碼圖像。當圖像遠大於視圖時應使用此方法而非 `scale`。

- `scale`: 圖像會被縮小或放大繪製。與 `resize` 相比，`scale` 更快（通常由硬體加速）且產生更高品質的圖像。若圖像小於視圖應使用此方法；若圖像略大於視圖也可使用。

關於 `resize` 和 `scale` 的更多細節請參閱 http://frescolib.org/docs/resizing。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'resize'`, `'scale'`) | `'auto'` |

---

### `resizeMode`

決定當框架與原始圖像尺寸不匹配時如何調整圖像大小。預設為 `cover`。

- `cover`: 均勻縮放圖片（保持圖片長寬比），使得：
  
  - 圖片的兩個維度（寬度和高度）都將等於或大於視圖的相應維度（減去內邊距）
  - 縮放後圖片的至少一個維度將等於視圖的相應維度（減去內邊距）

- `contain`: 均勻縮放圖片（保持圖片長寬比），使得圖片的兩個維度（寬度和高度）都將等於或小於視圖的相應維度（減去內邊距）。

- `stretch`: 獨立縮放寬度和高度，這可能會改變原始圖片的長寬比。

- `repeat`: 重複圖片以覆蓋視圖的框架。圖片將保持其大小和長寬比，除非它大於視圖，在這種情況下它將被均勻縮小以適應視圖。

- `center`: 在視圖的兩個維度上居中圖片。如果圖片大於視圖，則均勻縮小以使其適應視圖。

| Type                                                              | Default   |
| ----------------------------------------------------------------- | --------- |
| enum(`'cover'`, `'contain'`, `'stretch'`, `'repeat'`, `'center'`) | `'cover'` |

---

### `source`

圖片來源（可以是遠程URL或本地文件資源）。

此屬性也可以包含多個遠程URL，並指定它們的寬度和高度，以及可能的縮放比例或其他URI參數。本地端將根據圖片容器的測量大小選擇最佳的`uri`來顯示。可以添加`cache`屬性來控制網絡請求如何與本地緩存交互。（更多信息請參見[圖片的緩存控制（僅限iOS）](images#cache-control-ios-only)）。

目前支持的格式包括`png`、`jpg`、`jpeg`、`bmp`、`gif`、`webp`、`psd`（僅限iOS）。此外，iOS還支持多種RAW圖片格式。請參考Apple的文檔以獲取當前支持的相機型號列表（對於iOS 12，請參見https://support.apple.com/en-ca/HT208967）。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

---

### `style`

| Type                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Image Style Props](image-style-props#props), [Layout Props](layout-props#props), [Shadow Props](shadow-props#props), [Transforms](transforms#props) |

---

### `testID`

用於UI自動化測試腳本中識別此元素的唯一標識符。

| Type   |
| ------ |
| string |

## 方法

### `abortPrefetch()` <div class="label android">Android</div>

```jsx
Image.abortPrefetch(requestId);
```

中止預取請求。

**參數：**

| Name                                                       | Type   | Description                             |
| ---------------------------------------------------------- | ------ | --------------------------------------- |
| requestId <div class="label basic required">Required</div> | number | Request id as returned by `prefetch()`. |

---

### `getSize()`

```jsx
Image.getSize(uri, success, [failure]);
```

在顯示圖片之前獲取其寬度和高度（以像素為單位）。如果無法找到圖片或下載失敗，此方法可能會失敗。

為了獲取圖片的尺寸，可能需要先加載或下載圖片，之後它將被緩存。這意味著原則上您可以使用此方法預加載圖片，但這並非其優化目的，且未來可能會以不完全加載/下載圖片數據的方式實現。一個適當且受支持的預加載圖片方法將作為單獨的API提供。

**參數：**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `getSizeWithHeaders()`

```jsx
Image.getSizeWithHeaders(uri, headers, success, [failure]);
```

在顯示圖片之前獲取其寬度和高度（以像素為單位），並能夠提供請求的標頭。如果無法找到圖片或下載失敗，此方法可能會失敗。它也不適用於靜態圖片資源。

為了獲取圖像的尺寸，可能需要先加載或下載圖像，之後它會被緩存。這意味著原則上你可以使用此方法來預加載圖像，但此方法並非為此目的而優化，且未來可能會以不完全加載/下載圖像數據的方式實現。預加載圖像的正確且受支持的方式將作為一個單獨的API提供。

**參數：**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| headers <div class="label basic required">Required</div> | object   | The headers for the request.                                                                         |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `prefetch()`

```jsx
await Image.prefetch(url);
```

預取遠程圖像以供後續使用，方法是將其下載到磁盤緩存中。返回一個解析為布爾值的Promise。

**參數：**

| Name                                                 | Type                                              | Description                                            |
| ---------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| url <div class="label basic required">Required</div> | string                                            | The remote location of the image.                      |
| callback                                             | function <div class="label android">Android</div> | The function that will be called with the `requestId`. |

---

### `queryCache()`

```jsx
await Image.queryCache(urls);
```

執行緩存查詢。返回一個Promise，該Promise解析為一個從URL到緩存狀態的映射，例如"disk"、"memory"或"disk/memory"。如果請求的URL不在映射中，則表示它不在緩存中。

**參數：**

| Name                                                  | Type  | Description                                |
| ----------------------------------------------------- | ----- | ------------------------------------------ |
| urls <div class="label basic required">Required</div> | array | List of image URLs to check the cache for. |

---

### `resolveAssetSource()`

```jsx
Image.resolveAssetSource(source);
```

將資產引用解析為一個具有`uri`、`width`和`height`屬性的對象。

**參數：**

| <div className="wideColumn">Name</div>                  | Type                                     | Description                                                                  |
| ------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------- |
| source <div class="label basic required">Required</div> | [ImageSource](image#imagesource), number | A number (opaque type returned by `require('./foo.png')`) or an ImageSource. |

## 類型定義

### ImageCacheEnum <div class="label ios">iOS</div>

可用於設置緩存處理或策略的枚舉，用於處理可能被緩存的響應。

| Type                                                               | Default     |
| ------------------------------------------------------------------ | ----------- |
| enum(`'default'`, `'reload'`, `'force-cache'`, `'only-if-cached'`) | `'default'` |

- `default`: 使用原生平台的默認策略。
- `reload`: URL的數據將從原始源加載。不應使用現有的緩存數據來滿足URL加載請求。
- `force-cache`: 無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有與請求相對應的現有數據，則從原始源加載數據。
- `only-if-cached`: 無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有與URL加載請求相對應的現有數據，則不會嘗試從原始源加載數據，並且加載被視為失敗。

### ImageLoadEvent

在`onLoad`回調中返回的對象。

| Type   |
| ------ |
| object |

**屬性：**

| Name   | Type   | Description                         |
| ------ | ------ | ----------------------------------- |
| source | object | The [source object](#source-object) |

#### 源對象

**屬性：**

| Name   | Type   | Description                                                  |
| ------ | ------ | ------------------------------------------------------------ |
| width  | number | The width of loaded image.                                   |
| height | number | The height of loaded image.                                  |
| uri    | string | A string representing the resource identifier for the image. |

### ImageSource

| Type                             |
| -------------------------------- |
| object, array of objects, number |

**屬性（如果作為對象或對象數組傳遞）：**

| <div className="wideColumn">Name</div> | Type                                       | Description                                                                                                                                                                          |
| -------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| uri                                    | string                                     | A string representing the resource identifier for the image, which could be an http address, a local file path, or the name of a static image resource.                              |
| width                                  | number                                     | Can be specified if known at build time, in which case the value will be used to set the default `<Image/>` component dimension.                                                     |
| height                                 | number                                     | Can be specified if known at build time, in which case the value will be used to set the default `<Image/>` component dimension.                                                     |
| scale                                  | number                                     | Used to indicate the scale factor of the image. Defaults to `1.0` if unspecified, meaning that one image pixel equates to one display point / DIP.                                   |
| bundle<div class="label ios">iOS</div> | string                                     | The iOS asset bundle which the image is included in. This will default to `[NSBundle mainBundle]` if not set.                                                                        |
| method                                 | string                                     | The HTTP Method to use. Defaults to `'GET'` if not specified.                                                                                                                        |
| headers                                | object                                     | An object representing the HTTP headers to send along with the request for a remote image.                                                                                           |
| body                                   | string                                     | The HTTP body to send with the request. This must be a valid UTF-8 string, and will be sent exactly as specified, with no additional encoding (e.g. URL-escaping or base64) applied. |
| cache<div class="label ios">iOS</div>  | [ImageCacheEnum](image#imagecacheenum-ios) | Determines how the requests handles potentially cached responses.                                                                                                                    |

**如果傳遞數字：**

- `number` - 由類似`require('./image.jpg')`返回的不透明類型。