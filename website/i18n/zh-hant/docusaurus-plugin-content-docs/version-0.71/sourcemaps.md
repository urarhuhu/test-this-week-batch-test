---
id: sourcemaps
title: Source Maps
---

Source maps（原始碼映射）能將轉換後的檔案映射回原始碼檔案，主要用於輔助除錯及協助調查正式版建置中的問題。

若無原始碼映射，當正式版建置發生錯誤時，您將看到類似以下的堆疊追蹤：

```text
TypeError: Cannot read property 'data' of undefined
  at anonymous(app:///index.android.bundle:1:4277021)
  at call(native)
  at p(app:///index.android.bundle:1:227785)
```

若已生成原始碼映射，堆疊追蹤將包含原始碼檔案的路徑、檔名及行號：

```text
TypeError: Cannot read property 'data' of undefined
  at anonymous(src/modules/notifications/Permission.js:15:requestNotificationPermission)
  at call(native)
  at p(node_modules/regenerator-runtime/runtime.js:64:Generator)
```

This allows to triage release issues given a more accurate stacktrace.

請遵循以下指示開始使用原始碼映射。

## 在 Android 啟用原始碼映射

### Hermes 引擎

:::info
原始碼映射預設會在 Release 模式中建置。但若設定了 `hermesFlagsRelease` 參數，則需手動啟用原始碼映射功能。
:::

若有此需求，請確認您應用程式的 `android/app/build.gradle` 檔案中包含以下設定：

```groovy
project.ext.react = [
    enableHermes: true,
    hermesFlagsRelease: ["-O", "-output-source-map"], // plus whichever flag was required to set this away from default
]
```

若設定正確，您可在 Metro 建置輸出中看到原始碼映射的輸出位置。

```text
Writing bundle output to:, android/app/build/generated/assets/react/release/index.android.bundle
Writing sourcemap output to:, android/app/build/intermediates/sourcemaps/react/release/index.android.bundle.packager.map
```

開發版建置本身不會產生 bundle 且已包含符號資訊，但若開發版建置需打包 bundle，可參照前述方式使用 `hermesFlagsDebug` 啟用原始碼映射。

## 在 iOS 啟用原始碼映射

原始碼映射預設為關閉狀態。需定義 `SOURCEMAP_FILE` 環境變數來啟用。

操作方式：於 Xcode 中進入建置階段 - "Bundle React Native code and images"。

在檔案頂部其他 export 指令附近，加入 `SOURCEMAP_FILE` 設定並指定輸出路徑與名稱，如下所示：

```
export SOURCEMAP_FILE="$(pwd)/../main.jsbundle.map";

export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

若設定正確，您可在 Metro 建置輸出中看到原始碼映射的輸出位置。

```text
Writing bundle output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle
Writing sourcemap output to:, Build/Intermediates.noindex/ArchiveIntermediates/application/BuildProductsPath/Release-iphoneos/main.jsbundle.map
```

## 手動符號解析

:::info
您可於 [符號解析](symbolication.md) 頁面閱讀手動解析堆疊追蹤的相關說明。
:::