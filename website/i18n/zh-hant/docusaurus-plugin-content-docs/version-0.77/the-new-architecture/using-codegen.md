# 使用 Codegen

本指南將教導您如何：

- 配置 **Codegen**。
- 為每個平台手動調用它。

同時也描述生成的程式碼結構。

## 先決條件

即使手動調用 **Codegen**，您始終需要一個 React Native 應用程式才能正確生成程式碼。

**Codegen** 流程與應用程式的建置緊密耦合，相關腳本位於 `react-native` NPM 套件中。

為本指南之目的，請使用 React Native CLI 創建專案如下：

```bash
npx @react-native-community/cli@latest init SampleApp --version 0.76.0
```

**Codegen** 用於為您的自訂模組或元件生成黏合程式碼。詳見 Turbo Native Modules 和 Fabric Native Components 指南以了解如何創建它們。

<!-- TODO: add links -->

## 配置 **Codegen**

可透過修改 `package.json` 文件來配置 **Codegen**。其行為由名為 `codegenConfig` 的自訂欄位控制。

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
          // we don't have a good example for this, but it works in the same way. Pass the name of the class that implements the RCTImageDataDecoder. It must be a Native Module.
        ]
      },
      "componentProvider": {
        "<componentName>": "<iOS-class-implementing-the-component>"
      },
    }
  },
