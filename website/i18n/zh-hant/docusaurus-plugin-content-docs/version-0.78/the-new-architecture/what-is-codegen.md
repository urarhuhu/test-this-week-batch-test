# 什麼是 Codegen？

**Codegen** 是一個用來避免撰寫大量重複程式碼的工具。使用 **Codegen** 並非強制性的：你可以手動撰寫所有生成的程式碼。然而，**Codegen** 能生成框架程式碼，為你節省大量時間。

React Native 在每次建置 iOS 或 Android 應用程式時會自動調用 **Codegen**。偶爾，你可能會想手動執行 **Codegen** 腳本，以了解實際生成的類型和檔案：這在開發 Turbo Native Modules 和 Fabric Native Components 時是常見的情境。

<!-- TODO: Add links to TM and FC -->

## Codegen 的運作方式

**Codegen** 是一個與 React Native 應用程式緊密耦合的流程。**Codegen** 腳本位於 `react-native` NPM 套件中，應用程式在建置時會調用這些腳本。

**Codegen** 會從你在 `package.json` 中指定的目錄開始，爬取專案中的資料夾，尋找包含自訂模組和元件規格（或稱 specs）的特定 JS 檔案。規格檔案是以類型化方言撰寫的 JS 檔案：React Native 目前支援 Flow 和 TypeScript。

每當 **Codegen** 找到一個規格檔案時，它會生成與之相關的樣板程式碼。**Codegen** 會生成一些 C++ 黏合程式碼，然後針對特定平台生成程式碼，Android 使用 Java，iOS 則使用 Objective-C++。