---
id: navigation
title: Navigating Between Screens
---

行動應用程式很少由單一畫面組成。管理多個畫面的呈現與轉場通常由所謂的「導航器」處理。

本指南涵蓋 React Native 中提供的各種導航元件。若您剛開始接觸導航功能，可能會想使用 [React Navigation](navigation.md#react-navigation)。React Navigation 提供了直觀的導航解決方案，能在 Android 和 iOS 上實現常見的堆疊導航與分頁導航模式。

若您要將 React Native 整合至已具備原生導航功能的應用程式，或正在尋找 React Navigation 的替代方案，以下函式庫可在雙平台提供原生導航功能：[react-native-navigation](https://github.com/wix/react-native-navigation)。

## React Navigation

此社群導航解決方案是獨立函式庫，開發者僅需幾行程式碼即可設定應用程式的畫面結構。

### 安裝與設定

首先，您需要在專案中安裝它們：

```shell
npm install @react-navigation/native @react-navigation/native-stack
```

接著安裝必要的 peer dependencies。根據您的專案類型（Expo 託管專案或裸 React Native 專案），需執行不同指令：

- 若為 Expo 託管專案，請使用 `expo` 安裝依賴項：

  ```shell
  expo install react-native-screens react-native-safe-area-context
  ```

- 若為裸 React Native 專案，請使用 `npm` 安裝依賴項：

  ```shell
  npm install react-native-screens react-native-safe-area-context
  ```

  針對 iOS 裸專案，請確保已安裝 [CocoaPods](https://cocoapods.org/)。接著執行 pod install 完成安裝：

  ```shell
  cd ios
  pod install
  cd ..
  ```

:::note
安裝後可能會出現 peer dependencies 相關警告。這些通常是由某些套件指定的版本範圍不正確所導致。只要應用程式能正常建置，大多數警告可安全忽略。
:::

現在您需要將整個應用程式包裹在 `NavigationContainer` 中。通常會在入口檔案（如 `index.js` 或 `App.js`）進行此設定：

```jsx
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

現在您已準備好在裝置/模擬器上建置並執行應用程式。

### 使用方式

現在您可以建立包含首頁畫面與個人檔案畫面的應用程式：

```jsx
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

此範例中，我們使用 `Stack.Screen` 元件定義了兩個畫面（`Home` 和 `Profile`）。您同樣可以定義任意數量的畫面。

您可透過 `Stack.Screen` 的 `options` 屬性為每個畫面設定選項（例如畫面標題）。

每個畫面接受一個 React 元件作為 `component` 屬性。這些元件會收到名為 `navigation` 的 prop，其中包含連結至其他畫面的各種方法。例如，您可使用 `navigation.navigate` 跳轉至 `Profile` 畫面：

```jsx
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

此 `native-stack` 導航器使用原生 API：iOS 的 `UINavigationController` 與 Android 的 `Fragment`，因此透過 `createNativeStackNavigator` 建構的導航行為將與基於這些 API 的原生應用程式具有相同特性與效能表現。

React Navigation 還提供不同類型的導航器套件（如分頁與抽屜式導航），可用於實作應用程式中的各種導航模式。

完整入門教學請參閱 [React Navigation 入門指南](https://reactnavigation.org/docs/getting-started)。