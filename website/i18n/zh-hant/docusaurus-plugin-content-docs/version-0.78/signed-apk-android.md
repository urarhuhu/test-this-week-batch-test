---
id: signed-apk-android
title: Publishing to Google Play Store
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

Android 要求所有應用程式在安裝前都必須使用憑證進行數位簽署。若想透過 [Google Play 商店](https://play.google.com/store) 發佈您的 Android 應用程式，必須使用發布金鑰簽署，且後續所有更新都需沿用此金鑰。自 2017 年起，Google Play 可透過 [App Signing by Google Play](https://developer.android.com/studio/publish/app-signing#app-signing-google-play) 功能自動管理簽署發布流程。但在將應用程式二進位檔上傳至 Google Play 前，仍需使用上傳金鑰進行簽署。Android 開發者文件中的 [Signing Your Applications](https://developer.android.com/tools/publishing/app-signing.html) 頁面詳細說明了此主題。本指南將簡要介紹流程，並列出封裝 JavaScript bundle 所需的步驟。

:::info
若您使用 Expo，請參閱 Expo 的 [Deploying to App Stores](https://docs.expo.dev/distribution/app-stores/) 指南，以建置並提交應用程式至 Google Play 商店。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

## 產生上傳金鑰

您可以使用 `keytool` 產生私密簽署金鑰。

### Windows

在 Windows 上，必須以系統管理員身分從 `C:\Program Files\Java\jdkx.x.x_x\bin` 執行 `keytool`。

```shell
keytool -genkeypair -v -storetype PKCS12 -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

此命令會提示您輸入金鑰庫和金鑰的密碼，以及金鑰的 Distinguished Name 欄位。接著會產生名為 `my-upload-key.keystore` 的金鑰庫檔案。

該金鑰庫包含單一金鑰，有效期為 10000 天。別名是後續簽署應用程式時使用的名稱，請務必記下此別名。

### macOS

在 macOS 上，若不確定 JDK bin 資料夾的位置，可執行以下命令查詢：

```shell
/usr/libexec/java_home
```

命令會輸出 JDK 的目錄，格式類似：

```shell
/Library/Java/JavaVirtualMachines/jdkX.X.X_XXX.jdk/Contents/Home
```

使用 `cd /your/jdk/path` 命令切換至該目錄，並以 sudo 權限執行如下所示的 keytool 命令。

```shell
sudo keytool -genkey -v -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

:::caution
請妥善保管金鑰庫檔案。若遺失上傳金鑰或遭他人盜用，應 [遵循這些指示](https://support.google.com/googleplay/android-developer/answer/7384423#reset) 處理。
:::

## 設定 Gradle 變數

1. 將 `my-upload-key.keystore` 檔案置於專案目錄下的 `android/app` 資料夾內。
2. 編輯 `~/.gradle/gradle.properties` 或 `android/gradle.properties` 檔案，並加入以下內容（請將 `*****` 替換為正確的金鑰庫密碼、別名和金鑰密碼）：

```
MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=*****
MYAPP_UPLOAD_KEY_PASSWORD=*****
```

這些將成為全域 Gradle 變數，後續可在 Gradle 配置中用於簽署應用程式。

:::note[關於 git 使用的注意事項]
將上述 Gradle 變數儲存在 `~/.gradle/gradle.properties` 而非 `android/gradle.properties` 可避免其被提交至 git。您可能需要在使用者家目錄中建立 `~/.gradle/gradle.properties` 檔案後才能新增變數。
:::

:::note[關於安全性的注意事項]
若不願以明文儲存密碼，且使用 macOS，可 [將憑證儲存在 Keychain Access 應用程式](https://pilloxa.gitlab.io/posts/safer-passwords-in-gradle/) 中。如此便可省略 `~/.gradle/gradle.properties` 中的最後兩行。
:::

## 將簽署配置加入應用程式的 Gradle 配置

最後一個需要完成的配置步驟是設定使用上傳金鑰來簽署正式版建置。編輯專案目錄中的 `android/app/build.gradle` 檔案，並加入簽署設定：

```groovy
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_UPLOAD_STORE_FILE')) {
                storeFile file(MYAPP_UPLOAD_STORE_FILE)
                storePassword MYAPP_UPLOAD_STORE_PASSWORD
                keyAlias MYAPP_UPLOAD_KEY_ALIAS
                keyPassword MYAPP_UPLOAD_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
...
```

## 產生正式版 AAB

在終端機中執行以下指令：

```shell
npx react-native build-android --mode=release
```

此指令底層使用 Gradle 的 `bundleRelease` 功能，會將應用程式執行所需的所有 JavaScript 程式碼打包至 AAB ([Android App Bundle](https://developer.android.com/guide/app-bundle))。若需變更 JavaScript 套件或可繪製資源的打包方式（例如變更預設檔案/資料夾名稱或專案結構），請參閱 `android/app/build.gradle` 檔案以進行相應調整。

:::note
請確認 `gradle.properties` 中未包含 `org.gradle.configureondemand=true`，否則會導致正式版建置跳過將 JS 與資源打包至應用程式二進位檔的步驟。
:::

產生的 AAB 檔案位於 `android/app/build/outputs/bundle/release/app-release.aab`，可直接上傳至 Google Play。

為讓 Google Play 接受 AAB 格式，需在 Google Play 控制台為應用程式設定「Google Play 應用程式簽署」。若您要更新現有未使用此功能的應用程式，請參閱我們的[遷移章節](#migrating-old-android-react-native-apps-to-use-app-signing-by-google-play)了解如何進行配置變更。

## 測試應用程式的正式版建置

將正式版上傳至 Play 商店前，請務必徹底測試。先解除安裝裝置上所有舊版應用程式，接著在專案根目錄執行以下指令進行安裝：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android -- --mode="release"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android --mode release
```

</TabItem>
</Tabs>

注意：僅有當您已完成上述簽署設定時，才能使用 `--mode release` 參數。

您可以終止所有正在執行的打包程序，因為所有框架與 JavaScript 程式碼都已打包至 APK 的資源中。

## 發布至其他商店

預設情況下，產生的 APK 包含 `x86`、`x86_64`、`ARMv7a` 和 `ARM64-v8a` 四種 CPU 架構的原生程式碼，這使得 APK 能相容於絕大多數 Android 裝置。但此做法的缺點是裝置上會存在未使用的原生程式碼，導致 APK 體積不必要的增大。

您可以在 `android/app/build.gradle` 檔案中加入以下設定，為每種 CPU 架構分別產生 APK：

```diff
android {

    splits {
        abi {
            reset()
            enable true
            universalApk false
            include "armeabi-v7a", "arm64-v8a", "x86", "x86_64"
        }
    }

}
```

將這些檔案上傳至支援裝置篩選的商店（如 [Amazon AppStore](https://developer.amazon.com/docs/app-submission/device-filtering-and-compatibility.html) 或 [F-Droid](https://f-droid.org/en/)），使用者將自動取得對應的 APK。若需上傳至其他不支援多 APK 的商店（如 [APKFiles](https://www.apkfiles.com/)），請將 `universalApk false` 改為 `true` 以產生包含所有 CPU 架構的通用 APK。

請注意，您還需依照 [Android 官方文件建議](https://developer.android.com/studio/build/configure-apk-splits#configure-APK-versions) 設定不同的版本代碼。

## 啟用 Proguard 縮減 APK 體積（選用）

Proguard 是一款能略微縮減 APK 體積的工具，其原理是移除 React Native Java 位元碼（及其依賴項）中未被應用程式使用的部分。

:::caution[重要]
啟用 Proguard 後請務必徹底測試應用程式。Proguard 通常需要針對每個使用的原生函式庫進行特定配置，請參閱 `app/proguard-rules.pro`。
:::

要啟用 Proguard，請編輯 `android/app/build.gradle`：

```groovy
/**
 * Run Proguard to shrink the Java bytecode in release builds.
 */
def enableProguardInReleaseBuilds = true
```

## 遷移舊版 Android React Native 應用程式以使用 Google Play 應用程式簽署

如果您正在從舊版 React Native 遷移，您的應用程式可能尚未使用 Google Play 應用程式簽署功能。我們建議您啟用此功能，以充分利用自動應用程式拆分等優勢。要從舊版簽署方式遷移，您需要先[生成新的上傳金鑰](#generating-an-upload-key)，然後在 `android/app/build.gradle` 中將發佈簽署配置替換為使用上傳金鑰而非發佈金鑰（請參閱[將簽署配置添加到 Gradle](#adding-signing-config-to-your-apps-gradle-config) 部分）。完成後，請按照 [Google Play 幫助網站上的指示](https://support.google.com/googleplay/android-developer/answer/7384423) 將您的原始發佈金鑰發送給 Google Play。

## 預設權限

預設情況下，您的 Android 應用程式會添加 `INTERNET` 權限，因為幾乎所有應用程式都會使用它。`SYSTEM_ALERT_WINDOW` 權限會在偵錯模式下添加到您的 Android APK 中，但在生產環境中會被移除。