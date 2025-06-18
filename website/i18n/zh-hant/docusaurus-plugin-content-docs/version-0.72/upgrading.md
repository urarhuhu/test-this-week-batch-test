---
id: upgrading
title: Upgrading to new versions
---

升級至新版本的 React Native 將讓您獲得更多 API、視圖元件、開發者工具和其他優質功能。升級過程需要一些努力，但我們會盡量使其直觀簡便。

## Expo 專案

要將 Expo 專案升級至新版本的 React Native，需更新 `package.json` 中的 `react-native`、`react` 和 `expo` 套件版本。Expo 提供 `upgrade` 指令來自動處理這些核心套件與相關依賴的升級。詳見 [Expo SDK 升級指南](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/) 以獲取最新的專案升級資訊。

## React Native 專案

由於典型的 React Native 專案本質上由 Android 專案、iOS 專案和 JavaScript 專案組成，升級過程可能較為複雜。目前有兩種升級方式：使用 [React Native CLI](https://github.com/react-native-community/cli) 或透過 [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 手動升級。

### React Native CLI

[React Native CLI](https://github.com/react-native-community/cli) 內建的 `upgrade` 指令提供一步式操作，能以最小衝突升級原始碼檔案。其內部使用 [rn-diff-purge](https://github.com/react-native-community/rn-diff-purge) 專案來識別需新增、刪除或修改的檔案。

#### 1. 執行 `upgrade` 指令

> `upgrade` 指令基於 Git 的 `git apply` 三方合併機制運作，因此必須使用 Git 才能生效。若未使用 Git 但仍想採用此方案，請參閱 [疑難排解](#i-want-to-upgrade-with-react-native-cli-but-i-dont-use-git) 章節的解決方法。

執行以下指令開始升級至最新版本：

```shell
npx react-native upgrade
```

可透過參數指定 React Native 版本，例如升級至 `0.61.0-rc.0`：

```shell
npx react-native upgrade 0.61.0-rc.0
```

專案升級過程採用 `git apply` 三方合併，完成後可能需要解決部分衝突。

#### 2. 解決衝突

衝突檔案會包含明確標示變更來源的分隔符號，例如：

```
13B07F951A680F5B00A75B9A /* Release */ = {
  isa = XCBuildConfiguration;
  buildSettings = {
    ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon;
<<<<<<< ours
    CODE_SIGN_IDENTITY = "iPhone Developer";
    FRAMEWORK_SEARCH_PATHS = (
      "$(inherited)",
      "$(PROJECT_DIR)/HockeySDK.embeddedframework",
      "$(PROJECT_DIR)/HockeySDK-iOS/HockeySDK.embeddedframework",
    );
=======
    CURRENT_PROJECT_VERSION = 1;
>>>>>>> theirs
    HEADER_SEARCH_PATHS = (
      "$(inherited)",
      /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include,
      "$(SRCROOT)/../node_modules/react-native/React/**",
      "$(SRCROOT)/../node_modules/react-native-code-push/ios/CodePush/**",
    );
```

可將 "ours" 視為「您的團隊」，"theirs" 視為「React Native 開發團隊」。

### Upgrade Helper

[Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) 是協助升級應用的網頁工具，提供任意兩個版本間完整的變更集合，並在特定檔案附加註解說明變更原因。

#### 1. 選擇版本

首先需選擇要從哪個版本升級至哪個版本（預設選取最新主要版本），點擊「Show me how to upgrade」按鈕後會顯示升級指引。

💡 主要版本升級時，頂部會顯示「實用內容」區塊，包含升級輔助連結。

#### 2. 升級依賴套件

首先顯示的是 `package.json` 檔案，建議更新其中標示的依賴套件。例如若顯示 `react-native` 和 `react` 變更，可執行 `yarn add` 安裝：

```shell
# {{VERSION}} and {{REACT_VERSION}} are the release versions showing in the diff
yarn add react-native@{{VERSION}}
yarn add react@{{REACT_VERSION}}
```

#### 3. 升級專案檔案

新版本可能包含對執行 `npx react-native init` 時生成的其他文件的更新，這些文件會列在 Upgrade Helper 頁面的 `package.json` 之後。如果沒有其他變更，您只需重新建置專案即可繼續開發。

若有變更，您可以手動從頁面上的變更內容複製貼上進行更新，或是透過執行以下 React Native CLI 升級指令來完成：

```shell
npx react-native upgrade
```

此指令會將您的文件與最新模板進行比對，並執行以下操作：

- 若模板中有新文件，則會建立該文件。
- 若模板中的文件與您的文件完全相同，則會跳過。
- 若專案中的文件與模板不同，系統會提示您選擇保留原文件或覆寫為模板版本。

> 部分升級無法透過 React Native CLI 自動完成，需手動處理（例如從 `0.28` 升級至 `0.29`，或從 `0.56` 升級至 `0.57`）。升級時請務必查閱[版本發布說明](https://github.com/facebook/react-native/releases)，以確認專案可能需要的手動調整。

### 疑難排解

#### 我想使用 React Native CLI 升級，但未使用 Git

即使您的專案未使用 Git 版本控制系統（可能使用 Mercurial、SVN 或未使用任何版本控制），仍需在系統中[安裝 Git](https://git-scm.com/downloads) 才能執行 `npx react-native upgrade`。Git 必須可於 `PATH` 中存取。若專案未使用 Git，請初始化並提交：

```shell
git init # Initialize a Git repository
git add . # Stage all the current files
git commit -m "Upgrade react-native" # Save the current files in a commit
```

升級完成後可刪除 `.git` 目錄。

#### 已完成所有變更，但應用程式仍使用舊版本

此類錯誤通常與快取有關，建議安裝 [react-native-clean-project](https://github.com/pmadruga/react-native-clean-project) 清除專案所有快取，然後重新執行。