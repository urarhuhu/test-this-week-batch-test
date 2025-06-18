---
id: local-library-setup
title: Local libraries setup
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

本地函式庫是指包含視圖或模組且僅限於您應用程式內部使用、未發佈至註冊表的程式庫。與傳統視圖和模組的設置不同，本地函式庫與應用程式的原生代碼是解耦的。

本地函式庫建立於`android/`和`ios/`資料夾之外，並利用自動連結(autolinking)機制與您的應用程式整合。包含本地函式庫的專案結構可能如下：

```plaintext
MyApp
├── node_modules
├── modules <-- folder for your local libraries
│ └── awesome-module <-- your local library
├── android
├── ios
├── src
├── index.js
└── package.json
```

由於本地函式庫的代碼存在於`android/`和`ios/`資料夾之外，未來升級React Native版本、複製到其他專案等操作會更加容易。

我們將使用[create-react-native-library](https://callstack.github.io/react-native-builder-bob/create)工具來建立本地函式庫。此工具包含所有必要的模板。

### 開始使用

在您的React Native應用程式根目錄下執行以下命令：

```shell
npx create-react-native-library@latest awesome-module
```

其中`awesome-module`是您為新模組指定的名稱。完成提示後，專案根目錄會新增一個名為`modules`的資料夾，其中包含新建立的模組。

### 連結

預設情況下，生成的函式庫會自動透過Yarn的`link:`協議或npm的`file:`協議連結至專案：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>

<TabItem value="npm">

```json
"dependencies": {
  "awesome-module": "file:./modules/awesome-module"
}
```

</TabItem>
<TabItem value="yarn">

```json
"dependencies": {
  "awesome-module": "link:./modules/awesome-module"
}
```

</TabItem>
</Tabs>

這會在`node_modules`下建立指向函式庫的符號連結，使自動連結機制生效。

### 安裝依賴項

要連結模組，需先安裝其依賴項：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>

<TabItem value="npm">

```shell
npm install
```

</TabItem>
<TabItem value="yarn">

```shell
yarn install
```

</TabItem>
</Tabs>

### 在應用程式中使用模組

您可透過模組名稱直接導入使用：

```js
import {multiply} from 'awesome-module';
```