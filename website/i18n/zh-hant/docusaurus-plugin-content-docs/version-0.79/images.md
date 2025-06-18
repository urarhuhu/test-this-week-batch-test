---
id: images
title: Images
---

## 靜態圖片資源

React Native 提供統一方式管理 Android 和 iOS 應用中的圖片與其他媒體資源。要添加靜態圖片至應用，請將其置於原始碼目錄中並按以下方式引用：

```tsx
<Image source={require('./my-icon.png')} />
```

圖片名稱的解析方式與 JS 模組相同。上例中，打包工具會在同級目錄尋找組件所需的 `my-icon.png`。

您可使用 `@2x` 和 `@3x` 後綴為不同螢幕密度提供圖片。若文件結構如下：

```
.
├── button.js
└── img
    ├── check.png
    ├── check@2x.png
    └── check@3x.png
```

...且 `button.js` 代碼包含：

```tsx
<Image source={require('./img/check.png')} />
```

...打包工具將根據設備螢幕密度選擇對應圖片。例如 iPhone 7 會使用 `check@2x.png`，而 iPhone 7 Plus 或 Nexus 5 會使用 `check@3x.png`。若無完全匹配的圖片，則選擇最接近的選項。

在 Windows 系統中，若添加新圖片至專案，可能需要重啟打包工具。

此機制帶來以下優勢：

