---
id: getting-started-without-a-framework
title: Get Started Without a Framework
hide_table_of_contents: true
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import constants from '@site/core/TabsConstants';
import PlatformSupport from '@site/src/theme/PlatformSupport';

import RemoveGlobalCLI from './\_remove-global-cli.md';

<PlatformSupport platforms={['android', 'ios', 'macOS', 'tv', 'watchOS', 'web', 'windows', 'visionOS']} />

若現有[框架](/architecture/glossary#react-native-framework)無法滿足您的限制條件，或您偏好自行開發框架，您可以在不使用框架的情況下建立React Native應用程式。

為此，您首先需要[設定開發環境](set-up-your-environment)。完成設定後，請按照以下步驟建立應用程式並開始開發。

### 步驟1：建立新應用程式

<RemoveGlobalCLI />

您可以使用[React Native Community CLI](https://github.com/react-native-community/cli)來生成新專案。讓我們建立一個名為「AwesomeProject」的新React Native專案：

```shell
npx @react-native-community/cli@latest init AwesomeProject
```

若您正在將React Native整合至現有應用程式、或已在專案中安裝[Expo](https://docs.expo.dev/bare/installing-expo-modules/)、或正在為現有React Native專案添加Android支援（參見[與現有應用程式整合](integration-with-existing-apps.md)），則無需此步驟。您也可以使用第三方CLI（如[Ignite CLI](https://github.com/infinitered/ignite)）來設定React Native應用程式。

:::info

若在iOS環境遇到問題，請嘗試重新安裝依賴項：

1. 執行`cd ios`進入`ios`資料夾。
2. 執行`bundle install`安裝[Bundler](https://bundler.io/)。
3. 執行`bundle exec pod install`安裝由CocoaPods管理的iOS依賴項。

:::

#### [選用] 使用特定版本或模板

若需使用特定React Native版本建立新專案，可使用`--version`參數：

```shell
npx @react-native-community/cli@X.XX.X init AwesomeProject --version X.XX.X
```

您也可透過`--template`參數使用自訂React Native模板啟動專案，詳見[此處](https://github.com/react-native-community/cli/blob/main/docs/init.md#initializing-project-with-custom-template)。

### 步驟2：啟動Metro

[**Metro**](https://metrobundler.dev/)是React Native的JavaScript建構工具。請在專案資料夾中執行以下指令以啟動Metro開發伺服器：

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

:::note
若您熟悉網頁開發，Metro類似於Vite和webpack等打包工具，但專為React Native端到端設計。例如，Metro使用[Babel](https://babel.dev/)將JSX等語法轉換為可執行的JavaScript。
:::

### 步驟3：啟動應用程式

讓Metro Bundler在其專屬終端機中運行。在React Native專案資料夾內開啟新終端機，執行以下指令：

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

若一切設定正確，您將很快在Android模擬器中看到新應用程式運行。

這是運行應用程式的方式之一，您亦可直接透過Android Studio運行。

> 若無法順利運行，請參閱[疑難排解](troubleshooting.md)頁面。

### 步驟4：修改應用程式

現在您已成功運行應用程式，讓我們進行修改。

- 在您選擇的文字編輯器中開啟`App.tsx`並編輯部分內容。
- 按兩次<kbd>R</kbd>鍵，或從開發者選單（<kbd>Ctrl</kbd> + <kbd>M</kbd>）中選擇`重新載入`以查看變更！

### 完成！

恭喜！您已成功運行並修改了第一個基礎React Native應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

### 接下來呢？

- If you want to add this new React Native code to an existing application, check out the [Integration guide](integration-with-existing-apps.md).
- If you're curious to learn more about React Native, check out the [Introduction to React Native](getting-started).