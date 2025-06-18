---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖元件、開發者工具和其他優點。升級過程需要一些努力，但我們會盡量讓它變得簡單直接。

## Expo 專案

要將 Expo 專案升級至新版本的 React Native，您需要更新 `package.json` 檔案中的 `react-native`、`react` 和 `expo` 套件版本。Expo 建議逐步遞增升級 SDK 版本，一次升級一個版本。這樣做可以幫助您定位升級過程中出現的中斷和問題。請參閱 [升級 Expo SDK 逐步指南](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) 以獲取有關升級專案的最新資訊。

## React Native 專案

由於典型的 React Native 專案基本上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級可能會相當棘手。[升級助手](https://react-native-community.github.io/upgrade-helper/) 是一個網路工具，可以通過提供任何兩個版本之間發生的完整變更集來幫助您升級應用程式。它還顯示特定檔案的註解，以幫助理解為什麼需要該變更。

### 1. 選擇版本

首先，您需要選擇要從哪個版本升級到哪個版本，預設情況下會選擇最新的主要版本。選擇後，您可以點擊「顯示如何升級」按鈕。

💡 主要更新將在頂部顯示一個「有用內容」部分，其中包含幫助您升級的連結。

:::tip
或者，您可以執行 `npx react-native upgrade`，它會自動檢查您當前的版本和可用的最新版本，並顯示已選擇版本的升級助手頁面連結。
:::

### 2. 升級依賴項

顯示的第一個檔案是 `package.json`，最好更新其中顯示的依賴項。例如，如果 `react-native` 和 `react` 顯示為變更，則可以通過執行以下命令在專案中安裝它們：

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

新版本可能包含對執行 `npx react-native init` 時生成的其他檔案的更新，這些檔案列在升級助手頁面的 `package.json` 之後。如果沒有其他變更，則只需重新建置專案即可繼續開發。

如果有變更，您可以通過從頁面中的變更複製並貼上來手動更新它們，或者通過執行以下命令使用 React Native CLI 升級命令來完成：

```shell
npx react-native upgrade
```

這將根據最新模板檢查您的檔案並執行以下操作：

- 如果模板中有新檔案，則會創建它。
- 如果模板中的檔案與您的檔案相同，則會跳過它。
- 如果您的專案中的檔案與模板不同，您將收到提示；您可以選擇保留您的檔案或使用模板版本覆蓋它。

> 某些升級無法通過 React Native CLI 自動完成，需要手動操作，例如從 `0.28` 升級到 `0.29`，或從 `0.56` 升級到 `0.57`。升級時請務必檢查 [發行說明](https://github.com/facebook/react-native/releases)，以便識別您的特定專案可能需要的手動變更。

### 疑難排解

#### 我已完成所有變更，但我的應用程式仍在使用舊版本

這類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 來清除專案的所有快取，然後再次執行它。