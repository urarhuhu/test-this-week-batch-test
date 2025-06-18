# Codegen 命令行工具

手動調用 Gradle 或執行腳本可能難以記憶且流程繁瑣。

為簡化流程，我們開發了 **Codegen** 命令行工具來協助執行這些任務。此命令會為您的專案執行 [@react-native/codegen](https://www.npmjs.com/package/@react-native/codegen)，提供以下選項：

```sh
npx @react-native-community/cli codegen --help
Usage: rnc-cli codegen [options]

Options:
  --verbose            Increase logging verbosity
  --path <path>        Path to the React Native project root. (default: "/Users/MyUsername/projects/my-app")
  --platform <string>  Target platform. Supported values: "android", "ios", "all". (default: "all")
  --outputPath <path>  Path where generated artifacts will be output to.
  -h, --help           display help for command
```

## 範例

- 從當前工作目錄讀取 `package.json`，依據其 codegenConfig 生成程式碼。

```shell
npx @react-native-community/cli codegen
```

- 從當前工作目錄讀取 `package.json`，在 codegenConfig 定義的位置生成 iOS 程式碼。

```shell
npx @react-native-community/cli codegen --platform ios
```

- 從 `third-party/some-library` 讀取 `package.json`，在 `third-party/some-library/android/generated` 生成 Android 程式碼。

```shell
npx @react-native-community/cli codegen \
    --path third-party/some-library \
    --platform android \
    --outputPath third-party/some-library/android/generated
```

# 將生成程式碼整合至函式庫

Codegen 命令行工具對函式庫開發者極為實用，可預覽生成程式碼以確認需實作的介面。

通常生成程式碼不會包含在函式庫中，而是由使用該函式庫的應用程式在建置時執行 Codegen。此模式適用於多數情境，但 Codegen 也提供透過 `includesGeneratedCode` 屬性將生成程式碼直接打包進函式庫的機制。

需特別理解啟用 `includesGeneratedCode = true` 的影響。內嵌生成程式碼具有以下優勢：

- 無需依賴應用程式執行 **Codegen**，生成程式碼已預先存在
- 實作檔案始終與生成介面保持一致（增強函式庫程式碼對 Codegen API 變更的相容性）
- Android 無需維護兩套檔案支援新舊架構，僅需保留新架構版本即可確保向後相容
- 由於所有原生程式碼已內含，可將函式庫原生部分預先編譯後發布

但同時需注意一項限制：

- 生成程式碼將使用函式庫內定義的 React Native 版本。例如若函式庫基於 React Native 0.76 開發，生成程式碼將對應此版本，可能導致與使用**較舊** React Native 版本的應用程式（如運行於 0.75 版）不相容。

## 啟用 `includesGeneratedCode`

啟用步驟如下：

- 在函式庫 `package.json` 的 `codegenConfig` 欄位新增 `includesGeneratedCode` 屬性並設為 `true`
- 使用 Codegen 命令行工具在本機執行 **Codegen**
- 更新 `package.json` 以包含生成程式碼
- 更新 `podspec` 以包含生成程式碼
- 更新 `build.Gradle` 檔案以包含生成程式碼
- 在 `react-native.config.js` 中修改 `cmakeListsPath`，使 Gradle 從 outputDir 而非建置目錄尋找 CMakeLists 檔案