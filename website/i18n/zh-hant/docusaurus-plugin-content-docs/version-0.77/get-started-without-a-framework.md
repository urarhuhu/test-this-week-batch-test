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

若現有[框架](/architecture/glossary#react-native-framework)無法滿足需求，或您傾向自行開發框架，您可不依賴框架直接建立React Native應用程式。

首先需[設定開發環境](set-up-your-environment)，完成後請依以下步驟建立應用程式並開始開發。

### 步驟一：建立新專案

<RemoveGlobalCLI />

您可使用[React Native Community CLI](https://github.com/react-native-community/cli)生成新專案。以下建立名為「AwesomeProject」的新專案：

```shell
npx @react-native-community/cli@latest init AwesomeProject
```

若您正將React Native整合至現有應用程式、已安裝[Expo](https://docs.expo.dev/bare/installing-expo-modules/)，或為現有React Native專案添加Android支援（參見[整合既有應用程式](integration-with-existing-apps.md)），則無需此步驟。亦可使用第三方CLI工具如[Ignite CLI](https://github.com/infinitered/ignite)初始化專案。

:::info

若iOS環境遇到問題，請嘗試重新安裝依賴項：

1. 執行`cd ios`進入`ios`資料夾
2. 執行`bundle install`安裝[Bundler](https://bundler.io/)
3. 執行`bundle exec pod install`透過CocoaPods安裝iOS依賴項

:::

#### [選用]指定版本或模板

若要使用特定React Native版本建立專案，可加入`--version`參數：

```shell
npx @react-native-community/cli@X.XX.X init AwesomeProject --version X.XX.X
```

亦可透過`--template`參數使用自訂模板，詳見[此處說明](https://github.com/react-native-community/cli/blob/main/docs/init.md#initializing-project-with-custom-template)。

### 步驟二：啟動Metro

[**Metro**](https://metrobundler.dev/)是React Native的JavaScript建構工具。請在專案目錄執行以下指令啟動開發伺服器：

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
若您熟悉網頁開發，Metro功能類似Vite或webpack等打包工具，但專為React Native設計。例如Metro使用[Babel](https://babel.dev/)將JSX等語法轉換為可執行JavaScript。
:::

### 步驟三：啟動應用程式

讓Metro Bundler在獨立終端機運行。另開新終端機進入專案目錄後執行：

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

若設定正確，您將很快在Android模擬器中看到新應用程式運行。

此為運行方式之一，您亦可直接透過Android Studio啟動應用程式。

> 若遇到問題，請參閱[疑難排解](troubleshooting.md)頁面。

### 步驟四：修改應用程式

成功運行應用程式後，現在開始進行修改。

- 用文字編輯器開啟`App.tsx`並編輯部分內容
- 快速按兩次<kbd>R</kbd>鍵，或透過開發者選單（<kbd>Ctrl</kbd> + <kbd>M</kbd>）選擇「重新載入」即可查看變更！

### 完成！

恭喜！您已成功運行並修改了第一個基礎React Native應用程式。

<center><img src="/docs/assets/GettingStartedCongratulations.png" width="150"></img></center>

### 接下來？

- 若您想將新的 React Native 程式碼整合至現有應用程式，請參閱 [整合指南](integration-with-existing-apps.md)。
- 若您想進一步了解 React Native，請參閱 [React Native 簡介](getting-started)。