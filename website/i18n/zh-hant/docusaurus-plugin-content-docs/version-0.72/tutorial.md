---
id: tutorial
title: Learn the Basics
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 類似 React，但它使用原生元件而非網頁元件作為建構區塊。因此要理解 React Native 應用的基本結構，您需要掌握一些 React 的核心概念，如 JSX、元件、`state` 和 `props`。即使您已熟悉 React，仍需學習 React Native 特有的知識，例如原生元件。本教程適合所有讀者，無論您是否有 React 經驗。

讓我們開始吧。

## Hello World

依照程式設計的古老傳統，我們首先要建立一個只顯示「Hello, world!」的應用程式。程式碼如下：

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

若您感到好奇，可以直接在網頁模擬器中試玩這段範例程式碼。也可以將其貼到本機的 `App.js` 檔案中來建立實際應用。

## 這段程式碼做了什麼？

1. 首先需導入 `React` 才能使用 `JSX`，這些 JSX 之後會被轉換為各平台的原生元件。
2. 第二行從 `react-native` 導入 `Text` 和 `View` 元件

接著我們定義了 `HelloWorldApp` 函數，這是一個[函數式元件](https://reactjs.org/docs/components-and-props.html#function-and-class-components)，其行為與網頁版 React 完全相同。該函數返回一個帶有樣式的 `View` 元件，其中包含一個 `Text` 子元件。

`Text` 元件用於渲染文字，而 `View` 元件則渲染容器。這個容器應用了多種樣式，讓我們分析每個樣式的作用。

第一個樣式是 `flex: 1`，[`flex`](layout-props#flex) 屬性決定了子元件如何沿主軸「填滿」可用空間。由於我們只有一個容器，它會佔滿父元件的所有可用空間。在此例中，它是唯一元件，因此會佔據整個螢幕空間。

接下來的樣式是 [`justifyContent`](layout-props#justifycontent): "center"，這會將容器的子元件沿主軸居中對齊。最後是 [`alignItems`](layout-props#alignitems): "center"，使子元件沿交叉軸居中對齊。

這裡有些語法可能看起來不像 JavaScript。別擔心，_這是未來趨勢_。

首先，ES2015（又稱 ES6）是 JavaScript 的一系列改進，現已成為官方標準，但尚未被所有瀏覽器支援，因此在網頁開發中還不常見。React Native 內建支援 ES2015，所以您可以放心使用這些特性。上例中的 `import`、`export`、`const` 和 `from` 都是 ES2015 功能。如果您不熟悉 ES2015，可以透過閱讀此類教學範例來掌握。[此頁面](https://babeljs.io/learn-es2015/)詳細介紹了 ES2015 特性。

另一個特殊語法是 `<View><Text>Hello world!</Text></View>`。這是 JSX——一種在 JavaScript 中嵌入 XML 的語法。多數框架使用專屬的模板語言來在標記語言中嵌入程式碼，而 React 反其道而行。JSX 允許您在程式碼中編寫標記語言，看起來類似網頁的 HTML，但您使用的是 React 元件而非 `<div>` 或 `<span>` 等網頁元素。此處的 `<Text>` 是[核心元件](intro-react-native-components)用於顯示文字，`View` 則類似 `<div>` 或 `<span>`。

## 元件

這段程式碼定義了一個新的 `Component` 叫做 `HelloWorldApp`。當你開發 React Native 應用程式時，會經常創建新的組件。你在螢幕上看到的任何東西都是某種組件。

## Props 屬性

大多數組件在創建時可以通過不同的參數進行定制。這些創建參數被稱為 props（屬性）。

你自己的組件也可以使用 `props`。這讓你能創建一個可在應用程式多處重複使用的組件，每個地方只需稍微調整屬性即可。在函數式組件中通過 `props.YOUR_PROP_NAME` 訪問，在類組件中則通過 `this.props.YOUR_PROP_NAME` 訪問。這裡有個範例：

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

使用 `name` 作為 prop 讓我們能定制 `Greeting` 組件，這樣就能重複使用該組件來顯示不同的問候語。這個範例同時展示了如何在 JSX 中使用 `Greeting` 組件，這種能力正是 React 的強大之處。

這裡另一個新東西是 [`View`](view.md) 組件。[`View`](view.md) 作為其他組件的容器非常有用，能幫助控制樣式和佈局。

通過 `props` 和基本的 [`Text`](text.md)、[`Image`](image.md)、[`View`](view.md) 組件，你就能構建各種靜態畫面。要讓應用程式能隨時間變化，你需要[學習 State 狀態](#state)。

## State 狀態

與[只讀的 props](https://reactjs.org/docs/components-and-props.html#props-are-read-only)不同，`state` 允許 React 組件根據用戶操作、網絡響應等情況來改變其輸出。

#### React 中 state 和 props 有什麼區別？

在 React 組件中，props 是我們從父組件傳遞給子組件的變量。而 state 同樣是變量，不同之處在於它們不是作為參數傳遞的，而是由組件自身初始化和管理的。

#### React 和 React Native 在處理 state 方面有區別嗎？

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

如上所示，[React](https://reactjs.org/docs/state-and-lifecycle.html) 和 `React Native` 在處理 `state` 方面沒有區別。無論是在類組件還是函數式組件中（使用 [hooks](https://reactjs.org/docs/hooks-intro.html)），你都可以使用組件的狀態！

在接下來的範例中，我們將使用類組件來實現相同的計數器功能。

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