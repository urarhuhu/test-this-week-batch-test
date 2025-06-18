import RemoveGlobalCLI from './\_remove-global-cli.md';
import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<h2>安裝依賴項</h2>

您需要安裝 Node、React Native 命令行界面、JDK 和 Android Studio。

雖然您可以使用任何喜歡的編輯器開發應用程式，但必須安裝 Android Studio 才能設置必要的工具來為 Android 構建 React Native 應用程式。

<h3 id="jdk">Node 與 JDK</h3>

我們建議透過 [Chocolatey](https://chocolatey.org)（Windows 上流行的套件管理工具）來安裝 Node。

建議使用 Node 的 LTS 版本。若需切換不同版本，可考慮透過 [nvm-windows](https://github.com/coreybutler/nvm-windows)（Windows 的 Node 版本管理工具）安裝 Node。

React Native 還需要 [Java SE 開發套件 (JDK)](https://openjdk.java.net/projects/jdk/11/)，同樣可透過 Chocolatey 安裝。

以管理員身分開啟命令提示字元（右鍵點擊命令提示字元並選擇「以系統管理員身分執行」），然後執行以下命令：

```powershell
choco install -y nodejs-lts microsoft-openjdk11
```

若系統已安裝 Node，請確認版本為 16 或更新。若已安裝 JDK，建議使用 JDK11。使用更高版本的 JDK 可能會遇到問題。

> 您可以在 [Node 下載頁面](https://nodejs.org/en/download/) 找到其他安裝選項。

> 若使用最新版 Java 開發套件，需變更專案的 Gradle 版本以識別 JDK。方法是前往 `{project root folder}\android\gradle\wrapper\gradle-wrapper.properties`，修改 `distributionUrl` 值來升級 Gradle 版本。可參考 [Gradle 最新發布版本](https://gradle.org/releases/)。

<h3>Android 開發環境</h3>

若您是 Android 開發新手，設置開發環境可能較為繁瑣。若已有經驗，則只需配置部分項目。無論哪種情況，請仔細遵循後續步驟。

<h4 id="android-studio">1. 安裝 Android Studio</h4>

[下載並安裝 Android Studio](https://developer.android.com/studio/index.html)。在安裝精靈中，請確保勾選以下所有項目：

- `Android SDK`
- `Android SDK Platform`
- `Android Virtual Device`
- 若未使用 Hyper-V：`Performance (Intel ® HAXM)`（[AMD 或 Hyper-V 用戶請參閱此處](https://android-developers.googleblog.com/2018/07/android-emulator-amd-processor-hyper-v.html)）

接著點擊「下一步」安裝所有元件。

> 若選項呈灰色不可勾選，後續仍可安裝這些元件。

安裝完成並顯示歡迎畫面後，繼續下一步。

<h4 id="android-sdk">2. 安裝 Android SDK</h4>

Android Studio 預設會安裝最新版 Android SDK。但使用原生代碼構建 React Native 應用程式需要特定的 `Android 13 (Tiramisu)` SDK。可透過 Android Studio 的 SDK 管理器安裝其他版本。

開啟 Android Studio，點擊「More Actions」按鈕並選擇「SDK Manager」。

![Android Studio 歡迎畫面](/docs/assets/GettingStartedAndroidStudioWelcomeWindows.png)

> SDK 管理器也可在 Android Studio 的「設定」對話框中找到，路徑為 **Languages & Frameworks** → **Android SDK**。

在 SDK 管理器中選擇「SDK Platforms」標籤頁，然後勾選右下角的「顯示套件詳細資訊」。找到並展開 `Android 13 (Tiramisu)` 項目，確保以下項目已被勾選：

- `Android SDK Platform 33`
- `Intel x86 Atom_64 系統映像檔` 或 `Google APIs Intel x86 Atom 系統映像檔`

接著，選擇「SDK Tools」標籤頁並同樣勾選「顯示套件詳細資訊」。找到並展開 `Android SDK Build-Tools` 項目，確保已選取 `33.0.0` 版本。

最後，點擊「Apply」以下載並安裝 Android SDK 及相關建置工具。

<h4>3. 設定 ANDROID_HOME 環境變數</h4>

React Native 工具需要設定一些環境變數才能建置包含原生代碼的應用程式。

1. 開啟 **Windows 控制台**。
2. 點擊 **使用者帳戶**，然後再次點擊 **使用者帳戶**。
3. 點擊 **變更我的環境變數**。
4. 點擊 **新增...** 以建立一個指向 Android SDK 路徑的新使用者變數 `ANDROID_HOME`：

![ANDROID_HOME 環境變數](/docs/assets/GettingStartedAndroidEnvironmentVariableANDROID_HOME.png)

SDK 預設安裝在以下位置：

```powershell
%LOCALAPPDATA%\Android\Sdk
```

您可以在 Android Studio 的「設定」對話框中找到 SDK 的實際位置，路徑為 **Languages & Frameworks** → **Android SDK**。

開啟新的命令提示字元視窗，以確保新的環境變數已載入，然後繼續下一步。

1. 開啟 PowerShell
2. 複製並貼上 **Get-ChildItem -Path Env:\\** 到 PowerShell
3. 確認 `ANDROID_HOME` 已新增

<h4>4. 將 platform-tools 加入 Path</h4>

1. 開啟 **Windows 控制台**。
2. 點擊 **使用者帳戶**，然後再次點擊 **使用者帳戶**。
3. 點擊 **變更我的環境變數**。
4. 選擇 **Path** 變數。
5. 點擊 **編輯**。
6. 點擊 **新增** 並將 platform-tools 的路徑加入清單。

此資料夾的預設位置為：

```powershell
%LOCALAPPDATA%\Android\Sdk\platform-tools
```

<h3>React Native 命令列介面</h3>

React Native 內建命令列介面。與其全域安裝並管理特定版本的 CLI，我們建議您使用 Node.js 自帶的 `npx` 在執行時存取當前版本。透過 `npx react-native <command>`，系統會在執行命令時下載並執行當前穩定的 CLI 版本。

<h2>建立新應用程式</h2>

<RemoveGlobalCLI />

React Native 內建命令列介面，您可以用來產生新專案。您可以使用 Node.js 自帶的 `npx` 來存取它，而無需全域安裝任何東西。讓我們建立一個名為「AwesomeProject」的新 React Native 專案：

```shell
npx react-native@latest init AwesomeProject
```

如果您是將 React Native 整合到現有應用程式中，或已在專案中安裝 [Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或是為現有的 React Native 專案新增 Android 支援（請參閱 [與現有應用程式整合](integration-with-existing-apps.md)），則不需要此步驟。您也可以使用第三方 CLI（如 [Ignite CLI](https://github.com/infinitered/ignite)）來初始化您的 React Native 應用程式。

<h3>[選用] 使用特定版本或範本</h3>

如果您想使用特定的 React Native 版本開始新專案，可以使用 `--version` 參數：

```shell
npx react-native@X.XX.X init AwesomeProject --version X.XX.X
```

你也可以使用 `--template` 參數來以自訂的 React Native 模板啟動專案。

<h2>準備 Android 裝置</h2>

你需要一個 Android 裝置來執行你的 React Native Android 應用程式。這可以是實體的 Android 裝置，或者更常見的是使用 Android 虛擬裝置（AVD），讓你在電腦上模擬 Android 裝置。

無論哪種方式，你都需要準備裝置以執行開發用的 Android 應用程式。

<h3>使用實體裝置</h3>

如果你有實體的 Android 裝置，你可以透過 USB 線將其連接到電腦，並按照[這裡](running-on-device.md)的指示來代替 AVD 進行開發。

<h3>使用虛擬裝置</h3>

如果你使用 Android Studio 開啟 `./AwesomeProject/android`，可以透過 Android Studio 內的「AVD Manager」查看可用的 Android 虛擬裝置（AVDs）列表。尋找類似這樣的圖標：

<img src="/docs/assets/GettingStartedAndroidStudioAVD.svg" alt="Android Studio AVD Manager" width="100"/>

如果你最近安裝了 Android Studio，可能需要[建立一個新的 AVD](https://developer.android.com/studio/run/managing-avds.html)。選擇「Create Virtual Device...」，然後從列表中選擇任一手機並點擊「Next」，接著選擇 **Tiramisu** API Level 33 映像檔。

> 如果你尚未安裝 HAXM，點擊「Install HAXM」或按照[這些指示](https://github.com/intel/haxm/wiki/Installation-Instructions-on-Windows)進行設定，然後返回 AVD Manager。

點擊「Next」然後「Finish」來建立你的 AVD。此時，你應該可以點擊 AVD 旁邊的綠色三角形按鈕來啟動它，接著進行下一步。

<h2>執行你的 React Native 應用程式</h2>

<h3>步驟 1：啟動 Metro</h3>

首先，你需要啟動 Metro，這是 React Native 內建的 JavaScript 打包工具。Metro「接收一個入口檔案和各種選項，並返回一個包含所有程式碼及其依賴項的單一 JavaScript 檔案。」—[Metro 文件](https://metrobundler.dev/docs/concepts)

要啟動 Metro，請在你的 React Native 專案資料夾內執行以下命令：

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

> 如果你熟悉網頁開發，Metro 非常類似於 webpack——但用於 React Native 應用程式。與 Kotlin 或 Java 不同，JavaScript 不會被編譯——React Native 也不會。打包（bundling）與編譯不同，但它可以幫助提升啟動效能，並將某些平台特定的 JavaScript 轉換為更廣泛支援的 JavaScript。

<h3>步驟 2：啟動你的應用程式</h3>

讓 Metro Bundler 在自己的終端機中執行。在你的 React Native 專案資料夾內開啟一個新的終端機，並執行以下命令：

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

如果一切設定正確，你應該很快就能看到你的新應用程式在 Android 模擬器中執行。

![AwesomeProject on Android](/docs/assets/GettingStartedAndroidSuccessWindows.png)

這是執行應用程式的一種方式——你也可以直接從 Android Studio 內執行它。

> 如果無法成功執行，請參閱[疑難排解](troubleshooting.md)頁面。

<h3>修改你的應用程式</h3>

現在你已成功執行了應用程式，讓我們來修改它。

- 在文字編輯器中開啟 `App.tsx` 並編輯一些內容。
- 按兩下 <kbd>R</kbd> 鍵，或從開發者選單（<kbd>Ctrl</kbd> + <kbd>M</kbd>）中選擇 `Reload` 來查看變更！

<h3>大功告成！</h3>

恭喜！您已成功運行並修改了第一個 React Native 應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

<h2>接下來呢？</h2>

- 若想將此 React Native 程式碼整合至現有應用程式，請參閱[整合指南](integration-with-existing-apps.md)。

若想深入瞭解 React Native，請參閱[React Native 簡介](getting-started)。