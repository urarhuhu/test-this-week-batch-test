---
id: signed-apk-android
title: Publishing to Google Play Store
---

Android要求所有應用在安裝前必須經過數位憑證簽署。若需透過[Google Play商店](https://play.google.com/store)發佈應用，必須使用發布金鑰簽署，且後續更新皆需使用同一金鑰。自2017年起，Google Play可透過[App Signing by Google Play](https://developer.android.com/studio/publish/app-signing#app-signing-google-play)功能自動管理簽署流程。但上傳至Google Play前，應用程式仍需使用上傳金鑰簽署。Android開發者文件中的[應用程式簽署](https://developer.android.com/tools/publishing/app-signing.html)頁面有詳細說明。本指南將簡要介紹流程，並列出封裝JavaScript套件所需的步驟。

:::info
若使用Expo，請參閱Expo的[發佈至應用商店](https://docs.expo.dev/distribution/app-stores/)指南，以建置並提交應用至Google Play商店。本指南適用於任何React Native應用，可自動化部署流程。
:::

## 產生上傳金鑰

您可使用`keytool`產生私密簽署金鑰。

### Windows

在Windows上，需以管理員身分從`C:\Program Files\Java\jdkx.x.x_x\bin`執行`keytool`。

keytool -genkeypair -v -storetype PKCS12 -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

此命令會提示您輸入金鑰庫密碼、金鑰密碼及金鑰的辨別名稱欄位。接著會產生名為`my-upload-key.keystore`的金鑰庫檔案。

該金鑰庫包含單一金鑰，有效期為10000天。別名是後續簽署應用時使用的名稱，請務必記下。

### macOS

在macOS上，若不確定JDK bin目錄位置，可執行以下命令查詢：

/usr/libexec/java_home

該命令會輸出JDK目錄，路徑類似：

/Library/Java/JavaVirtualMachines/jdkX.X.X_XXX.jdk/Contents/Home

使用`cd /your/jdk/path`命令切換至該目錄，並以sudo權限執行如下keytool命令。

sudo keytool -genkey -v -keystore my-upload-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000

:::caution
請妥善保管金鑰庫檔案。若遺失上傳金鑰或遭洩露，應[遵循此指引](https://support.google.com/googleplay/android-developer/answer/7384423#reset)處理。
:::

## 設定Gradle變數

1. 將`my-upload-key.keystore`檔案置於專案目錄的`android/app`下。
2. 編輯`~/.gradle/gradle.properties`或`android/gradle.properties`檔案，加入以下內容（請將`*****`替換為正確的金鑰庫密碼、別名及金鑰密碼）：

```
MYAPP_UPLOAD_STORE_FILE=my-upload-key.keystore
MYAPP_UPLOAD_KEY_ALIAS=my-key-alias
MYAPP_UPLOAD_STORE_PASSWORD=*****
MYAPP_UPLOAD_KEY_PASSWORD=*****
```

這些將作為全域Gradle變數，後續可用於Gradle配置中以簽署應用。

:::note[關於git使用的注意事項]
將上述Gradle變數儲存在`~/.gradle/gradle.properties`而非`android/gradle.properties`中，可避免其被提交至git。您可能需先在用戶家目錄建立`~/.gradle/gradle.properties`檔案才能新增變數。
:::

:::note[安全性注意事項]
若您不傾向以明文儲存密碼，且使用 macOS 系統，可改為[將憑證儲存在 Keychain Access 應用程式](https://pilloxa.gitlab.io/posts/safer-passwords-in-gradle/)。如此便能省略 `~/.gradle/gradle.properties` 檔案中的最後兩行設定。
:::

## 為應用程式的 Gradle 設定添加簽署配置

最後需完成的配置步驟是設定正式版建構使用上傳金鑰進行簽署。編輯專案目錄中的 `android/app/build.gradle` 檔案，並添加簽署配置：

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

在終端機執行以下指令：

```shell
cd android
./gradlew bundleRelease
```

Gradle 的 `bundleRelease` 會將應用程式運作所需的所有 JavaScript 程式碼打包至 AAB ([Android App Bundle](https://developer.android.com/guide/app-bundle))。若需調整 JavaScript 套件或繪圖資源的打包方式（例如變更預設檔案/目錄名稱或專案結構），請參閱 `android/app/build.gradle` 檔案以更新對應設定。

:::note
請確認 `gradle.properties` 未包含 `org.gradle.configureondemand=true` 設定，否則正式版建構會略過將 JS 與資源打包至應用程式二進位檔的步驟。
:::

產生的 AAB 檔案位於 `android/app/build/outputs/bundle/release/app-release.aab`，可直接上傳至 Google Play。

Google Play 需為您的應用程式設定「Google Play 應用程式簽署」功能才能接受 AAB 格式。若您要更新尚未採用此功能的既有應用程式，請參閱我們的[遷移章節](#migrating-old-android-react-native-apps-to-use-app-signing-by-google-play)了解如何進行配置變更。

## 測試應用程式的正式版建構

將正式版上傳至 Play 商店前，請務必徹底測試。先解除安裝裝置上所有舊版應用程式，再於專案根目錄執行以下指令進行安裝：

```shell
npx react-native run-android --mode=release
```

注意：僅有當您完成前述簽署設定後，才能使用 `--mode release` 參數。

您可以終止所有正在執行的打包程序，因為所有框架與 JavaScript 程式碼都已打包至 APK 的資源中。

## 發布至其他商店

預設產生的 APK 會包含 x86 與 ARMv7a 兩種 CPU 架構的原生程式碼，這能讓 APK 相容於絕大多數 Android 裝置，但代價是裝置上會存在未使用的原生程式碼，導致 APK 檔案大小不必要地增加。

您可以透過修改 android/app/build.gradle 中的以下設定，為每種 CPU 架構分別產生 APK：

```diff
- ndk {
-   abiFilters "armeabi-v7a", "x86"
- }
- def enableSeparateBuildPerCPUArchitecture = false
+ def enableSeparateBuildPerCPUArchitecture = true
```

將這兩個檔案上傳至支援裝置定向的市集（如 [Google Play](https://developer.android.com/google/play/publishing/multiple-apks.html) 和 [Amazon AppStore](https://developer.amazon.com/docs/app-submission/device-filtering-and-compatibility.html)），使用者便會自動取得對應的 APK。若要上傳至其他市集（如 [APKFiles](https://www.apkfiles.com/)）這類不支援單一應用程式多 APK 的平台，請同時修改以下設定以產生包含雙 CPU 架構的通用 APK。

```diff
- universalApk false  // If true, also generate a universal APK
+ universalApk true  // If true, also generate a universal APK
```

## 啟用 Proguard 縮減 APK 大小（選用）

Proguard 工具能透過移除應用程式未使用的 React Native Java 位元碼（及其依賴項），略微縮減 APK 檔案大小。

:::caution[重要]
若您已啟用 Proguard，請務必徹底測試您的應用程式。Proguard 通常需要針對您使用的每個原生函式庫進行特定配置。請參閱 `app/proguard-rules.pro`。
:::

要啟用 Proguard，請編輯 `android/app/build.gradle`：

```groovy
/**
 * Run Proguard to shrink the Java bytecode in release builds.
 */
def enableProguardInReleaseBuilds = true
```

## 將舊版 Android React Native 應用程式遷移至使用 Google Play 應用程式簽署

若您正從舊版 React Native 遷移，您的應用程式可能尚未使用 Google Play 應用程式簽署功能。我們建議您啟用此功能以充分利用自動應用程式拆分等優勢。要從舊版簽署方式遷移，您需先[產生新的上傳金鑰](#generating-an-upload-key)，接著在 `android/app/build.gradle` 中將發佈簽署配置替換為使用上傳金鑰（請參閱[將簽署配置加入 Gradle](#adding-signing-config-to-your-apps-gradle-config) 章節）。完成後，請依照 [Google Play 說明網站](https://support.google.com/googleplay/android-developer/answer/7384423)的指示，將原始發佈金鑰傳送給 Google Play。

## 預設權限

預設情況下，您的 Android 應用程式會加入 `INTERNET` 權限，因絕大多數應用程式皆需使用。`SYSTEM_ALERT_WINDOW` 權限會在偵錯模式中加入至您的 Android APK，但在正式環境中會被移除。