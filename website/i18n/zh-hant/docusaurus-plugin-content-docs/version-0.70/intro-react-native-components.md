---
id: intro-react-native-components
title: Core Components and Native Components
description: 'React Native lets you compose app interfaces using Native Components. Conveniently, it comes with a set of these components for you to get started with right now—the Core Components!'
---

import ThemedImage from '@theme/ThemedImage';

React Native 是一個開源框架，用於使用 [React](https://reactjs.org/) 和應用平台的本地功能來構建 Android 和 iOS 應用程式。透過 React Native，你可以使用 JavaScript 來存取平台的 API，並使用 React 元件來描述 UI 的外觀和行為：這些是可重複使用、可嵌套的程式碼包。你可以在下一節中了解更多關於 React 的內容。但首先，讓我們來看看 React Native 中的元件是如何運作的。

## 視圖與移動開發

在 Android 和 iOS 開發中，**視圖**是 UI 的基本構建塊：螢幕上的一個小矩形元素，可以用來顯示文字、圖片或響應用戶輸入。即使是應用中最小的視覺元素，如一行文字或一個按鈕，也是視圖的一種。某些類型的視圖可以包含其他視圖。這就是所謂的「視圖層層嵌套」！

<figure>
  <img src="/docs/assets/diagram_ios-android-views.svg" width="1000" alt="Diagram of Android and iOS app showing them both built on top of atomic elements called views." />
  <figcaption>Just a sampling of the many views used in Android and iOS apps.</figcaption>
</figure>

## 本地元件

在 Android 開發中，你使用 Kotlin 或 Java 來編寫視圖；在 iOS 開發中，你使用 Swift 或 Objective-C。透過 React Native，你可以使用 React 元件以 JavaScript 來調用這些視圖。在運行時，React Native 會為這些元件創建相應的 Android 和 iOS 視圖。因為 React Native 元件是由與 Android 和 iOS 相同的視圖支持的，所以 React Native 應用程式的外觀、感覺和性能與其他應用程式一樣。我們稱這些由平台支持的元件為**本地元件**。

React Native 提供了一組基本的、即開即用的本地元件，你可以立即開始構建你的應用程式。這些是 React Native 的**核心元件**。

React Native 還允許你為 [Android](native-components-android.md) 和 [iOS](native-components-ios.md) 構建自己的本地元件，以滿足應用的獨特需求。我們還有一個繁榮的生態系統，這些是**社區貢獻的元件**。查看 [Native Directory](https://reactnative.directory) 以了解社區的創作。

## 核心元件

React Native 有許多核心元件，從控制項到活動指示器應有盡有。你可以在 [API 部分](components-and-apis) 找到它們的所有文檔。你主要會使用以下核心元件：

| React Native UI Component | Android View   | iOS View         | Web Analog              | Description                                                                                           |
| ------------------------- | -------------- | ---------------- | ----------------------- | ----------------------------------------------------------------------------------------------------- |
| `<View>`                  | `<ViewGroup>`  | `<UIView>`       | A non-scrolling `<div>` | A container that supports layout with flexbox, style, some touch handling, and accessibility controls |
| `<Text>`                  | `<TextView>`   | `<UITextView>`   | `<p>`                   | Displays, styles, and nests strings of text and even handles touch events                             |
| `<Image>`                 | `<ImageView>`  | `<UIImageView>`  | `<img>`                 | Displays different types of images                                                                    |
| `<ScrollView>`            | `<ScrollView>` | `<UIScrollView>` | `<div>`                 | A generic scrolling container that can contain multiple components and views                          |
| `<TextInput>`             | `<EditText>`   | `<UITextField>`  | `<input type="text">`   | Allows the user to enter text                                                                         |

在下一節中，你將開始結合這些核心元件來了解 React 的工作原理。現在就來試試它們吧！

```SnackPlayer name=Hello%20World
import React from 'react';
import { View, Text, Image, ScrollView, TextInput } from 'react-native';

const App = () => {
  return (
    <ScrollView>
      <Text>Some text</Text>
      <View>
        <Text>Some more text</Text>
        <Image
          source={{
            uri: 'https://reactnative.dev/docs/assets/p_cat2.png',
          }}
          style={{ width: 200, height: 200 }}
        />
      </View>
      <TextInput
        style={{
          height: 40,
          borderColor: 'gray',
          borderWidth: 1
        }}
        defaultValue="You can type in me"
      />
    </ScrollView>
  );
}

export default App;
```

---

由於 React Native 使用與 React 元件相同的 API 結構，你需要了解 React 元件 API 才能開始。[下一節](intro-react) 是一個快速的介紹或複習。不過，如果你已經熟悉 React，可以[跳過](handling-text-input)。

<ThemedImage
alt="A diagram showing React Native's Core Components are a subset of React Components that ship with React Native."
sources={{
  light: '/docs/assets/diagram_react-native-components.svg',
  dark: '/docs/assets/diagram_react-native-components_dark.svg',
}}
/>