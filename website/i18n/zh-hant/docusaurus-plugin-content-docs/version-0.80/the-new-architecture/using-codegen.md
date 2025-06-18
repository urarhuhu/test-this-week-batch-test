# 使用 Codegen

本指南將教您如何：

- 配置 **Codegen**。
- 為每個平台手動調用它。

同時也描述了生成的程式碼。

## 先決條件

即使手動調用 **Codegen**，您始終需要一個 React Native 應用程式來正確生成程式碼。

**Codegen** 過程與應用程式的構建緊密耦合，且腳本位於 `react-native` NPM 套件中。

為了本指南的目的，請使用 React Native CLI 創建一個專案，如下所示：

```bash
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

**Codegen** 用於為您的自訂模組或組件生成粘合程式碼。有關如何創建它們的詳細信息，請參閱 Turbo Native Modules 和 Fabric Native Components 的指南。

<!-- TODO: add links -->

## 配置 **Codegen**

可以通過修改 `package.json` 文件來配置 **Codegen**。**Codegen** 由一個名為 `codegenConfig` 的自訂字段控制。

```json title="package.json"
  "codegenConfig": {
    "name": "<SpecName>",
    "type": "<types>",
    "jsSrcsDir": "<source_dir>",
    "android": {
      "javaPackageName": "<java.package.name>"
    },
    "ios": {
      "modules": {
        "TestModule": {
          "className": "<iOS-class-implementing-the-RCTModuleProvider-protocol>",
          "unstableRequiresMainQueueSetup": false
          "conformsToProtocols": ["RCTImageURLLoader", "RCTURLRequestHandler", "RCTImageDataDecoder"],
        }
      },
      "components": {
        "TestComponent": {
          "className": "<iOS-class-implementing-the-component>"
        }
      }
    }
  },
