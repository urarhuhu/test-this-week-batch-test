---
id: image
title: Image
---

一個用於顯示不同類型圖片的 React 元件，包括網路圖片、靜態資源、暫存本地圖片以及來自本地磁碟的圖片（例如相機膠卷）。

此範例展示如何從本地儲存空間、網路甚至透過 `'data:'` URI 方案提供的資料中獲取並顯示圖片。

> 請注意，對於網路和資料圖片，您需要手動指定圖片的尺寸！

## 範例

```SnackPlayer name=Image%20Example
import React from 'react';
import {Image, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const styles = StyleSheet.create({
  container: {
    flex: 1,
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

const DisplayAnImage = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
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
    </SafeAreaView>
  </SafeAreaProvider>
);

export default DisplayAnImage;
```

您也可以為圖片添加 `style`：

```SnackPlayer name=Styled%20Image%20Example
import React from 'react';
import {Image, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  stretch: {
    width: 50,
    height: 200,
    resizeMode: 'stretch',
  },
});

const DisplayAnImageWithStyle = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
      <Image
        style={styles.stretch}
        source={require('@expo/snack-static/react-native-logo.png')}
      />
    </SafeAreaView>
  </SafeAreaProvider>
);

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
  implementation 'com.facebook.fresco:animated-gif:3.2.0'

  // For WebP support, including animated WebP
  implementation 'com.facebook.fresco:animated-webp:3.2.0'
  implementation 'com.facebook.fresco:webpsupport:3.2.0'

  // For WebP support, without animations
  implementation 'com.facebook.fresco:webpsupport:3.2.0'
}
```

