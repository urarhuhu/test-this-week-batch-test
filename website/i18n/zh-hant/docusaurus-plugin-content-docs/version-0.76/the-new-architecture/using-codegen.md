# 使用 Codegen

本指南將教您如何：

- 配置 **Codegen**。
- 為每個平台手動調用它。

同時也描述生成的程式碼。

## 先決條件

即使手動調用 **Codegen**，您始終需要一個 React Native 應用程式才能正確生成程式碼。

**Codegen** 過程與應用程式的構建緊密耦合，腳本位於 `react-native` NPM 套件中。

為了本指南的緣故，請使用 React Native CLI 創建一個專案，如下所示：

```bash
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

**Codegen** 用於為您的自定義模組或組件生成粘合程式碼。有關如何創建它們的詳細信息，請參閱 Turbo Native Modules 和 Fabric Native Components 的指南。

<!-- TODO: add links -->

## 配置 **Codegen**

可以通過修改 `package.json` 文件來配置 **Codegen**。**Codegen** 由一個名為 `codegenConfig` 的自定義字段控制。

```json title="package.json"
  "codegenConfig": {
    "name": "<SpecName>",
    "type": "<types>",
    "jsSrcsDir": "<source_dir>",
    "android": {
      "javaPackageName": "<java.package.name>"
    },
    "ios": {
      "modulesConformingToProtocol": {
        "RCTImageURLLoader": [
          "<iOS-class-conforming-to-RCTImageURLLoader>",
          // example from react-native-camera-roll: https://github.com/react-native-cameraroll/react-native-cameraroll/blob/8a6d1b4279c76e5682a4b443e7a4e111e774ec0a/package.json#L118-L127
          // "RNCPHAssetLoader",
        ],
        "RCTURLRequestHandler": [
          "<iOS-class-conforming-to-RCTURLRequestHandler>",
          // example from react-native-camera-roll: https://github.com/react-native-cameraroll/react-native-cameraroll/blob/8a6d1b4279c76e5682a4b443e7a4e111e774ec0a/package.json#L118-L127
          // "RNCPHAssetUploader",
        ],
        "RCTImageDataDecoder": [
          "<iOS-class-conforming-to-RCTImageDataDecoder>",
          // we don't have a good example for this, but it works in the same way. Pass the name of the class that implements the RCTImageDataDecoder. It must be a Native Modules.
        ]
      }
    }
  },
