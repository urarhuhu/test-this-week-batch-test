---
id: running-on-simulator-ios
title: Running On Simulator
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 啟動模擬器

當您的 React Native 專案初始化完成後，您可以在新建立的專案目錄中執行以下指令。

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios
```

</TabItem>
</Tabs>

如果所有設定都正確無誤，您應該很快就能在 iOS 模擬器中看到您的新應用程式運行。

## 指定裝置

您可以使用 `--simulator` 參數來指定模擬器運行的裝置，後面接上裝置名稱的字串。預設值為 `"iPhone 14"`。如果您希望在 iPhone SE（第三代）上運行您的應用程式，請執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios -- --simulator="iPhone SE (3rd generation)"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios --simulator "iPhone SE (3rd generation)"
```

</TabItem>
</Tabs>

裝置名稱對應於 Xcode 中可用的裝置列表。您可以通過在控制台中執行 `xcrun simctl list devices` 來檢查您可用的裝置。

### 指定裝置版本

如果您安裝了多個 iOS 版本，您還需要指定適當的版本。例如，要在 iPhone 14 Pro（16.0 版本）上運行您的應用程式，請執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios -- --simulator="iPhone 14 Pro (16.0)"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios --simulator "iPhone 14 Pro (16.0)"
```

</TabItem>
</Tabs>

## 指定 UDID

您可以指定從 `xcrun simctl list devices` 指令返回的裝置 UDID。例如，要在 UDID 為 `AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA` 的裝置上運行您的應用程式，請執行以下指令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm run ios -- --udid="AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA"
```

</TabItem>
<TabItem value="yarn">

```shell
yarn ios --udid "AAAAAAAA-AAAA-AAAA-AAAA-AAAAAAAAAAAA"
```

</TabItem>
</Tabs>