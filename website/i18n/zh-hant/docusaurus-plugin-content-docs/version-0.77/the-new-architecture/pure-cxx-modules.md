# 跨平台原生模組 (C++)

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

使用 C++ 編寫模組是實現 Android 和 iOS 平台無關程式碼共享的最佳方式。透過純 C++ 模組，您只需編寫一次邏輯，即可在所有平台上直接重用，無需編寫平台特定程式碼。

本指南將逐步說明如何建立純 C++ Turbo 原生模組：

1. 建立 JS 規格文件
2. 配置 Codegen 生成框架程式碼
3. 實作原生邏輯
4. 在 Android 和 iOS 應用中註冊模組
5. 在 JS 中測試變更

本指南後續內容假設您已透過以下指令建立應用程式：

```shell
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

## 1. 建立 JS 規格文件

純 C++ Turbo 原生模組屬於 Turbo 原生模組，需要規格文件（spec file）讓 Codegen 為我們生成框架程式碼。該規格文件也是我們在 JS 中存取 Turbo 原生模組的介面。

規格文件需使用類型化 JS 方言編寫。React Native 目前支援 Flow 或 TypeScript。

1. 在應用程式根目錄下建立名為 `specs` 的新資料夾
2. 建立名為 `NativeSampleModule.ts` 的新檔案，內容如下：

:::warning
所有原生 Turbo 模組規格文件必須以 `Native` 前綴開頭，否則 Codegen 將忽略它們。
:::

<Tabs groupId="tnm-specs" queryString defaultValue={constants.defaultJavaScriptSpecLanguages} values={constants.javaScriptSpecLanguages}>
<TabItem value="flow">

```ts title="specs/NativeSampleModule.ts"
// @flow
import type {TurboModule} from 'react-native'
import { TurboModuleRegistry } from "react-native";

export interface Spec extends TurboModule {
  +reverseString: (input: string) => string;
}

export default (TurboModuleRegistry.getEnforcing<Spec>(
  "NativeSampleModule"
): Spec);
```

</TabItem>
<TabItem value="typescript">

```ts title="specs/NativeSampleModule.ts"
import {TurboModule, TurboModuleRegistry} from 'react-native';

export interface Spec extends TurboModule {
  readonly reverseString: (input: string) => string;
}