```

您可以將此代碼片段添加到您的應用程式中並自定義各個字段：

- `name:` 這將用於創建包含您的規格的文件的名稱。按照慣例，它應該有後綴 `Spec`，但這不是強制性的。
- `type`: 我們需要生成的程式碼類型。允許的值為 `modules`、`components`、`all`。
  - `modules:` 如果您只需要為 Turbo Native Modules 生成程式碼，請使用此值。
  - `components:` 如果您只需要為 Native Fabric Components 生成程式碼，請使用此值。
  - `all`: 如果您有混合的組件和模組，請使用此值。
- `jsSrcsDir`: 這是所有規格所在的根文件夾。
- `android.javaPackageName`: 這是一個 Android 特定的設置，讓 **Codegen** 在自定義包中生成文件。
- `ios`: `ios` 字段是一個對象，應用程式開發人員和庫維護人員可以使用它來提供一些高級功能。以下所有字段都是**可選的**。
  - `ios.modulesConformingToProtocol`: React Native 提供了一些協議，原生模組可以實現這些協議來自定義某些行為。這些字段允許您定義符合這些協議的模組列表。這些模組將在應用程式啟動時注入到 React Native 運行時中。
    - `ios.modulesConformingToProtocol.RCTImageURLLoader`: 實現 [`RCTImageURLLoader` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageURLLoader.h#L26-L81) 的 iOS 原生模組列表。您需要傳遞實現 `RCTImageURLLoader` 的 iOS 類的類名。它們必須是 Native Modules。
    - `ios.modulesConformingToProtocol.RCTURLRequestHandler`: 實現 [`RCTURLRequestHandler` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/React/Base/RCTURLRequestHandler.h#L11-L52) 的 iOS 原生模組列表。您需要傳遞實現 `RCTURLRequestHandler` 的 iOS 類的類名。它們必須是 Native Modules。
    - `ios.modulesConformingToProtocol.RCTImageDataDecoder`: 實現 [`RCTImageDataDecoder` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageDataDecoder.h#L15-L53) 的 iOS 原生模組列表。您需要傳遞實現 `RCTImageDataDecoder` 的 iOS 類的類名。它們必須是 Native Modules。

當 **Codegen** 運行時，它會在應用程式的所有依賴項中搜索，尋找符合某些特定約定的 JS 文件，並生成所需的程式碼：

- Turbo Native Modules 要求規格檔案必須以 `Native` 作為前綴。例如，`NativeLocalStorage.ts` 就是一個有效的規格檔案名稱。
- Native Fabric Components 則要求規格檔案必須以 `NativeComponent` 作為後綴。例如，`WebViewNativeComponent.ts` 就是一個有效的規格檔案名稱。

## 執行 **Codegen**

本指南後續內容假設您已在專案中設置好 Native Turbo Module、Native Fabric Component 或兩者兼具。同時也假設您已在 `package.json` 指定的 `jsSrcsDir` 目錄中準備好有效的規格檔案。

### Android

Android 的 **Codegen** 已整合至 React Native Gradle Plugin (RNGP)。RNGP 包含一個可執行的任務，該任務會讀取 `package.json` 中的配置並執行 **Codegen**。要執行此 gradle 任務，請先進入專案的 `android` 資料夾，然後執行：

```bash
./gradlew generateCodegenArtifactsFromSchema
```

此任務會在應用程式所有導入的專案（包含應用程式本身及所有連結的 node modules）上呼叫 `generateCodegenArtifactsFromSchema` 指令。產生的程式碼會存放在對應的 `node_modules/<dependency>` 資料夾中。舉例來說，若您有一個 Node module 名為 `my-fabric-component` 的 Fabric Native Component，產生的程式碼會位於 `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` 路徑。而應用程式本身的程式碼則會生成在 `android/app/build/generated/source/codegen` 資料夾。

#### 產生的程式碼

執行上述 gradle 指令後，您可以在 `SampleApp/android/app/build` 資料夾中找到 codegen 產生的程式碼。其結構如下：

```
build
└── generated
    └── source
        └── codegen
            ├── java
            │   └── com
            │       ├── facebook
            │       │   └── react
            │       │       └── viewmanagers
            │       │           ├── <nativeComponent>ManagerDelegate.java
            │       │           └── <nativeComponent>ManagerInterface.java
            │       └── sampleapp
            │           └── NativeLocalStorageSpec.java
            ├── jni
            │   ├── <codegenConfig.name>-generated.cpp
            │   ├── <codegenConfig.name>.h
            │   ├── CMakeLists.txt
            │   └── react
            │       └── renderer
            │           └── components
            │               └── <codegenConfig.name>
            │                   ├── <codegenConfig.name>JSI-generated.cpp
            │                   ├── <codegenConfig.name>.h
            │                   ├── ComponentDescriptors.cpp
            │                   ├── ComponentDescriptors.h
            │                   ├── EventEmitters.cpp
            │                   ├── EventEmitters.h
            │                   ├── Props.cpp
            │                   ├── Props.h
            │                   ├── ShadowNodes.cpp
            │                   ├── ShadowNodes.h
            │                   ├── States.cpp
            │                   └── States.h
            └── schema.json
