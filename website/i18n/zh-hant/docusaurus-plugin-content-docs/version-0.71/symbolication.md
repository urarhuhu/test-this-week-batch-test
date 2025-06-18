---
id: symbolication
title: Symbolicating a stack trace
---

若 React Native 應用程式在正式版本中拋出未處理的例外，輸出內容可能經過混淆而難以閱讀：

```shell
07-15 10:58:25.820 18979 18998 E AndroidRuntime: FATAL EXCEPTION: mqt_native_modules
07-15 10:58:25.820 18979 18998 E AndroidRuntime: Process: com.awesomeproject, PID: 18979 07-15 10:58:25.820 18979 18998 E AndroidRuntime: com.facebook.react.common.JavascriptException: Failed, js engine: hermes, stack:
07-15 10:58:25.820 18979 18998 E AndroidRuntime: p@1:132161
07-15 10:58:25.820 18979 18998 E AndroidRuntime: p@1:132084
07-15 10:58:25.820 18979 18998 E AndroidRuntime: f@1:131854
07-15 10:58:25.820 18979 18998 E AndroidRuntime: anonymous@1:131119
```

區段如 `p@1:132161` 是經過最小化的函式名稱與位元組碼偏移量。若要偵錯此問題，您會希望將其轉換為檔案、行號與函式名稱格式：`AwesomeProject/App.js:54:initializeMap`。此過程稱為**符號化 (symbolication)**。您可以透過將產生的原始碼對應檔 (source map) 與堆疊追蹤傳遞給 `metro-symbolicate`，來對上述最小化函式名稱與位元組碼進行符號化。

> `metro-symbolicate` 套件預設已安裝於 React Native 範本專案中（參見[設定開發環境](environment-setup)）。

從包含堆疊追蹤的檔案：

```shell
npx metro-symbolicate android/app/build/generated/sourcemaps/react/release/index.android.bundle.map < stacktrace.txt
```

直接從 `adb logcat`：

```shell
adb logcat -d | npx metro-symbolicate android/app/build/generated/sourcemaps/react/release/index.android.bundle.map
```

此操作會將每個最小化函式名稱與偏移量（如 `p@1:132161`）轉換為實際的檔案與函式名稱（如 `AwesomeProject/App.js:54:initializeMap`）。

## 原始碼對應檔注意事項

- 建置過程可能產生多個原始碼對應檔，請確保使用範例中所示位置的對應檔。
- 請確認使用的原始碼對應檔與當機應用程式的提交版本完全對應，原始碼的微小變更可能導致偏移量大幅差異。
- 若 `metro-symbolicate` 立即成功退出，請確認輸入來源為管道或重新導向，而非終端機直接輸入。