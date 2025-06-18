---
id: intro-react
title: React Fundamentals
description: To understand React Native fully, you need a solid foundation in React. This short introduction to React can help you get started or get refreshed.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

React Native 基於 [React](https://reactjs.org/) 運行，這是一個用 JavaScript 構建用戶界面的熱門開源庫。要充分發揮 React Native 的優勢，理解 React 本身至關重要。本節可作為入門指南或複習課程。

我們將涵蓋 React 的核心概念：

- 組件 (components)
- JSX
- 屬性 (props)
- 狀態 (state)

若需深入學習，建議查閱 [React 官方文檔](https://reactjs.org/docs/getting-started.html)。

## 你的第一個組件

本篇 React 入門將以貓咪為例：這些友好親人的生物需要名字和工作的咖啡館。以下是你的第一個 Cat 組件：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Your%20Cat
import React from 'react';
import { Text } from 'react-native';

const Cat = () => {
  return (
    <Text>Hello, I am your cat!</Text>
  );
}

export default Cat;
```

Here is how you do it: To define your `Cat` component, first use JavaScript’s [`import`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) to import React and React Native’s [`Text`](/docs/next/text) Core Component:

```jsx
import React from 'react';
import {Text} from 'react-native';
```

Your component starts as a function:

```jsx
const Cat = () => {};
```

You can think of components as blueprints. Whatever a function component returns is rendered as a **React element.** React elements let you describe what you want to see on the screen.

Here the `Cat` component will render a `<Text>` element:

```jsx
const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};
```

You can export your function component with JavaScript’s [`export default`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) for use throughout your app like so:

```jsx
const Cat = () => {
  return <Text>Hello, I am your cat!</Text>;
};

export default Cat;
```

</TabItem>
<TabItem value="classical">

Class components tend to be a bit more verbose than function components.

```SnackPlayer name=Your%20Cat
import React, { Component } from 'react';
import { Text } from 'react-native';

class Cat extends Component {
  render() {
    return (
      <Text>Hello, I am your cat!</Text>
    );
  }
}

export default Cat;
```

You additionally import `Component` from React:

```jsx
import React, {Component} from 'react';
```

Your component starts as a class extending `Component` instead of as a function:

```jsx
class Cat extends Component {}
```

Class components have a `render()` function. Whatever is returned inside it is rendered as a React element:

```jsx
class Cat extends Component {
  render() {
    return <Text>Hello, I am your cat!</Text>;
  }
}
```

And as with function components, you can export your class component:

```jsx
class Cat extends Component {
  render() {
    return <Text>Hello, I am your cat!</Text>;
  }
}

export default Cat;
```

</TabItem>
</Tabs>

> 這是導出組件的多種方式之一。此類導出方式與 Snack Player 相容性良好。但根據應用文件結構，你可能需要採用其他慣例。這份 [JavaScript 導入導出速查表](https://medium.com/dailyjs/javascript-module-cheatsheet-7bd474f1d829) 可提供協助。

現在仔細觀察 `return` 語句。`<Text>Hello, I am your cat!</Text>` 使用了一種讓元素編寫更便捷的 JavaScript 語法：JSX。

## JSX

React 與 React Native 使用 **JSX**，這種語法允許你在 JavaScript 中直接編寫元素：`<Text>Hello, I am your cat!</Text>`。React 文檔提供 [詳盡的 JSX 指南](https://reactjs.org/docs/jsx-in-depth.html) 可供深入學習。由於 JSX 本質是 JavaScript，你可以在其中使用變量。此處我們宣告貓咪的名字 `name`，並用大括號將其嵌入 `<Text>`。

```SnackPlayer name=Curly%20Braces
import React from 'react';
import { Text } from 'react-native';

const Cat = () => {
  const name = "Maru";
  return (
    <Text>Hello, I am {name}!</Text>
  );
}

export default Cat;
```

任何 JavaScript 表達式都可置於大括號內，包括函數調用如 `{getFullName("Rum", "Tum", "Tugger")}`：

```SnackPlayer name=Curly%20Braces
import React from 'react';
import { Text } from 'react-native';

const getFullName = (firstName, secondName, thirdName) => {
  return firstName + " " + secondName + " " + thirdName;
}

const Cat = () => {
  return (
    <Text>
      Hello, I am {getFullName("Rum", "Tum", "Tugger")}!
    </Text>
  );
}

export default Cat;
```

你可以將大括號視為通往 JSX 中 JavaScript 功能的傳送門！

> 由於 JSX 包含在 React 庫中，若文件頂部沒有 `import React from 'react'`，它將無法工作！

## 自定義組件

你已認識 [React Native 核心組件](intro-react-native-components)。React 允許將這些組件相互嵌套以創建新組件，這種可嵌套、可複用的特性正是 React 範式的核心。

例如，你可以將 [`Text`](text) 和 [`TextInput`](textinput) 嵌套在 [`View`](view) 中，React Native 會將它們一併渲染：

```SnackPlayer name=Custom%20Components
import React from 'react';
import { Text, TextInput, View } from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>Hello, I am...</Text>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1
        }}
        defaultValue="Name me!"
      />
    </View>
  );
}