1. Android 與 iOS 使用相同系統
2. 圖片與 JavaScript 代碼位於同目錄，組件自包含
3. 無需擔心全局命名空間衝突
4. 僅實際使用的圖片會被打包進應用
5. 增改圖片無需重新編譯應用，可如常刷新模擬器
6. 打包工具知曉圖片尺寸，無需在代碼中重複聲明
7. 可透過 [npm](https://www.npmjs.com/) 分發圖片資源

需注意：`require` 中的圖片名稱必須是靜態已知的。

```tsx
// GOOD
<Image source={require('./my-icon.png')} />;

// BAD
const icon = this.props.active
  ? 'my-icon-active'
  : 'my-icon-inactive';
<Image source={require('./' + icon + '.png')} />;

// GOOD
const icon = this.props.active
  ? require('./my-icon-active.png')
  : require('./my-icon-inactive.png');
<Image source={icon} />;
```

注意：此方式引入的圖片源包含尺寸（寬高）信息。若需動態縮放圖片（如透過 flex），可能需手動設置樣式屬性為 `{width: undefined, height: undefined}`。

## 靜態非圖片資源

上述 `require` 語法亦可用於靜態引入音頻、視頻或文檔文件。支援常見格式如 `.mp3`、`.wav`、`.mp4`、`.mov`、`.html` 和 `.pdf`。完整列表請參閱 [打包工具默認配置](https://github.com/facebook/metro/blob/master/packages/metro-config/src/defaults/defaults.js#L14-L44)。

可透過在 [Metro 配置](https://metrobundler.dev/docs/configuration) 中添加 [`assetExts` 解析器選項](https://metrobundler.dev/docs/configuration#resolver-options) 來支援其他類型。

需注意：視頻必須使用絕對定位而非 `flexGrow`，因目前非圖片資源不傳遞尺寸信息。此限制不適用於直接鏈接至 Xcode 或 Android Assets 文件夾的視頻。

## 混合應用資源中的圖片

若開發混合應用（部分 UI 使用 React Native，部分使用平台原生代碼），仍可使用已打包至應用的圖片。

對於透過 Xcode 資產目錄或 Android drawable 文件夾引入的圖片，使用不帶擴展名的圖片名稱：

```tsx
<Image
  source={{uri: 'app_icon'}}
  style={{width: 40, height: 40}}
/>
```

對於 Android assets 文件夾中的圖片，使用 `asset:/` 協議：

```tsx
<Image
  source={{uri: 'asset:/app_icon.png'}}
  style={{width: 40, height: 40}}
/>
```

這些方法不進行安全性檢查。您需自行確保圖片存在於應用中，且需手動指定圖片尺寸。

## 網絡圖片

您應用中顯示的許多圖片在編譯時並不可用，或者您可能希望動態加載某些圖片以減少二進位文件大小。與靜態資源不同，_您需要手動指定圖片的尺寸_。強烈建議同時使用 https 以滿足 iOS 上 [App Transport Security](publishing-to-app-store.md#1-enable-app-transport-security) 的要求。

```tsx
// GOOD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}} />
```

### 圖片的網路請求

如果您想為圖片請求設定 HTTP 方法、標頭或請求體，可以通過在來源物件中定義這些屬性來實現：

```tsx
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    method: 'POST',
    headers: {
      Pragma: 'no-cache',
    },
    body: 'Your Body goes here',
  }}
  style={{width: 400, height: 400}}
/>
```

## URI 數據圖片

有時，您可能從 REST API 呼叫中獲取編碼的圖片數據。您可以使用 `'data:'` URI 方案來使用這些圖片。與網路資源相同，_您需要手動指定圖片的尺寸_。

:::info
建議僅用於非常小且動態的圖片，例如來自資料庫的清單圖標。
:::

```tsx
// include at least width and height!
<Image
  style={{
    width: 51,
    height: 51,
    resizeMode: 'contain',
  }}
  source={{
    uri: 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADMAAAAzCAYAAAA6oTAqAAAAEXRFWHRTb2Z0d2FyZQBwbmdjcnVzaEB1SfMAAABQSURBVGje7dSxCQBACARB+2/ab8BEeQNhFi6WSYzYLYudDQYGBgYGBgYGBgYGBgYGBgZmcvDqYGBgmhivGQYGBgYGBgYGBgYGBgYGBgbmQw+P/eMrC5UTVAAAAABJRU5ErkJggg==',
  }}
/>
```

### 快取控制

在某些情況下，您可能只想顯示本地快取中已有的圖片，例如在更高解析度的圖片可用之前顯示低解析度的佔位圖。在其他情況下，您不介意圖片是否過時，並願意顯示過時的圖片以節省頻寬。`cache` 來源屬性讓您可以控制網路層如何與快取互動。

- `default`: 使用原生平台的預設策略。
- `reload`: 從原始來源加載 URL 的數據。不應使用現有的快取數據來滿足 URL 加載請求。
- `force-cache`: 無論其年齡或過期日期如何，將使用現有的快取數據來滿足請求。如果快取中沒有對應請求的數據，則從原始來源加載數據。
- `only-if-cached`: 無論其年齡或過期日期如何，將使用現有的快取數據來滿足請求。如果快取中沒有對應 URL 加載請求的數據，則不會嘗試從原始來源加載數據，並且加載被視為失敗。

```tsx
<Image
  source={{
    uri: 'https://reactjs.org/logo-og.png',
    cache: 'only-if-cached',
  }}
  style={{width: 400, height: 400}}
/>
```

## 本地文件系統圖片

請參閱 [CameraRoll](https://github.com/react-native-community/react-native-cameraroll) 以了解如何使用 `Images.xcassets` 之外的本地資源的範例。

### Drawable 資源

Android 支援通過 `xml` 文件類型加載 [drawable 資源](https://developer.android.com/guide/topics/resources/drawable-resource)。這意味著您可以使用 [向量 drawables](https://developer.android.com/develop/ui/views/graphics/vector-drawable-resources) 來渲染圖標，或使用 [形狀 drawables](https://developer.android.com/guide/topics/resources/drawable-resource#Shape) 來繪製形狀！您可以像導入和使用任何其他 [靜態資源](#static-image-resources) 或 [混合資源](#images-from-hybrid-apps-resources) 一樣導入和使用這些資源類型。您需要手動指定圖片尺寸。

對於與您的 JS 代碼位於同一位置的靜態 drawables，請使用 `require` 或 `import` 語法（兩者效果相同）：

```tsx
<Image
  source={require('./img/my_icon.xml')}
  style={{width: 40, height: 40}}
/>
```

對於包含在 Android drawable 文件夾（即 `res/drawable`）中的 drawables，請使用不帶擴展名的資源名稱：

```tsx
<Image
  source={{uri: 'my_icon'}}
  style={{width: 40, height: 40}}
/>
```

drawable 資源與其他圖片類型的一個關鍵區別在於，必須在 Android 應用程序的編譯時引用該資源，因為 Android 需要運行 [Android Asset Packaging Tool (AAPT)](https://developer.android.com/tools/aapt2) 來打包資源。AAPT 創建的二進位 XML 文件格式無法通過 Metro 在網路上加載。如果您更改資源的目錄或名稱，則每次都需要重新構建 Android 應用程序。

#### 創建 XML drawable 資源

Android 在其[可繪製資源指南](https://developer.android.com/guide/topics/resources/drawable-resource)中提供了對每種支援的可繪製資源類型的完整說明，並附有原始 XML 檔案的範例。您可以使用 Android Studio 的工具，如[向量資源工作室](https://developer.android.com/studio/write/vector-asset-studio)，從可縮放向量圖形 (SVG) 和 Adobe Photoshop 文件 (PSD) 檔案創建向量可繪製資源。

:::info
如果您希望將 XML 檔案視為靜態圖片資源（即使用 `import` 或 `require` 語句），應盡量避免在創建的 XML 檔案中引用其他資源。若需引用其他可繪製資源或屬性，如[顏色狀態列表](https://developer.android.com/guide/topics/resources/color-list-resource)或[尺寸資源](https://developer.android.com/guide/topics/resources/more-resources#Dimension)，則應將可繪製資源作為[混合應用資源](#images-from-hybrid-apps-resources)包含，並通過名稱導入。
:::

### 最佳相機膠卷圖片

iOS 會在相機膠卷中為同一張圖片保存多種尺寸，出於性能考量，選擇最接近的尺寸至關重要。例如，顯示 200x200 縮略圖時，您不會想使用全解析度的 3264x2448 圖片作為來源。如果有完全匹配的尺寸，React Native 會自動選擇它；否則，它會使用至少大 50% 的第一個尺寸，以避免從相近尺寸調整時產生模糊效果。這一切默認自動完成，您無需編寫繁瑣（且容易出錯）的代碼來處理。

## 為何不自動調整所有內容的尺寸？

_在瀏覽器中_，如果您未指定圖片的尺寸，瀏覽器會渲染一個 0x0 的元素，下載圖片後再根據正確尺寸重新渲染。這種行為的主要問題在於，圖片載入時會導致 UI 四處跳動，造成糟糕的使用者體驗。這被稱為[累積版面偏移](https://web.dev/cls/)。

_在 React Native 中_，我們刻意不實作此行為。雖然開發者需事先知道遠端圖片的尺寸（或長寬比）會增加工作量，但我們認為這能帶來更好的使用者體驗。透過 `require('./my-icon.png')` 語法從應用程式套件載入的靜態圖片_可以自動調整尺寸_，因為它們的尺寸在掛載時即可立即取得。

例如，`require('./my-icon.png')` 的結果可能是：

```tsx
{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}
```

## 來源作為物件

在 React Native 中，一個有趣的設計決策是將 `src` 屬性命名為 `source`，且不接受字串，而是接受一個帶有 `uri` 屬性的物件。

```tsx
<Image source={{uri: 'something.jpg'}} />
```

在基礎架構層面，這樣做讓我們能為此物件附加元數據。例如，當您使用 `require('./my-icon.png')` 時，我們會添加其實際位置和大小的資訊（請勿依賴此特性，未來可能變更！）。這也具有前瞻性，例如未來我們可能想支援精靈圖（sprites），此時輸出的不是 `{uri: ...}`，而是 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}`，從而透明地支援所有現有呼叫點的精靈圖功能。

對使用者而言，這讓您能為物件添加實用屬性，例如圖片的尺寸，以計算其顯示大小。您可以自由地將其作為資料結構，儲存更多關於圖片的資訊。

## 透過嵌套實現背景圖片

熟悉網頁開發的開發者常提出 `background-image` 的功能需求。為滿足此需求，您可以使用 `<ImageBackground>` 組件，其屬性與 `<Image>` 相同，並可在其上疊加任何子組件。

在某些情況下，您可能不想使用 `<ImageBackground>`，因為其實現較為基礎。如需更多資訊，請參考 `<ImageBackground>` 的[文件](imagebackground.md)，並在需要時創建自己的自訂組件。

```tsx
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

請注意，您必須指定一些寬度和高度的樣式屬性。

## iOS 邊框半徑樣式

請注意，iOS 的圖片組件可能會忽略以下特定角落的邊框半徑樣式屬性：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

## 線程外解碼

圖片解碼可能需要超過一幀的時間。這是網頁上幀率下降的主要原因之一，因為解碼是在主線程中完成的。在 React Native 中，圖片解碼是在不同的線程中進行的。實際上，您已經需要處理圖片尚未下載的情況，因此在解碼過程中多顯示幾幀的佔位圖不需要任何代碼更改。

## 配置 iOS 圖片緩存限制

在 iOS 上，我們提供了一個 API 來覆蓋 React Native 的默認圖片緩存限制。這應該從您的原生 AppDelegate 代碼中調用（例如在 `didFinishLaunchingWithOptions` 中）。

```objectivec
RCTSetImageCacheLimits(4*1024*1024, 200*1024*1024);
```

**參數：**

| Name           | Type   | Required | Description             |
| -------------- | ------ | -------- | ----------------------- |
| imageSizeLimit | number | Yes      | Image cache size limit. |
| totalCostLimit | number | Yes      | Total cache cost limit. |

在上述代碼示例中，圖片大小限制設置為 4 MB，總成本限制設置為 200 MB。