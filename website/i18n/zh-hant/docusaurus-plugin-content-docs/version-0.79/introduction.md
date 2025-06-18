---
id: getting-started
title: Introduction
description: This helpful guide lays out the prerequisites for learning React Native, using these docs, and setting up your environment.
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

<div className="content-banner">
  Welcome to the very start of your React Native journey! If you're looking for getting started instructions, they've moved to <a href="environment-setup">their own section</a>. Continue reading for an introduction to the documentation, Native Components, React, and more!
  <img className="content-banner-img" src="/docs/assets/p_android-ios-devices.svg" alt=" " />
</div>

React Native 的使用者背景多元：從資深 iOS 開發者、React 初學者到職業生涯首次接觸程式設計的人都有。本文件面向所有學習者撰寫，無論其經驗水平或背景為何。

## 如何使用本文件

您可以從這裡開始，像閱讀書籍般線性瀏覽；或直接查閱特定章節。若已熟悉 React，可跳過[該章節](intro-react)或快速複習。

## 前置知識

使用 React Native 需具備 JavaScript 基礎知識。若需入門或複習，可參考 Mozilla Developer Network 的[完整指南](https://developer.mozilla.org/en-US/docs/Web/JavaScript)或[快速回顧](https://developer.mozilla.org/en-US/docs/Web/JavaScript/A_re-introduction_to_JavaScript)。

> 雖然我們盡量避免預設讀者具備 React、Android 或 iOS 開發知識，但這些都是進階學習的重要主題。適當時機我們會提供深入資源連結。

## 互動式範例

本篇導覽讓您能直接在瀏覽器中透過互動範例立即上手，例如：

```SnackPlayer name=Hello%20World
import React from 'react';
import {Text, View} from 'react-native';

const YourApp = () => {
  return (
    <View
      style={{
        flex: 1,
        justifyContent: 'center',
        alignItems: 'center',
      }}>
      <Text>Try editing me! 🎉</Text>
    </View>
  );
};

export default YourApp;
```

上方是 Expo 開發的 Snack Player 工具，可嵌入並運行 React Native 專案，展示其在 Android/iOS 平台的渲染效果。程式碼為即時可編輯狀態，您可直接在瀏覽器中修改。試著將「Try editing me!」文字改為「Hello, world!」吧！

> 若需建立本地開發環境，可[遵循本地環境設置指南](set-up-your-environment)並將程式碼範例貼入專案（網頁開發者可能已具備行動瀏覽器測試環境）。

## 開發者須知

React Native 學習者來自多元技術背景，可能具備網頁、Android、iOS 等開發經驗。我們嘗試兼顧不同背景的開發者，有時會提供特定平台的說明如下：

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

選單路徑以粗體標示並用箭頭表示子選單導覽，例如：**Android Studio > Preferences**。

---

了解本指南運作方式後，接下來將認識 React Native 的核心基礎：[原生元件](intro-react-native-components.md)。