> 注意：上述列出的版本可能未及時更新。請查閱主倉庫中的 [`packages/react-native/gradle/libs.versions.toml`](https://github.com/facebook/react-native/blob/main/packages/react-native/gradle/libs.versions.toml)，以查看特定標記版本中使用的 fresco 版本。

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

當使用者與圖片互動時，螢幕閱讀器會朗讀的文字。

| Type   |
| ------ |
| string |

---

### `alt`

定義圖片替代文字說明的字串，當使用者與圖片互動時，螢幕閱讀器會朗讀此文字。使用此屬性會自動將此元素標記為無障礙。

| Type   |
| ------ |
| string |

---

### `blurRadius`

blurRadius：添加到圖片的模糊濾鏡的模糊半徑。

| Type   |
| ------ |
| number |

> 提示：在 IOS 上，您需要將 `blurRadius` 增加到超過 `5`。

---

### `capInsets` <div class="label ios">iOS</div>

當圖片調整大小時，由 `capInsets` 指定的尺寸角落將保持固定大小，但圖片的中心內容和邊框將被拉伸。這對於創建可調整大小的圓角按鈕、陰影和其他可調整大小的資源非常有用。更多資訊請參閱 [官方 Apple 文件](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIImage_Class/index.html#//apple_ref/occ/instm/UIImage/resizableImageWithCapInsets)。

| Type         |
| ------------ |
| [Rect](rect) |

---

### `crossOrigin`

一個關鍵字字串，指定在獲取圖片資源時使用的 CORS 模式。其功能類似於 HTML 中的 crossorigin 屬性。

- `anonymous`：在圖片請求中不交換使用者憑證。
- `use-credentials`：在圖片請求中將 `Access-Control-Allow-Credentials` 標頭值設為 `true`。

| Type                                     | Default       |
| ---------------------------------------- | ------------- |
| enum(`'anonymous'`, `'use-credentials'`) | `'anonymous'` |

---

### `defaultSource`

在載入圖片來源時顯示的靜態圖片。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

> **注意：** 在 Android 上，debug 版本會忽略預設的 source 屬性。

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

在圖片部分載入完成時觸發。「部分載入」的定義取決於載入器，但這主要用於漸進式 JPEG 載入。

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

設為 `true` 時，啟用漸進式 JPEG 串流 - https://frescolib.org/docs/progressive-jpegs。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `referrerPolicy`

一個字串，指示在獲取資源時使用哪個 referrer。設定圖片請求中 `Referrer-Policy` 標頭的值。功能類似於 HTML 中的 `referrerpolicy` 屬性。

| Type                                                                                                                                                                                     | Default                             |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| enum(`'no-referrer'`, `'no-referrer-when-downgrade'`, `'origin'`, `'origin-when-cross-origin'`, `'same-origin'`, `'strict-origin'`, `'strict-origin-when-cross-origin'`, `'unsafe-url'`) | `'strict-origin-when-cross-origin'` |

---

### `resizeMethod` <div class="label android">Android</div>

當圖片的尺寸與圖片視圖的尺寸不同時，用於調整圖片大小的機制。預設為 `auto`。

- `auto`：使用啟發式方法在 `resize` 和 `scale` 之間選擇。

- `resize`：一種軟體操作，在圖像解碼前於記憶體中改變編碼圖像。當圖像遠大於視圖時，應使用此方法而非 `scale`。

- `scale`：圖像會被縮小或放大繪製。與 `resize` 相比，`scale` 更快（通常由硬體加速）且產生更高品質的圖像。若圖像小於視圖時應使用此方法。若圖像略大於視圖時也應使用。

- `none`：不進行採樣，圖像以完整解析度顯示。僅在極少數情況下使用，因為這被視為不安全，當 Android 嘗試渲染消耗過多記憶體的圖像時會拋出運行時例外。

關於 `resize` 和 `scale` 的更多細節可參閱 https://frescolib.org/docs/resizing。

| Type                                            | Default  |
| ----------------------------------------------- | -------- |
| enum(`'auto'`, `'resize'`, `'scale'`, `'none'`) | `'auto'` |

---

### `resizeMode`

決定當框架與原始圖像尺寸不符時如何調整圖像大小。默認為 `cover`。

- `cover`：均勻縮放圖像（保持圖像的寬高比），使得：
  
  - 圖像的寬度和高度都會等於或大於視圖的對應維度（減去內邊距）
  - 縮放後圖像至少有一個維度會等於視圖的對應維度（減去內邊距）

- `contain`：均勻縮放圖像（保持寬高比），使得圖像的寬度和高度都會等於或小於視圖的對應維度（減去內邊距）。

- `stretch`：獨立縮放寬度和高度，這可能改變圖像的寬高比。

- `repeat`：重複圖像以覆蓋視圖框架。圖像會保持其大小和寬高比，除非它大於視圖，此時會均勻縮小以使其包含在視圖中。

- `center`：在視圖中沿兩個維度居中圖像。若圖像大於視圖，則均勻縮小以使其包含在視圖中。

| Type                                                              | Default   |
| ----------------------------------------------------------------- | --------- |
| enum(`'cover'`, `'contain'`, `'stretch'`, `'repeat'`, `'center'`) | `'cover'` |

---

### `resizeMultiplier` <div class="label android">Android</div>

當 `resizeMethod` 設為 `resize` 時，目標尺寸會乘以此值。`scale` 方法用於執行剩餘的調整。默認值 `1.0` 表示點陣圖大小設計為符合目標尺寸。大於 `1.0` 的乘數會將調整選項設為大於目標尺寸，生成的點陣圖會從硬體大小縮小。默認為 `1.0`。

此屬性在目標尺寸非常小且源圖像明顯較大時最有用。`resize` 調整方法會進行降採樣，源圖像與目標圖像尺寸之間會損失大量圖像品質，通常導致圖像模糊。通過使用乘數，解碼後的圖像會略大於目標尺寸但小於源圖像（若源圖像足夠大）。這允許透過縮放操作在乘數圖像上產生偽品質的混疊偽影。

若源圖像尺寸為 200x200 而目標尺寸為 24x24，`2.0` 的 resizeMultiplier 會指示 Fresco 將圖像降採樣至 48x48。Fresco 會選擇最接近的 2 的冪次（即 50x50）並將圖像解碼為該大小的點陣圖。若無乘數，最接近的 2 的冪次會是 25x25。生成的圖像會由系統縮小。

| Type   | Default |
| ------ | ------- |
| number | `1.0`   |

---

### `source`

圖像來源（可以是遠程 URL 或本地文件資源）。

此屬性也可包含多個遠端 URL，並指定其寬度、高度及可能的縮放比例/其他 URI 參數。原生端會根據圖片容器的測量尺寸選擇最佳的 `uri` 來顯示。可添加 `cache` 屬性來控制網路請求與本地快取的互動方式（更多資訊請參閱[圖片快取控制](images#cache-control)）。

目前支援的格式包括 `png`、`jpg`、`jpeg`、`bmp`、`gif`、`webp`、`psd`（僅限 iOS）。此外，iOS 支援多種 RAW 圖片格式。請參閱 Apple 官方文件以獲取當前支援的相機型號清單（iOS 12 請見 https://support.apple.com/en-ca/HT208967）。

請注意，`webp` 格式在 iOS 上**僅**在與 JavaScript 代碼捆綁時才受支援。

| Type                             |
| -------------------------------- |
| [ImageSource](image#imagesource) |

---

### `src`

表示圖片遠端 URL 的字串。此屬性優先於 `source` 屬性。

**範例：** `src={'https://reactnative.dev/img/tiny_logo.png'}`

| Type   |
| ------ |
| string |

---

### `srcSet`

表示以逗號分隔的候選圖片來源清單的字串。每個圖片來源包含圖片的 URL 和像素密度描述符。若未指定描述符，則預設為 `1x` 描述符。

若 `srcSet` 未包含 `1x` 描述符，則會使用 `src` 中的值作為 `1x` 描述符的圖片來源（如有提供）。

此屬性優先於 `src` 和 `source` 屬性。

**範例：** `srcSet={'https://reactnative.dev/img/tiny_logo.png 1x, https://reactnative.dev/img/header_logo.svg 2x'}`

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

將所有非透明像素的顏色更改為 `tintColor`。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `width`

圖片元件的寬度。

| Type   |
| ------ |
| number |

## 方法

### `abortPrefetch()` <div class="label android">Android</div>

```tsx
static abortPrefetch(requestId: number);
```

中止預取請求。

**參數：**

| Name                                                       | Type   | Description                             |
| ---------------------------------------------------------- | ------ | --------------------------------------- |
| requestId <div class="label basic required">Required</div> | number | Request id as returned by `prefetch()`. |

---

### `getSize()`

```tsx
static getSize(uri: string): Promise<{width: number, height: number}>;
```

在顯示圖片前獲取其寬度和高度（以像素為單位）。若圖片無法找到或下載失敗，此方法可能會失敗。

為獲取圖片尺寸，可能需要先加載或下載圖片，之後會將其快取。這意味著原則上您可以使用此方法預載圖片，但此方法並非為此目的優化，未來可能會以不完全加載/下載圖片數據的方式實現。預載圖片的正式支援方式將作為獨立 API 提供。

**參數：**

| <div className="wideColumn">Name</div>               | Type   | Description                |
| ---------------------------------------------------- | ------ | -------------------------- |
| uri <div class="label basic required">Required</div> | string | The location of the image. |

---

### `getSizeWithHeaders()`

```tsx
static getSizeWithHeaders(
  uri: string,
  headers: {[index: string]: string}
): Promise<{width: number, height: number}>;
```

在顯示圖片之前，先擷取其寬度和高度（以像素為單位），並可提供請求的標頭。若圖片無法找到或下載失敗，此方法可能會失敗。此外，它不適用於靜態圖片資源。

為了擷取圖片的尺寸，可能需要先載入或下載圖片，之後會將其快取。這意味著原則上你可以使用此方法來預載圖片，但這並非其最佳用途，且未來可能會以不完全載入/下載圖片資料的方式實作。預載圖片的正式支援方法將作為獨立的API提供。

**參數：**

| <div className="wideColumn">Name</div>                   | Type   | Description                  |
| -------------------------------------------------------- | ------ | ---------------------------- |
| uri <div class="label basic required">Required</div>     | string | The location of the image.   |
| headers <div class="label basic required">Required</div> | object | The headers for the request. |

---

### `prefetch()`

```tsx
await Image.prefetch(url);
```

預取遠端圖片以供後續使用，方法是將其下載至磁碟快取。回傳一個解析為布林值的Promise。

**參數：**

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

執行快取查詢。回傳一個Promise，解析為從URL到快取狀態的映射，例如「disk」、「memory」或「disk/memory」。若請求的URL不在映射中，表示其不在快取內。

**參數：**

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

將資產參考解析為一個物件，該物件具有`uri`、`scale`、`width`和`height`等屬性。

**參數：**

| <div className="wideColumn">Name</div>                  | Type                                     | Description                                                                  |
| ------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------- |
| source <div class="label basic required">Required</div> | [ImageSource](image#imagesource), number | A number (opaque type returned by `require('./foo.png')`) or an ImageSource. |

## 類型定義

### ImageCacheEnum <div class="label ios">iOS</div>

枚舉，可用於設定快取處理或策略，以應對可能已快取的回應。

| Type                                                               | Default     |
| ------------------------------------------------------------------ | ----------- |
| enum(`'default'`, `'reload'`, `'force-cache'`, `'only-if-cached'`) | `'default'` |

- `default`：使用原生平台的預設策略。
- `reload`：URL的資料將從原始來源載入。不應使用現有的快取資料來滿足URL載入請求。
- `force-cache`：無論其年齡或過期日期如何，將使用現有的快取資料來滿足請求。若快取中沒有對應請求的資料，則從原始來源載入資料。
- `only-if-cached`：無論其年齡或過期日期如何，將使用現有的快取資料來滿足請求。若快取中沒有對應URL載入請求的資料，則不會嘗試從原始來源載入資料，且載入視為失敗。

### ImageLoadEvent

在`onLoad`回調中回傳的物件。

| Type   |
| ------ |
| object |

**屬性：**

| Name   | Type   | Description                         |
| ------ | ------ | ----------------------------------- |
| source | object | The [source object](#source-object) |

#### 來源物件

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

**屬性（若傳遞物件或物件陣列）：**

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

**若傳遞數字：**

- `number` - 不透明類型，由類似`require('./image.jpg')`的東西回傳。