```

產生的程式碼分為兩個資料夾：

- `java` 包含平台專屬的程式碼
- `jni` 包含讓 JavaScript 與 Java 正確互動所需的 C++ 程式碼

在 `java` 資料夾中，您可以在 `com/facebook/viewmanagers` 子資料夾找到 Fabric Native component 產生的程式碼。

- `<nativeComponent>ManagerDelegate.java` 包含 `ViewManager` 可呼叫的自訂 Native Component 方法
- `<nativeComponent>ManagerInterface.java` 包含 `ViewManager` 的介面

在 `codegenConfig.android.javaPackageName` 設定的名稱對應資料夾中，則可找到 Turbo Native Module 需實作的抽象類別，以執行其任務。

最後在 `jni` 資料夾中，包含所有連接 JavaScript 與 Android 的樣板程式碼。

- `<codegenConfig.name>.h` 包含自訂 C++ Turbo Native Modules 的介面
- `<codegenConfig.name>-generated.cpp` 包含自訂 C++ Turbo Native Modules 的黏合程式碼
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含自訂元件所需的所有黏合程式碼

此結構是使用 `codegenConfig.type` 欄位值為 `all` 時所產生。若使用 `modules` 值，將不會看到 `react/renderer/components/` 資料夾；若使用 `components` 值，則不會看到其他檔案。

### iOS

iOS 的 **Codegen** 依賴建置過程中呼叫的 Node 腳本。這些腳本位於 `SampleApp/node_modules/react-native/scripts/` 資料夾中。

主要腳本為 `generate-codegen-artifacts.js`。要呼叫此腳本，您可從應用程式根目錄執行以下指令：

```bash
node node_modules/react-native/scripts/generate-codegen-artifacts.js

Usage: generate-codegen-artifacts.js -p [path to app] -t [target platform] -o [output path]

Options:
      --help            Show help                                      [boolean]
      --version         Show version number                            [boolean]
  -p, --path            Path to the React Native project root.        [required]
  -t, --targetPlatform  Target platform. Supported values: "android", "ios",
                        "all".                                        [required]
  -o, --outputPath      Path where generated artifacts will be output to.
```

其中：

- `--path` 是應用程式根目錄的路徑
- `--outputPath` 是 **Codegen** 寫入產生檔案的目標位置
- `--targetPlatform` 是您想產生程式碼的目標平台

#### 產生的程式碼

使用以下參數執行腳本：

```shell
node node_modules/react-native/scripts/generate-codegen-artifacts.js \
    --path . \
    --outputPath ios/ \
    --targetPlatform ios
```

將會在 `ios/build` 資料夾中生成以下檔案：

```
build
└── generated
    └── ios
        ├── <codegenConfig.name>
        │   ├── <codegenConfig.name>-generated.mm
        │   └── <codegenConfig.name>.h
        ├── <codegenConfig.name>JSI-generated.cpp
        ├── <codegenConfig.name>JSI.h
        ├── FBReactNativeSpec
        │   ├── FBReactNativeSpec-generated.mm
        │   └── FBReactNativeSpec.h
        ├── FBReactNativeSpecJSI-generated.cpp
        ├── FBReactNativeSpecJSI.h
        ├── RCTModulesConformingToProtocolsProvider.h
        ├── RCTModulesConformingToProtocolsProvider.mm
        └── react
            └── renderer
                └── components
                    └── <codegenConfig.name>
                        ├── ComponentDescriptors.cpp
                        ├── ComponentDescriptors.h
                        ├── EventEmitters.cpp
                        ├── EventEmitters.h
                        ├── Props.cpp
                        ├── Props.h
                        ├── RCTComponentViewHelpers.h
                        ├── ShadowNodes.cpp
                        ├── ShadowNodes.h
                        ├── States.cpp
                        └── States.h
```

這些生成的檔案中，部分會被 React Native 核心使用。其餘檔案則包含你在 package.json 的 `codegenConfig.name` 欄位中指定的名稱。

- `<codegenConfig.name>/<codegenConfig.name>.h`：此檔案包含你自訂 iOS Turbo Native Modules 的介面。
- `<codegenConfig.name>/<codegenConfig.name>-generated.mm`：此檔案包含你自訂 iOS Turbo Native Modules 的黏合程式碼。
- `<codegenConfig.name>JSI.h`：此檔案包含你自訂 C++ Turbo Native Modules 的介面。
- `<codegenConfig.name>JSI-generated.h`：此檔案包含你自訂 C++ Turbo Native Modules 的黏合程式碼。
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含你自訂元件所需的所有黏合程式碼。

此結構是透過將 `codegenConfig.type` 欄位設為 `all` 所生成。若使用 `modules` 值，將不會看到 `react/renderer/components/` 資料夾。若使用 `components` 值，則不會看到其他檔案。