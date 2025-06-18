---
id: tutorial
title: Learn the Basics
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 類似於 React，但它使用原生組件而非網頁組件作為建構基礎。因此，要理解 React Native 應用的基本結構，你需要掌握一些 React 的基本概念，如 JSX、組件、`state` 和 `props`。如果你已經熟悉 React，仍需要學習一些 React Native 特有的知識，例如原生組件。本教程適合所有讀者，無論你是否具有 React 經驗。

讓我們開始吧。

## 你好，世界

依照我們祖先的傳統，我們首先需要構建一個只顯示「Hello, world!」的應用。以下是範例：

```SnackPlayer name=Hello%20World
import React from 'react';
import {Text, View} from 'react-native';

const HelloWorldApp = () => {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
      }}>
      <Text>Hello, world!</Text>
    </View>
  );
};
export default HelloWorldApp;
```

如果你感到好奇，可以直接在網頁模擬器中試玩這段範例代碼。你也可以將其貼到你的 `App.js` 文件中，在本地機器上創建一個真實的應用。

## 這裡發生了什麼？

1. 首先，我們需要導入 `React` 才能使用 `JSX`，它隨後會被轉換為各平台的原生組件。
2. 在第 2 行，我們從 `react-native` 導入 `Text` 和 `View` 組件

接著我們定義了 `HelloWorldApp` 函數，這是一個[函數式組件](https://reactjs.org/docs/components-and-props.html#function-and-class-components)，其行為與網頁版 React 中的相同。此函數返回一個帶有某些樣式的 `View` 組件，並以 `Text` 作為其子組件。

`Text` 組件讓我們可以渲染文字，而 `View` 組件則渲染一個容器。這個容器應用了多種樣式，讓我們來分析每個樣式的作用。

第一個樣式是 `flex: 1`，[`flex`](layout-props#flex) 屬性會定義你的項目如何沿主軸「填滿」可用空間。由於我們只有一個容器，它將佔據父組件的所有可用空間。在這個例子中，它是唯一的組件，因此會佔據整個屏幕的可用空間。

接下來的樣式是 [`justifyContent`](layout-props#justifycontent): "center"。這會將容器的子組件沿主軸居中對齊。最後，我們有 [`alignItems`](layout-props#alignitems): "center"，這會將容器的子組件沿交叉軸居中對齊。

這裡的一些內容可能看起來不像 JavaScript。別擔心。_這就是未來_。

首先，ES2015（也稱為 ES6）是 JavaScript 的一系列改進，現已成為官方標準的一部分，但尚未被所有瀏覽器支持，因此在網頁開發中還不常使用。React Native 自帶 ES2015 支持，因此你可以放心使用這些功能。上面範例中的 `import`、`export`、`const` 和 `from` 都是 ES2015 的特性。如果你不熟悉 ES2015，可以通過閱讀像本教程這樣的範例代碼來掌握。如果需要，[這個頁面](https://babeljs.io/learn-es2015/)提供了 ES2015 特性的詳細概述。

這段代碼中另一個不尋常的地方是 `<View><Text>Hello world!</Text></View>`。這是 JSX——一種在 JavaScript 中嵌入 XML 的語法。許多框架使用專門的模板語言，讓你在標記語言中嵌入代碼。在 React 中，這被反轉了。JSX 讓你在代碼中編寫標記語言。它看起來像網頁上的 HTML，但你不是使用 `<div>` 或 `<span>` 這樣的網頁元素，而是使用 React 組件。在這個例子中，`<Text>` 是一個[核心組件](intro-react-native-components)，用於顯示文字，而 `View` 則類似於 `<div>` 或 `<span>`。

## 組件

這段程式碼定義了一個新的「元件」`HelloWorldApp`。當你開發 React Native 應用程式時，會經常創建新元件。螢幕上看到的任何內容都是某種元件。

## 屬性(Props)

多數元件在創建時可透過不同參數進行客製化，這些創建參數稱為「props」(屬性)。

自訂元件也能使用`props`。這讓你可以創建單一元件，並在應用程式的多個地方使用，每個地方只需微調屬性即可。在函式元件中透過`props.YOUR_PROP_NAME`存取，類別元件則用`this.props.YOUR_PROP_NAME`。範例如下：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Hello%20Props&ext=js
import React from 'react';
import {Text, View, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center',
  },
});

const Greeting = props => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Hello%20Props&ext=tsx
import React from 'react';
import {Text, View, StyleSheet} from 'react-native';

const styles = StyleSheet.create({
  center: {
    alignItems: 'center',
  },
});

type GreetingProps = {
  name: string;
};

const Greeting = (props: GreetingProps) => {
  return (
    <View style={styles.center}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={[styles.center, {top: 50}]}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
</Tabs>

使用`name`作為屬性(prop)讓我們能客製化`Greeting`元件，得以重複使用該元件來顯示不同問候語。此範例也展示了如何在JSX中使用`Greeting`元件，這種靈活性正是React的強大之處。

這裡另一個新概念是[`View`](view.md)元件。[`View`](view.md)作為其他元件的容器非常實用，能協助控制樣式與佈局。

透過`props`與基礎的[`Text`](text.md)、[`Image`](image.md)和[`View`](view.md)元件，你已能建構各種靜態畫面。若要讓應用程式能隨時間變化，請繼續[學習狀態(State)](#state)。

## 狀態(State)

與[唯讀的props](https://reactjs.org/docs/components-and-props.html#props-are-read-only)不同，`state`允許React元件根據用戶操作、網路回應等情況動態改變輸出內容。

#### React中state和props有何區別？

在React元件中，props是從父元件傳遞給子元件的變數。而state同樣是變數，關鍵差異在於state不由外部傳入，而是由元件自行初始化與管理。

#### React與React Native處理state的方式有差異嗎？

<div className="two-columns">

```tsx
// ReactJS Counter Example using Hooks!

import React, {useState} from 'react';



const App = () => {
  const [count, setCount] = useState(0);

  return (
    <div className="container">
      <p>You clicked {count} times</p>
      <button
        onClick={() => setCount(count + 1)}>
        Click me!
      </button>
    </div>
  );
};


// CSS
.container {
  display: flex;
  justify-content: center;
  align-items: center;
}

```

```tsx
// React Native Counter Example using Hooks!

import React, {useState} from 'react';
import {View, Text, Button, StyleSheet} from 'react-native';

const App = () => {
  const [count, setCount] = useState(0);

  return (
    <View style={styles.container}>
      <Text>You clicked {count} times</Text>
      <Button
        onPress={() => setCount(count + 1)}
        title="Click me!"
      />
    </View>
  );
};

// React Native Styles
const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});
```

</div>

如上所示，[React](https://reactjs.org/docs/state-and-lifecycle.html)與React Native在處理`state`方面完全一致。無論是類別元件或使用[hooks](https://reactjs.org/docs/hooks-intro.html)的函式元件，都能運用元件狀態！

以下範例將用類別(class)實現相同的計數器功能：

```SnackPlayer name=Hello%20Classes
import React, {Component} from 'react';
import {StyleSheet, TouchableOpacity, Text, View} from 'react-native';

class App extends Component {
  state = {
    count: 0,
  };

  onPress = () => {
    this.setState({
      count: this.state.count + 1,
    });
  };

  render() {
    return (
      <View style={styles.container}>
        <TouchableOpacity style={styles.button} onPress={this.onPress}>
          <Text>Click me</Text>
        </TouchableOpacity>
        <View>
          <Text>You clicked {this.state.count} times</Text>
        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
    marginBottom: 10,
  },
});

export default App;
```