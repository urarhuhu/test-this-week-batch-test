---
id: navigation
title: Navigating Between Screens
---

行動應用程式很少由單一畫面組成。管理多個畫面的呈現與轉場過渡，通常由所謂的「導航器」來處理。

本指南涵蓋 React Native 中各種可用的導航元件。如果你是導航功能的初學者，可能會想使用 [React Navigation](navigation.md#react-navigation)。React Navigation 提供直觀的導航解決方案，能夠在 Android 和 iOS 上呈現常見的堆疊導航與分頁導航模式。

若你正在將 React Native 整合至已使用原生方式管理導航的應用程式，或正在尋找 React Navigation 的替代方案，以下函式庫可在雙平台提供原生導航功能：[react-native-navigation](https://github.com/wix/react-native-navigation)。

## React Navigation

這個由社群提供的導航解決方案是獨立函式庫，開發者僅需幾行程式碼即可設定應用程式的畫面結構。

### 安裝與設定

首先，你需要在專案中安裝它們：

```shell
npm install @react-navigation/native @react-navigation/native-stack
```

接著安裝必要的同儕相依套件。根據你的專案是 Expo 託管專案或純 React Native 專案，需執行不同的指令。

- 若為 Expo 託管專案，請使用 `expo` 安裝相依套件：

  ```shell
  npx expo install react-native-screens react-native-safe-area-context
  ```

- 若為純 React Native 專案，請使用 `npm` 安裝相依套件：

  ```shell
  npm install react-native-screens react-native-safe-area-context
  ```

  若是純 React Native 專案的 iOS 平台，請確認已安裝 [CocoaPods](https://cocoapods.org/)。接著執行以下指令完成安裝：

  ```shell
  cd ios
  pod install
  cd ..
  ```

:::note
安裝後可能會看到與同儕相依套件相關的警告。這些警告通常是由某些套件指定的版本範圍不正確所導致。只要應用程式能正常建置，大多數警告可安全忽略。
:::

現在你需要將整個應用程式包裹在 `NavigationContainer` 中。通常會在入口檔案（如 `index.js` 或 `App.js`）進行此設定：

```tsx
import * as React from 'react';
import {NavigationContainer} from '@react-navigation/native';

const App = () => {
  return (
    <NavigationContainer>
      {/* Rest of your app code */}
    </NavigationContainer>
  );
};

export default App;
```

現在你的應用程式已準備好可在裝置/模擬器上建置並執行。

### 使用方式

現在你可以建立包含首頁畫面與個人檔案畫面的應用程式：

```tsx
import * as React from 'react';
import {NavigationContainer} from '@react-navigation/native';
import {createNativeStackNavigator} from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

const MyStack = () => {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen
          name="Home"
          component={HomeScreen}
          options={{title: 'Welcome'}}
        />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
};
```

在此範例中，我們使用 `Stack.Screen` 元件定義了兩個畫面（`Home` 和 `Profile`）。同理，你可以定義任意數量的畫面。

你可以透過 `Stack.Screen` 的 `options` 屬性為每個畫面設定選項，例如畫面標題。

每個畫面接受一個 React 元件作為 `component` 屬性。這些元件會接收到名為 `navigation` 的屬性，其中包含連結至其他畫面的各種方法。例如，你可以使用 `navigation.navigate` 跳轉至 `Profile` 畫面：

```tsx
const HomeScreen = ({navigation}) => {
  return (
    <Button
      title="Go to Jane's profile"
      onPress={() =>
        navigation.navigate('Profile', {name: 'Jane'})
      }
    />
  );
};
const ProfileScreen = ({navigation, route}) => {
  return <Text>This is {route.params.name}'s profile</Text>;
};
```

此 `native-stack` 導航器使用原生 API：iOS 上的 `UINavigationController` 與 Android 上的 `Fragment`，因此透過 `createNativeStackNavigator` 建構的導航行為將與基於這些 API 原生建置的應用程式具有相同特性與效能表現。

React Navigation 也提供不同類型的導航器套件，例如分頁與抽屜式導航。你可以使用它們在應用程式中實作各種導航模式。

完整的 React Navigation 入門指南，請參閱 [React Navigation 入門指南](https://reactnavigation.org/docs/getting-started)。