export default Cat;
```

#### 開發者須知

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["android", "web"])}>

<TabItem value="web">

> If you’re familiar with web development, `<View>` and `<Text>` might remind you of HTML! You can think of them as the `<div>` and `<p>` tags of application development.

</TabItem>
<TabItem value="android">

> On Android, you usually put your views inside `LinearLayout`, `FrameLayout`, `RelativeLayout`, etc. to define how the view’s children will be arranged on the screen. In React Native, `View` uses Flexbox for its children’s layout. You can learn more in [our guide to layout with Flexbox](flexbox).

</TabItem>
</Tabs>

通過使用 `<Cat>`，你可以在多處重複渲染此組件而無需重複代碼：

```SnackPlayer name=Multiple%20Components
import React from 'react';
import { Text, View } from 'react-native';

const Cat = () => {
  return (
    <View>
      <Text>I am also a cat!</Text>
    </View>
  );
}

const Cafe = () => {
  return (
    <View>
      <Text>Welcome!</Text>
      <Cat />
      <Cat />
      <Cat />
    </View>
  );
}

export default Cafe;
```

任何渲染其他組件的組件稱為 **父組件**。此處 `Cafe` 是父組件，每個 `Cat` 則是 **子組件**。

你可以在咖啡館放置任意數量的貓咪。每個 `<Cat>` 都會渲染獨立的元素——並可通過屬性進行定制。

## 屬性 (Props)

**Props** 是 "properties" 的簡稱。通過 Props 可定制 React 組件。例如，此處我們為每個 `<Cat>` 傳遞不同的 `name` 供其渲染：

```SnackPlayer name=Multiple%20Props
import React from 'react';
import { Text, View } from 'react-native';

const Cat = (props) => {
  return (
    <View>
      <Text>Hello, I am {props.name}!</Text>
    </View>
  );
}

const Cafe = () => {
  return (
    <View>
      <Cat name="Maru" />
      <Cat name="Jellylorum" />
      <Cat name="Spot" />
    </View>
  );
}

export default Cafe;
```

大多數 React Native 核心組件也可通過 Props 定制。例如使用 [`Image`](image) 時，可傳遞 [`source`](image#source) 屬性來指定顯示的圖片：

```SnackPlayer name=Props
import React from 'react';
import { Text, View, Image } from 'react-native';

const CatApp = () => {
  return (
    <View>
      <Image
        source={{uri: "https://reactnative.dev/docs/assets/p_cat1.png"}}
        style={{width: 200, height: 200}}
      />
      <Text>Hello, I am your cat!</Text>
    </View>
  );
}

export default CatApp;
```

`Image` 擁有[許多不同的 props](image#props)，包括 [`style`](image#style)，它接受一個包含設計和佈局相關屬性-值對的 JS 物件。

> 請注意 `style` 的寬度和高度周圍的雙大括號 `{{ }}`。在 JSX 中，JavaScript 值是用 `{}` 來引用的。這在傳遞非字串的 props 時非常方便，比如陣列或數字：`<Cat food={["fish", "kibble"]} age={2} />`。然而，JS 物件**也**是用大括號表示的：`{width: 200, height: 200}`。因此，要在 JSX 中傳遞一個 JS 物件，你必須用**另一對**大括號將物件包起來：`{{width: 200, height: 200}}`

你可以用 props 和核心元件 [`Text`](text)、[`Image`](image) 和 [`View`](view) 來構建許多東西！但要構建互動式的東西，你需要狀態。

## 狀態

雖然你可以將 props 視為配置元件渲染方式的參數，但**狀態**就像是元件的個人數據存儲。狀態對於處理隨時間變化的數據或用戶交互產生的數據非常有用。狀態賦予你的元件記憶！

> 一般來說，當元件渲染時，使用 props 來配置它。使用狀態來跟踪你預期會隨時間變化的任何元件數據。

以下示例發生在一家貓咖啡館，兩隻飢餓的貓正在等待餵食。它們的飢餓程度（我們預期會隨時間變化，不像它們的名字）被存儲為狀態。要餵貓，按下它們的按鈕——這將更新它們的狀態。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

You can add state to a component by calling [React’s `useState` Hook](https://reactjs.org/docs/hooks-state.html). A Hook is a kind of function that lets you “hook into” React features. For example, `useState` is a Hook that lets you add state to function components. You can learn more about [other kinds of Hooks in the React documentation.](https://reactjs.org/docs/hooks-intro.html)

```SnackPlayer name=State
import React, { useState } from "react";
import { Button, Text, View } from "react-native";

const Cat = (props) => {
  const [isHungry, setIsHungry] = useState(true);

  return (
    <View>
      <Text>
        I am {props.name}, and I am {isHungry ? "hungry" : "full"}!
      </Text>
      <Button
        onPress={() => {
          setIsHungry(false);
        }}
        disabled={!isHungry}
        title={isHungry ? "Pour me some milk, please!" : "Thank you!"}
      />
    </View>
  );
}

