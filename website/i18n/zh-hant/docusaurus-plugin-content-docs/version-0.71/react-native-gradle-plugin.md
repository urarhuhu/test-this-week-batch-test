---
id: react-native-gradle-plugin
title: React Native Gradle Plugin
---

本指南說明如何配置 **React Native Gradle 插件**（通常簡稱為 RNGP），以便為 Android 構建 React Native 應用程式。

## 使用插件

React Native Gradle 插件作為獨立的 NPM 套件分發，會隨 `react-native` 自動安裝。

使用 `npx react-native init` 創建的新專案已**預設配置**該插件。若您以此指令創建應用，無需進行額外安裝步驟。

若需將 React Native 整合至現有專案，請參閱[對應頁面](/docs/next/integration-with-existing-apps#configuring-gradle)，其中包含插件安裝的具體指引。

## 配置插件

預設情況下，插件將以**開箱即用**的合理配置運作。僅在需要時才參考本指南進行客製化調整。

您可透過修改 `android/app/build.gradle` 中的 `react` 區塊來配置插件：

```groovy
apply plugin: "com.facebook.react"

/**
 * This is the configuration block to customize your React Native Android app.
 * By default you don't need to apply any configuration, just uncomment the lines you need.
 */
react {
  // Custom configuration goes here.
}
```

各配置參數說明如下：

### `root`

此為 React Native 專案的根目錄（即存放 `package.json` 的資料夾）。預設值為 `..`。可依需求調整：

```groovy
root = file("../")
```

### `reactNativeDir`

此為 `react-native` 套件的存放路徑。預設值為 `../node_modules/react-native`。
若您使用 monorepo 或其他套件管理器，可依環境調整 `reactNativeDir`。

客製化範例：

```groovy
reactNativeDir = file("../node_modules/react-native")
```

### `codegenDir`

此為 `react-native-codegen` 套件的存放路徑。預設值為 `../node_modules/react-native-codegen`。
若使用 monorepo 或其他套件管理器，請相應調整 `codegenDir`。

客製化範例：

```groovy
reactNativeDir = file("../node_modules/@react-native/codegen")
```

### `cliFile`

此為 React Native CLI 的入口檔案。預設值為 `../node_modules/react-native/cli.js`。
插件需透過此檔案調用 CLI 進行應用打包與構建。

若使用 monorepo 或其他套件管理器，請調整 `cliFile` 路徑。
客製化範例：

```groovy
cliFile = file("../node_modules/react-native/cli.js")
```

### `debuggableVariants`

此為可調試變體清單（關於變體的詳細說明請參見[使用變體](#using-variants)）。

預設情況下，插件僅將 `debug` 變體視為可調試，而 `release` 則否。若有其他變體（如 `staging`、`lite` 等），需相應調整此設定。

列為 `debuggableVariants` 的變體不會包含預編譯套件包，需依賴 Metro 運行。

客製化範例：

```groovy
debuggableVariants = ["liteDebug", "prodDebug"]
```

### `nodeExecutableAndArgs`

此為執行所有腳本時調用的 node 指令與參數清單。預設值為 `[node]`，可添加額外參數：

```groovy
nodeExecutableAndArgs = ["node"]
```

### `bundleCommand`

這是建立應用程式套件時要呼叫的 `bundle` 指令名稱。若您使用 [RAM Bundles](https://reactnative.dev/docs/0.74/ram-bundles-inline-requires) 時會很有用。預設為 `bundle`，但可透過以下方式自訂並添加額外參數：

```groovy
bundleCommand = "ram-bundle"
```

### `bundleConfig`

此為傳遞給 `bundle --config <file>` 的設定檔案路徑。預設為空（不提供設定檔）。更多關於套件設定檔的資訊可參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。可透過以下方式自訂：

```groovy
bundleConfig = file(../rn-cli.config.js)
```

### `bundleAssetName`

此為應生成的套件檔案名稱。預設為 `index.android.bundle`。可透過以下方式自訂：

```groovy
bundleAssetName = "MyApplication.android.bundle"
```

### `entryFile`

用於套件生成的入口檔案。預設會搜尋 `index.android.js` 或 `index.js`。可透過以下方式自訂：

```groovy
entryFile = file("../js/MyApplication.android.js")
```

### `extraPackagerArgs`

傳遞給 `bundle` 指令的額外參數列表。可用參數清單請參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。預設為空。可透過以下方式自訂：

```groovy
extraPackagerArgs = []
```

### `hermesCommand`

`hermesc` 指令（Hermes 編譯器）的路徑。React Native 自帶 Hermes 編譯器版本，因此通常無需自訂此項。插件預設會為您的系統使用正確的編譯器。

### `hermesFlags`

傳遞給 `hermesc` 的參數列表。預設為 `["-O", "-output-source-map"]`。可透過以下方式自訂：

```groovy
hermesFlags = ["-O", "-output-source-map"]
```

## 使用風味與建置變體

建置 Android 應用時，您可能會想使用 [自訂風味](https://developer.android.com/studio/build/build-variants#product-flavors) 來從同一專案產生不同版本的應用。

請參閱 [官方 Android 指南](https://developer.android.com/studio/build/build-variants) 來設定自訂建置類型（如 `staging`）或自訂風味（如 `full`、`lite` 等）。
新應用預設會建立兩種建置類型（`debug` 和 `release`）且無自訂風味。

所有建置類型與風味的組合會生成一組**建置變體**。例如 `debug`/`staging`/`release` 建置類型與 `full`/`lite` 風味會產生 6 種建置變體：`fullDebug`、`fullStaging`、`fullRelease` 等。

若您使用 `debug` 和 `release` 以外的自訂變體，需透過 [`debuggableVariants`](#debuggablevariants) 設定指示 React Native Gradle 插件哪些變體為**可偵錯**：

```diff
apply plugin: "com.facebook.react"

react {
+ debuggableVariants = ["fullStaging", "fullDebug"]
}
```

此設定必要的原因是插件會跳過所有 `debuggableVariants` 的 JS 套件生成：您需要 Metro 來執行它們。例如若將 `fullStaging` 列入 `debuggableVariants`，將無法發布至商店，因該版本會缺少套件。

## 插件在底層做了什麼？

React Native Gradle 插件負責設定您的應用建置，以將 React Native 應用發布至生產環境。
該插件也用於第三方函式庫中，以執行新架構所需的 [Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。

以下是該插件的主要職責概述：

- 為每個不可調試的變體添加 `createBundle<Variant>JsAndAssets` 任務，負責調用 `bundle`、`hermesc` 和 `compose-source-map` 命令。
- 根據 `react-native` 的 `package.json` 設置正確版本的 `com.facebook.react:react-android` 和 `com.facebook.react:hermes-android` 依賴項。
- 配置必要的 Maven 倉庫（如 Maven Central、Google Maven Repo、JSC 本地 Maven 倉庫等）以獲取所有必需的 Maven 依賴項。
- 設置 NDK 以支持使用新架構（New Architecture）的應用程式建置。
- 配置 `buildConfigFields`，讓您能在運行時判斷是否啟用了 Hermes 或新架構。
- 將 Metro 開發伺服器端口設置為 Android 資源，使應用程式知道連接的端口。
- 若函式庫或應用程式使用新架構的 Codegen，則調用 [React Native Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。