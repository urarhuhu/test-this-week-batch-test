import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## 核心概念

將 React Native 元件整合至 iOS 應用的關鍵在於：

1. 設定 React Native 依賴項與目錄結構。
2. 了解應用中將使用的 React Native 元件。
3. 使用 CocoaPods 添加這些元件作為依賴項。
4. 以 JavaScript 開發 React Native 元件。
5. 在 iOS 應用中添加 `RCTRootView`，該視圖將作為 React Native 元件的容器。
6. 啟動 React Native 伺服器並運行原生應用。
7. 驗證應用的 React Native 部分是否如預期運作。

## 必要條件

依照[環境設定指南](environment-setup)中的 React Native CLI 快速入門，配置開發環境以建構 iOS 適用的 React Native 應用。

### 1. 設定目錄結構

為確保流程順暢，請為整合的 React Native 專案建立新資料夾，並將現有 iOS 專案複製至 `/ios` 子資料夾中。

### 2. 安裝 JavaScript 依賴項

進入專案的根目錄，建立包含以下內容的新 `package.json` 檔案：

```
{
  "name": "MyReactNativeApp",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "yarn react-native start"
  }
}
```

接著，請確認已[安裝 yarn 套件管理工具](https://yarnpkg.com/lang/en/docs/install/)。

安裝 `react` 與 `react-native` 套件。開啟終端機或命令提示字元，導航至含 `package.json` 的目錄並執行：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install react-native
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add react-native
```

</TabItem>
</Tabs>

此操作將輸出類似以下的訊息（請在 yarn 輸出中向上滾動查看）：

> 警告："`react-native@0.52.2`" 有未滿足的對等依賴 "`react@16.2.0`"。

這屬正常現象，表示我們還需安裝 React：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm install react@version_printed_above
```

</TabItem>
<TabItem value="yarn">

```shell
yarn add react@version_printed_above
```

</TabItem>
</Tabs>

安裝過程會建立新的 `/node_modules` 資料夾，此資料夾儲存建構專案所需的所有 JavaScript 依賴項。

將 `node_modules/` 加入你的 `.gitignore` 檔案。

### 3. 安裝 CocoaPods

[CocoaPods](http://cocoapods.org) 是 iOS 與 macOS 開發的套件管理工具，我們用它將實際的 React Native 框架程式碼本地化加入當前專案。

建議透過 [Homebrew](http://brew.sh/) 安裝 CocoaPods。

```shell
brew install cocoapods
```

> 技術上雖可不使用 CocoaPods，但這將需要手動添加函式庫與連結器設定，使流程過度複雜化。

## 將 React Native 加入應用

假設[待整合的應用](https://github.com/JoelMarcey/iOS-2048)為 [2048](https://en.wikipedia.org/wiki/2048_%28video_game%29) 遊戲。下圖為未整合 React Native 時原生應用的主選單畫面。

![整合 RN 前的畫面](/docs/assets/react-native-existing-app-integration-ios-before.png)

### Xcode 命令列工具

安裝命令列工具。在 Xcode 選單中選擇 **Settings... (或 Preferences...)**，進入 Locations 面板，從 Command Line Tools 下拉選單中選擇最新版本進行安裝。

![Xcode 命令列工具](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### 配置 CocoaPods 依賴項

在將 React Native 整合至應用前，需決定要整合框架中的哪些部分。我們將使用 CocoaPods 來指定應用所依賴的這些「子規格」(subspecs)。

The list of supported `subspec`s is available in [`/node_modules/react-native/React.podspec`](https://github.com/facebook/react-native/blob/main/packages/react-native/React.podspec). They are generally named by functionality. For example, you will generally always want the `Core` `subspec`. That will get you the `AppRegistry`, `StyleSheet`, `View` and other core React Native libraries. If you want to add the React Native `Text` library (e.g., for `<Text>` elements), then you will need the `RCTText` `subspec`. If you want the `Image` library (e.g., for `<Image>` elements), then you will need the `RCTImage` `subspec`.

You can specify which `subspec`s your app will depend on in a `Podfile` file. The easiest way to create a `Podfile` is by running the CocoaPods `init` command in the `/ios` subfolder of your project:

```shell
pod init
```

The `Podfile` will contain a boilerplate setup that you will tweak for your integration purposes.

> The `Podfile` version changes depending on your version of `react-native`. Refer to https://react-native-community.github.io/upgrade-helper/ for the specific version of `Podfile` you should be using.

Ultimately, your `Podfile` should look something similar to this:

```
# The target name is most likely the name of your project.
target 'NumberTileGame' do

  # Your 'node_modules' directory is probably in the root of your project,
  # but if not, adjust the `:path` accordingly
  pod 'FBLazyVector', :path => "../node_modules/react-native/Libraries/FBLazyVector"
  pod 'FBReactNativeSpec', :path => "../node_modules/react-native/Libraries/FBReactNativeSpec"
  pod 'RCTRequired', :path => "../node_modules/react-native/Libraries/RCTRequired"
  pod 'RCTTypeSafety', :path => "../node_modules/react-native/Libraries/TypeSafety"
  pod 'React', :path => '../node_modules/react-native/'
  pod 'React-Core', :path => '../node_modules/react-native/'
  pod 'React-CoreModules', :path => '../node_modules/react-native/React/CoreModules'
  pod 'React-Core/DevSupport', :path => '../node_modules/react-native/'
  pod 'React-RCTActionSheet', :path => '../node_modules/react-native/Libraries/ActionSheetIOS'
  pod 'React-RCTAnimation', :path => '../node_modules/react-native/Libraries/NativeAnimation'
  pod 'React-RCTBlob', :path => '../node_modules/react-native/Libraries/Blob'
  pod 'React-RCTImage', :path => '../node_modules/react-native/Libraries/Image'
  pod 'React-RCTLinking', :path => '../node_modules/react-native/Libraries/LinkingIOS'
  pod 'React-RCTNetwork', :path => '../node_modules/react-native/Libraries/Network'
  pod 'React-RCTSettings', :path => '../node_modules/react-native/Libraries/Settings'
  pod 'React-RCTText', :path => '../node_modules/react-native/Libraries/Text'
  pod 'React-RCTVibration', :path => '../node_modules/react-native/Libraries/Vibration'
  pod 'React-Core/RCTWebSocket', :path => '../node_modules/react-native/'

  pod 'React-cxxreact', :path => '../node_modules/react-native/ReactCommon/cxxreact'
  pod 'React-jsi', :path => '../node_modules/react-native/ReactCommon/jsi'
  pod 'React-jsiexecutor', :path => '../node_modules/react-native/ReactCommon/jsiexecutor'
  pod 'React-jsinspector', :path => '../node_modules/react-native/ReactCommon/jsinspector'
  pod 'ReactCommon/callinvoker', :path => "../node_modules/react-native/ReactCommon"
  pod 'ReactCommon/turbomodule/core', :path => "../node_modules/react-native/ReactCommon"
  pod 'Yoga', :path => '../node_modules/react-native/ReactCommon/yoga'

  pod 'DoubleConversion', :podspec => '../node_modules/react-native/third-party-podspecs/DoubleConversion.podspec'
  pod 'glog', :podspec => '../node_modules/react-native/third-party-podspecs/glog.podspec'
  pod 'Folly', :podspec => '../node_modules/react-native/third-party-podspecs/Folly.podspec'

end
```

After you have created your `Podfile`, you are ready to install the React Native pod.

```shell
$ pod install
```

You should see output such as:

```
Analyzing dependencies
Fetching podspec for `React` from `../node_modules/react-native`
Downloading dependencies
Installing React (0.62.0)
Generating Pods project
Integrating client project
Sending stats
Pod installation complete! There are 3 dependencies from the Podfile and 1 total pod installed.
```

> If this fails with errors mentioning `xcrun`, make sure that in Xcode in **Settings... (or Preferences...) > Locations** the Command Line Tools are assigned.

### Code integration

Now we will actually modify the native iOS application to integrate React Native. For our 2048 sample app, we will add a "High Score" screen in React Native.

#### The React Native component

The first bit of code we will write is the actual React Native code for the new "High Score" screen that will be integrated into our application.

##### 1. Create a `index.js` file

First, create an empty `index.js` file in the root of your React Native project.

`index.js` is the starting point for React Native applications, and it is always required. It can be a small file that `require`s other file that are part of your React Native component or application, or it can contain all the code that is needed for it. In our case, we will put everything in `index.js`.

##### 2. Add your React Native code

In your `index.js`, create your component. In our sample here, we will add a `<Text>` component within a styled `<View>`

```jsx
import React from 'react';
import {AppRegistry, StyleSheet, Text, View} from 'react-native';

const RNHighScores = ({scores}) => {
  const contents = scores.map(score => (
    <Text key={score.name}>
      {score.name}:{score.value}
      {'\n'}
    </Text>
  ));
  return (
    <View style={styles.container}>
      <Text style={styles.highScoresTitle}>
        2048 High Scores!
      </Text>
      <Text style={styles.scores}>{contents}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#FFFFFF',
  },
  highScoresTitle: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  scores: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
});

// Module name
AppRegistry.registerComponent('RNHighScores', () => RNHighScores);
```

> `RNHighScores` is the name of your module that will be used when you add a view to React Native from within your iOS application.

#### The Magic: `RCTRootView`

Now that your React Native component is created via `index.js`, you need to add that component to a new or existing `ViewController`. The easiest path to take is to optionally create an event path to your component and then add that component to an existing `ViewController`.

We will tie our React Native component with a new native view in the `ViewController` that will actually contain it called `RCTRootView` .

##### 1. Create an Event Path

You can add a new link on the main game menu to go to the "High Score" React Native page.

![Event Path](/docs/assets/react-native-add-react-native-integration-link.png)

##### 2. Event Handler

我們現在將為選單連結添加事件處理器。一個方法會被添加到應用程式的主`ViewController`中。這裡就是`RCTRootView`發揮作用的地方。

當你建構React Native應用程式時，會使用[Metro打包工具][metro]來建立由React Native伺服器提供的`index.bundle`。在`index.bundle`中會有我們的`RNHighScore`模組。因此，我們需要將`RCTRootView`指向`index.bundle`資源的位置（透過`NSURL`）並將其與模組綁定。

為了除錯目的，我們會記錄事件處理器被調用的情況。接著，我們會建立一個字串，包含存在於`index.bundle`中的React Native程式碼位置。最後，我們會建立主要的`RCTRootView`。請注意我們如何提供`RNHighScores`作為`moduleName`，這是我們在[前面](#the-react-native-component)撰寫React Native元件程式碼時所建立的。

首先`import` `RCTRootView`的標頭檔。

```objectivec
#import <React/RCTRootView.h>
```

> 這裡的`initialProperties`是為了示範目的，讓我們的高分畫面有一些資料。在我們的React Native元件中，我們會使用`this.props`來存取這些資料。

```objectivec
- (IBAction)highScoreButtonPressed:(id)sender {
    NSLog(@"High Score Button Pressed");
    NSURL *jsCodeLocation = [NSURL URLWithString:@"http://localhost:8081/index.bundle?platform=ios"];

    RCTRootView *rootView =
      [[RCTRootView alloc] initWithBundleURL: jsCodeLocation
                                  moduleName: @"RNHighScores"
                           initialProperties:
                             @{
                               @"scores" : @[
                                 @{
                                   @"name" : @"Alex",
                                   @"value": @"42"
                                  },
                                 @{
                                   @"name" : @"Joel",
                                   @"value": @"10"
                                 }
                               ]
                             }
                               launchOptions: nil];
    UIViewController *vc = [[UIViewController alloc] init];
    vc.view = rootView;
    [self presentViewController:vc animated:YES completion:nil];
}
```

> 請注意`RCTRootView initWithBundleURL`會啟動一個新的JSC虛擬機器。為了節省資源並簡化原生應用中不同部分RN視圖之間的通信，你可以讓多個由React Native驅動的視圖與單一JS運行時關聯。要做到這一點，可以使用[`RCTBridge initWithBundleURL`](https://github.com/facebook/react-native/blob/main/packages/react-native/React/Base/RCTBridge.h#L94)來建立橋接器，然後使用`RCTRootView initWithBridge`，而不是使用`[RCTRootView alloc] initWithBundleURL`。

> 當將應用程式移至生產環境時，`NSURL`可以指向磁碟上的預打包檔案，例如透過`[[NSBundle mainBundle] URLForResource:@"main" withExtension:@"jsbundle"];`。你可以使用`node_modules/react-native/scripts/`中的`react-native-xcode.sh`腳本來生成該預打包檔案。

##### 3. 連接

將主選單中的新連結與新添加的事件處理器方法連接起來。

![事件路徑](/docs/assets/react-native-add-react-native-integration-wire-up.png)

> 其中一個較簡單的方法是：在storyboard中打開視圖，右鍵點擊新連結。選擇類似`Touch Up Inside`的事件，將其拖曳到storyboard，然後從提供的列表中選擇已建立的方法。

### 測試你的整合

你現在已完成將React Native與現有應用程式整合的所有基本步驟。現在我們將啟動[Metro打包工具][metro]來建構`index.bundle`套件，並啟動運行在`localhost`上的伺服器來提供它。

##### 1. 添加App Transport Security例外

Apple已禁止隱式的明文HTTP資源加載。因此我們需要在專案的`Info.plist`（或等效）檔案中添加以下內容。

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>localhost</key>
        <dict>
            <key>NSTemporaryExceptionAllowsInsecureHTTPLoads</key>
            <true/>
        </dict>
    </dict>
</dict>
```

> App Transport Security對你的用戶有好處。請確保在發布應用程式至生產環境前重新啟用它。

##### 2. 執行打包工具

要運行你的應用程式，首先需要啟動開發伺服器。為此，請在React Native專案的根目錄中執行以下命令：

<Tabs groupId="package-manager" queryString defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```shell
npm start
```

</TabItem>
<TabItem value="yarn">

```shell
yarn start
```

</TabItem>
</Tabs>

##### 3. 執行應用程式

如果你使用Xcode或你喜歡的編輯器，像平常一樣建構並運行你的原生iOS應用程式。或者，你也可以透過命令列執行應用程式：

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

在我們的範例應用程式中，您應該會看到「高分排行榜」的連結，點擊後將看到 React Native 元件的渲染畫面。

這是原生應用程式的主畫面：

![主畫面](/docs/assets/react-native-add-react-native-integration-example-home-screen.png)

這是 React Native 的高分排行榜畫面：

![高分排行榜](/docs/assets/react-native-add-react-native-integration-example-high-scores.png)

> 若執行應用程式時遇到模組解析問題，請參閱 [GitHub 議題](https://github.com/facebook/react-native/issues/4968) 以獲取解決方案。[此評論](https://github.com/facebook/react-native/issues/4968#issuecomment-220941717) 可能是最新的解決方法。

### 接下來？

至此，您可以繼續照常開發應用程式。請參考我們的[除錯指南](debugging)和[部署文件](running-on-device)以了解更多 React Native 開發相關資訊。

[metro]: https://metrobundler.dev/