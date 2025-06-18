---
title: Open Source Roadmap
author: Héctor Ramos
authorTitle: Engineer at Facebook
authorURL: 'https://hectorramos.com/about'
authorImageURL: 'https://s.gravatar.com/avatar/f2223874e66e884c99087e452501f2da?s=128'
authorTwitter: hectorramos
tags: [announcement]
---

![](/blog/assets/oss-roadmap-hero.jpg)

今年，React Native 團隊致力於大規模的 [React Native 架構重構](https://github.com/react-native-community/discussions-and-proposals/issues/4)。正如 Sophie 在她的 [React Native 現狀文章](/blog/2018/06/14/state-of-react-native-2018) 中所提到的，我們已擬定計劃以更好地支持 Facebook 外部蓬勃發展的 React Native 用戶和協作者群體。現在是時候分享我們的工作細節了。在此之前，我想先闡明我們對開源 React Native 的長期願景。

我們對 React Native 的願景是...

- **健康的 GitHub 倉庫。** 問題和拉取請求能在合理時間內得到處理。
  - 提高測試覆蓋率。
  - 從 Facebook 代碼倉庫同步的提交不應破壞開源測試。
  - 更高規模的社區有意義貢獻。
- **穩定的 API，** 使其更容易與開源依賴項對接。
  - Facebook 使用與開源相同的公共 API。
  - React Native 發布遵循語意化版本控制。
- **活躍的生態系統。** 由社區維護的高質量 ViewManagers、原生模塊和多平台支持。
- **優秀的文檔。** 專注於幫助用戶創建高質量體驗，以及最新的 API 參考文檔。

我們已確定以下重點領域以幫助實現這一願景。

## ✂️ 精簡核心

我們的目標是通過移除非核心和未使用的組件來 [減少 React Native 的表面積](https://github.com/react-native-community/discussions-and-proposals/issues/6)。我們將把非核心組件移交給社區，使其能夠更快地發展。減少表面積將使管理對 React Native 的貢獻變得更加容易。

[`WebView`](https://github.com/react-native-community/discussions-and-proposals/blob/master/proposals/0001-webview.md) 是我們移交給社區的組件之一。我們正在開發一種工作流程，允許內部團隊在我們從倉庫中移除這些組件後繼續使用它們。我們已識別出 [數十個更多組件](https://github.com/react-native-community/discussions-and-proposals/issues/6)，將交由社區負責。

## 🎁 開源內部工具和 🛠 更新工具鏈

Facebook 產品團隊的 React Native 開發體驗可能與開源社區大不相同。開源社區中流行的工具在 Facebook 內部可能並未使用。可能存在實現相同目的的內部工具。在某些情況下，Facebook 團隊已習慣使用外部不存在的工具。這些差異在我們開源即將到來的架構工作時可能帶來挑戰。

我們將致力於發布部分這些內部工具。同時也將改進對開源社區流行工具的支持。以下是我們將處理的非詳盡項目列表：

- 開源 JSI 並使社區能夠引入自己的 JavaScript 虛擬機，取代 RN 初始版本中的 JavaScriptCore。我們將在未來的文章中詳細介紹 JSI，在此期間您可以通過 [Parashuram 在 React Conf 的演講](https://www.youtube.com/watch?v=UcqRXTriUVI) 了解更多。
- 支持 Android 上的 64 位庫。
- 在新架構下啟用調試功能。
- 改進對 CocoaPods、Gradle、Maven 和新 Xcode 構建系統的支持。

## ✅ 測試基礎設施

當 Facebook 工程師發布代碼時，如果通過所有測試則被認為可以安全合併。這些測試識別變更是否可能破壞我們自己的 React Native 界面。然而，Facebook 使用 React Native 的方式存在差異。這使我們可能在不知情的情況下破壞開源 React Native。

我們將加強內部測試，確保它們在盡可能接近開源的環境中運行。這將有助於防止破壞這些測試的代碼進入開源。我們還將致力於基礎設施改進，以更好地在 GitHub 上測試核心倉庫，使未來的拉取請求能夠輕鬆包含測試。

Combined with the reduced surface area, this will allow contributors to merge pull requests quicker, with confidence.

## 📜 Public API

Facebook will consume React Native via the public API, the same way open source does, to reduce unintentional breaking changes. We have started converting internal call sites to address this. Our goal is to converge on a stable, public API, leading to the adoption of semantic versioning in version 1.0.

## 📣 Communication

React Native is one of the [top open source projects on GitHub](https://octoverse.github.com/#top-and-trending-projects) by contributor count. That makes us really happy, and we'd like to keep it going. We'll continue working on initiatives that lead to involved contributors, such as increased transparency and open discussion. The documentation is one of the first things someone new to React Native will encounter, yet it has not been a priority. We'd like to fix that, starting with bringing back auto-generated API reference docs, creating additional content focused on creating [quality user experiences](/docs/improvingux), and improving our [release notes](https://github.com/react-native-community/react-native-releases/issues/47).

## Timeline

We're planning to land these projects throughout the next year or so. Some of these efforts are already ongoing, such as [JSI which has already landed in open source](https://github.com/facebook/react-native/compare/e337bcafb0b017311c37f2dbc24e5a757af0a205...8427f64e06456f171f9df0316c6ca40613de7a20). Others will take a bit longer to complete, such as reducing the surface area. We'll do our best to keep the community up to date with our progress. Please join us in the [Discussions and Proposals](https://github.com/react-native-community/discussions-and-proposals) repository, an initiative from the React Native community that has led to the creation of several of the initiatives discussed in this roadmap.