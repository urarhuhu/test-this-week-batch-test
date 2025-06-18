---
id: hermes
title: Using Hermes
---

<a href="https://hermesengine.dev">
  <img width={300} height={300} className="hermes-logo" src="/docs/assets/HermesLogo.svg" style={{height: "auto"}}/>
</a>

[Hermes](https://hermesengine.dev) 是一個專為 React Native 優化的開源 JavaScript 引擎。對多數應用而言，相較於 JavaScriptCore，使用 Hermes 能帶來更快的啟動時間、更低的記憶體使用量以及更小的應用體積。
React Native 預設已整合 Hermes，無需額外配置即可啟用。

## 內建版 Hermes

React Native 附帶 **內建版本** 的 Hermes。
每當我們發布新版本 React Native 時，都會為您構建對應的 Hermes 版本。這確保您使用的 Hermes 版本與當前 React Native 版本完全相容。

過去我們曾遇到 Hermes 與 React Native 版本匹配問題。此方案徹底解決了該問題，為用戶提供與特定 React Native 版本完全相容的 JS 引擎。

此變更對 React Native 用戶完全透明。您仍可透過本頁面所述指令停用 Hermes。
[點此閱讀技術實現詳情](/architecture/bundled-hermes)。

## 確認 Hermes 已啟用

若您近期新建專案，應能在歡迎畫面查看 Hermes 啟用狀態：

![AwesomeProject 中 JS 引擎狀態顯示位置](/docs/assets/HermesApp.jpg)

JavaScript 環境中會存在 `HermesInternal` 全域變數，可用於驗證 Hermes 是否運作：

```jsx
const isHermes = () => !!global.HermesInternal;
```

:::caution
若您採用非標準方式載入 JS bundle，可能出現 `HermesInternal` 變數存在但未使用高效能預編譯位元組碼的情況。
請確認您使用的是 `.hbc` 檔案，並參照下文進行效能比較測試。
:::

要驗證 Hermes 的優勢，請建立正式版應用進行比較。例如：

```shell
$ npx react-native run-android --mode release
```

或 iOS 平台：

```shell
$ npx react-native run-ios --mode Release
```

此操作會在建置階段將 JavaScript 編譯為位元組碼，從而提升裝置上的應用啟動速度。

## 使用 Chrome DevTools 調試 Hermes 上的 JS

Hermes 透過實作 Chrome 檢測器協議支援 Chrome 調試器。這意味著可直接使用 Chrome 開發者工具來調試運行在模擬器或實體裝置上 Hermes 環境中的 JavaScript。

:::info
請注意，這與[調試章節](debugging#debugging-using-a-custom-javascript-debugger)所述「遠端 JS 調試」（透過應用內開發者選單啟用）有本質差異，後者實際是在開發機（筆電或桌機）的 Chrome V8 引擎上運行 JS 程式碼。
:::

Chrome 透過 Metro 與裝置上的 Hermes 建立連接，因此需知曉 Metro 監聽位置。通常為 `localhost:8081`，但此設定可[調整](https://metrobundler.dev/docs/configuration)。執行 `yarn start` 時，該地址會顯示在啟動時的 stdout 輸出中。

確認 Metro 伺服器監聽位置後，請依以下步驟連接 Chrome：

1. 在 Chrome 瀏覽器中訪問 `chrome://inspect`

2. 點擊 `Configure...` 按鈕新增 Metro 伺服器地址（通常為上述的 `localhost:8081`）

![Chrome DevTools 裝置頁面的設定按鈕](/docs/assets/HermesDebugChromeConfig.png)

![新增 Chrome DevTools 網路目標的對話框](/docs/assets/HermesDebugChromeMetroAddress.png)

3. 現在您應該會看到一個標示為「Hermes React Native」的目標項目，並附有「inspect」連結，點擊即可開啟偵錯工具。若未顯示「inspect」連結，請確認 Metro 伺服器是否正在運作。![目標檢查連結](/docs/assets/HermesDebugChromeInspect.png)

4. 您現在可以使用 Chrome 的偵錯工具。例如，若要在下次執行 JavaScript 時中斷點，請點擊暫停按鈕，並在應用程式中觸發會導致 JavaScript 執行的動作。![偵錯工具中的暫停按鈕](/docs/assets/HermesDebugChromePause.png)

## 在舊版 React Native 中啟用 Hermes

自 React Native 0.70 起，Hermes 已成為預設引擎。本節說明如何在舊版 React Native 中啟用 Hermes。首先，請確保您至少使用 React Native 0.60.4（Android）或 0.64（iOS）版本以啟用 Hermes。

若您的現有應用程式基於較舊的 React Native 版本，您必須先進行升級。請參閱[升級至新版 React Native](/docs/upgrading)了解升級方式。升級應用程式後，請確認一切運作正常，再嘗試切換至 Hermes。

:::caution[React Native 相容性注意事項]
每個 Hermes 版本皆針對特定 RN 版本設計。基本原則是嚴格遵循 [Hermes 發佈版本](https://github.com/facebook/hermes/releases)。
版本不相容可能導致應用程式在最糟情況下立即崩潰。
:::

:::info[Windows 使用者注意事項]
Hermes 需安裝 [Microsoft Visual C++ 2015 Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145)。
:::

### Android

編輯您的 `android/gradle.properties` 檔案，並確認 `hermesEnabled` 設為 true：

```diff
# Use this property to enable or disable the Hermes JS engine.
# If set to false, you will be using JSC instead.
hermesEnabled=true
```

:::note
此屬性於 React Native 0.71 新增。若在您的 `gradle.properties` 檔案中找不到該屬性，請參考您所使用的 React Native 版本對應文件。
:::

此外，若您使用 ProGuard，需在 `proguard-rules.pro` 中加入以下規則：

```
-keep class com.facebook.hermes.unicode.** { *; }
-keep class com.facebook.jni.** { *; }
```

接著，若您已至少建置過應用程式一次，請先清除建置：

```shell
$ cd android && ./gradlew clean
```

完成！您現在應可如常開發與部署應用程式：

```shell
$ npx react-native run-android
```

### iOS

自 React Native 0.64 起，Hermes 亦支援 iOS。若要為 iOS 啟用 Hermes，請編輯您的 `ios/Podfile` 檔案並進行以下變更：

```diff
   use_react_native!(
     :path => config[:reactNativePath],
     # to enable hermes on iOS, change `false` to `true` and then install pods
     # By default, Hermes is disabled on Old Architecture, and enabled on New Architecture.
     # You can enable/disable it manually by replacing `flags[:hermes_enabled]` with `true` or `false`.
-    :hermes_enabled => flags[:hermes_enabled],
+    :hermes_enabled => true
   )
```

預設情況下，若您使用新架構（New Architecture），將會使用 Hermes。透過指定如 `true` 或 `false` 的值，您可自行啟用/停用 Hermes。

設定完成後，您可透過以下指令安裝 Hermes pods：

```shell
$ cd ios && pod install
```

完成！您現在應可如常開發與部署應用程式：

```shell
$ npx react-native run-ios
```

## 切換回 JavaScriptCore

React Native 亦支援使用 JavaScriptCore 作為 [JavaScript 引擎](javascript-environment)。請遵循以下指示以退出 Hermes。

### Android

編輯您的 `android/gradle.properties` 檔案，並將 `hermesEnabled` 設回 false：

```diff
# Use this property to enable or disable the Hermes JS engine.
# If set to false, you will be using JSC instead.
hermesEnabled=false
```

### iOS

編輯您的 `ios/Podfile` 檔案並進行以下變更：

```diff
   use_react_native!(
     :path => config[:reactNativePath],
     # Hermes is now enabled by default. Disable by setting this flag to false.
     # Upcoming versions of React Native may rely on get_default_flags(), but
     # we make it explicit here to aid in the React Native upgrade process.
-    :hermes_enabled => flags[:hermes_enabled],
+    :hermes_enabled => false,
   )
```