export default TurboModuleRegistry.getEnforcing<Spec>(
  'NativeSampleModule',
);
```

</TabItem>
</Tabs>

## 2. 配置 Codegen

下一步是在 `package.json` 中配置 [Codegen](what-is-codegen.md)。更新檔案內容如下：

```json title="package.json"
     "start": "react-native start",
     "test": "jest"
   },
   // highlight-add-start
   "codegenConfig": {
     "name": "AppSpecs",
     "type": "modules",
     "jsSrcsDir": "specs",
     "android": {
       "javaPackageName": "com.sampleapp.specs"
     }
   },
   // highlight-add-end
   "dependencies": {
```

此配置指示 Codegen 在 `specs` 資料夾中尋找規格文件，並僅為 `modules` 生成程式碼，同時將生成的程式碼命名空間設為 `AppSpecs`。

## 3. 編寫原生程式碼

編寫 C++ Turbo 原生模組可實現 Android 與 iOS 的程式碼共享。因此我們只需編寫一次程式碼，後續將說明如何調整各平台配置以載入這些 C++ 程式碼。

1. 在與 `android` 和 `ios` 資料夾同級目錄下建立名為 `shared` 的資料夾
2. 在 `shared` 資料夾中建立新檔案 `NativeSampleModule.h`

   ```cpp title="shared/NativeSampleModule.h"
   #pragma once

   #include <AppSpecsJSI.h>

   #include <memory>
   #include <string>

   namespace facebook::react {

   class NativeSampleModule : public NativeSampleModuleCxxSpec<NativeSampleModule> {
   public:
     NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker);

     std::string reverseString(jsi::Runtime& rt, std::string input);
   };

   } // namespace facebook::react

   ```

3. 在 `shared` 資料夾中建立新檔案 `NativeSampleModule.cpp`

   ```cpp title="shared/NativeSampleModule.cpp"
   #include "NativeSampleModule.h"

   namespace facebook::react {

   NativeSampleModule::NativeSampleModule(std::shared_ptr<CallInvoker> jsInvoker)
       : NativeSampleModuleCxxSpec(std::move(jsInvoker)) {}

   std::string NativeSampleModule::reverseString(jsi::Runtime& rt, std::string input) {
     return std::string(input.rbegin(), input.rend());
   }

   } // namespace facebook::react
   ```

讓我們檢視這兩個新建的檔案：

- `NativeSampleModule.h` 檔案是一個純 C++ TurboModule 的標頭檔。`include` 語句確保我們包含了由 Codegen 生成的規格檔，其中包含我們需要實作的介面與基礎類別。
- 該模組位於 `facebook::react` 命名空間中，以便存取該命名空間內的所有類型。
- `NativeSampleModule` 類別是實際的 Turbo Native Module 類別，它繼承了 `NativeSampleModuleCxxSpec` 類別，後者包含了一些黏合程式碼與樣板程式碼，讓這個類別能作為 Turbo Native Module 運作。
- 最後，我們有建構函式，它接受一個指向 `CallInvoker` 的指標，以便在需要時與 JS 溝通，以及我們需要實作的函式原型。

`NativeSampleModule.cpp` 檔案是我們 Turbo Native Module 的實際實作，它實作了我們在規格檔中宣告的建構函式與方法。

## 4. 在平台中註冊模組

接下來的步驟將讓我們在平台中註冊模組。這一步驟將原生程式碼暴露給 JS，以便 React Native 應用程式最終可以從 JS 層呼叫原生方法。

這是我們唯一需要撰寫一些平台特定程式碼的時候。

### Android

為了確保 Android 應用程式能夠有效地建置 C++ Turbo Native Module，我們需要：

1. 建立一個 `CMakeLists.txt` 來存取我們的 C++ 程式碼。
2. 修改 `build.gradle` 以指向新建立的 `CMakeLists.txt` 檔案。
3. 在我們的 Android 應用程式中建立一個 `OnLoad.cpp` 檔案來註冊新的 Turbo Native Module。

#### 1. 建立 `CMakeLists.txt` 檔案

Android 使用 CMake 來建置。CMake 需要存取我們在共享資料夾中定義的檔案才能建置它們。

1. 建立一個新資料夾 `SampleApp/android/app/src/main/jni`。`jni` 資料夾是 Android 的 C++ 部分所在的位置。
2. 建立一個 `CMakeLists.txt` 檔案並加入以下內容：

```shell title="CMakeLists.txt"
cmake_minimum_required(VERSION 3.13)

# Define the library name here.
project(appmodules)

# This file includes all the necessary to let you build your React Native application
include(${REACT_ANDROID_DIR}/cmake-utils/ReactNative-application.cmake)

# Define where the additional source code lives. We need to crawl back the jni, main, src, app, android folders
target_sources(${CMAKE_PROJECT_NAME} PRIVATE ../../../../../shared/NativeSampleModule.cpp)

# Define where CMake can find the additional header files. We need to crawl back the jni, main, src, app, android folders
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ../../../../../shared)
```

CMake 檔案做了以下事情：

- 定義 `appmodules` 函式庫，所有應用程式的 C++ 程式碼都將包含在其中。
- 載入基礎的 React Native CMake 檔案。
- 使用 `target_sources` 指令加入我們需要建置的模組 C++ 原始碼。預設情況下，React Native 會將預設原始碼填入 `appmodules` 函式庫，這裡我們加入我們自訂的部分。你可以看到我們需要從 `jni` 資料夾回溯到 `shared` 資料夾，那裡有我們的 C++ Turbo Module。
- 指定 CMake 可以找到模組標頭檔的位置。同樣地，我們需要從 `jni` 資料夾回溯。

#### 2. 修改 `build.gradle` 以包含自訂的 C++ 程式碼

Gradle 是協調 Android 建置的工具。我們需要告訴它哪裡可以找到建置 Turbo Native Module 的 `CMake` 檔案。

1. 開啟 `SampleApp/android/app/build.gradle` 檔案。
2. 在現有的 `android` 區塊中加入以下區塊：

```diff title="android/app/build.gradle"
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
        }
        release {
            // Caution! In production, you need to generate your own keystore file.
            // see https://reactnative.dev/docs/signed-apk-android.
            signingConfig signingConfigs.debug
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
    }

