---
id: images
title: Images
---

## 靜態圖片資源

React Native 提供了一種統一的方式來管理 Android 和 iOS 應用中的圖片及其他媒體資源。若要添加靜態圖片到你的應用，請將其放置於原始碼目錄樹中的某處，並像這樣引用它：

```tsx
<Image source={require('./my-icon.png')} />
```

圖片名稱的解析方式與 JS 模組相同。在上面的例子中，打包工具會在同一個資料夾中尋找 `my-icon.png`，該資料夾與引用它的組件位於同一位置。

你可以使用 `@2x` 和 `@3x` 後綴來為不同的螢幕密度提供圖片。如果你有以下文件結構：

```
.
├── button.js
└── img
    ├── check.png
    ├── check@2x.png
    └── check@3x.png
```

...而 `button.js` 代碼中包含：

```tsx
<Image source={require('./img/check.png')} />
```

...打包工具將會根據設備的螢幕密度來打包和提供對應的圖片。例如，`check@2x.png` 將會在 iPhone 7 上使用，而 `check@3x.png` 則會在 iPhone 7 Plus 或 Nexus 5 上使用。如果沒有與螢幕密度匹配的圖片，則會選擇最接近的最佳選項。

在 Windows 上，如果你向項目中添加了新圖片，可能需要重新啟動打包工具。

以下是你能獲得的一些好處：