```

您可以將此片段添加到您的應用程式中，並自訂各個字段：

- `name:` Codegen 配置的名稱。這將自訂 codgen 的輸出：文件名和程式碼。
- `type:`
  - `modules:` 僅為模組生成程式碼。
  - `components:` 僅為組件生成程式碼。
  - `all`: 為所有內容生成程式碼。
- `jsSrcsDir`: 所有規格文件所在的根目錄。
- `android`: Android 的 Codegen 配置（全部可選）：
  - `.javaPackageName`: 配置 Android Java 程式碼生成輸出的包名。
- `ios`: iOS 的 Codegen 配置（全部可選）：
  - `.modules[moduleName]:`
    - `.className`: 此模組的 ObjC 類。或者，如果它是 [僅 C++ 模組](/docs/next/the-new-architecture/pure-cxx-modules)，則是其 `RCTModuleProvider` 類。
    - `.unstableRequiresMainQueueSetup`: 在運行任何 JavaScript 之前，在 UI 線程上初始化此模組。
    - `.conformsToProtocols`: 註釋此模組符合以下哪些協議：[`RCTImageURLLoader`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageURLLoader.h#L26-L81)、[`RCTURLRequestHandler`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/React/Base/RCTURLRequestHandler.h#L11-L52)、[`RCTImageDataDecoder`](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageDataDecoder.h#L15-L53)。
  - `.components[componentName]`:
    - `.className`: 此組件的 ObjC 類（例如：`TextInput` -> `RCTTextInput`）。

當 **Codegen** 運行時，它會在應用程式的所有依賴項中搜索符合特定約定的 JS 文件，並生成所需的程式碼：

- Turbo Native Modules 要求規格文件以 `Native` 為前綴。例如，`NativeLocalStorage.ts` 是一個有效的規格文件名。
- Native Fabric Components 要求規格文件以 `NativeComponent` 為後綴。例如，`WebViewNativeComponent.ts` 是一個有效的規格文件名。

## 運行 **Codegen**

本指南的其餘部分假設您已經在專案中設置了 Native Turbo Module、Native Fabric Component 或兩者。我們還假設您在 `package.json` 中指定的 `jsSrcsDir` 中有有效的規格文件。

### Android

Android 平台的 **Codegen** 已整合至 React Native Gradle 插件 (RNGP)。RNGP 包含一個可執行任務，該任務會讀取 `package.json` 中定義的配置並執行 **Codegen**。要運行此 gradle 任務，請先進入專案的 `android` 資料夾，然後執行：

```bash
./gradlew generateCodegenArtifactsFromSchema
```

此任務會對所有導入的專案（主應用及所有連結的 node 模組）執行 `generateCodegenArtifactsFromSchema` 指令。生成的程式碼會存放在對應的 `node_modules/<dependency>` 資料夾中。例如，若您有一個名為 `my-fabric-component` 的 Fabric 原生元件模組，生成的程式碼將位於 `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` 路徑下。主應用的程式碼則會生成在 `android/app/build/generated/source/codegen` 資料夾中。

#### 生成的程式碼

執行上述 gradle 指令後，您可以在 `SampleApp/android/app/build` 資料夾中找到生成的程式碼，其結構如下：

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

生成的程式碼分為兩個資料夾：

- `java` 包含平台專屬程式碼
- `jni` 包含讓 JavaScript 與 Java 正確互動所需的 C++ 程式碼

在 `java` 資料夾中，您可以在 `com/facebook/viewmanagers` 子資料夾找到 Fabric 原生元件的生成程式碼。

- `<nativeComponent>ManagerDelegate.java` 包含 `ViewManager` 可調用的自訂原生元件方法
- `<nativeComponent>ManagerInterface.java` 包含 `ViewManager` 的介面定義

在 `codegenConfig.android.javaPackageName` 指定的套件名稱對應資料夾中，則可找到 Turbo 原生模組需實現的抽象類別。

最後，`jni` 資料夾內包含所有連接 JavaScript 與 Android 的樣板程式碼。

- `<codegenConfig.name>.h` 包含自訂 C++ Turbo 原生模組的介面
- `<codegenConfig.name>-generated.cpp` 包含自訂 C++ Turbo 原生模組的黏合程式碼
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含自訂元件所需的所有黏合程式碼

此結構是使用 `codegenConfig.type` 設為 `all` 時生成的。若設為 `modules`，將不會看到 `react/renderer/components/` 資料夾；若設為 `components`，則不會看到其他檔案。

### iOS

iOS 平台的 **Codegen** 依賴建置過程中調用的 Node 腳本，這些腳本位於 `SampleApp/node_modules/react-native/scripts/` 資料夾。

主要腳本為 `generate-codegen-artifacts.js`。要執行此腳本，請從應用根目錄運行以下指令：

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

- `--path` 指定應用根目錄路徑
- `--outputPath` 指定 **Codegen** 生成檔案的輸出位置
- `--targetPlatform` 指定目標生成平台

#### 生成的程式碼

使用以下參數執行腳本：

```shell
node node_modules/react-native/scripts/generate-codegen-artifacts.js \
    --path . \
    --outputPath ios/ \
    --targetPlatform ios
```

將在 `ios/build` 資料夾生成以下檔案：

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

部分生成檔案會被 React Native 核心使用，其餘檔案名稱則對應 `package.json` 中 `codegenConfig.name` 欄位的設定值。

- `<codegenConfig.name>/<codegenConfig.name>.h`：此檔案包含您自訂 iOS Turbo 原生模組的介面。
- `<codegenConfig.name>/<codegenConfig.name>-generated.mm`：此檔案包含您自訂 iOS Turbo 原生模組的黏合程式碼。
- `<codegenConfig.name>JSI.h`：此檔案包含您自訂 C++ Turbo 原生模組的介面。
- `<codegenConfig.name>JSI-generated.h`：此檔案包含您自訂 C++ Turbo 原生模組的黏合程式碼。
- `react/renderer/components/<codegenConfig.name>`：此資料夾包含您自訂元件所需的所有黏合程式碼。

此結構是透過在 `codegenConfig.type` 欄位使用 `all` 值所生成。若使用 `modules` 值，則不會看到 `react/renderer/components/` 資料夾；若使用 `components` 值，則不會看到其他檔案。