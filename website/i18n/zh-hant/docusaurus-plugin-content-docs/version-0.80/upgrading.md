---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖、開發者工具和其他好處。升級需要一些努力，但我們會盡量讓這個過程簡單明瞭。

## Expo 專案

將您的 Expo 專案升級至新版本的 React Native 需要更新 `package.json` 檔案中的 `react-native`、`react` 和 `expo` 套件版本。Expo 建議逐步升級 SDK 版本，一次一個版本。這樣做可以幫助您找出升級過程中出現的中斷和問題。請參閱 [Upgrading Expo SDK Walkthrough](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) 以獲取有關升級專案的最新資訊。

## React Native 專案

由於典型的 React Native 專案基本上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級可能會相當棘手。[Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 是一個網頁工具，可以幫助您在升級應用程式時提供兩個版本之間的所有變更。它還會顯示特定檔案的註解，以幫助理解為什麼需要這些變更。

### 1. 選擇版本

首先，您需要選擇要從哪個版本升級到哪個版本，預設會選擇最新的主要版本。選擇後，您可以點擊「Show me how to upgrade」按鈕。

💡 主要更新會在頂部顯示一個「useful content」區塊，其中包含幫助您升級的連結。

### 2. 升級依賴項

第一個顯示的檔案是 `package.json`，最好更新其中顯示的依賴項。例如，如果 `react-native` 和 `react` 顯示為變更，您可以通過運行以下命令在專案中安裝它們：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
npm install react-native@{{VERSION}}
npm install react@{{REACT_VERSION}}
```

</TabItem>
<TabItem value="yarn">

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
yarn add react-native@{{VERSION}}
yarn add react@{{REACT_VERSION}}
```

</TabItem>
</Tabs>

### 3. 升級專案檔案

新版本可能包含對其他檔案的更新，這些檔案在您運行 `npx react-native init` 時生成，這些檔案會在 `package.json` 之後列在 [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 頁面上。如果沒有其他變更，您只需要重新建置專案即可繼續開發。如果有變更，您需要手動將它們應用到您的專案中。

### 疑難排解

#### 我已經完成了所有變更，但我的應用程式仍然使用舊版本

這類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 來清除專案的所有快取，然後再次運行。