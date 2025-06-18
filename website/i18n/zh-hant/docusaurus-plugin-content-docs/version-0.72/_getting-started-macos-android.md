import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## Installing dependencies

You will need Node, Watchman, the React Native command line interface, a JDK, and Android Studio.

While you can use any editor of your choice to develop your app, you will need to install Android Studio in order to set up the necessary tooling to build your React Native app for Android.

<h3>Node &amp; Watchman</h3>

We recommend installing Node and Watchman using [Homebrew](http://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install node
brew install watchman
```

If you have already installed Node on your system, make sure it is Node 16 or newer.

[Watchman](https://facebook.github.io/watchman) is a tool by Facebook for watching changes in the filesystem. It is highly recommended you install it for better performance.

<h3>Java Development Kit</h3>

We recommend installing the OpenJDK distribution called Azul **Zulu** using [Homebrew](http://brew.sh/). Run the following commands in a Terminal after installing Homebrew:

```shell
brew install --cask zulu@11

# Get path to where cask was installed to double-click installer
brew info --cask zulu@11
```

After you install the JDK, update your `JAVA_HOME` environment variable. If you used above steps, JDK will likely be at `/Library/Java/JavaVirtualMachines/zulu-11.jdk/Contents/Home`

The Zulu OpenJDK distribution offers JDKs for **both Intel and M1 Macs**. This will make sure your builds are faster on M1 Macs compared to using an Intel-based JDK.

If you have already installed JDK on your system, we recommend JDK 11. You may encounter problems using higher JDK versions.

<h3>Android development environment</h3>

Setting up your development environment can be somewhat tedious if you're new to Android development. If you're already familiar with Android development, there are a few things you may need to configure. In either case, please make sure to carefully follow the next few steps.

<h4 id="android-studio">1. Install Android Studio</h4>

[Download and install Android Studio](https://developer.android.com/studio/index.html). While on Android Studio installation wizard, make sure the boxes next to all of the following items are checked:

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`

Then, click "Next" to install all of these components.

> If the checkboxes are grayed out, you will have a chance to install these components later on.

Once setup has finalized and you're presented with the Welcome screen, proceed to the next step.

<h4 id="android-sdk">2. Install the Android SDK</h4>

Android Studio installs the latest Android SDK by default. Building a React Native app with native code, however, requires the `Android 13 (Tiramisu)` SDK in particular. Additional Android SDKs can be installed through the SDK Manager in Android Studio.

To do that, open Android Studio, click on "More Actions" button and select "SDK Manager".

![Android Studio Welcome](/docs/assets/GettingStartedAndroidStudioWelcomeMacOS.png)

> The SDK Manager can also be found within the Android Studio "Settings" dialog, under **Languages & Frameworks** → **Android SDK**.

Select the "SDK Platforms" tab from within the SDK Manager, then check the box next to "Show Package Details" in the bottom right corner. Look for and expand the `Android 13 (Tiramisu)` entry, then make sure the following items are checked:

- `Android SDK Platform 33`
- `Intel x86 Atom_64 系統映像` 或 `Google APIs Intel x86 Atom 系統映像` 或 (適用於 Apple M1 晶片) `Google APIs ARM 64 v8a 系統映像`

接著，選擇「SDK Tools」標籤頁，同樣勾選「顯示套件詳細資訊」。找到並展開「Android SDK Build-Tools」項目，確保已選取 `33.0.0` 版本。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. 設定 ANDROID_HOME 環境變數</h4>

React Native 工具需要設定某些環境變數才能建置包含原生代碼的應用程式。

將以下內容加入您的 `~/.zprofile` 或 `~/.zshrc` 設定檔（若使用 `bash`，則為 `~/.bash_profile` 或 `~/.bashrc`）：

```shell
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

執行 `source ~/.zprofile`（或 `source ~/.bash_profile` 適用於 `bash`）以載入設定至當前 shell。透過執行 `echo $ANDROID_HOME` 確認 ANDROID_HOME 已設定，並執行 `echo $PATH` 檢查相關目錄是否已加入路徑。

> 請確保使用正確的 Android SDK 路徑。您可以在 Android Studio 的「Settings」對話框中，於 **Languages & Frameworks** → **Android SDK** 下找到 SDK 的實際位置。

<h3>React Native 命令列介面</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議透過 Node.js 自帶的 `npx` 在執行時存取當前版本。使用 `npx react-native <command>` 時，將自動下載並執行當前穩定版的 CLI。

<h2>建立新應用程式</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，可用於生成新專案。您無需全域安裝，透過 Node.js 自帶的 `npx` 即可存取。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

若您要將 React Native 整合至現有應用程式、或已在專案中安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)、或需為現有 React Native 專案添加 Android 支援（參見 [與現有應用程式整合](integration-with-existing-apps.md)），則此步驟非必要。您也可使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）初始化 React Native 應用程式。

<h3>[選用] 使用特定版本或模板</h3>

若要以特定 React Native 版本建立新專案，可使用 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

您也可透過 `--template` 參數使用自訂 React Native 模板建立專案。

<h2>準備 Android 裝置</h2>

執行 React Native Android 應用程式需準備 Android 裝置，可以是實體裝置，或更常見的 Android 虛擬裝置（AVD），後者允許您在電腦上模擬 Android 裝置。

無論採用何種方式，均需準備裝置以執行開發版 Android 應用程式。

<h3>使用實體裝置</h3>

若有實體 Android 裝置，可透過 USB 連接電腦並遵循[此處說明](running-on-device.md)，即可用於開發以取代 AVD。

<h3>使用虛擬裝置</h3>

如果你使用 Android Studio 開啟 `./AwesomeProject/android` 專案，可以透過點選 Android Studio 內的「AVD Manager」圖示（如下所示）查看可用的 Android 虛擬裝置 (AVD) 清單：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

若你剛安裝 Android Studio，可能需要[建立新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。點選「Create Virtual Device...」後，從清單中選擇任一手機型號並點擊「Next」，接著選擇 **Tiramisu** API Level 33 系統映像檔。

點擊「Next」後再按「Finish」完成 AVD 建立。此時你應能點選 AVD 旁的綠色三角形按鈕啟動模擬器，並繼續後續步驟。

<h2>執行你的 React Native 應用程式</h2>

<h3>步驟 1：啟動 Metro</h3>

首先需啟動 Metro——React Native 內建的 JavaScript 打包工具。根據 [Metro 文件](https://metrobundler.dev/docs/concepts)說明，Metro「接收入口檔案與各項參數，最終輸出一个包含所有程式碼與依賴項的單一 JavaScript 檔案」。

請在 React Native 專案目錄下執行以下指令啟動 Metro：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm start
```

</TabItem>
<TabItem value="yarn">

```shell
yarn start
```

</TabItem>
</Tabs>

> 若你有網頁開發經驗，可將 Metro 視為 React Native 的 webpack。與 Kotlin 或 Java 不同，JavaScript 不需編譯——React Native 也是。打包雖不等同於編譯，但能提升啟動效能並將某些平台特定的 JavaScript 轉換為更通用的版本。

<h3>步驟 2：啟動應用程式</h3>

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run android
```

</TabItem>
<TabItem value="yarn">

```shell
yarn android
```

</TabItem>
</Tabs>

若所有設定正確，你應該很快能在 Android 模擬器中看到新應用程式運行。

![Android 上的 AwesomeProject](/docs/assets/GettingStartedAndroidSuccessMacOS.png)

這是執行應用程式的方式之一——你也可以直接透過 Android Studio 啟動。

> 若遇到問題，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>修改應用程式</h3>

成功運行應用程式後，現在來修改它：

- 用文字編輯器開啟 `App.tsx` 並編輯部分程式碼
- 快速按兩次 <kbd>R</kbd> 鍵，或透過開發者選單（<kbd>Cmd ⌘</kbd> + <kbd>M</kbd>）選擇「Reload」即可查看變更！

<h3>大功告成！</h3>

恭喜！你已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>接下來？</h2>

- 若要將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)

若想深入瞭解 React Native，請查看 [React Native 簡介](getting-started)