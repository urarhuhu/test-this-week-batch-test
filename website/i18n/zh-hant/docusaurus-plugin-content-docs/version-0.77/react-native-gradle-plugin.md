---
id: react-native-gradle-plugin
title: React Native Gradle Plugin
---

本指南說明如何配置 **React Native Gradle 插件**（通常簡稱為 RNGP），以建構 Android 平台的 React Native 應用程式。

## 使用插件

React Native Gradle 插件以獨立的 NPM 套件形式發佈，會隨 `react-native` 自動安裝。

透過 `npx react-native init` 建立的新專案已**預設配置**此插件。若以此指令建立應用程式，無需進行額外安裝步驟。

若需將 React Native 整合至現有專案，請參閱[對應頁面](/docs/next/integration-with-existing-apps#configuring-gradle)，該文件包含插件安裝的具體指引。

## 配置插件

插件預設採用**開箱即用**的合理配置。僅在需要調整行為時，才需參考本指南進行自訂。

配置插件需修改 `android/app/build.gradle` 中的 `react` 區塊：

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

此參數指向 React Native 專案的根目錄（即包含 `package.json` 的目錄）。預設值為 `..`。可依下列方式自訂：

```groovy
root = file("../")
```

### `reactNativeDir`

此參數指向 `react-native` 套件的安裝目錄。預設值為 `../node_modules/react-native`。
若處於 monorepo 環境或使用不同套件管理器，可依專案結構調整此路徑。

自訂範例如下：

```groovy
reactNativeDir = file("../node_modules/react-native")
```

### `codegenDir`

此參數指向 `react-native-codegen` 套件的安裝目錄。預設值為 `../node_modules/react-native-codegen`。
若處於 monorepo 環境或使用不同套件管理器，可依專案結構調整此路徑。

自訂範例如下：

```groovy
codegenDir = file("../node_modules/@react-native/codegen")
```

### `cliFile`

此參數指向 React Native CLI 的入口檔案。預設值為 `../node_modules/react-native/cli.js`。
插件需透過此入口檔案呼叫 CLI，以執行應用程式的打包與建立。

若處於 monorepo 環境或使用不同套件管理器，可依專案結構調整此路徑。
自訂範例如下：

```groovy
cliFile = file("../node_modules/react-native/cli.js")
```

### `debuggableVariants`

此參數列出可偵錯的變體（關於變體的詳細說明，請參閱[使用變體](#using-variants)章節）。

插件預設僅將 `debug` 變體視為可偵錯，`release` 則否。若存在其他變體（如 `staging`、`lite` 等），需相應調整此清單。

列為 `debuggableVariants` 的變體不會包含預打包的 bundle，需依賴 Metro 執行。

自訂範例如下：

```groovy
debuggableVariants = ["liteDebug", "prodDebug"]
```

### `nodeExecutableAndArgs`

此參數指定執行所有腳本時使用的 node 指令與參數。預設值為 `[node]`，可依下列方式添加額外參數：

```groovy
nodeExecutableAndArgs = ["node"]
```

### `bundleCommand`

這是在為應用程式建立打包檔案時要調用的 `bundle` 指令名稱。若您使用 [RAM Bundles](https://reactnative.dev/docs/0.74/ram-bundles-inline-requires) 時會很有用。預設為 `bundle`，但可透過以下方式自訂以添加額外參數：

```groovy
bundleCommand = "ram-bundle"
```

### `bundleConfig`

這是傳遞給 `bundle --config <file>` 的設定檔案路徑（若提供）。預設為空（不提供設定檔）。更多關於打包設定檔的資訊可參閱 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。可透過以下方式自訂：

```groovy
bundleConfig = file(../rn-cli.config.js)
```

### `bundleAssetName`

這是應生成的打包檔案名稱。預設為 `index.android.bundle`。可透過以下方式自訂：

```groovy
bundleAssetName = "MyApplication.android.bundle"
```

### `entryFile`

用於打包生成的入口檔案。預設會搜尋 `index.android.js` 或 `index.js`。可透過以下方式自訂：

```groovy
entryFile = file("../js/MyApplication.android.js")
```

### `extraPackagerArgs`

傳遞給 `bundle` 指令的額外參數列表。可用參數清單請見 [CLI 文件](https://github.com/react-native-community/cli/blob/main/docs/commands.md#bundle)。預設為空。可透過以下方式自訂：

```groovy
extraPackagerArgs = []
```

### `hermesCommand`

`hermesc` 指令（Hermes 編譯器）的路徑。React Native 自帶了 Hermes 編譯器版本，因此通常無需自訂此項。插件預設會為您的系統使用正確的編譯器。

### `hermesFlags`

傳遞給 `hermesc` 的參數列表。預設為 `["-O", "-output-source-map"]`。可透過以下方式自訂：

```groovy
hermesFlags = ["-O", "-output-source-map"]
```

## 使用風味與建置變體

在建置 Android 應用程式時，您可能會想使用 [自訂風味](https://developer.android.com/studio/build/build-variants#product-flavors) 來從同一專案產生不同版本的應用程式。

請參閱 [官方 Android 指南](https://developer.android.com/studio/build/build-variants) 來設定自訂建置類型（如 `staging`）或自訂風味（如 `full`、`lite` 等）。
新建立的應用程式預設會有兩種建置類型（`debug` 和 `release`）且無自訂風味。

所有建置類型與風味的組合會生成一組**建置變體**。例如對於 `debug`/`staging`/`release` 建置類型和 `full`/`lite` 風味，您將有 6 個建置變體：`fullDebug`、`fullStaging`、`fullRelease` 等。

若您使用 `debug` 和 `release` 以外的自訂變體，需透過 [`debuggableVariants`](#debuggablevariants) 設定指示 React Native Gradle 插件哪些變體是**可除錯的**，如下所示：

```diff
apply plugin: "com.facebook.react"

react {
+ debuggableVariants = ["fullStaging", "fullDebug"]
}
```

這是必要的，因為插件會跳過所有 `debuggableVariants` 的 JS 打包：您需要 Metro 來執行它們。例如，若您在 `debuggableVariants` 中列出 `fullStaging`，將無法將其發布到商店，因為它會缺少打包檔案。

## 插件在底層做了什麼？

React Native Gradle 插件負責設定您的應用程式建置，以將 React Native 應用程式發布至生產環境。
該插件也用於第三方函式庫中，以執行新架構所需的 [Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。

以下是該插件的主要職責概述：

- 為每個非可調試變體添加 `createBundle<Variant>JsAndAssets` 任務，負責調用 `bundle`、`hermesc` 和 `compose-source-map` 命令。
- 根據 `react-native` 的 `package.json` 設置正確版本的 `com.facebook.react:react-android` 和 `com.facebook.react:hermes-android` 依賴項。
- 配置必要的 Maven 倉庫（如 Maven Central、Google Maven Repo、JSC 本地 Maven 倉庫等），以獲取所有必需的 Maven 依賴項。
- 設置 NDK，讓您能夠構建使用新架構的應用程式。
- 配置 `buildConfigFields`，以便在運行時判斷是否啟用了 Hermes 或新架構。
- 將 Metro 開發伺服器端口設置為 Android 資源，讓應用程式知道連接的端口。
- 如果函式庫或應用程式使用新架構的代碼生成器（Codegen），則調用 [React Native Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md)。