+   externalNativeBuild {
+       cmake {
+           path "src/main/jni/CMakeLists.txt"
+       }
+   }
}
```

這個區塊告訴 Gradle 檔案去哪裡尋找 CMake 檔案。路徑是相對於 `build.gradle` 檔案所在的資料夾，所以我們需要加入指向 `jni` 資料夾中 `CMakeLists.txt` 檔案的路徑。

#### 3. 註冊新的 Turbo Native Module

最後一步是在執行時期註冊新的 C++ Turbo Native Module，這樣當 JS 要求 C++ Turbo Native Module 時，應用程式知道去哪裡找到它並回傳。

1. 從資料夾 `SampleApp/android/app/src/main/jni` 執行以下命令：

```sh
curl -O https://raw.githubusercontent.com/facebook/react-native/v0.76.0/packages/react-native/ReactAndroid/cmake-utils/default-app-setup/OnLoad.cpp
```

2. 接著，修改此檔案如下：

```diff title="android/app/src/main/jni/OnLoad.cpp"
#include <DefaultComponentsRegistry.h>
#include <DefaultTurboModuleManagerDelegate.h>
#include <autolinking.h>
#include <fbjni/fbjni.h>
#include <react/renderer/componentregistry/ComponentDescriptorProviderRegistry.h>
#include <rncore.h>

+ // Include the NativeSampleModule header
+ #include <NativeSampleModule.h>

//...

std::shared_ptr<TurboModule> cxxModuleProvider(
    const std::string& name,
    const std::shared_ptr<CallInvoker>& jsInvoker) {
  // Here you can provide your CXX Turbo Modules coming from
  // either your application or from external libraries. The approach to follow
  // is similar to the following (for a module called `NativeCxxModuleExample`):
  //
  // if (name == NativeCxxModuleExample::kModuleName) {
  //   return std::make_shared<NativeCxxModuleExample>(jsInvoker);
  // }

+  // This code register the module so that when the JS side asks for it, the app can return it
+  if (name == NativeSampleModule::kModuleName) {
+    return std::make_shared<NativeSampleModule>(jsInvoker);
+  }

  // And we fallback to the CXX module providers autolinked
  return autolinking_cxxModuleProvider(name, jsInvoker);
}

// leave the rest of the file
```

這些步驟會從 React Native 下載原始的 `OnLoad.cpp` 檔案，以便我們能安全地覆寫它，將 C++ Turbo Native Module 載入應用程式中。

下載檔案後，我們可以透過以下方式修改它：

- 引入指向我們模組的標頭檔
- 註冊 Turbo Native Module，讓當 JavaScript 要求時，應用程式能回傳它。

現在，你可以在專案根目錄執行 `yarn android`，看到應用程式成功建置。

### iOS

為了確保 iOS 應用程式能有效建置 C++ Turbo Native Module，我們需要：

1. 安裝 Pods 並執行 Codegen。
2. 將 `shared` 資料夾加入我們的 iOS 專案。
3. 在應用程式中註冊 C++ Turbo Native Module。

#### 1. 安裝 Pods 並執行 Codegen

我們需要執行的第一步是每次準備 iOS 應用程式時的常規步驟。CocoaPods 是我們用來設定和安裝 React Native 相依套件的工具，在此過程中，它也會為我們執行 Codegen。

```bash
cd ios
bundle install
bundle exec pod install
```

#### 2. 將 shared 資料夾加入 iOS 專案

此步驟將 `shared` 資料夾加入專案，使其對 Xcode 可見。

1. 開啟 CocoPods 產生的 Xcode Workspace。

```bash
cd ios
open SampleApp.xcworkspace
```

2. 點擊左側的 `SampleApp` 專案，選擇 `Add files to "Sample App"...`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode1.png)

3. 選擇 `shared` 資料夾並點擊 `Add`。

![Add Files to Sample App...](/docs/assets/AddFilesToXcode2.png)

如果一切正確，左側的專案應該會如下所示：

![Xcode Project](/docs/assets/CxxTMGuideXcodeProject.png)

#### 3. 在應用程式中註冊 Cxx Turbo Native Module

:::warning
如果你的應用程式有一些用 C++ 編寫的本地模組，你將無法使用我們在 React Native 0.77 中提供的 Swift 版 AppDelegate。

如果你的應用程式屬於此類別，請跳過將 AppDelegate 遷移到 Swift 的步驟，繼續為你的應用程式使用 Objective-C++ 的 AppDelegate。

React Native 核心主要使用 C++ 開發，以鼓勵 iOS 和 Android 及其他平台間的程式碼共享。Swift 和 C++ 之間的互通性尚未成熟或穩定。我們正在尋找方法填補這一空白，讓你也遷移到 Swift。
:::

透過這最後一步，我們將告訴 iOS 應用程式去哪裡尋找純 C++ Turbo Native Module。

在 Xcode 中，開啟 `AppDelegate.mm` 檔案並如下修改：

```diff title="SampleApp/AppDelegate.mm"
#import <React/RCTBundleURLProvider.h>
+ #import <RCTAppDelegate+Protected.h>
+ #import "NativeSampleModule.h"

