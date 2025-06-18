---
title: Toward Better Documentation
author: Kevin Lacker
authorTitle: Engineering Manager at Facebook
authorURL: 'https://twitter.com/lacker'
authorImageURL: 'https://www.gravatar.com/avatar/9b790592be15d4f55a5ed7abb5103304?s=128'
authorTwitter: lacker
tags: [announcement]
---

優質的開發者體驗部分來自於完善的技術文件。要產出優秀的文件需要投入大量心力——理想的文件應當簡潔、實用、準確、完整且令人愉悅。近期我們根據您的反饋持續改進文件品質，在此分享部分優化成果。

## 內嵌範例

當您學習新函式庫、程式語言或框架時，最美好的時刻莫過於首次編寫的程式碼成功運行——您親手創造了真實可用的東西。我們希望將這種直觀體驗直接融入文件中。例如：

```ReactNativeWebPlayer
import React, { Component } from 'react';
import { AppRegistry, Text, View } from 'react-native';

class ScratchPad extends Component {
  render() {
    return (
      <View style={{flex: 1}}>
        <Text style={{fontSize: 30, flex: 1, textAlign: 'center'}}>
          Isn't this cool?
        </Text>
        <Text style={{fontSize: 100, flex: 1, textAlign: 'center'}}>
          👍
        </Text>
      </View>
    );
  }
}

AppRegistry.registerComponent('ScratchPad', () => ScratchPad);
```

這些內嵌範例採用[`react-native-web-player`](https://github.com/dabbott/react-native-web-player)模組（由[Devin Abbott](https://twitter.com/devinaabbott)協助開發），是學習React Native基礎的絕佳方式。我們已更新[新手教學文件](/docs/tutorial)，盡可能採用此形式。試試看吧——若您曾好奇修改範例程式碼會發生什麼，這種互動形式能讓您輕鬆實驗。此外，若您正在開發工具並希望展示即時React Native範例，[`react-native-web-player`](https://github.com/dabbott/react-native-web-player)能簡化此流程。

核心模擬引擎由[Nicolas Gallagher](https://twitter.com/necolas)的[`react-native-web`](https://github.com/necolas/react-native-web)專案提供，該專案能讓`Text`、`View`等React Native元件在網頁端顯示。如果您有興趣建構共享程式碼的跨平台（行動裝置與網頁）體驗，請參閱[`react-native-web`](https://github.com/necolas/react-native-web)。

## 強化指南

React Native某些功能存在多種實作方式，我們收到反饋指出需要更明確的指引。

新版[導航指南](/docs/navigation)比較了不同方案（`Navigator`、`NavigatorIOS`、`NavigationExperimental`）並提供選用建議。中期規劃中我們將改進並整合這些介面，短期目標是透過更完善的指南減輕您的決策負擔。

我們新增了[觸控處理指南](/docs/handling-touches)，解釋按鈕式介面的基礎實作，並簡述各種觸控事件處理方式。

另一項重點是Flexbox佈局，包含[Flexbox佈局教學](/docs/flexbox)、[元件尺寸控制](/docs/height-and-width)，以及雖不炫目但實用的[React Native佈局屬性全集](/docs/layout-props)。

## 入門指引

在本地環境設定React Native開發工具時，難免需要安裝與配置多項元件。雖然難以讓安裝過程變得有趣，但我們至少能使其快速且無痛。

新版[入門指引流程](/docs/next/getting-started)讓您預先選擇開發環境與目標行動平台，集中呈現所有設定步驟。我們實際走遍安裝流程以確保每個環節運作正常，並在決策點提供明確建議。經過同事實際測試後，我們確信這項改進卓有成效。

我們也強化了[現有應用整合指南](/docs/integration-with-existing-apps)。許多大型應用程式（如Facebook主應用）實際採用混合開發模式——部分功能使用React Native，其餘則沿用傳統工具開發。希望本指南能幫助更多人實踐此開發模式。

## 需要您的協助

您的反饋讓我們知道應該優先處理哪些事項。我知道有些人讀完這篇部落格文章後會想：「改進文件？哼！X 的文件還是很糟糕！」這很好——我們需要這種聲音。根據反饋類型的不同，提供反饋的最佳方式也有所不同。

如果您在文件中發現錯誤，例如描述不準確或程式碼無法運作，請[提交 issue](https://github.com/facebook/react-native/issues)。標記為「Documentation」，以便更容易將其轉交給相關人員。

如果沒有具體錯誤，但文件中的某些內容根本令人困惑，這不太適合提交 GitHub issue。相反，請在 [Canny](https://react-native.canny.io/feature-requests) 上發文討論需要改進的文件部分。這有助於我們在進行指南撰寫等一般性工作時排定優先順序。

感謝您閱讀至此，也感謝您使用 React Native！