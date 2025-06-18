---
id: native-modules-setup
title: Native Modules NPM Package Setup
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

原生模組通常以 npm 套件形式發布，除了常規的 JavaScript 代碼外，還會包含各平台的本地代碼。若想更了解 npm 套件，可參考[此指南](https://docs.npmjs.com/packages-and-modules/contributing-packages-to-the-registry)。

要建立原生模組的基本專案結構，我們將使用社群工具 [create-react-native-library](https://github.com/callstack/react-native-builder-bob)。您可以進一步深入研究該工具的工作原理，但我們目前只需執行基本指令：

```shell
npx create-react-native-library@latest react-native-awesome-module
```

其中 `react-native-awesome-module` 是您為新模組指定的名稱。執行後請進入 `react-native-awesome-module` 資料夾，並透過以下指令初始化範例專案：

```shell
yarn
```

初始化完成後，您可透過以下任一指令啟動範例應用程式：

```shell
# Android app
yarn example android
# iOS app
yarn example ios
```

完成上述所有步驟後，您即可繼續參閱 [Android 原生模組](native-modules-android) 或 [iOS 原生模組](native-modules-ios) 指南來添加代碼。

> 若需要較不受限的設定方式，可參考第三方工具 [create-react-native-module](https://github.com/brodybits/create-react-native-module)。