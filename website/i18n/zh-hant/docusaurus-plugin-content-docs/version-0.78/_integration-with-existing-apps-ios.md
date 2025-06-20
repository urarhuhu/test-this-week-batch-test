import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

## Key Concepts

The keys to integrating React Native components into your iOS application are to:

1. Set up the correct directory structure.
2. Install the necessary NPM dependencies.
3. Adding React Native to your Podfile configuration.
4. Writing the TypeScript code for your first React Native screen.
5. Integrate React Native with your iOS code using a `RCTRootView`.
6. Testing your integration by running the bundler and seeing your app in action.

## Using the Community Template

While you follow this guide, we suggest you to use the [React Native Community Template](https://github.com/react-native-community/template/) as reference. The template contains a **minimal iOS app** and will help you understanding how to integrate React Native into an existing iOS app.

## Prerequisites

Follow the guide on [setting up your development environment](set-up-your-environment) and using [React Native without a framework](getting-started-without-a-framework) to configure your development environment for building React Native apps for iOS.
This guide also assumes you're familiar with the basics of iOS development such as creating a `UIViewController` and editing the `Podfile` file.

### 1. Set up directory structure

To ensure a smooth experience, create a new folder for your integrated React Native project, then **move your existing iOS project** to the `/ios` subfolder.

## 2. Install NPM dependencies

Go to the root directory and run the following command:

```shell
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.78-stable/template/package.json
```

This will copy the `package.json` [file from the Community template](https://github.com/react-native-community/template/blob/0.78-stable/template/package.json) to your project.

Next, install the NPM packages by running:

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

Installation process has created a new `node_modules` folder. This folder stores all the JavaScript dependencies required to build your project.

Add `node_modules/` to your `.gitignore` file (here the [Community default one](https://github.com/react-native-community/template/blob/0.78-stable/template/_gitignore)).

### 3. Install Development tools

### Command Line Tools for Xcode

Install the Command Line Tools. Choose **Settings... (or Preferences...)** in the Xcode menu. Go to the Locations panel and install the tools by selecting the most recent version in the Command Line Tools dropdown.

![Xcode Command Line Tools](/docs/assets/GettingStartedXcodeCommandLineTools.png)

### CocoaPods

[CocoaPods](https://cocoapods.org) is a package management tool for iOS and macOS development. We use it to add the actual React Native framework code locally into your current project.

We recommend installing CocoaPods using [Homebrew](https://brew.sh/):

```shell
brew install cocoapods
```

## 4. Adding React Native to your app

### Configuring CocoaPods

To configure CocoaPods, we need two files:

- A **Gemfile** that defines which Ruby dependencies we need.
- A **Podfile** that defines how to properly install our dependencies.

For the **Gemfile**, go to the root directory of your project and run this command

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.78-stable/template/Gemfile
```

This will download the Gemfile from the template.

:::note
If you created your project with Xcode 16, you need to update the Gemfile as it follows:

```diff
-gem 'cocoapods', '>= 1.13', '!= 1.15.0', '!= 1.15.1'
+gem 'cocoapods', '1.16.2'
gem 'activesupport', '>= 6.1.7.5', '!= 7.1.0'
-gem 'xcodeproj', '< 1.26.0'
+gem 'xcodeproj', '1.27.0'
```

Xcode 16 generates a project in a slightly different ways from previous versions of Xcode, and you need the latest CocoaPods and Xcodeproj gems to make it work properly.
:::

Similarly, for the **Podfile**, go to the `ios` folder of your project and run

```sh
curl -O https://raw.githubusercontent.com/react-native-community/template/refs/heads/0.78-stable/template/ios/Podfile
```

Please use the Community Template as a reference point for the [Gemfile](https://github.com/react-native-community/template/blob/0.78-stable/template/Gemfile) and for the [Podfile](https://github.com/react-native-community/template/blob/0.78-stable/template/ios/Podfile).

:::note
Remember to change [this line](https://github.com/react-native-community/template/blob/0.78-stable/template/ios/Podfile#L17).
:::

Now, we need to run a couple of extra commands to install the Ruby gems and the Pods.
Navigate to the `ios` folder and run the following commands:

```sh
bundle install
bundle exec pod install
```

The first command will install the Ruby dependencies and the second command will actually integrate the React Native code in your application so that your iOS files can import the React Native headers.

## 5. Writing the TypeScript Code

Now we will actually modify the native iOS application to integrate React Native.

The first bit of code we will write is the actual React Native code for the new screen that will be integrated into our application.

### Create a `index.js` file

First, create an empty `index.js` file in the root of your React Native project.

`index.js` is the starting point for React Native applications, and it is always required. It can be a small file that `import`s other file that are part of your React Native component or application, or it can contain all the code that is needed for it.

Our `index.js` should look as follows (here the [Community template file as reference](https://github.com/react-native-community/template/blob/0.78-stable/template/index.js)):

```js
import {AppRegistry} from 'react-native';
import App from './App';

AppRegistry.registerComponent('HelloWorld', () => App);
```

### Create a `App.tsx` file

Let's create an `App.tsx` file. This is a [TypeScript](https://www.typescriptlang.org/) file that can have [JSX](<https://en.wikipedia.org/wiki/JSX_(JavaScript)>) expressions. It contains the root React Native component that we will integrate into our iOS application ([link](https://github.com/react-native-community/template/blob/0.78-stable/template/App.tsx)):

```tsx
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  StatusBar,
  StyleSheet,
  Text,
  useColorScheme,
  View,
} from 'react-native';

import {
  Colors,
  DebugInstructions,
  Header,
  ReloadInstructions,
} from 'react-native/Libraries/NewAppScreen';

function App(): React.JSX.Element {
  const isDarkMode = useColorScheme() === 'dark';

  const backgroundStyle = {
    backgroundColor: isDarkMode ? Colors.darker : Colors.lighter,
  };

  return (
    <SafeAreaView style={backgroundStyle}>
      <StatusBar
        barStyle={isDarkMode ? 'light-content' : 'dark-content'}
        backgroundColor={backgroundStyle.backgroundColor}
      />
      <ScrollView
        contentInsetAdjustmentBehavior="automatic"
        style={backgroundStyle}>
        <Header />
        <View
          style={{
            backgroundColor: isDarkMode
              ? Colors.black
              : Colors.white,
            padding: 24,
          }}>
          <Text style={styles.title}>Step One</Text>
          <Text>
            Edit <Text style={styles.bold}>App.tsx</Text> to
            change this screen and see your edits.
          </Text>
          <Text style={styles.title}>See your changes</Text>
          <ReloadInstructions />
          <Text style={styles.title}>Debug</Text>
          <DebugInstructions />
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  title: {
    fontSize: 24,
    fontWeight: '600',
  },
  bold: {
    fontWeight: '700',
  },
});

export default App;
```

Here the [Community template file as reference](https://github.com/react-native-community/template/blob/0.78-stable/template/App.tsx)

## 5. Integrating with your iOS code

We now need to add some native code in order to start the React Native runtime and tell it to render our React components.

### Requirements

React Native intialization is now unbound to any specific part of an iOS app.

React Native can be initialized using a class called `RCTReactNativeFactory`, that takes care of handling the React Native lifecycle for you.

Once the class is initialized, you can either start a React Native view providing a `UIWindow` object, or you can ask for the factory to generate a `UIView` that you can load in any `UIViewController.`

In the following example, we will create a ViewController that can load a React Native view as it's `view`.

#### Create the ReactViewController

Create a new file from template (<kbd>⌘</kbd>+<kbd>N</kbd>) and choose the Cocoa Touch Class template.

Make sure to select `UIViewController` as the "Subclass of" field.

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

Now open the `ReactViewController.m` file and apply the following changes

```diff title="ReactViewController.m"
#import "ReactViewController.h"
+#import <React/RCTBundleURLProvider.h>
+#import <RCTReactNativeFactory.h>
+#import <RCTDefaultReactNativeFactoryDelegate.h>
+#import <RCTAppDependencyProvider.h>

@interface ReactViewController ()

@end

+@interface ReactNativeFactoryDelegate: RCTDefaultReactNativeFactoryDelegate
+@end

-@implementation ReactViewController
+@implementation ReactViewController {
+  RCTReactNativeFactory *_factory;
+  id<RCTReactNativeFactoryDelegate> _factoryDelegate;
+}

 - (void)viewDidLoad {
     [super viewDidLoad];
     // Do any additional setup after loading the view.
+    _factoryDelegate = [ReactNativeFactoryDelegate new];
+    _factoryDelegate.dependencyProvider = [RCTAppDependencyProvider new];
+    _factory = [[RCTReactNativeFactory alloc] initWithDelegate:_factoryDelegate];
+    self.view = [_factory.rootViewFactory viewWithModuleName:@"HelloWorld"];
 }

@end

+@implementation ReactNativeFactoryDelegate
+
+- (NSURL *)sourceURLForBridge:(RCTBridge *)bridge
+{
+  return [self bundleURL];
+}
+
+- (NSURL *)bundleURL
+{
+#if DEBUG
+  return [RCTBundleURLProvider.sharedSettings jsBundleURLForBundleRoot:@"index"];
+#else
+  return [NSBundle.mainBundle URLForResource:@"main" withExtension:@"jsbundle"];
+#endif
+}

@end

```

</TabItem>
<TabItem value="swift">

Now open the `ReactViewController.swift` file and apply the following changes

```diff title="ReactViewController.swift"
import UIKit
+import React
+import React_RCTAppDelegate

class ReactViewController: UIViewController {
+  var reactNativeFactory: RCTReactNativeFactory?
+  var reactNativeFactoryDelegate: RCTReactNativeFactoryDelegate?

  override func viewDidLoad() {
    super.viewDidLoad()
+    reactNativeFactoryDelegate = ReactNativeDelegate()
+    reactNativeFactory = RCTReactNativeFactory(delegate: reactNativeFactoryDelegate!)
+    view = reactNativeFactory!.rootViewFactory.view(withModuleName: "HelloWorld")

  }
}

+class ReactNativeDelegate: RCTDefaultReactNativeFactoryDelegate {
+    override func sourceURL(for bridge: RCTBridge) -> URL? {
+      self.bundleURL()
+    }
+
+    override func bundleURL() -> URL? {
+      #if DEBUG
+      RCTBundleURLProvider.sharedSettings().jsBundleURL(forBundleRoot: "index")
+      #else
+      Bundle.main.url(forResource: "main", withExtension: "jsbundle")
+      #endif
+    }
+
+}
```

</TabItem>
</Tabs>

#### Presenting a React Native view in a rootViewController

Finally, we can present our React Native view. To do so, we need a new View Controller that can host a view in which we can load the JS content.
We already have the initial `ViewController`, and we can make it present the `ReactViewController`. There are several ways to do so, depending on your app. For this example, we assume that you have a button that presents React Native modally.

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

```diff title="ViewController.m"
#import "ViewController.h"
+#import "ReactViewController.h"

@interface ViewController ()

@end

- @implementation ViewController
+@implementation ViewController {
+  ReactViewController *reactViewController;
+}

 - (void)viewDidLoad {
   [super viewDidLoad];
   // Do any additional setup after loading the view.
   self.view.backgroundColor = UIColor.systemBackgroundColor;
+  UIButton *button = [UIButton new];
+  [button setTitle:@"Open React Native" forState:UIControlStateNormal];
+  [button setTitleColor:UIColor.systemBlueColor forState:UIControlStateNormal];
+  [button setTitleColor:UIColor.blueColor forState:UIControlStateHighlighted];
+  [button addTarget:self action:@selector(presentReactNative) forControlEvents:UIControlEventTouchUpInside];
+  [self.view addSubview:button];

+  button.translatesAutoresizingMaskIntoConstraints = NO;
+  [NSLayoutConstraint activateConstraints:@[
+    [button.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor],
+    [button.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor],
+    [button.centerYAnchor constraintEqualToAnchor:self.view.centerYAnchor],
+    [button.centerXAnchor constraintEqualToAnchor:self.view.centerXAnchor],
+  ]];
 }

+- (void)presentReactNative
+{
+  if (reactViewController == NULL) {
+    reactViewController = [ReactViewController new];
+  }
+  [self presentViewController:reactViewController animated:YES];
+}

@end
```

</TabItem>
<TabItem value="swift">

```diff title="ViewController.swift"
import UIKit

class ViewController: UIViewController {

+  var reactViewController: ReactViewController?

  override func viewDidLoad() {
    super.viewDidLoad()
    // Do any additional setup after loading the view.
    self.view.backgroundColor = .systemBackground

+    let button = UIButton()
+    button.setTitle("Open React Native", for: .normal)
+    button.setTitleColor(.systemBlue, for: .normal)
+    button.setTitleColor(.blue, for: .highlighted)
+    button.addAction(UIAction { [weak self] _ in
+      guard let self else { return }
+      if reactViewController == nil {
+       reactViewController = ReactViewController()
+      }
+      present(reactViewController!, animated: true)
+    }, for: .touchUpInside)
+    self.view.addSubview(button)
+
+    button.translatesAutoresizingMaskIntoConstraints = false
+    NSLayoutConstraint.activate([
+      button.leadingAnchor.constraint(equalTo: self.view.leadingAnchor),
+      button.trailingAnchor.constraint(equalTo: self.view.trailingAnchor),
+      button.centerXAnchor.constraint(equalTo: self.view.centerXAnchor),
+      button.centerYAnchor.constraint(equalTo: self.view.centerYAnchor),
+    ])
  }
}
```

</TabItem>
</Tabs>

Make sure to disable the Sandbox scripting. To achieve this, in Xcode, click on your app, then on build settings. Filter for script and set the `User Script Sandboxing` to `NO`. This step is needed to properly switch between the Debug and Release version of the [Hermes engine](https://github.com/facebook/hermes/blob/main/README.md) that we ship with React Native.

![Disable Sandboxing](/docs/assets/disable-sandboxing.png)

Finally, make sure to add the `UIViewControllerBasedStatusBarAppearance` key into your `Info.plist` file, with value of `NO`.

![Disable UIViewControllerBasedStatusBarAppearance](/docs/assets/disable-UIViewControllerBasedStatusBarAppearance.png)

## 6. Test your integration

You have completed all the basic steps to integrate React Native with your application. Now we will start the [Metro bundler](https://metrobundler.dev/) to build your TypeScript application code into a bundle. Metro's HTTP server shares the bundle from `localhost` on your developer environment to a simulator or device. This allows for [hot reloading](https://reactnative.dev/blog/2016/03/24/introducing-hot-reloading).

First, you need to create a `metro.config.js` file in the root of your project as follows:

```js
const {getDefaultConfig} = require('@react-native/metro-config');
module.exports = getDefaultConfig(__dirname);
```

You can checkout the [metro.config.js file](https://github.com/react-native-community/template/blob/0.78-stable/template/metro.config.js) from the Community template file as reference.

Then, you need to create a `.watchmanconfig` file in the root of your project. The file must contain an empty json object:

```sh
echo {} > .watchmanconfig
```

Once you have the configurations file in place, you can run the bundler. Run the following command in the root directory of your project:

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

Now build and run your iOS app as normal.

Once you reach your React-powered Activity inside the app, it should load the JavaScript code from the development server and display:

<center><img src="/docs/assets/EmbeddedAppIOS078.gif" width="300" /></center>

### Creating a release build in Xcode

You can use Xcode to create your release builds too! The only additional step is to add a script that is executed when the app is built to package your JS and images into the iOS application.

1. In Xcode, select your application
2. Click on `Build Phases`
3. Click on the `+` in the top left corner and select `New Run Script Phase`
4. Click on the `Run Script` line and rename the Script to `Bundle React Native code and images`
5. Paste in the text box the following script

```sh title="Build React Native code and image"
set -e

WITH_ENVIRONMENT="$REACT_NATIVE_PATH/scripts/xcode/with-environment.sh"
REACT_NATIVE_XCODE="$REACT_NATIVE_PATH/scripts/react-native-xcode.sh"

/bin/sh -c "$WITH_ENVIRONMENT $REACT_NATIVE_XCODE"
```

6. Drag and drop the script before the one called `[CP] Embed Pods Frameworks`.

Now, if you build your app for Release, it will work as expected.

### Now what?

At this point you can continue developing your app as usual. Refer to our [debugging](debugging) and [deployment](running-on-device) docs to learn more about working with React Native.