---
id: debugging-release-builds
title: Debugging Release Builds
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 符號化堆疊追蹤

若 React Native 應用程式在正式版建置中拋出未處理的例外，輸出內容可能會經過混淆而難以閱讀。

```shell
07-15 10:58:25.820 18979 18998 E AndroidRuntime: FATAL EXCEPTION: mqt_native_modules
07-15 10:58:25.820 18979 18998 E AndroidRuntime: Process: com.awesomeproject, PID: 18979 07-15 10:58:25.820 18979 18998 E AndroidRuntime: com.facebook.react.common.JavascriptException: Failed, js engine: hermes, stack:
07-15 10:58:25.820 18979 18998 E AndroidRuntime: p@1:132161
07-15 10:58:25.820 18979 18998 E AndroidRuntime: p@1:132084
07-15 10:58:25.820 18979 18998 E AndroidRuntime: f@1:131854
07-15 10:58:25.820 18979 18998 E AndroidRuntime: anonymous@1:131119
```

在上述堆疊追蹤中，類似 `p@1:132161` 的條目是經過最小化的函式名稱與位元組碼偏移量。要除錯這些呼叫，我們需要將其轉換為檔案、行數與函式名稱，例如 `AwesomeProject/App.js:54:initializeMap`。此過程稱為**符號化**。

您可以透過將堆疊追蹤與生成的原始碼映射傳遞給 [`metro-symbolicate`](http://npmjs.com/package/metro-symbolicate)，來符號化上述的最小化函式名稱與位元組碼。

### 啟用原始碼映射

符號化堆疊追蹤需要原始碼映射。請確認在目標平台的建置設定中已啟用原始碼映射。

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms} className="pill-tabs">
<TabItem value="android">

:::info
On Android, source maps are **enabled** by default.
:::

To enable source map generation, ensure the following `hermesFlags` are present in `android/app/build.gradle`.

```groovy
react {
    hermesFlags = ["-O", "-output-source-map"]
}
```

If done correctly you should see the output location of the source map during Metro build output.

```text
Writing bundle output to:, android/app/build/generated/assets/react/release/index.android.bundle
Writing sourcemap output to:, android/app/build/intermediates/sourcemaps/react/release/index.android.bundle.packager.map
```

</TabItem>
<TabItem value="ios">

:::info
On iOS, source maps are **disabled** by default. Use the following instructions to enable them.
:::

To enable source map generation:

- Open Xcode and edit the build phase "Bundle React Native code and images".
- Above the other exports, add a `SOURCEMAP_FILE` entry with the desired output path.

```diff
+ SOURCEMAP_FILE="$(pwd)/../main.jsbundle.map";
  WITH_ENVIRONMENT="../node_modules/react-native/scripts/xcode/with-environment.sh"
```

If done correctly you should see the output location of the source map during Metro build output.

```text
Writing bundle output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle
Writing sourcemap output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle.map
```

</TabItem>
</Tabs>

### 使用 `metro-symbolicate`

當原始碼映射生成後，我們現在可以轉譯堆疊追蹤。

```shell
# Print usage instructions
npx metro-symbolicate

# From a file containing the stack trace
npx metro-symbolicate android/app/build/generated/sourcemaps/react/release/index.android.bundle.map < stacktrace.txt

# From adb logcat (Android)
adb logcat -d | npx metro-symbolicate android/app/build/generated/sourcemaps/react/release/index.android.bundle.map
```

### 原始碼映射注意事項

- 建置過程可能會生成多個原始碼映射。請確保使用範例中所示位置的對應映射。
- 確保使用的原始碼映射與當機應用程式的確切提交版本相符。原始碼的微小變更可能導致偏移量大幅差異。
- 若 `metro-symbolicate` 立即成功退出，請確認輸入來自管道或重新導向，而非終端機。