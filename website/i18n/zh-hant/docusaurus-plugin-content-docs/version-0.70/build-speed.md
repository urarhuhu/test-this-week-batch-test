---
id: build-speed
title: Speeding up your Build phase
---

建構你的 React Native 應用程式可能**相當耗時**，會佔用開發者數分鐘的時間。
隨著專案規模增長，特別是在擁有多位 React Native 開發者的大型組織中，這個問題會更加明顯。

隨著[新 React Native 架構](the-new-architecture/landing-page.md)的推出，這個問題變得更加關鍵，
因為除了 iOS 和 Android 平台原本需要的原生程式碼外，你可能還需要使用 Android NDK 來編譯專案中的一些原生 C++ 程式碼。

為了減輕效能上的衝擊，本頁面將分享一些**提升建構速度**的建議。

## 開發期間僅建構單一 ABI (僅限 Android)

當你在本地建構 Android 應用程式時，預設會建構所有 4 種[應用程式二進位介面 (ABI)](https://developer.android.com/ndk/guides/abis)：`armeabi-v7a`、`arm64-v8a`、`x86` 和 `x86_64`。

然而，如果你只是在本地建構並在模擬器或實體裝置上測試，可能不需要建構所有這些 ABI。

這樣做可以將建構時間減少**約 75%**。

如果你使用 React Native CLI，可以在 `run-android` 指令中加入 `--active-arch-only` 旗標。
這個旗標會確保從運行的模擬器或連接的手機中選取正確的 ABI。
要確認這個方法運作正常，你會在控制台看到類似 `info Detected architectures arm64-v8a` 的訊息。

```
$ yarn react-native run-android --active-arch-only

[ ... ]
info Running jetifier to migrate libraries to AndroidX. You can disable it using "--no-jetifier" flag.
Jetifier found 1037 file(s) to forward-jetify. Using 32 workers...
info JS server already running.
info Detected architectures arm64-v8a
info Installing the app...
```

這個機制依賴於 `reactNativeArchitectures` Gradle 屬性。

因此，如果你直接從命令列使用 Gradle 建構而不透過 CLI，可以像這樣指定要建構的 ABI：

```
$ ./gradlew :app:assembleDebug -PreactNativeArchitectures=x86,x86_64
```

這在你想於 CI 上建構 Android 應用程式並使用矩陣來平行化不同架構的建構時特別有用。

如果需要，你也可以使用專案[頂層目錄](https://github.com/facebook/react-native/blob/19cf70266eb8ca151aa0cc46ac4c09cb987b2ceb/template/android/gradle.properties#L30-L33)中的 `gradle.properties` 檔案來覆寫這個值：

```
# Use this property to specify which architecture you want to build.
# You can also override it from the CLI using
# ./gradlew <task> -PreactNativeArchitectures=x86_64
reactNativeArchitectures=armeabi-v7a,arm64-v8a,x86,x86_64
```

當你建構應用程式的**正式版本**時，請記得移除這些旗標，因為你會希望建構出能支援所有 ABI 的 apk/app bundle，而不僅是日常開發工作流程中使用的那一種。

## 使用編譯器快取

如果你經常執行原生建構（無論是 C++ 或 Objective-C），使用**編譯器快取**可能會帶來好處。

具體來說，你可以使用兩種類型的快取：本地編譯器快取和分散式編譯器快取。

### 本地快取

:::info
以下說明適用於**Android 和 iOS**。
如果你僅建構 Android 應用程式，應該可以直接使用。
如果你也需要建構 iOS 應用程式，請遵循下方[XCode 特定設定](#xcode-specific-setup)章節的說明。
:::

我們建議使用 [**ccache**](https://ccache.dev/) 來快取原生建構的編譯結果。
Ccache 的工作原理是包裝 C++ 編譯器，儲存編譯結果，並在發現中間編譯結果已存在時跳過編譯步驟。

安裝方式可參考[官方安裝說明](https://github.com/ccache/ccache/blob/master/doc/INSTALL.md)。

在 macOS 上，我們可以使用 `brew install ccache` 來安裝 ccache。
安裝完成後，可以透過以下設定來快取 NDK 的編譯結果：

```
ln -s $(which ccache) /usr/local/bin/gcc
ln -s $(which ccache) /usr/local/bin/g++
ln -s $(which ccache) /usr/local/bin/cc
ln -s $(which ccache) /usr/local/bin/c++
ln -s $(which ccache) /usr/local/bin/clang
ln -s $(which ccache) /usr/local/bin/clang++
```

這將在 `/usr/local/bin/` 目錄下建立指向 `ccache` 的符號連結，命名為 `gcc`、`g++` 等。

只要 `/usr/local/bin/` 在你的 `$PATH` 變數中排在 `/usr/bin/` 之前（這是預設情況），此設定即可生效。

你可以使用 `which` 指令來驗證是否生效：

```
$ which gcc
/usr/local/bin/gcc
```

若結果顯示 `/usr/local/bin/gcc`，則表示你實際呼叫的是 `ccache`，它會包裹 `gcc` 的呼叫。

:::caution
請注意，此 `ccache` 設定會影響你機器上所有的編譯行為，不僅限於 React Native 相關的編譯。使用時請自行承擔風險。若你遇到其他軟體安裝或編譯失敗的情況，這可能是原因之一。若發生此狀況，你可以移除已建立的符號連結來恢復原始狀態：

```
unlink /usr/local/bin/gcc
unlink /usr/local/bin/g++
unlink /usr/local/bin/cc
unlink /usr/local/bin/c++
unlink /usr/local/bin/clang
unlink /usr/local/bin/clang++
```

以還原機器設定並使用預設編譯器。
:::

接著你可以執行兩次完整建置（例如在 Android 上先執行 `yarn react-native run-android`，刪除 `android/app/build` 資料夾後再次執行相同指令）。你會注意到第二次建置速度大幅提升（應只需數秒而非數分鐘）。建置過程中可透過 `ccache -s` 驗證 `ccache` 是否正常運作並查看快取命中率。

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

注意 `ccache` 會累積所有建置的統計數據。可使用 `ccache --zero-stats` 在建置前重置數據以驗證快取命中率。

若需清除快取，可執行 `ccache --clear`

#### XCode 專屬設定

要確保 `ccache` 在 iOS 和 XCode 環境正常運作，需額外執行以下步驟：

1. 必須修改 Xcode 和 `xcodebuild` 呼叫編譯器指令的方式。預設情況下它們會使用編譯器二進位檔的_絕對路徑_，因此 `/usr/local/bin` 中的符號連結不會被使用。可透過以下任一方式設定 Xcode 使用_相對_名稱：

- 若直接使用命令列，可在指令前綴環境變數：`CLANG=clang CLANGPLUSPLUS=clang++ LD=clang LDPLUSPLUS=clang++ xcodebuild <xcodebuild 指令其餘部分>`
- 在 `ios/Podfile` 中加入 `post_install` 區段，於 `pod install` 階段修改 Xcode workspace 中的編譯器設定：

```ruby
  post_install do |installer|
    react_native_post_install(installer)

    # ...possibly other post_install items here

    installer.pods_project.targets.each do |target|
      target.build_configurations.each do |config|
        # Using the un-qualified names means you can swap in different implementations, for example ccache
        config.build_settings["CC"] = "clang"
        config.build_settings["LD"] = "clang"
        config.build_settings["CXX"] = "clang++"
        config.build_settings["LDPLUSPLUS"] = "clang++"
      end
    end

    __apply_Xcode_12_5_M1_post_install_workaround(installer)
  end
```

2. 需要配置 ccache 以允許一定程度的寬鬆性和快取行為，使 Xcode 編譯時能註冊快取命中。若透過環境變數配置，標準配置與以下參數不同：

```bash
export CCACHE_SLOPPINESS=clang_index_store,file_stat_matches,include_file_ctime,include_file_mtime,ivfsoverlay,pch_defines,modules,system_headers,time_macros
export CCACHE_FILECLONE=true
export CCACHE_DEPEND=true
export CCACHE_INODECACHE=true
```

相同配置可寫入 `ccache.conf` 檔案或透過 ccache 提供的其他機制設定。詳情請參閱 [官方 ccache 手冊](https://ccache.dev/manual/4.3.html)。

#### 在 CI 上使用此方法

在 macOS 上，ccache 使用 `/Users/$USER/Library/Caches/ccache` 資料夾儲存快取。因此你也可以在 CI 上保存與還原此資料夾來加速建置。

但需注意以下事項：

1. 在 CI 環境中，我們建議進行完整的乾淨建置，以避免快取污染問題。如果您遵循前文提到的方法，應該能夠將原生建置平行化到 4 種不同的 ABI 上，這樣在 CI 上很可能就不需要用到 `ccache`。

2. `ccache` 依賴時間戳記來計算快取命中率。這在 CI 環境中效果不佳，因為每次 CI 執行時檔案都會重新下載。為了解決這個問題，您需要使用 `compiler_check content` 選項，該選項改為[透過檔案內容的雜湊值](https://ccache.dev/manual/4.3.html)來判斷。

### 分散式快取

與本地快取類似，您可能會想考慮為原生建置使用分散式快取。
這對於經常進行原生建置的大型組織特別有用。

我們建議使用 [sccache](https://github.com/mozilla/sccache) 來實現此目的。
關於如何設定和使用此工具，我們參考 sccache 的[分散式編譯快速入門指南](https://github.com/mozilla/sccache/blob/main/docs/DistributedQuickstart.md)。