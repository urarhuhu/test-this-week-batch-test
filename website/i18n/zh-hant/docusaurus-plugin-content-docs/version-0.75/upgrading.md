---
id: upgrading
title: Upgrading to new versions
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

升級至新版本的 React Native 將讓您獲得更多 API、視圖元件、開發者工具及其他功能。升級過程需要少量調整，但我們會盡可能使其直觀易操作。

## Expo 專案

升級 Expo 專案的 React Native 版本需更新 `package.json` 中的 `react-native`、`react` 和 `expo` 套件版本。Expo 建議採用漸進式升級策略，逐次更新 SDK 版本以準確定位升級過程中出現的問題。詳見 [Expo SDK 升級指南](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) 獲取最新升級資訊。

## React Native 專案

由於標準 React Native 專案本質上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級過程較為複雜。[升級助手](https://react-native-community.github.io/upgrade-helper/) 工具可完整顯示任意兩個版本間的所有變更，並附帶檔案修改註解說明變更原因。

### 1. 選擇版本

首先需選擇起始版本與目標版本（預設為最新主要版本），點擊「顯示升級路徑」按鈕即可查看升級指引。

💡 主要版本升級時，頁面頂部會顯示「實用資源」區塊，包含升級輔助連結。

### 2. 升級依賴套件

首先顯示的是 `package.json` 檔案，建議先更新其中標示的依賴套件。例如若顯示需變更 `react-native` 和 `react` 版本，可執行以下指令進行安裝：

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

新版本可能包含執行 `npx react-native init` 時生成的檔案更新，這些檔案會列在 [升級助手](https://react-native-community.github.io/upgrade-helper/) 頁面的 `package.json` 之後。若無其他變更，僅需重新建置專案即可繼續開發；若有變更則需手動套用至專案中。

### 疑難排解

#### 已完成所有變更但應用仍使用舊版本

此類問題通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 清除專案所有快取後重新執行。