// ...
  return [[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];
#endif
}

+- (std::shared_ptr<facebook::react::TurboModule>)getTurboModule:(const std::string &)name
+                                                      jsInvoker:(std::shared_ptr<facebook::react::CallInvoker>)jsInvoker
+{
+  if (name == "NativeSampleModule") {
+    return std::make_shared<facebook::react::NativeSampleModule>(jsInvoker);
+  }
+
+  return [super getTurboModule:name jsInvoker:jsInvoker];
+}

@end
```

這些變更做了以下幾件事：

1. 引入 `RCTAppDelegate+Protected` 標頭檔，讓 AppDelegate 知道它符合 `RCTTurboModuleManagerDelegate` 協議。
2. 引入純 C++ Native Turbo Module 介面 `NativeSampleModule.h`
3. 覆寫 C++ 模組的 `getTurboModule` 方法，讓當 JavaScript 端要求名為 `NativeSampleModule` 的模組時，應用程式知道該回傳哪個模組。

如果你現在從 Xcode 建置應用程式，應該能成功建置。

## 5. 測試你的程式碼

現在是時候從 JS 存取我們的 C++ Turbo 原生模組了。為此，我們需要修改 `App.tsx` 檔案，導入 Turbo 原生模組並在程式碼中呼叫它。

1. 開啟 `App.tsx` 檔案。
2. 將範本內容替換為以下程式碼：

```tsx title="App.tsx"
import React from 'react';
import {
  Button,
  SafeAreaView,
  StyleSheet,
  Text,
  TextInput,
  View,
} from 'react-native';
import SampleTurboModule from './specs/NativeSampleModule';

function App(): React.JSX.Element {
  const [value, setValue] = React.useState('');
  const [reversedValue, setReversedValue] = React.useState('');

  const onPress = () => {
    const revString = SampleTurboModule.reverseString(value);
    setReversedValue(revString);
  };

  return (
    <SafeAreaView style={styles.container}>
      <View>
        <Text style={styles.title}>
          Welcome to C++ Turbo Native Module Example
        </Text>
        <Text>Write down here he text you want to revert</Text>
        <TextInput
          style={styles.textInput}
          placeholder="Write your text here"
          onChangeText={setValue}
          value={value}
        />
        <Button title="Reverse" onPress={onPress} />
        <Text>Reversed text: {reversedValue}</Text>
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 18,
    marginBottom: 20,
  },
  textInput: {
    borderColor: 'black',
    borderWidth: 1,
    borderRadius: 5,
    padding: 10,
    marginTop: 10,
  },
});

export default App;
```

這個應用程式中值得注意的程式碼行包括：

- `import SampleTurboModule from './specs/NativeSampleModule';`：這行程式碼在應用程式中導入 Turbo 原生模組，
- `onPress` 回調中的 `const revString = SampleTurboModule.reverseString(value);`：這是您在應用程式中使用 Turbo 原生模組的方式。

:::warning
為了這個範例的簡潔性，我們直接在應用程式中導入規格檔案。
最佳實踐是建立一個單獨的檔案來封裝規格，並在應用程式中使用該檔案。
這樣可以讓您準備規格的輸入，並在 JS 中對它們有更多控制權。
:::

恭喜，您已經寫出了第一個 C++ Turbo 原生模組！

| <center>Android</center>                                                                             | <center>iOS</center>                                                                          |
| ---------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| <center><img src="/docs/assets/CxxGuideAndroidVideo.gif" alt="Android Video" height="600"/></center> | <center><img src="/docs/assets/CxxGuideIOSVideo.gif" alt="iOS video" height="600" /></center> |