---
id: debugging-native-code
title: Debugging Native Code
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3><p>
    The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/workflow/prebuild/" target="_blank">prebuild</a> to use this API.</p>
</div>

## 存取日誌

當應用程式運行時，您可以在終端機中使用以下指令來顯示 iOS 或 Android 應用的原生日誌：

```shell
# For Android:
npx react-native log-android
# Or, for iOS:
npx react-native log-ios
```

您也可以透過 iOS 模擬器中的「Debug > Open System Log…」選項，或當 Android 應用在裝置或模擬器上運行時，在終端機中執行 `adb logcat "*:S" ReactNative:V ReactNativeJS:V` 來存取這些日誌。

## 在原生 IDE 中進行除錯

當處理原生程式碼時，例如編寫原生模組，您可以從 Android Studio 或 Xcode 啟動應用，並利用原生除錯功能（設定中斷點等），就像在開發標準原生應用時一樣。

另一個選項是使用 React Native CLI 運行您的應用，並將原生 IDE（Android Studio 或 Xcode）的原生除錯器附加到該程序。

### Android Studio

在 Android Studio 中，您可以透過點選選單列上的「Run」選項，點擊「Attach to Process…」，然後選擇正在運行的 React Native 應用來完成此操作。

### Xcode

在 Xcode 中，點擊頂部選單列上的「Debug」，選擇「Attach to process」選項，然後在「Likely Targets」列表中選擇應用程式。