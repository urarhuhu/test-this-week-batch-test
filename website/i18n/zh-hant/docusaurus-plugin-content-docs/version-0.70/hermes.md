---
id: hermes
title: Using Hermes
---

<a href="https://hermesengine.dev">
  <img width={300} height={300} className="hermes-logo" src="/docs/assets/HermesLogo.svg" style={{height: "auto"}}/>
</a>

[Hermes](https://hermesengine.dev) 是一個專為 React Native 優化的開源 JavaScript 引擎。對多數應用而言，相較於 JavaScriptCore，使用 Hermes 能帶來更快的啟動速度、更低的記憶體消耗以及更小的應用體積。
React Native 預設已整合 Hermes，無需額外配置即可啟用。

## 內建 Hermes

React Native 附帶**內建版本**的 Hermes。
每當發布新版本 React Native 時，我們都會為您構建對應的 Hermes 版本。這確保您使用的 Hermes 與當前 React Native 版本完全相容。

過去我們曾遇到 Hermes 與 React Native 版本匹配的問題。此方案徹底解決了該問題，為用戶提供與特定 React Native 版本完美適配的 JS 引擎。

此變更對 React Native 用戶完全透明。您仍可透過本頁面描述的指令停用 Hermes。
[點此閱讀技術實現詳情](/architecture/bundled-hermes)。

## 確認 Hermes 已啟用

若您近期新建專案，應能在歡迎畫面查看 Hermes 啟用狀態：

![AwesomeProject 中查看 JS 引擎狀態的位置](/docs/assets/HermesApp.jpg)

JavaScript 環境中會存在全域變量 `HermesInternal` 用於驗證 Hermes 是否啟用：

```jsx
const isHermes = () => !!global.HermesInternal;
```

:::caution
若您使用非標準方式載入 JS 套件包，可能出現 `HermesInternal` 變量存在但未實際使用高效能預編譯位元碼的情況。
請確認您使用的是 `.hbc` 檔案，並參照下文進行效能對比測試。
:::

要驗證 Hermes 的優勢，請建立正式環境版本進行對比測試。例如：

```shell
$ npx react-native run-android --variant release
```

或 iOS 平台：

```shell
$ npx react-native run-ios --configuration Release
```

此操作會在建置階段將 JavaScript 編譯為位元碼，從而提升裝置上的應用啟動速度。

## 使用 Chrome DevTools 調試 Hermes 上的 JS

Hermes 透過實作 Chrome 檢測器協議支援 Chrome 調試器。這意味著可直接使用 Chrome 開發者工具調試運行在 Hermes 上的 JavaScript，無論是模擬器或實體裝置。

:::info
請注意，這與[調試](debugging#debugging-using-a-custom-javascript-debugger)章節記載的「遠端 JS 調試」（透過應用內開發者選單啟用）有本質區別，後者實際是在開發機（筆電或桌機）的 Chrome V8 引擎上運行 JS 代碼。
:::

Chrome 透過 Metro 與裝置上的 Hermes 建立連接，因此需確認 Metro 監聽位置。通常為 `localhost:8081`，但此設定可[配置](https://metrobundler.dev/docs/configuration)。執行 `yarn start` 時，該地址會顯示在啟動時的 stdout 輸出中。

確認 Metro 伺服器監聽位置後，請依以下步驟連接 Chrome：

1. 在 Chrome 瀏覽器訪問 `chrome://inspect`
2. 點擊 `Configure...` 按鈕添加 Metro 伺服器地址（通常為上述的 `localhost:8081`）

![Chrome DevTools 裝置頁面的配置按鈕](/docs/assets/HermesDebugChromeConfig.png)

![添加 Chrome DevTools 網路目標的對話框](/docs/assets/HermesDebugChromeMetroAddress.png)

3. 現在您應該會看到一個標有「Hermes React Native」的目標，並帶有「inspect」連結，可用於啟動偵錯工具。如果沒有看到「inspect」連結，請確認 Metro 伺服器正在運行。![目標檢查連結](/docs/assets/HermesDebugChromeInspect.png)

4. 您現在可以使用 Chrome 偵錯工具。例如，要在下次執行 JavaScript 時設定中斷點，點擊暫停按鈕並觸發應用程式中會導致 JavaScript 執行的動作。![偵錯工具中的暫停按鈕](/docs/assets/HermesDebugChromePause.png)

## 在舊版 React Native 中啟用 Hermes

自 React Native 0.70 起，Hermes 已成為預設引擎。本節說明如何在舊版 React Native 中啟用 Hermes。
首先，請確保您至少使用 React Native 0.60.4 版本以在 Android 上啟用 Hermes，或 React Native 0.64 版本以在 iOS 上啟用 Hermes。

如果您有一個基於較早版本 React Native 的現有應用程式，您需要先升級它。請參閱[升級到新版 React Native](/docs/upgrading)了解如何進行升級。升級應用程式後，請確保一切正常運作，再嘗試切換到 Hermes。

:::caution[React Native 相容性注意事項]
每個 Hermes 版本都針對特定的 RN 版本。一般規則是嚴格遵循 [Hermes 版本發布](https://github.com/facebook/hermes/releases)。
版本不匹配可能導致應用程式在最壞情況下立即崩潰。
:::

:::info[Windows 使用者注意事項]
Hermes 需要 [Microsoft Visual C++ 2015 Redistributable](https://www.microsoft.com/en-us/download/details.aspx?id=48145)。
:::

### Android

編輯您的 `android/app/build.gradle` 檔案並進行如下所示的變更：

```diff
  project.ext.react = [
      entryFile: "index.js",
-     enableHermes: false  // clean and rebuild if changing
+     enableHermes: true  // clean and rebuild if changing
  ]

// ...

if (enableHermes) {
-    def hermesPath = "../../node_modules/hermes-engine/android/";
-    debugImplementation files(hermesPath + "hermes-debug.aar")
-    releaseImplementation files(hermesPath + "hermes-release.aar")
+    //noinspection GradleDynamicVersion
+    implementation("com.facebook.react:hermes-engine:+") { // From node_modules
+        exclude group:'com.facebook.fbjni'
+    }
} else {
```

此外，如果您使用 ProGuard，則需要在 `proguard-rules.pro` 中添加這些規則：

```
-keep class com.facebook.hermes.unicode.** { *; }
-keep class com.facebook.jni.** { *; }
```

接下來，如果您已經至少構建過一次應用程式，請清理構建：

```shell
$ cd android && ./gradlew clean
```

就是這樣！您現在應該能夠像往常一樣開發和部署應用程式：

```shell
$ npx react-native run-android
```

:::note[關於 Android App Bundles 的注意事項]
Android app bundles 從 React Native 0.62 開始支援。
:::

### iOS

自 React Native 0.64 起，Hermes 也可在 iOS 上運行。要為 iOS 啟用 Hermes，請編輯您的 `ios/Podfile` 檔案並進行如下所示的變更：

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

預設情況下，如果您使用新架構，則會使用 Hermes。通過指定如 `true` 或 `false` 的值，您可以根據需要啟用/停用 Hermes。

配置完成後，您可以通過以下命令安裝 Hermes pods：

```shell
$ cd ios && pod install
```

就是這樣！您現在應該能夠像往常一樣開發和部署應用程式：

```shell
$ npx react-native run-ios
```

## 切換回 JavaScriptCore

React Native 也支援使用 JavaScriptCore 作為 [JavaScript 引擎](javascript-environment)。請按照以下說明選擇退出 Hermes。

### Android

編輯您的 `android/app/build.gradle` 檔案並進行如下所示的變更：

```diff
  project.ext.react = [
-     enableHermes: true,  // clean and rebuild if changing
+     enableHermes: false,  // clean and rebuild if changing
  ]
```

### iOS

編輯您的 `ios/Podfile` 檔案並進行如下所示的變更：

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