1. Android 和 iOS 使用相同的系統。
2. 圖片與你的 JavaScript 代碼位於同一資料夾中。組件是自包含的。
3. 沒有全局命名空間，即你不必擔心名稱衝突。
4. 只有實際使用的圖片才會被打包到你的應用中。
5. 添加和更改圖片不需要重新編譯應用，你可以像平常一樣刷新模擬器。
6. 打包工具知道圖片的尺寸，無需在代碼中重複指定。
7. 圖片可以通過 [npm](https://www.npmjs.com/) 包分發。

為了使這一切正常工作，`require` 中的圖片名稱必須是靜態已知的。

```tsx
// GOOD
<Image source={require('./my-icon.png')} />;

// BAD
var icon = this.props.active
  ? 'my-icon-active'
  : 'my-icon-inactive';
<Image source={require('./' + icon + '.png')} />;

// GOOD
var icon = this.props.active
  ? require('./my-icon-active.png')
  : require('./my-icon-inactive.png');
<Image source={icon} />;
```

請注意，通過這種方式引用的圖片源包含圖片的尺寸（寬度、高度）信息。如果你需要動態縮放圖片（例如通過 flex），你可能需要在樣式屬性中手動設置 `{width: undefined, height: undefined}`。

## 靜態非圖片資源

上述的 `require` 語法也可以用於靜態包含音頻、視頻或文檔文件到你的項目中。支持大多數常見的文件類型，包括 `.mp3`、`.wav`、`.mp4`、`.mov`、`.html` 和 `.pdf`。完整列表請參見 [打包工具默認設置](https://github.com/facebook/metro/blob/master/packages/metro-config/src/defaults/defaults.js#L14-L44)。

你可以通過在 [Metro 配置](https://metrobundler.dev/docs/configuration) 中添加 [`assetExts` 解析器選項](https://metrobundler.dev/docs/configuration#resolver-options) 來支持其他類型。

需要注意的是，視頻必須使用絕對定位而不是 `flexGrow`，因為目前沒有為非圖片資源傳遞尺寸信息。對於直接鏈接到 Xcode 或 Android 的 Assets 文件夾中的視頻，這一限制不適用。

## 來自混合應用資源的圖片

如果你正在構建一個混合應用（部分 UI 使用 React Native，部分 UI 使用平台代碼），你仍然可以使用已經打包到應用中的圖片。

對於通過 Xcode 資源目錄或 Android drawable 文件夾包含的圖片，請使用不帶擴展名的圖片名稱：

```tsx
<Image
  source={{uri: 'app_icon'}}
  style={{width: 40, height: 40}}
/>
```

對於 Android assets 文件夾中的圖片，請使用 `asset:/` 方案：

```tsx
<Image
  source={{uri: 'asset:/app_icon.png'}}
  style={{width: 40, height: 40}}
/>
```

這些方法不提供安全性檢查。你需要自己保證這些圖片在應用中是可用的。此外，你還需要手動指定圖片的尺寸。

## 網絡圖片

您應用中顯示的許多圖片在編譯時並不可用，或者您可能希望動態加載某些圖片以減少二進制文件大小。與靜態資源不同，_您需要手動指定圖片的尺寸_。強烈建議您同時使用 https 以滿足 iOS 上 [App Transport Security](publishing-to-app-store.md#1-enable-app-transport-security) 的要求。

```tsx
// GOOD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}}
       style={{width: 400, height: 400}} />

// BAD
<Image source={{uri: 'https://reactjs.org/logo-og.png'}} />
```

### 圖片的網路請求

如果您希望在圖片請求中設置 HTTP 方法、標頭或請求體，可以通過在來源物件中定義這些屬性來實現：

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

有時，您可能會從 REST API 調用中獲取編碼的圖片數據。您可以使用 `'data:'` URI 方案來使用這些圖片。與網路資源相同，_您需要手動指定圖片的尺寸_。

:::info
這僅建議用於非常小且動態的圖片，例如來自數據庫列表中的圖標。
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

### 緩存控制（僅限 iOS）

在某些情況下，您可能只希望在本地緩存中已有圖片時顯示它，例如在更高解析度的圖片可用之前顯示低解析度的佔位圖。在其他情況下，您可能不關心圖片是否過時，並願意顯示過時的圖片以節省帶寬。`cache` 來源屬性讓您可以控制網路層如何與緩存交互。

- `default`：使用原生平台的默認策略。
- `reload`：URL 的數據將從原始來源加載。不應使用現有的緩存數據來滿足 URL 加載請求。
- `force-cache`：無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有對應請求的現有數據，則從原始來源加載數據。
- `only-if-cached`：無論其年齡或過期日期如何，都將使用現有的緩存數據來滿足請求。如果緩存中沒有對應 URL 加載請求的現有數據，則不會嘗試從原始來源加載數據，並且加載被視為失敗。

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

請參閱 [CameraRoll](https://github.com/react-native-community/react-native-cameraroll) 以了解如何使用 `Images.xcassets` 之外本地資源的示例。

### 最佳相機膠卷圖片

iOS 會在您的相機膠卷中為同一張圖片保存多個尺寸，為了性能考慮，選擇最接近的尺寸非常重要。您不會希望使用全解析度的 3264x2448 圖片作為來源來顯示 200x200 的縮略圖。如果有完全匹配的尺寸，React Native 會選擇它，否則它會使用至少大 50% 的第一個尺寸，以避免從接近的尺寸調整大小時出現模糊。所有這些都是默認完成的，因此您無需擔心編寫繁瑣（且容易出錯）的代碼來自行處理。

## 為什麼不自動調整所有內容的大小？

_在瀏覽器中_，如果您不給圖片指定大小，瀏覽器將渲染一個 0x0 的元素，下載圖片，然後根據正確的尺寸渲染圖片。這種行為的主要問題是，您的 UI 會在圖片加載時四處跳動，這會導致非常糟糕的用戶體驗。這被稱為 [Cumulative Layout Shift](https://web.dev/cls/)。

_在 React Native 中_，這種行為是故意不實現的。開發人員需要提前知道遠程圖片的尺寸（或寬高比）會增加工作量，但我們相信這會帶來更好的用戶體驗。通過 `require('./my-icon.png')` 語法從應用包加載的靜態圖片_可以自動調整大小_，因為它們的尺寸在掛載時立即可用。

舉例來說，`require('./my-icon.png')` 的執行結果可能如下：

```tsx
{"__packager_asset":true,"uri":"my-icon.png","width":591,"height":573}
```

## 以物件形式指定來源

在 React Native 中，一個有趣的設計決策是將 `src` 屬性命名為 `source`，且不接受字串值，而是接受一個帶有 `uri` 屬性的物件。

```tsx
<Image source={{uri: 'something.jpg'}} />
```

從基礎架構的角度來看，這種設計讓我們能為該物件附加元數據。例如當您使用 `require('./my-icon.png')` 時，我們會加入其實際位置和尺寸的資訊（請勿依賴此特性，未來可能變更！）。這同時也是為未來功能預留空間，比方說未來若需支援雪碧圖（sprites），我們可以輸出 `{uri: ..., crop: {left: 10, top: 50, width: 20, height: 40}}` 而非單純的 `{uri: ...}`，就能無縫在所有現有呼叫點支援雪碧圖技術。

對使用者而言，這讓您能為物件添加實用屬性，例如圖像尺寸以計算顯示大小。您可自由將其作為資料結構，來儲存更多關於圖像的資訊。

## 透過嵌套實現背景圖像

熟悉網頁開發的開發者常要求 `background-image` 功能。為滿足此需求，您可使用 `<ImageBackground>` 元件，其擁有與 `<Image>` 相同的屬性，並可在其上疊加任意子元件。

在某些情況下，您可能不想使用 `<ImageBackground>`，因為其實作較為基礎。請參考 `<ImageBackground>` 的[說明文件](imagebackground.md)獲取更多見解，並在需要時建立自訂元件。

```tsx
return (
  <ImageBackground source={...} style={{width: '100%', height: '100%'}}>
    <Text>Inside</Text>
  </ImageBackground>
);
```

請注意您必須指定某些寬度和高度的樣式屬性。

## iOS 圓角邊框樣式

請注意以下針對特定角落的圓角邊框樣式屬性，可能會被 iOS 的圖片元件忽略：

- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomLeftRadius`
- `borderBottomRightRadius`

## 線程外解碼

圖片解碼可能耗時超過一幀。這在網頁上是導致幀率下降的主因之一，因為解碼是在主線程進行。React Native 中，圖片解碼會在獨立線程處理。實務上您本來就需要處理圖片尚未下載完成的情況，因此在解碼期間多顯示幾幀佔位圖，並不需要任何程式碼變更。

## 設定 iOS 圖片快取限制

在 iOS 平台，我們提供了覆寫 React Native 預設圖片快取限制的 API。此方法應在原生 AppDelegate 程式碼中呼叫（例如在 `didFinishLaunchingWithOptions` 方法內）。

```objectivec
RCTSetImageCacheLimits(4*1024*1024, 200*1024*1024);
```

**參數：**

| Name           | Type   | Required | Description             |
| -------------- | ------ | -------- | ----------------------- |
| imageSizeLimit | number | Yes      | Image cache size limit. |
| totalCostLimit | number | Yes      | Total cache cost limit. |

上述程式碼範例中，單一圖片大小限制設為 4 MB，總成本限制設為 200 MB。