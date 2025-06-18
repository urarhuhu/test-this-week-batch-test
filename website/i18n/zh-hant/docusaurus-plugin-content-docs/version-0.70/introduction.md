---
id: getting-started
title: Introduction
description: This helpful guide lays out the prerequisites for learning React Native, using these docs, and setting up your environment.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<div className="content-banner">
  Welcome to the very start of your React Native journey! If you're looking for environment setup instructions, they've moved to <a href="environment-setup">their own section</a>. Continue reading for an introduction to the documentation, Native Components, React, and more!
  <img className="content-banner-img" src="/docs/assets/p_android-ios-devices.svg" alt=" " />
</div>

React Native 的使用者背景多元，從資深 iOS 開發者、React 初學者到首次接觸程式設計的人都有。本文件適用於所有學習者，無論其經驗水平或背景為何。

## 如何使用這些文件

您可以從這裡開始，像閱讀書籍一樣線性瀏覽；或直接查閱特定章節。若已熟悉 React，可跳過[該章節](intro-react)或快速複習。

## 前置知識

使用 React Native 需具備 JavaScript 基礎知識。若需學習或複習，可參考 [Mozilla 開發者網絡](https://developer.mozilla.org/en-US/docs/Web/JavaScript)的[入門指南](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)。

> 雖然我們盡量避免預設讀者具備 React、Android 或 iOS 開發知識，但這些主題對 React Native 開發者至關重要。適當時機我們會提供進階資源連結。

## 互動式範例

本篇導覽提供可直接在瀏覽器中運行的互動範例，例如：

```SnackPlayer name=Hello%20World
import React from 'react';
import { Text, View } from 'react-native';

const YourApp = () => {
  return (
    <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
      <Text>
        Try editing me! 🎉
      </Text>
    </View>
  );
}

export default YourApp;
```

上方是 Expo 開發的 Snack Player 工具，可嵌入並運行 React Native 專案，預覽其在 Android/iOS 平台的渲染效果。程式碼可即時編輯，請試著將「Try editing me!」文字改為「Hello, world!」。

> 若需設定本地開發環境，可[參閱環境設定指南](environment-setup)，並將範例程式碼貼至本機的 `App.js` 檔案（網頁開發者可能已具備行動瀏覽器測試環境）。

## 函式元件與類別元件

React 允許使用類別或函式定義元件。原本只有類別元件能持有狀態，但自 React Hooks API 推出後，函式元件也能管理狀態及其他功能。

[Hooks 於 React Native 0.59 版本引入](/blog/2019/03/12/releasing-react-native-059)。由於 Hooks 代表未來趨勢，本文件範例主要採用函式元件。必要時會以下列切換方式補充類別元件說明：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Hello%20World%20Function%20Component
import React from 'react';
import { Text, View } from 'react-native';

const HelloWorldApp = () => {
  return (
    <View style={{
      flex: 1,
      justifyContent: 'center',
      alignItems: 'center'
    }}>
      <Text>Hello, world!</Text>
    </View>
  );
}

export default HelloWorldApp;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Hello%20World%20Class%20Component
import React, { Component } from 'react';
import { Text, View } from 'react-native';

class HelloWorldApp extends Component {
  render() {
    return (
      <View style={{
          flex: 1,
          justifyContent: "center",
          alignItems: "center"
        }}>
        <Text>Hello, world!</Text>
      </View>
    );
  }
}

export default HelloWorldApp;
```

</TabItem>
</Tabs>

更多類別元件範例可查閱[舊版文件](/versions)。

## 開發者須知

React Native 學習者可能具備網頁、Android 或 iOS 等不同技術背景。我們會適時提供平台專屬說明，例如：

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["android","ios","web"])}>

<TabItem value="android">

> Android developers may be familiar with this concept.

</TabItem>
<TabItem value="ios">

> iOS developers may be familiar with this concept.

</TabItem>
<TabItem value="web">

> Web developers may be familiar with this concept.

</TabItem>
</Tabs>

## 格式規範

選單路徑以粗體顯示並用「>」符號表示層級，例如：**Android Studio > Preferences**。

---

了解本指南結構後，接下來可認識 React Native 的核心基礎：[原生元件](intro-react-native-components.md)。