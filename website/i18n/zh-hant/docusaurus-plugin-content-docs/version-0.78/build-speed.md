---
id: build-speed
title: Speeding up your Build phase
---

建構 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。隨著專案規模增長，特別是在擁有多位 React Native 開發者的大型組織中，這個問題會更加明顯。

為緩解此效能問題，本頁面提供數項**提升建構速度**的建議。

## 開發期間僅建構單一 ABI (僅限 Android)

本地建構 Android 應用時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

但若僅需在本地建構並於模擬器或實體裝置測試，您可能無需建構全部架構。

此舉可將**原生建構時間**縮短約 75%。

若使用 React Native CLI，可在 `run-android` 指令加入 `--active-arch-only` 參數。此參數會自動偵測運行中模擬器或連接裝置的 ABI 架構。成功時，控制台將顯示如 `info Detected architectures arm64-v8a` 的確認訊息。

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

因此若直接透過 Gradle 指令列建構（不使用 CLI），可透過以下方式指定目標 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

此方式適用於在 CI 環境建構 Android 應用時，透過矩陣平行處理不同架構的建構任務。

您亦可於本地專案[頂層目錄](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33)的 `gradle.properties` 檔案覆寫此值：

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

請注意建構**正式版**應用時，務必移除此參數，以確保產出的 apk/app bundle 支援所有 ABI 架構，而非僅限日常開發使用的單一架構。

## 使用編譯器快取

若需頻繁執行原生建構（C++ 或 Objective-C），使用**編譯器快取**將能帶來顯著效益。

具體而言，可採用兩類快取：本地編譯器快取與分散式編譯器快取。

### 本地快取

:::info
以下說明同時適用於 **Android 與 iOS** 建構。
若僅建構 Android 應用，直接依指示操作即可。
若需建構 iOS 應用，請參閱下方 [XCode 專屬設定](#xcode-specific-setup)章節。
:::

建議使用 [**ccache**](https://ccache.dev/) 快取原生建構的編譯結果。Ccache 透過封裝 C++ 編譯器運作，儲存編譯結果並在偵測到相同的中間編譯結果時直接跳過編譯階段。

多數作業系統的套件管理器皆提供 Ccache。在 macOS 可透過 `brew install ccache` 安裝，或參閱[官方安裝指南](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)從原始碼編譯安裝。

您可執行兩次完整建構（例如 Android 專案先執行 `yarn react-native run-android`，刪除 `android/app/build` 資料夾後再次執行）。會發現第二次建構速度大幅提升（耗時應從分鐘級縮短至秒級）。建構過程中可透過 `ccache -s` 指令驗證快取命中率。

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

Note that `ccache` aggregates the stats over all builds. You can use `ccache --zero-stats` to reset them before a build to verify the cache-hit ratio.

Should you need to wipe your cache, you can do so with `ccache --clear`

#### XCode Specific Setup

To make sure `ccache` works correctly with iOS and XCode, you need to enable React Native support for ccache in `ios/Podfile`.

Open `ios/Podfile` in your editor and uncomment the `ccache_enabled` line.

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

#### Using this approach on a CI

Ccache uses the `/Users/$USER/Library/Caches/ccache` folder on macOS to store the cache.
Therefore you could save & restore the corresponding folder also on CI to speedup your builds.

However, there are a couple of things to be aware:

1. On CI, we recommend to do a full clean build, to avoid poisoned cache problems. If you follow the approach mentioned in the previous paragraph, you should be able to parallelize the native build on 4 different ABIs and you will most likely not need `ccache` on CI.

2. `ccache` relies on timestamps to compute a cache hit. This doesn't work well on CI as files are re-downloaded at every CI run. To overcome this, you'll need to use the `compiler_check content` option which relies instead on [hashing the content of the file](https://ccache.dev/manual/4.3.html).

### Distributed caches

Similar to local caches, you might want to consider using a distributed cache for your native builds.
This could be specifically useful in bigger organizations that are doing frequent native builds.

We recommend to use [sccache](https://github.com/mozilla/sccache) to achieve this.
We defer to the sccache [distributed compilation quickstart](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md) for instructions on how to setup and use this tool.