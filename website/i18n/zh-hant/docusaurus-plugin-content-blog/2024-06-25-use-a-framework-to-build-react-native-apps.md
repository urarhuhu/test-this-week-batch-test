---
title: 'Use a framework to build React Native apps'
authors: [cortinico]
tags: [announcement]
date: 2024-06-25
---

在[React Conf](https://www.youtube.com/live/0ckOUBiuxVY?si=pU4qP4eB5iWfY0IG&t=2320)上，我們更新了關於開始構建React Native應用程式的最佳工具指南：**React Native框架**——一個包含所有必要API的工具箱，讓您能構建生產就緒的應用程式。

使用React Native框架（如Expo）現在是創建新應用程式的**推薦**方法。

在這篇部落格文章中，我們將詳細介紹它們是什麼，以及對於作為React Native開發者的您開始新專案意味著什麼。

<!-- truncate  -->

## 什麼是React Native框架？

如果您一直在構建生產應用程式，您可能知道有一系列常見問題遲早需要解決。

無論是在網頁還是原生平台上構建任何應用程式，您可能希望用戶能瀏覽不同畫面、獲取數據並存儲用戶狀態。但對於原生應用程式，還有更多需要處理的事項：您需要工具來升級React Native版本之間的原生代碼、管理所有依賴項的兼容版本，以及處理原生建構工具。

沒有合適的工具，將一個應用程式從想法帶到生產環境需要付出巨大努力。

我們希望您能專注於為用戶編寫精美的應用程式和功能，而不是反覆解決這些常見問題。

這就是為什麼我們認為，體驗React Native的最佳方式是透過一個提供所有必要工具的工具箱框架來構建生產就緒的應用程式。

我們發現**您要麼使用框架，要麼在構建自己的框架**。

構建自己的框架並沒有錯，透過為路由、導航、部署等問題打造自己的解決方案。像Meta和Microsoft這樣的大型企業會在內部構建自己的框架，以深度整合到他們的既有應用程式中。但我們相信大多數人使用現有框架會更好。

如果您一直在網頁上使用React，您可能熟悉類似的[生產級React框架](https://react.dev/learn/start-a-new-react-project#production-grade-react-frameworks)概念。

截至目前，唯一推薦的React Native社群框架是[Expo](https://docs.expo.dev/)。Expo團隊從React Native早期就開始投資於React Native生態系統，我們認為Expo提供的開發者體驗是目前最優秀的。

:::note

Expo框架將保持免費和開源，而Expo應用程式服務（EAS）是可選的付費服務。

:::

<!--alex ignore he-she retext-equality-->

如果您最近沒有使用Expo，請不要錯過[Kadi在Expo的這場演講](https://www.youtube.com/live/0ckOUBiuxVY?si=N-WSfmAJSMfd6wDL&t=3888)，她展示了2024年您能用Expo做什麼。

我們也更新了網站上的[入門頁面](https://reactnative.dev/docs/environment-setup)以反映這一推薦。

## 框架將如何影響您？

如果您已經在使用推薦的框架（如Expo），那麼您已經準備就緒！

如果您想將現有應用程式遷移到Expo，可以在[Expo官方網站](https://docs.expo.dev/bare/overview/)上找到相關說明。Expo提供了許多好處，例如更簡單的[升級React Native版本](https://docs.expo.dev/workflow/upgrading-expo-sdk-walkthrough/)方式、更好的開發者體驗等等。

However, if you can't or don't want to migrate to Expo, that's fine too. Using React Native without an official framework will continue to be supported. The tools you’ve been using such as React Native Community CLI, Template and [Upgrade Helper](https://react-native-community.github.io/upgrade-helper/) will keep on working as usual.

The `react-native init` command has moved out of core and is now accessible via:

```
npx @react-native-community/cli@latest init
```

and on GitHub at [react-native-community/cli](https://github.com/react-native-community/cli).

If you’re a React Native library developer, we collected a list of recommendations on which APIs to use. [Read more in the RFC](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#what-do-we-recommend-to-react-native-library-developers).

## Further reading

If you’re interested in learning more about the reasoning behind this decision, we invite you to read the [RFC0759: React Native Frameworks](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#what-do-we-recommend-to-react-native-library-developers). This RFC is a result of a multi-month effort involving countless discussions and brainstorming among different partners and players of the React Native ecosystem.

While Expo today is the only recommended framework, the RFC also contains [guidelines](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0759-react-native-frameworks.md#becoming-a-react-native-framework) on how to become a recommended framework, as we hope to see more competition and innovation in this space.

Moreover, you should check out the talk [useFrameworks()](https://www.youtube.com/watch?v=lifGTznLBcw) at App.js 2024 where we presented this RFC and the necessary changes in a short format.

We believe that by clarifying the respective responsibilities of React Native Core and the Frameworks, we can foster a healthier ecosystem and drive growth & innovation for React Native.