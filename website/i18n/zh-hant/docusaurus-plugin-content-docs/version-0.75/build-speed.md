---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會耗費開發者數分鐘的時間。隨著專案規模擴大，特別是在擁有多位 React Native 開發者的大型組織中，這個問題會更加明顯。

為緩解此效能問題，本頁面提供了一些**加速建構時間**的建議。

## 開發期間僅建構單一 ABI（僅限 Android）

當您在本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但若您僅在本地建構並於模擬器或實體裝置上測試，實際上可能不需要建構所有 ABI。

此舉可將**原生建構時間**縮短約 75%。

若使用 React Native CLI，可在 `run-android` 指令中加入 `--active-arch-only` 旗標。此旗標會確保從運行的模擬器或連接的手機中選取正確的 ABI。若此方法運作正常，您將在控制台看到類似 `info Detected architectures arm64-v8a` 的訊息。

```
$ yarn react-native run-android --active-arch-only

[ ... ]
info Running jetifier to migrate libraries to AndroidX. You can disable it using "--no-jetifier" flag.
Jetifier found 1037 file(s) to forward-jetify. Using 32 workers...
info JS server already running.
info Detected architectures arm64-v8a
info Installing the app...
```

此機制依賴於 `reactNativeArchitectures` Gradle 屬性。

因此，若您直接透過命令列使用 Gradle 建構（不使用 CLI），可透過以下方式指定要建構的 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

這在您希望於 CI 上建構 Android 應用程式，並使用矩陣平行化不同架構的建構時特別有用。

您也可以選擇在本機透過專案[頂層目錄](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33)中的 `gradle.properties` 檔案覆寫此值：

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

請注意，當建構**正式版**應用程式時，務必移除這些旗標，因為您需要建構支援所有 ABI 的 apk/app bundle，而非僅限日常開發工作流程中使用的單一 ABI。

## 使用編譯器快取

若您頻繁執行原生建構（無論是 C++ 或 Objective-C），使用**編譯器快取**可能會帶來顯著效益。

具體而言，您可使用兩種類型的快取：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下說明適用於**Android 與 iOS**。
若您僅建構 Android 應用程式，可直接遵循這些指示。
若您也需要建構 iOS 應用程式，請參閱下方的[XCode 特定設定](#xcode-specific-setup)章節。
:::

我們建議使用 [**ccache**](https://ccache.dev/) 來快取原生建構的編譯結果。Ccache 的工作原理是包裝 C++ 編譯器，儲存編譯結果，並在偵測到相同的中間編譯結果時跳過重新編譯。

Ccache 可透過多數作業系統的套件管理器取得。在 macOS 上，可透過 `brew install ccache` 安裝。您也可以遵循[官方安裝說明](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)從原始碼安裝。

接著您可以執行兩次完整建構（例如在 Android 上先執行 `yarn react-native run-android`，刪除 `android/app/build` 資料夾後再次執行相同指令）。您會注意到第二次建構速度大幅提升（應僅需數秒而非數分鐘）。建構過程中，可透過 `ccache -s` 驗證 ccache 是否正常運作並檢查快取命中/遺失率。

```
$ ccache -s
Summary:
  Hits:             196 /  3068 (6.39 %)
    Direct:           0 /  3068 (0.00 %)
    Preprocessed:   196 /  3068 (6.39 %)
  Misses:          2872
    Direct:        3068
    Preprocessed:  2872
  Uncacheable:        1
Primary storage:
  Hits:             196 /  6136 (3.19 %)
  Misses:          5940
  Cache size (GB): 0.60 / 20.00 (3.00 %)
```

請注意，`ccache` 會累積所有建置過程的統計數據。您可以使用 `ccache --zero-stats` 在建置前重置統計數據，以驗證快取命中率。

若需清除快取，可執行 `ccache --clear`。

#### XCode 專屬設定

為確保 `ccache` 在 iOS 和 XCode 環境中正常運作，您需要在 `ios/Podfile` 中啟用 React Native 對 ccache 的支援。

開啟 `ios/Podfile` 並取消註解 `ccache_enabled` 這一行。

```ruby
  post_install do |installer|
    # https://github.com/facebook/react-native/blob/main/packages/react-native/scripts/react_native_pods.rb#L197-L202
    react_native_post_install(
      installer,
      config[:reactNativePath],
      :mac_catalyst_enabled => false,
      # TODO: Uncomment the line below
      :ccache_enabled => true
    )
  end
```

#### 在 CI 上使用此方法

Ccache 在 macOS 上會使用 `/Users/$USER/Library/Caches/ccache` 資料夾來儲存快取。因此，您也可以在 CI 上保存並還原對應的資料夾以加速建置。

但需注意以下幾點：

1. 在 CI 上，建議執行完整的乾淨建置，以避免快取污染問題。若您遵循前文提到的方法，應能將原生建置平行化到 4 種不同的 ABI，這樣在 CI 上很可能就不需要 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中。這在 CI 上效果不佳，因為每次 CI 執行時檔案都會重新下載。為解決此問題，您需要使用 `compiler_check content` 選項，該選項改為[對檔案內容進行雜湊計算](https://ccache.dev/manual/4.3.html)。

### 分散式快取

與本地快取類似，您可能會考慮對原生建置使用分散式快取。這對於經常進行原生建置的大型組織特別有用。

我們建議使用 [sccache](https://github.com/mozilla/sccache) 來實現此目的。有關如何設定和使用此工具的說明，請參考 sccache 的[分散式編譯快速入門指南](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。