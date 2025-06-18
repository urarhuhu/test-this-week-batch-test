## 安裝依賴套件

您需要安裝 Node、React Native 命令行界面、JDK 以及 Android Studio。

雖然您可以使用任何喜歡的編輯器來開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3>Node</h3>

請依照 [Linux 發行版的安裝說明](https://nodejs.org/en/download/package-manager/) 安裝 Node 18.18 或更新版本。

<h3>Java 開發套件 (JDK)</h3>

React Native 目前推薦使用 Java SE 開發套件 (JDK) 第 17 版。使用更高版本的 JDK 可能會遇到問題。您可以從 [AdoptOpenJDK](https://adoptopenjdk.net/) 或系統套件管理員下載並安裝 [OpenJDK](https://openjdk.java.net)。

<h3>Android 開發環境</h3>

如果您是 Android 開發的新手，設置開發環境可能會有些繁瑣。如果您已經熟悉 Android 開發，則可能需要配置一些東西。無論哪種情況，請務必仔細遵循接下來的幾個步驟。

<h4 id="android-studio">1. 安裝 Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在 Android Studio 安裝精靈中，請確保勾選以下所有項目旁邊的方框：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

然後，點擊「下一步」安裝所有這些元件。

> 如果複選框呈灰色顯示，您稍後仍有機會安裝這些元件。

安裝完成並顯示歡迎畫面後，請繼續進行下一步。

<h4 id="android-sdk">2. 安裝 Android SDK</h4>

Android Studio 預設會安裝最新的 Android SDK。然而，使用原生代碼構建 React Native 應用程式需要特定的 `Android 15 (VanillaIceCream)` SDK。可以通過 Android Studio 中的 SDK 管理器安裝其他 Android SDK。

為此，請打開 Android Studio，點擊「Configure」按鈕並選擇「SDK Manager」。

> SDK 管理器也可以在 Android Studio 的「設定」對話框中找到，位於 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤頁，然後勾選右下角的「Show Package Details」。找到並展開 `Android 15 (VanillaIceCream)` 條目，確保勾選以下項目：

- `Android SDK Platform 35`
- `Intel x86 Atom_64 System Image` 或 `Google APIs Intel x86 Atom System Image`

接著，選擇「SDK Tools」標籤頁，同樣勾選「Show Package Details」。找到並展開「Android SDK Build-Tools」條目，確保選擇 `35.0.0` 版本。

最後，點擊「Apply」下載並安裝 Android SDK 及相關構建工具。

<h4>3. 配置 ANDROID_HOME 環境變數</h4>

React Native 工具需要設置一些環境變數才能使用原生代碼構建應用程式。

將以下行添加到您的 `$HOME/.bash_profile` 或 `$HOME/.bashrc` 配置文件中（如果您使用 `zsh`，則是 `~/.zprofile` 或 `~/.zshrc`）：

```shell
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

> `.bash_profile` 是 `bash` 專用的。如果您使用其他 shell，則需要編輯相應的 shell 專用配置文件。

對於 `bash` 使用者請輸入 `source $HOME/.bash_profile`，或輸入 `source $HOME/.zprofile` 以載入設定檔至當前 shell。執行 `echo $ANDROID_HOME` 確認 ANDROID_HOME 環境變數已設定，並執行 `echo $PATH` 確認相關目錄已正確加入路徑。

> 請務必使用正確的 Android SDK 路徑。您可以在 Android Studio 的「設定」對話框中找到 SDK 實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

<h3>Watchman</h3>

請遵循 [Watchman 安裝指南](https://facebook.github.io/watchman/docs/install#buildinstall) 從原始碼編譯並安裝 Watchman。

> [Watchman](https://facebook.github.io/watchman/docs/install) 是 Facebook 開發的檔案系統監控工具。強烈建議安裝以獲得更好效能，並提升特定邊緣案例的相容性（譯註：雖然不安裝可能仍可運作，但實際效果因人而異；現在安裝可避免後續潛在問題）。

<h2>準備 Android 裝置</h2>

您需要 Android 裝置來執行 React Native Android 應用程式。可使用實體 Android 裝置，或更常見的做法是使用 Android 虛擬裝置 (AVD) 在電腦上模擬 Android 裝置。

無論採用哪種方式，都需要準備裝置以執行開發版 Android 應用程式。

<h3>使用實體裝置</h3>

若有實體 Android 裝置，可透過 USB 連接線連接電腦後，依照[此處說明](running-on-device.md)設定，即可用於開發並取代 AVD。

<h3>使用虛擬裝置</h3>

若使用 Android Studio 開啟 `./AwesomeProject/android` 專案，可透過 Android Studio 內的「AVD Manager」查看可用虛擬裝置清單。尋找如下圖示的按鈕：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若剛安裝 Android Studio，可能需要[建立新 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」後，從清單中任選手機型號並點擊「Next」，接著選擇 **VanillaIceCream** API Level 35 映像檔。

> 建議設定系統的 [VM 加速](https://developer.android.com/studio/run/emulator-acceleration.html#vm-linux)以提升效能。完成設定後，請返回 AVD Manager。

點擊「Next」後選擇「Finish」建立 AVD。此時應可點擊 AVD 旁的綠色三角形按鈕來啟動模擬器。

<h3>大功告成！</h3>

恭喜！您已成功設定開發環境。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>接下來？</h2>

- 若要將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。
- 若想進一步了解 React Native，請查看 [React Native 簡介](getting-started)。