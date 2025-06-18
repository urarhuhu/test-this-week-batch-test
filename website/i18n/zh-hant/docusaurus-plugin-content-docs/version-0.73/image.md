---
id: image
title: Image
---

一個用於顯示不同類型圖片的 React 元件，包括網路圖片、靜態資源、暫存本地圖片以及來自本地磁碟的圖片（例如相機膠卷中的圖片）。

此範例展示如何從本地儲存空間、網路甚至使用 `'data:'` URI 方案提供的資料中獲取並顯示圖片。

> 請注意，對於網路和資料圖片，您需要手動指定圖片的尺寸！

## 範例

```SnackPlayer name=Example
import React from 'react';
import {View, Image, StyleSheet} from 'react-native';

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
};

export default DisplayAnImage;
```

您也可以為圖片添加 `style`：

```SnackPlayer name=Example
import React from 'react';
import {View, Image, StyleSheet} from 'react-native';

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
};

export default DisplayAnImageWithStyle;
```

## Android 上的 GIF 和 WebP 支援

在自行建置原生程式碼時，Android 預設不支援 GIF 和 WebP。

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

> 注意：上述列出的版本可能未及時更新。請查閱主倉庫中的 [`ReactAndroid/gradle.properties`](https://github.com/facebook/react-native/blob/main/packages/react-native/ReactAndroid/gradle.properties) 以查看特定標記版本中使用的 fresco 版本。

---

# 參考

## 屬性

### [View 屬性](view.md#props)

繼承 [View 屬性](view#props)。

---

### `accessible`

當為 true 時，表示該圖片是一個無障礙元素。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `accessibilityLabel`

當使用者與圖片互動時，螢幕閱讀器會讀取的文字。

| Type   |
| ------ |
| string |

---

### `alt`

定義圖片替代文字描述的字符串，當使用者與圖片互動時，螢幕閱讀器會讀取此文字。使用此屬性會自動將此元素標記為可存取。

| Type   |
| ------ |
| string |

---

### `blurRadius`

blurRadius：添加到圖片的模糊濾鏡的模糊半徑。

| Type   |
| ------ |
| number |

> 提示：在 iOS 上，您需要將 `blurRadius` 增加到超過 `5`。

---

### `capInsets` <div class="label ios">iOS</div>

當圖片被調整大小時，由 `capInsets` 指定的尺寸的角落將保持固定大小，但圖片的中心內容和邊框將被拉伸。這對於創建可調整大小的圓角按鈕、陰影和其他可調整大小的資源非常有用。更多資訊請參閱 [Apple 官方文件](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/index.html#//apple_ref/occ/instm/UIImage/resizableImageWithCapInsets)。

| Type         |
| ------------ |
| [Rect](rect) |

---

### `crossOrigin`

一個指定在獲取圖片資源時使用的 CORS 模式的關鍵字字符串。其功能類似於 HTML 中的 crossorigin 屬性。

- `anonymous`：在圖片請求中不交換使用者憑證。
- `use-credentials`：在圖片請求中將 `Access-Control-Allow-Credentials` 標頭值設置為 `true`。

| Type                                     | Default       |
| ---------------------------------------- | ------------- |
| enum(`'anonymous'`, `'use-credentials'`) | `'anonymous'` |

---

### `defaultSource`

在載入圖片來源時顯示的靜態圖片。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

> **注意：** 在 Android 上，預設的 source prop 在除錯版本中會被忽略。

---

### `fadeDuration` <div class="label android">Android</div>

淡入淡出動畫的持續時間（毫秒）。

| Type   | Default |
| ------ | ------- |
| number | `300`   |

---

### `height`

圖片元件的高度。

| Type   |
| ------ |
| number |

---

### `loadingIndicatorSource`

與 `source` 類似，此屬性代表用於渲染圖片載入指示器的資源。載入指示器會顯示直到圖片準備好顯示，通常是在圖片下載完成後。

| Type                                                  |
| ----------------------------------------------------- |
| [ImageSource](image#imagesource) (`uri` only), number |

---

### `onError`

在載入錯誤時觸發。

| Type                                |
| ----------------------------------- |
| (`{nativeEvent: {error} }`) => void |

---

### `onLayout`

在掛載時和佈局變化時觸發。

| Type                                                    |
| ------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)} => void` |

---

### `onLoad`

在載入成功完成時觸發。

**範例：** `onLoad={({nativeEvent: {source: {width, height}}}) => setImageRealSize({width, height})}`

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

**範例：** `onLoadStart={() => this.setState({loading: true})}`

| Type       |
| ---------- |
| () => void |

---

### `onPartialLoad` <div class="label ios">iOS</div>

在圖片部分載入完成時觸發。「部分載入」的定義取決於載入器，但這通常用於漸進式 JPEG 載入。

| Type       |
| ---------- |
| () => void |

---

### `onProgress`

在下載進度更新時觸發。

| Type                                        |
| ------------------------------------------- |
| (`{nativeEvent: {loaded, total} }`) => void |

---

### `progressiveRenderingEnabled` <div class="label android">Android</div>

當設為 `true` 時，啟用漸進式 JPEG 串流 - https://frescolib.org/docs/progressive-jpegs。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `resizeMethod` <div class="label android">Android</div>

當圖片的尺寸與圖片視圖的尺寸不同時，用於調整圖片大小的機制。預設為 `auto`。

- `auto`：根據啟發式方法在 `resize` 和 `scale` 之間選擇。

- `resize`：一種軟體操作，在解碼前改變記憶體中的編碼圖片。當圖片比視圖大得多時，應使用此方法而非 `scale`。

- `scale`：圖片會被縮小或放大繪製。與 `resize` 相比，`scale` 更快（通常是硬體加速）且產生更高品質的圖片。如果圖片比視圖小，應使用此方法。如果圖片比視圖稍大，也應使用此方法。

有關 `resize` 和 `scale` 的更多細節，請參閱 https://frescolib.org/docs/resizing。

| Type                                  | Default  |
| ------------------------------------- | -------- |
| enum(`'auto'`, `'resize'`, `'scale'`) | `'auto'` |

---

### `referrerPolicy`

一個字串，指示在獲取資源時使用哪個引用來源。設定圖片請求中 `Referrer-Policy` 標頭的值，功能類似於 HTML 中的 `referrerpolicy` 屬性。

| Type                                                                                                                                                                                     | Default                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| enum(`'no-referrer'`, `'no-referrer-when-downgrade'`, `'origin'`, `'origin-when-cross-origin'`, `'same-origin'`, `'strict-origin'`, `'strict-origin-when-cross-origin'`, `'unsafe-url'`) | `'strict-origin-when-cross-origin'` |

---

### `resizeMode`

決定當框架與原始圖片尺寸不符時如何調整圖片大小。預設為 `cover`。

- `cover`: 均勻縮放圖片（保持圖片長寬比），使：
  
  - 圖片的兩個維度（寬度和高度）都等於或大於視圖的相應維度（減去內邊距）
  - 至少一個維度的縮放圖片會等於視圖的相應維度（減去內邊距）

- `contain`: 均勻縮放圖片（保持圖片長寬比），使圖片的兩個維度（寬度和高度）都等於或小於視圖的相應維度（減去內邊距）。

- `stretch`: 獨立縮放寬度和高度，這可能會改變原始圖片的長寬比。

- `repeat`: 重複圖片以覆蓋視圖的框架。圖片將保持其大小和長寬比，除非它比視圖大，在這種情況下它將被均勻縮小以使其包含在視圖中。

- `center`: 在兩個維度上將圖片置中於視圖中。如果圖片比視圖大，則均勻縮小以使其包含在視圖中。

| Type                                                              | Default   |
| ----------------------------------------------------------------- | --------- |
| enum(`'cover'`, `'contain'`, `'stretch'`, `'repeat'`, `'center'`) | `'cover'` |

---

### `source`

圖片來源（可以是遠端 URL 或本地檔案資源）。

此屬性也可以包含多個遠端 URL，並指定它們的寬度和高度，以及可能的縮放比例/其他 URI 參數。原生端將根據圖片容器的測量大小選擇最佳的 `uri` 來顯示。可以添加 `cache` 屬性來控制網路請求如何與本地緩存互動。（更多資訊請參閱 [圖片快取控制（僅限 iOS）](images#cache-control-ios-only)）。

目前支援的格式有 `png`、`jpg`、`jpeg`、`bmp`、`gif`、`webp`、`psd`（僅限 iOS）。此外，iOS 還支援多種 RAW 圖片格式。請參考 Apple 的文件以獲取當前支援的相機型號列表（對於 iOS 12，請參閱 https://support.apple.com/en-ca/HT208967）。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

---

### `src`

一個表示圖片遠端 URL 的字串。此屬性優先於 `source` 屬性。

**範例:** `src={'https://reactnative.dev/img/tiny_logo.png'}`

| Type   |
| ------ |
| string |

---

### `srcSet`

一個表示逗號分隔的潛在候選圖片來源清單的字串。每個圖片來源包含圖片的 URL 和像素密度描述符。如果未指定描述符，則預設為 `1x` 描述符。

如果 `srcSet` 不包含 `1x` 描述符，則 `src` 中的值將用作具有 `1x` 描述符的圖片來源（如果提供）。

此屬性優先於 `src` 和 `source` 屬性。

**範例:** `srcSet={'https://reactnative.dev/img/tiny_logo.png 1x, https://reactnative.dev/img/header_logo.svg 2x'}`

| Type   |
| ------ |
| string |

---

### `style`

| Type                                                                                                                                                 |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Image Style Props](image-style-props#props), [Layout Props](layout-props#props), [Shadow Props](shadow-props#props), [Transforms](transforms#props) |

---

### `testID`

用於 UI 自動化測試腳本中識別此元素的唯一標識符。

| Type   |
| ------ |
| string |

---

### `tintColor`

Changes the color of all non-transparent pixels to the `tintColor`.

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `width`

Width of the image component.

| Type   |
| ------ |
| number |

## Methods

### `abortPrefetch()` <div class="label android">Android</div>

```tsx
static abortPrefetch(requestId: number);
```

Abort prefetch request.

**Parameters:**

| Name                                                       | Type   | Description                             |
| ---------------------------------------------------------- | ------ | --------------------------------------- |
| requestId <div class="label basic required">Required</div> | number | Request id as returned by `prefetch()`. |

---

### `getSize()`

```tsx
static getSize(
  uri: string,
  success: (width: number, height: number) => void,
  failure?: (error: any) => void,
): any;
```

Retrieve the width and height (in pixels) of an image prior to displaying it. This method can fail if the image cannot be found, or fails to download.

In order to retrieve the image dimensions, the image may first need to be loaded or downloaded, after which it will be cached. This means that in principle you could use this method to preload images, however it is not optimized for that purpose, and may in future be implemented in a way that does not fully load/download the image data. A proper, supported way to preload images will be provided as a separate API.

**Parameters:**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `getSizeWithHeaders()`

```tsx
static getSizeWithHeaders(
  uri: string,
  headers: {[index: string]: string},
  success: (width: number, height: number) => void,
  failure?: (error: any) => void,
): any;
```

Retrieve the width and height (in pixels) of an image prior to displaying it with the ability to provide the headers for the request. This method can fail if the image cannot be found, or fails to download. It also does not work for static image resources.

In order to retrieve the image dimensions, the image may first need to be loaded or downloaded, after which it will be cached. This means that in principle you could use this method to preload images, however it is not optimized for that purpose, and may in future be implemented in a way that does not fully load/download the image data. A proper, supported way to preload images will be provided as a separate API.

**Parameters:**

| <div className="wideColumn">Name</div>                   | Type     | Description                                                                                          |
| -------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| uri <div class="label basic required">Required</div>     | string   | The location of the image.                                                                           |
| headers <div class="label basic required">Required</div> | object   | The headers for the request.                                                                         |
| success <div class="label basic required">Required</div> | function | The function that will be called if the image was successfully found and width and height retrieved. |
| failure                                                  | function | The function that will be called if there was an error, such as failing to retrieve the image.       |

---

### `prefetch()`

```tsx
await Image.prefetch(url);
```

Prefetches a remote image for later use by downloading it to the disk cache. Returns a promise which resolves to a boolean.

**Parameters:**

| Name                                                 | Type                                              | Description                                            |
| ---------------------------------------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| url <div class="label basic required">Required</div> | string                                            | The remote location of the image.                      |
| callback                                             | function <div class="label android">Android</div> | The function that will be called with the `requestId`. |

---

### `queryCache()`

```tsx
static queryCache(
  urls: string[],
): Promise<Record<string, 'memory' | 'disk' | 'disk/memory'>>;
```

Perform cache interrogation. Returns a promise which resolves to a mapping from URL to cache status, such as "disk", "memory" or "disk/memory". If a requested URL is not in the mapping, it means it's not in the cache.

**Parameters:**

| Name                                                  | Type  | Description                                |
| ----------------------------------------------------- | ----- | ------------------------------------------ |
| urls <div class="label basic required">Required</div> | array | List of image URLs to check the cache for. |

---

### `resolveAssetSource()`

```tsx
static resolveAssetSource(source: ImageSourcePropType): {
  height: number;
  width: number;
  scale: number;
  uri: string;
};
```

Resolves an asset reference into an object which has the properties `uri`, `scale`, `width`, and `height`.

**Parameters:**

| <div className="wideColumn">Name</div>                  | Type                                     | Description                                                                  |
| ------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------- |
| source <div class="label basic required">Required</div> | [ImageSource](image#imagesource), number | A number (opaque type returned by `require('./foo.png')`) or an ImageSource. |

## Type Definitions

### ImageCacheEnum <div class="label ios">iOS</div>

Enum which can be used to set the cache handling or strategy for the potentially cached responses.

| Type                                                               | Default     |
| ------------------------------------------------------------------ | ----------- |
| enum(`'default'`, `'reload'`, `'force-cache'`, `'only-if-cached'`) | `'default'` |

- `default`: 使用原生平台的預設策略。
- `reload`: URL 的資料將從原始來源載入。不應使用任何現有的快取資料來滿足 URL 載入請求。
- `force-cache`: 無論快取資料的新舊或過期日期為何，都將使用現有的快取資料來滿足請求。如果快取中沒有對應請求的現有資料，則從原始來源載入資料。
- `only-if-cached`: 無論快取資料的新舊或過期日期為何，都將使用現有的快取資料來滿足請求。如果快取中沒有對應 URL 載入請求的現有資料，則不會嘗試從原始來源載入資料，且載入將被視為失敗。

### ImageLoadEvent

在 `onLoad` 回調中返回的物件。

| Type   |
| ------ |
| object |

**屬性:**

| Name   | Type   | Description                         |
| ------ | ------ | ----------------------------------- |
| source | object | The [source object](#source-object) |

#### 來源物件

**屬性:**

| Name   | Type   | Description                                                  |
| ------ | ------ | ------------------------------------------------------------ |
| width  | number | The width of loaded image.                                   |
| height | number | The height of loaded image.                                  |
| uri    | string | A string representing the resource identifier for the image. |

### ImageSource

| Type                             |
| -------------------------------- |
| object, array of objects, number |

**屬性（如果傳遞物件或物件陣列）:**

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

**如果傳遞數字:**

- `number` - 由類似 `require('./image.jpg')` 返回的不透明類型。