---
title: 'React Native Monthly #3'
authors: [grabbou]
tags: [engineering]
---

React Native 每月會議持續進行！本月的會議稍短，因為多數團隊正忙於產品發布。下個月，我們將在波蘭弗羅茨瓦夫舉辦的 [React Native EU](https://react-native.eu/) 大會上見面。記得搶購門票，現場見！現在，讓我們看看各團隊的最新動態。

## 團隊

第三次會議共有 5 個團隊參與：

- [Callstack](https://github.com/callstack)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)

## 會議記錄

以下是各團隊的分享摘要：

### Callstack

- 近期開源了 [`react-native-material-palette`](https://github.com/callstack-io/react-native-material-palette)，該工具能從圖像提取主色調以打造視覺吸引力的應用。目前僅支援 Android，未來計劃擴展至 iOS。
- 我們已將熱模組替換（HMR）功能整合進 [`haul`](https://github.com/callstack-io/haul)，並加入多項新特性！請查看最新版本。
- React Native EU 2017 即將登場！下個月將是 React Native 與波蘭的主場！最後門票請至[官網](https://react-native.eu/)搶購。

### Expo

- 在 [Snack](https://snack.expo.io) 上新增了 npm 套件安裝功能（需符合 Expo 內建原生 API 限制）。我們正開發多檔案支援與資源上傳功能。[Satyajit](https://github.com/satya164) 將於 [React Native Europe](https://react-native.eu/) 大會分享 Snack 相關內容。
- 發布 SDK20，新增相機、支付、安全儲存、磁力計、暫停/恢復檔案下載等功能，並優化啟動/載入畫面。
- 持續與 [Krzysztof](https://github.com/kmagiera) 合作開發 [react-native-gesture-handler](https://github.com/kmagiera/react-native-gesture-handler)。歡迎試用並回饋使用 PanResponder 或原生手勢識別器的遷移問題。
- 實驗性開發 JSC 除錯協議，並處理 [Canny](https://expo.canny.io/feature-requests) 上的功能請求。

### Facebook

- 上個月討論了 GitHub issue 追蹤器的管理策略，目標提升專案維護效率。
- 目前未解決 issue 數量穩定維持約 600 件。過去一個月，我們關閉了 690 個長期無活動（定義為 60 天無評論）的 issue，其中 58 件因維護者承諾修復或貢獻者提出充分理由被重新開啟。
- 我們將持續自動清理停滯 issue，目標是確保每個重要 issue 都能獲得處理。現階段亟需維護者協助分類 issue，避免遺漏回歸問題或重大變更（尤其影響新專案者）。有意協助者可參閱 Facebook GitHub Bot 使用指南，並加入 [issue 任務小組](https://github.com/facebook/react-native/blob/master/bots/IssueCommands.txt)。

### Microsoft

- 新版 Skype 應用程式基於 React Native 開發，以實現跨平台最大程度的程式碼共享。目前採用 React Native 的 Skype 應用已在 Android 和 iOS 應用商店上架。
- 在基於 React Native 開發 Skype 應用過程中，我們針對遇到的錯誤和缺失功能向 React Native 提交了 Pull Request。截至目前，已有約 [70 個 PR 被合併](https://github.com/facebook/react-native/pulls?utf8=%E2%9C%93&q=is%3Apr%20author%3Arigdern%20)。
- React Native 讓我們能透過單一程式碼庫同時驅動 Android 和 iOS 版 Skype 應用。我們還希望利用該程式碼庫支援 Skype 網頁版應用。為實現此目標，我們開發並開源了建構於 React/React Native 之上的輕量層 [ReactXP](https://microsoft.github.io/reactxp/blog/2017/04/06/introducing-reactxp.html)。ReactXP 提供一組跨平台元件，在 iOS/Android 平台會映射到 React Native，在網頁平台則映射到 react-dom。ReactXP 的目標與另一個開源庫 React Native for Web 相似。[ReactXP 常見問題](https://microsoft.github.io/reactxp/docs/faq.html) 中簡要說明了這些庫在方法上的差異。

### Shoutem

- 我們持續致力於改善和簡化使用 [Shoutem](https://shoutem.github.io/) 建構應用程式的開發者體驗。
- 已開始將所有應用遷移至 react-navigation，但最終決定延後此計劃，直到更穩定版本發布或某個原生導航解決方案趨於穩定。
- 正在將所有 [擴充功能](https://github.com/shoutem/extensions) 和大多數開源庫（[動畫](https://github.com/shoutem/animation)、[主題](https://github.com/shoutem/theme)、[UI](https://github.com/shoutem/ui)）更新至 React Native 0.47.1 版本。

## 下次會議

下次會議定於 2017 年 9 月 13 日星期三舉行。由於這僅是我們的第三次會議，我們想了解這些記錄如何使 React Native 社群受益。若您對如何改進會議成果有任何建議，歡迎透過 [Twitter](https://twitter.com/grabbou) 與我聯繫。