const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
}

export default Cafe;
```

First, you will want to import `useState` from React like so:

```jsx
import React, {useState} from 'react';
```

Then you declare the component’s state by calling `useState` inside its function. In this example, `useState` creates an `isHungry` state variable:

```jsx
const Cat = props => {
  const [isHungry, setIsHungry] = useState(true);
  // ...
};
```

> You can use `useState` to track any kind of data: strings, numbers, Booleans, arrays, objects. For example, you can track the number of times a cat has been petted with `const [timesPetted, setTimesPetted] = useState(0)`!

Calling `useState` does two things:

- it creates a “state variable” with an initial value—in this case the state variable is `isHungry` and its initial value is `true`
- it creates a function to set that state variable’s value—`setIsHungry`

It doesn’t matter what names you use. But it can be handy to think of the pattern as `[<getter>, <setter>] = useState(<initialValue>)`.

Next you add the [`Button`](button) Core Component and give it an `onPress` prop:

```jsx
<Button
  onPress={() => {
    setIsHungry(false);
  }}
  //..
/>
```

Now, when someone presses the button, `onPress` will fire, calling the `setIsHungry(false)`. This sets the state variable `isHungry` to `false`. When `isHungry` is false, the `Button`’s `disabled` prop is set to `true` and its `title` also changes:

```jsx
<Button
  //..
  disabled={!isHungry}
  title={isHungry ? 'Pour me some milk, please!' : 'Thank you!'}
/>
```

> You might’ve noticed that although `isHungry` is a [const](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/const), it is seemingly reassignable! What is happening is when a state-setting function like `setIsHungry` is called, its component will re-render. In this case the `Cat` function will run again—and this time, `useState` will give us the next value of `isHungry`.

Finally, put your cats inside a `Cafe` component:

```jsx
const Cafe = () => {
  return (
    <>
      <Cat name="Munkustrap" />
      <Cat name="Spot" />
    </>
  );
};
```

</TabItem>
<TabItem value="classical">

The older class components approach is a little different when it comes to state.

```SnackPlayer name=State%20and%20Class%20Components
import React, { Component } from "react";
import { Button, Text, View } from "react-native";

class Cat extends Component {
  state = { isHungry: true };

  render() {
    return (
      <View>
        <Text>
          I am {this.props.name}, and I am
          {this.state.isHungry ? " hungry" : " full"}!
        </Text>
        <Button
          onPress={() => {
            this.setState({ isHungry: false });
          }}
          disabled={!this.state.isHungry}
          title={
            this.state.isHungry ? "Pour me some milk, please!" : "Thank you!"
          }
        />
      </View>
    );
  }
}

class Cafe extends Component {
  render() {
    return (
      <>
        <Cat name="Munkustrap" />
        <Cat name="Spot" />
      </>
    );
  }
}

export default Cafe;
```

As always with class components, you must import the `Component` class from React:

```jsx
import React, {Component} from 'react';
```

In class components, state is stored in a state object:

```jsx
export class Cat extends Component {
  state = {isHungry: true};
  //..
}
```

As with accessing props with `this.props`, you access this object inside your component with `this.state`:

```jsx
<Text>
  I am {this.props.name}, and I am
  {this.state.isHungry ? ' hungry' : ' full'}!
</Text>
```

And you set individual values inside the state object by passing an object with the key value pair for state and its new value to `this.setState()`:

```jsx
<Button
  onPress={() => {
    this.setState({isHungry: false});
  }}
  // ..
/>
```

> Do not change your component's state directly by assigning it a new value with `this.state.isHungry = false`. Calling `this.setState()` allows React to track changes made to state that trigger rerendering. Setting state directly can break your app's reactivity!

When `this.state.isHungry` is false, the `Button`’s `disabled` prop is set to `true` and its `title` also changes:

```jsx
<Button
  // ..
  disabled={!this.state.isHungry}
  title={
    this.state.isHungry
      ? 'Pour me some milk, please!'
      : 'Thank you!'
  }
/>
```

Finally, put your cats inside a `Cafe` component:

```jsx
class Cafe extends Component {
  render() {
    return (
      <>
        <Cat name="Munkustrap" />
        <Cat name="Spot" />
      </>
    );
  }
}

export default Cafe;
```

</TabItem>
</Tabs>

> 看到上面的 `<>` 和 `</>` 了嗎？這些 JSX 片段是 [fragments](https://reactjs.org/docs/fragments.html)。相鄰的 JSX 元素必須包裹在一個封閉標籤中。Fragments 讓你可以做到這一點，而無需嵌套一個額外的、不必要的包裹元素，如 `View`。

---

現在你已經涵蓋了 React 和 React Native 的核心元件，讓我們通過查看[處理 `<TextInput>`](handling-text-input) 來更深入地了解其中一些核心元件。