```

您可將此程式碼片段加入應用程式並自訂各欄位：

- `name:` 用於創建包含規格的文件名稱。依慣例應包含後綴 `Spec`，但非強制。
- `type`: 需生成的程式碼類型。允許值為 `modules`、`components`、`all`。
  - `modules:` 僅需為 Turbo Native Modules 生成程式碼時使用。
  - `components:` 僅需為 Native Fabric Components 生成程式碼時使用。
  - `all`: 當同時存在元件和模組時使用。
- `jsSrcsDir`: 存放所有規格的根目錄。
- `android.javaPackageName`: Android 專用設定，用於指定生成文件的套件名稱。
- `ios`: `ios` 欄位為物件，可供應用開發者和函式庫維護者實現進階功能。以下所有欄位均為**可選**。
  - `ios.modulesConformingToProtocol`: React Native 提供了一些協議，原生模組可透過實現這些協議來客製化行為。此欄位用於定義符合這些協議的模組列表，這些模組將在應用啟動時注入 React Native 運行環境。
    - `ios.modulesConformingToProtocol.RCTImageURLLoader`: 實現 [`RCTImageURLLoader` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageURLLoader.h#L26-L81) 的 iOS 原生模組列表。需傳入實現該協議的 iOS 類別名稱，且必須為 Native Modules。
    - `ios.modulesConformingToProtocol.RCTURLRequestHandler`: 實現 [`RCTURLRequestHandler` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/React/Base/RCTURLRequestHandler.h#L11-L52) 的 iOS 原生模組列表。需傳入實現該協議的 iOS 類別名稱，且必須為 Native Modules。
    - `ios.modulesConformingToProtocol.RCTImageDataDecoder`: 實現 [`RCTImageDataDecoder` 協議](https://github.com/facebook/react-native/blob/00d5caee9921b6c10be8f7d5b3903c6afe8dbefa/packages/react-native/Libraries/Image/RCTImageDataDecoder.h#L15-L53) 的 iOS 原生模組列表。需傳入實現該協議的 iOS 類別名稱，且必須為 Native Modules。
  - `ios.componentProvider`: 此欄位為映射表，用於生成自訂 JS React 元件與實現它的原生類別之間的關聯。映射表的鍵為元件的 JS 名稱（例如 `TextInput`），值為實現該元件的 iOS 類別（例如 `RCTTextInput`）。

When **Codegen** runs, it searches among all the dependencies of the app, looking for JS files that respects some specific conventions, and it generates the required code:

- Turbo Native Modules require that the spec files are prefixed with `Native`. For example, `NativeLocalStorage.ts` is a valid name for a spec file.
- Native Fabric Components require that the spec files are suffixed with `NativeComponent`. For example, `WebViewNativeComponent.ts` is a valid name for a spec file.

## Running **Codegen**

The rest of this guide assumes that you have a Native Turbo Module, a Native Fabric Component or both already set up in your project. We also assume that you have valid specification files in the `jsSrcsDir` specified in the `package.json`.

### Android

**Codegen** for Android is integrated with the React Native Gradle Plugin (RNGP). The RNGP contains a task that can be invoked that reads the configurations defined in the `package.json` file and execute **Codegen**. To run the gradle task, first navigate inside the `android` folder of your project. Then run:

```bash
./gradlew generateCodegenArtifactsFromSchema
```

This task invokes the `generateCodegenArtifactsFromSchema` command on all the imported projects of the app (the app and all the node modules which are linked to it). It generates the code in the corresponding `node_modules/<dependency>` folder. For example, if you have a Fabric Native Component whose Node module is called `my-fabric-component`, the generated code is located in the `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` path. For the app, the code is generated in the `android/app/build/generated/source/codegen` folder.

#### The Generated Code

After running the gradle command above, you will find the codegen code in the `SampleApp/android/app/build` folder. The structure will look like this:

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

The generated code is split in two folders:

- `java` which contains the platform specific code
- `jni` which contains the C++ code required to let JS and Java interact correctly.

In the `java` folder, you can find the Fabric Native component generated code in the `com/facebook/viewmanagers` subfolder.

- the `<nativeComponent>ManagerDelegate.java` contains the methods that the `ViewManager` can call on the custom Native Component
- the `<nativeComponent>ManagerInterface.java` contains the interface of the `ViewManager`.

In the folder whose name was set up in the `codegenConfig.android.javaPackageName`, instead, you can find the abstract class that a Turbo Native Module has to implement to carry out its tasks.

In the `jni` folder, finally, there is all the boilerplate code to connect JS to Android.

- `<codegenConfig.name>.h` this contains the interface of your custom C++ Turbo Native Modules.
- `<codegenConfig.name>-generated.cpp` this contains the glue code of your custom C++ Turbo Native Modules.
- `react/renderer/components/<codegenConfig.name>`: this folder contains all the glue-code required by your custom component.

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.

### iOS

**Codegen** for iOS relies on some Node scripts that are invoked during the build process. The scripts are located in the `SampleApp/node_modules/react-native/scripts/` folder.

The main script is the `generate-codegen-artifacts.js` script. To invoke the script, you can run this command from the root folder of your app:

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

where:

- `--path` is the path to the root folder of your app.
- `--outputPath` is the destination where **Codegen** will write the generated files.
- `--targetPlatform` is the platform you'd like to generate the code for.

#### The Generated Code

Running the script with these arguments:

```shell
node node_modules/react-native/scripts/generate-codegen-artifacts.js \
    --path . \
    --outputPath ios/ \
    --targetPlatform ios
```

Will generate these files in the `ios/build` folder:

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

Part of these generated files are used by React Native in the Core. Then there is a set of files which contains the same name you specified in the package.json `codegenConfig.name` field.

- `<codegenConfig.name>/<codegenConfig.name>.h`: this contains the interface of your custom iOS Turbo Native Modules.
- `<codegenConfig.name>/<codegenConfig.name>-generated.mm`: this contains the glue code of your custom iOS Turbo Native Modules.
- `<codegenConfig.name>JSI.h`: this contains the interface of your custom C++ Turbo Native Modules.
- `<codegenConfig.name>JSI-generated.h`: this contains the glue code of your custom C++ Turbo Native Modules.
- `react/renderer/components/<codegenConfig.name>`: this folder contains all the glue-code required by your custom component.

This structure has been generated by using the value `all` for the `codegenConfig.type` field. If you use the value `modules`, expect to see no `react/renderer/components/` folder. If you use the value `components`, expect not to see any of the other files.