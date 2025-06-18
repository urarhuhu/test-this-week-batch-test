---
id: linking
title: Linking
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`Linking` 提供了一個通用介面來處理應用程式的進出連結。

每個連結(URL)都有一個URL Scheme，例如網址前綴的`https://`或`http://`中的`http`就是URL Scheme，我們簡稱為scheme。

除了`https`，您可能也熟悉`mailto` scheme。當您開啟帶有mailto scheme的連結時，作業系統會啟動已安裝的郵件應用程式。同樣地，也有用於撥打電話和發送簡訊的scheme。更多關於[內建URL](#built-in-url-schemes) scheme的說明請見下文。

如同使用mailto scheme，也可以透過自訂url scheme連結到其他應用程式。例如當您收到Slack的**Magic Link**郵件時，**Launch Slack**按鈕其實是一個錨點標籤，其href類似：`slack://secret/magic-login/other-secret`。和Slack一樣，您可以告知作業系統要處理某個自訂scheme。當Slack應用程式開啟時，會收到用來開啟它的URL。這通常被稱為深度連結(deep linking)。更多關於如何[獲取深度連結](#get-the-deep-link)到您應用程式的方法請見下文。

自訂URL scheme並非在行動裝置上開啟應用程式的唯一方式。舉例來說，若您想透過電子郵件發送一個在行動裝置上開啟的連結，使用自訂URL scheme並不理想，因為使用者可能在桌機上開啟郵件，而連結將無法運作。此時您應該使用標準的`https`連結，例如`https://www.myapp.io/records/1234546`。在行動裝置上，這些連結可設定為開啟您的應用程式。在Android上此功能稱為**深度連結**(Deep Links)，而在iOS上則稱為**通用連結**(Universal Links)。

### 內建URL Schemes

如前言所述，各平台都有一些核心功能的URL scheme。以下是非詳盡清單，但涵蓋了最常用的scheme。

| Scheme           | Description                                | iOS | Android |
| ---------------- | ------------------------------------------ | --- | ------- |
| `mailto`         | Open mail app, eg: mailto: support@expo.io | ✅  | ✅      |
| `tel`            | Open phone app, eg: tel:+123456789         | ✅  | ✅      |
| `sms`            | Open SMS app, eg: sms:+123456789           | ✅  | ✅      |
| `https` / `http` | Open web browser app, eg: https://expo.io  | ✅  | ✅      |

### 啟用深度連結

<div className="banner-native-code-required">
  <h3>Projects with Native Code Only</h3>
  <p>The following section only applies to projects with native code exposed. If you are using the managed Expo workflow, see the guide on <a href="https://docs.expo.dev/guides/linking/">Linking</a> in the Expo documentation for the appropriate alternative.</p>
</div>

若要在應用程式中啟用深度連結，請閱讀以下指南：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultPlatform} values={constants.platforms}>
<TabItem value="android">

> For instructions on how to add support for deep linking on Android, refer to [Enabling Deep Links for App Content - Add Intent Filters for Your Deep Links](https://developer.android.com/training/app-indexing/deep-linking.html#adding-filters).

If you wish to receive the intent in an existing instance of MainActivity, you may set the `launchMode` of MainActivity to `singleTask` in `AndroidManifest.xml`. See [`<activity>`](https://developer.android.com/guide/topics/manifest/activity-element.html) documentation for more information.

```xml
<activity
  android:name=".MainActivity"
  android:launchMode="singleTask">
```

</TabItem>
<TabItem value="ios">

> **NOTE:** On iOS, you'll need to add the `LinkingIOS` folder into your header search paths as described in step 3 [here](linking-libraries-ios#step-3). If you also want to listen to incoming app links during your app's execution, you'll need to add the following lines to your `*AppDelegate.m`:

<Tabs groupId="ios-language" queryString defaultValue={constants.defaultAppleLanguage} values={constants.appleLanguages}>
<TabItem value="objc">

```objc title="AppDelegate.mm"
// iOS 9.x or newer
#import <React/RCTLinkingManager.h>

- (BOOL)application:(UIApplication *)application
   openURL:(NSURL *)url
   options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
  return [RCTLinkingManager application:application openURL:url options:options];
}
```

If your app is using [Universal Links](https://developer.apple.com/ios/universal-links/), you'll need to add the following code as well:

```objc title="AppDelegate.mm"
- (BOOL)application:(UIApplication *)application continueUserActivity:(nonnull NSUserActivity *)userActivity
 restorationHandler:(nonnull void (^)(NSArray<id<UIUserActivityRestoring>> * _Nullable))restorationHandler
{
 return [RCTLinkingManager application:application
                  continueUserActivity:userActivity
                    restorationHandler:restorationHandler];
}
```

</TabItem>
<TabItem value="swift">

```swift title="AppDelegate.swift"
override func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
  return RCTLinkingManager.application(app, open: url, options: options)
}
```

If your app is using [Universal Links](https://developer.apple.com/ios/universal-links/), you'll need to add the following code as well:

```swift title="AppDelegate.swift"
override func application(
  _ application: UIApplication,
  continue userActivity: NSUserActivity,
  restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    return RCTLinkingManager.application(
      application,
      continue: userActivity,
      restorationHandler: restorationHandler
    )
  }
```

</TabItem>
</Tabs>

</TabItem>
</Tabs>

### 處理深度連結

有兩種方式可以處理開啟您應用程式的URL。

#### 1. 若應用程式已開啟，則會將應用程式帶到前景並觸發Linking的'url'事件

您可以使用`Linking.addEventListener('url', callback)`處理這些事件 - 它會呼叫`callback({url})`並傳入連結的URL

#### 2. 若應用程式尚未開啟，則會開啟應用程式並將url作為initialURL傳入

您可以使用`Linking.getInitialURL()`處理這些事件 - 它會回傳一個解析為URL的Promise（如果有的話）。

---

## 範例

### 開啟連結與深度連結（通用連結）

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const supportedURL = 'https://google.com';

const unsupportedURL = 'slack://open?team=123456';

const OpenURLButton = ({url, children}) => {
  const handlePress = useCallback(async () => {
    // Checking if the link is supported for links with custom URL scheme.
    const supported = await Linking.canOpenURL(url);

    if (supported) {
      // Opening the link with some app, if the URL scheme is "http" the web link should be opened
      // by some browser in the mobile
      await Linking.openURL(url);
    } else {
      Alert.alert(`Don't know how to open this URL: ${url}`);
    }
  }, [url]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenURLButton url={supportedURL}>Open Supported URL</OpenURLButton>
      <OpenURLButton url={unsupportedURL}>Open Unsupported URL</OpenURLButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const supportedURL = 'https://google.com';

const unsupportedURL = 'slack://open?team=123456';

type OpenURLButtonProps = {
  url: string;
  children: string;
};

const OpenURLButton = ({url, children}: OpenURLButtonProps) => {
  const handlePress = useCallback(async () => {
    // Checking if the link is supported for links with custom URL scheme.
    const supported = await Linking.canOpenURL(url);

    if (supported) {
      // Opening the link with some app, if the URL scheme is "http" the web link should be opened
      // by some browser in the mobile
      await Linking.openURL(url);
    } else {
      Alert.alert(`Don't know how to open this URL: ${url}`);
    }
  }, [url]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenURLButton url={supportedURL}>Open Supported URL</OpenURLButton>
      <OpenURLButton url={unsupportedURL}>Open Unsupported URL</OpenURLButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 開啟自訂設定

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useCallback} from 'react';
import {Button, Linking, StyleSheet, View} from 'react-native';

const OpenSettingsButton = ({children}) => {
  const handlePress = useCallback(async () => {
    // Open the custom settings if the app has one
    await Linking.openSettings();
  }, []);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenSettingsButton>Open Settings</OpenSettingsButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useCallback} from 'react';
import {Button, Linking, StyleSheet, View} from 'react-native';

type OpenSettingsButtonProps = {
  children: string;
};

const OpenSettingsButton = ({children}: OpenSettingsButtonProps) => {
  const handlePress = useCallback(async () => {
    // Open the custom settings if the app has one
    await Linking.openSettings();
  }, []);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <OpenSettingsButton>Open Settings</OpenSettingsButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 獲取深度連結

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=js
import React, {useState, useEffect} from 'react';
import {Linking, StyleSheet, Text, View} from 'react-native';

const useInitialURL = () => {
  const [url, setUrl] = useState(null);
  const [processing, setProcessing] = useState(true);

  useEffect(() => {
    const getUrlAsync = async () => {
      // Get the deep link used to open the app
      const initialUrl = await Linking.getInitialURL();

      // The setTimeout is just for testing purpose
      setTimeout(() => {
        setUrl(initialUrl);
        setProcessing(false);
      }, 1000);
    };

    getUrlAsync();
  }, []);

  return {url, processing};
};

const App = () => {
  const {url: initialUrl, processing} = useInitialURL();

  return (
    <View style={styles.container}>
      <Text>
        {processing
          ? 'Processing the initial url from a deep link'
          : `The deep link is: ${initialUrl || 'None'}`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=ios,android&ext=tsx
import React, {useState, useEffect} from 'react';
import {Linking, StyleSheet, Text, View} from 'react-native';

const useInitialURL = () => {
  const [url, setUrl] = useState<string | null>(null);
  const [processing, setProcessing] = useState(true);

  useEffect(() => {
    const getUrlAsync = async () => {
      // Get the deep link used to open the app
      const initialUrl = await Linking.getInitialURL();

      // The setTimeout is just for testing purpose
      setTimeout(() => {
        setUrl(initialUrl);
        setProcessing(false);
      }, 1000);
    };

    getUrlAsync();
  }, []);

  return {url, processing};
};

const App = () => {
  const {url: initialUrl, processing} = useInitialURL();

  return (
    <View style={styles.container}>
      <Text>
        {processing
          ? 'Processing the initial url from a deep link'
          : `The deep link is: ${initialUrl || 'None'}`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

### 發送Intent（Android）

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Linking%20Example&supportedPlatforms=android&ext=js
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

const SendIntentButton = ({action, extras, children}) => {
  const handlePress = useCallback(async () => {
    try {
      await Linking.sendIntent(action, extras);
    } catch (e) {
      Alert.alert(e.message);
    }
  }, [action, extras]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <SendIntentButton action="android.intent.action.POWER_USAGE_SUMMARY">
        Power Usage Summary
      </SendIntentButton>
      <SendIntentButton
        action="android.settings.APP_NOTIFICATION_SETTINGS"
        extras={[
          {
            key: 'android.provider.extra.APP_PACKAGE',
            value: 'com.facebook.katana',
          },
        ]}>
        App Notification Settings
      </SendIntentButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Linking%20Example&ext=tsx
import React, {useCallback} from 'react';
import {Alert, Button, Linking, StyleSheet, View} from 'react-native';

type SendIntentButtonProps = {
  action: string;
  children: string;
  extras?: Array<{
    key: string;
    value: string | number | boolean;
  }>;
};

const SendIntentButton = ({
  action,
  extras,
  children,
}: SendIntentButtonProps) => {
  const handlePress = useCallback(async () => {
    try {
      await Linking.sendIntent(action, extras);
    } catch (e: any) {
      Alert.alert(e.message);
    }
  }, [action, extras]);

  return <Button title={children} onPress={handlePress} />;
};

const App = () => {
  return (
    <View style={styles.container}>
      <SendIntentButton action="android.intent.action.POWER_USAGE_SUMMARY">
        Power Usage Summary
      </SendIntentButton>
      <SendIntentButton
        action="android.settings.APP_NOTIFICATION_SETTINGS"
        extras={[
          {
            key: 'android.provider.extra.APP_PACKAGE',
            value: 'com.facebook.katana',
          },
        ]}>
        App Notification Settings
      </SendIntentButton>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default App;
```

</TabItem>
</Tabs>

# 參考資料

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: 'url',
  handler: (event: {url: string}) => void,
): EmitterSubscription;
```

透過監聽`url`事件類型並提供處理程序，來為Linking變更添加處理程序。

---

### `canOpenURL()`

```tsx
static canOpenURL(url: string): Promise<boolean>;
```

判斷已安裝的應用程式是否能處理指定的URL。

此方法會回傳一個`Promise`物件。當系統判定該URL能否被處理時，Promise會被解析，且第一個參數會表示該URL能否被打開。

在Android上，若無法檢查URL是否能被打開，或是當目標為Android 11 (SDK 30)時未在`AndroidManifest.xml`中指定相關的intent查詢，Promise會被拒絕。同樣地，在iOS上，若未在`Info.plist`中的`LSApplicationQueriesSchemes`鍵內添加特定scheme（見下文），Promise也會被拒絕。

**參數：**

| Name                                                     | Type   | Description      |
| -------------------------------------------------------- | ------ | ---------------- |
| url <div className="label basic required">Required</div> | string | The URL to open. |

> 對於網頁URL，必須正確設定協定（`"http://"`、`"https://"`）！

> 此方法在iOS 9+上有其限制。根據[Apple官方文件](https://developer.apple.com/documentation/uikit/uiapplication/1622952-canopenurl)：
>
> - 若你的應用程式連結至較早版本的iOS，但在iOS 9.0或更新版本上運行，此方法最多可呼叫50次。達到此限制後，後續呼叫將一律解析為`false`。若使用者重新安裝或升級應用程式，iOS會重置此限制。
>
> 自iOS 9起，你的應用程式還需在`Info.plist`中提供`LSApplicationQueriesSchemes`鍵，否則`canOpenURL()`將一律解析為`false`。

> 當目標為Android 11 (SDK 30)時，你必須在`AndroidManifest.xml`中指定你想處理的scheme對應的intent。常見的intent列表可參考[此處](https://developer.android.com/guide/components/intents-common)。
>
> 例如，要處理`https` scheme，需在manifest中添加以下內容：
>
> ```
> <manifest ...>
>     <queries>
>         <intent>
>             <action android:name="android.intent.action.VIEW" />
>             <data android:scheme="https"/>
>         </intent>
>     </queries>
> </manifest>
> ```

---

### `getInitialURL()`

```tsx
static getInitialURL(): Promise<string | null>;
```

若應用程式的啟動是由應用程式連結觸發，則會回傳該連結的URL，否則回傳`null`。

> 要在Android上支援深層連結，請參考 https://developer.android.com/training/app-indexing/deep-linking.html#handling-intents

> 當遠端JS除錯功能啟用時，getInitialURL可能會回傳`null`。請停用除錯器以確保能正確傳遞。

---

### `openSettings()`

```tsx
static openSettings(): Promise<void>;
```

開啟設定應用程式並顯示該應用程式的自訂設定（若有）。

---

### `openURL()`

```tsx
static openURL(url: string): Promise<any>;
```

嘗試使用任何已安裝的應用程式開啟指定的`url`。

你也可以使用其他URL，例如位置（在Android上使用`"geo:37.484847,-122.148386"`或在iOS上使用`"https://maps.apple.com/?ll=37.484847,-122.148386"`）、聯絡人，或任何可透過已安裝應用程式開啟的URL。

此方法會回傳一個`Promise`物件。若使用者確認開啟對話框或URL自動開啟，Promise會被解析。若使用者取消開啟對話框或沒有已註冊的應用程式可處理該URL，Promise會被拒絕。

**參數：**

| Name                                                     | Type   | Description      |
| -------------------------------------------------------- | ------ | ---------------- |
| url <div className="label basic required">Required</div> | string | The URL to open. |

> 若系統不知道如何開啟指定的URL，此方法將會失敗。若你傳入非http(s)的URL，最好先檢查`canOpenURL()`。

> 對於網頁URL，必須正確設定協定（`"http://"`、`"https://"`）！

> 此方法在模擬器中的行為可能有所不同，例如 iOS 模擬器無法處理 `"tel:"` 連結，因為無法存取撥號應用程式。

---

### `sendIntent()` <div class="label android">Android</div>

```tsx
static sendIntent(
  action: string,
  extras?: Array<{key: string; value: string | number | boolean}>,
): Promise<void>;
```

啟動一個帶有附加參數的 Android intent。

**參數：**

| Name                                                        | Type                                                       |
| ----------------------------------------------------------- | ---------------------------------------------------------- |
| action <div className="label basic required">Required</div> | string                                                     |
| extras                                                      | `Array<{key: string, value: string ｜ number ｜ boolean}>` |