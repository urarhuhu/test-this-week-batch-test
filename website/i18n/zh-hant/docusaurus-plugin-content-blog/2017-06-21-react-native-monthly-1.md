---
title: 'React Native Monthly #1'
author: Tomislav Tenodi
authorTitle: Product Manager at Shoutem
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

At [Shoutem](https://shoutem.github.io/), we've been fortunate enough to work with React Native from its very beginnings. We decided we wanted to be part of the amazing community from day one. Soon enough, we realized it's almost impossible to keep up with the pace the community was growing and improving. That's why we decided to organize a monthly meeting where all major React Native contributors can briefly present what their efforts and plans are.

## Monthly meetings

We had our first session of the monthly meeting on June 14, 2017. The mission for React Native Monthly is simple and straightforward: **improve the React Native community**. Presenting teams' efforts eases collaboration between teams done offline.

## Teams

On the first meeting, we had 8 teams join us:

- [Airbnb](https://github.com/airbnb)
- [Callstack](https://github.com/callstack-io)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [GeekyAnts](https://github.com/GeekyAnts)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)
- [Wix](https://github.com/wix)

We hope to have more core contributors join the upcoming sessions!

## Notes

As teams' plans might be of interest to a broader audience, we'll be sharing them here, on the React Native blog. So, here they are:

### Airbnb

- Plans to add some A11y (accessibility) APIs to `View` and the `AccessibilityInfo` native module.
- Will be investigating adding some APIs to native modules on Android to allow for specifying threads for them to run on.
- Have been investigating potential initialization performance improvements.
- Have been investigating some more sophisticated bundling strategies to use on top of "unbundle".

### Callstack

- Looking into improving the release process by using [Detox](https://github.com/wix/detox) for E2E testing. Pull request should land soon.
- Blob pull request they have been working on has been merged, subsequent pull requests coming up.
- Increasing [Haul](https://github.com/callstack-io/haul) adoption across internal projects to see how it performs compared to [Metro Bundler](https://github.com/facebook/metro-bundler). Working on better multi-threaded performance with the webpack team.
- Internally, they have implemented a better infrastructure to manage open source projects. Plans to be getting more stuff out in upcoming weeks.
- The React Native Europe conference is coming along, nothing interesting yet, but y'all invited!
- Stepped back from [react-navigation](https://github.com/react-community/react-navigation) for a while to investigate alternatives (especially native navigations).

### Expo

- Working on making it possible to install npm modules in [Snack](https://snack.expo.io/), will be useful for libraries to add examples to documentation.
- Working with [Krzysztof](https://github.com/kmagiera) and other people at [Software Mansion](https://github.com/software-mansion) on a JSC update on Android and a gesture handling library.
- [Adam Miskiewicz](https://github.com/skevy) is transitioning his focus towards [react-navigation](https://github.com/react-community/react-navigation).
- [Create React Native App](https://github.com/react-community/create-react-native-app) is in the [Getting Started guide](/docs/getting-started) in the docs. Expo wants to encourage library authors to explain clearly whether their lib works with CRNA or not, and if so, explain how to set it up.

### Facebook

- React Native 的打包工具現已獨立為 [Metro Bundler](https://github.com/facebook/metro) 專案。倫敦的 Metro Bundler 團隊致力於滿足社群需求，提升模組化以支援 React Native 以外的使用情境，並加快對問題與 PR 的回應速度。
- 未來幾個月，React Native 團隊將著重優化基礎元件的 API。預計將改善佈局異常、無障礙功能與 Flow 型別檢查等問題。
- 團隊也計劃在今年提升核心模組化程度，透過重構全面支援 Windows 和 macOS 等第三方平台。

### GeekyAnts

- 團隊正在開發一款直接操作 `.js` 檔案的 UI/UX 設計工具（代號：Builder），目前僅支援 React Native。功能類似 Adobe XD 和 Sketch。
- 團隊致力於實現：在編輯器中載入現有 React Native 應用程式後，設計師可視覺化修改並直接保存變更至 JS 檔案。
- 目標是縮短設計師與開發者之間的鴻溝，讓兩者能在同一個程式庫中協作。
- 此外，[NativeBase](https://github.com/GeekyAnts/NativeBase) 近期已突破 5,000 GitHub 星標。

### Microsoft

- [CodePush](https://github.com/Microsoft/code-push) 已整合至 [Mobile Center](https://mobile.azure.com/)，這是強化與分發、分析等服務整合的第一步。詳見[公告](https://microsoft.github.io/code-push/articles/CodePushOnMobileCenter.html)。
- [VS Code](https://github.com/Microsoft/vscode) 的偵錯功能存在問題，團隊正在修復並將發布新版本。
- 評估使用 [Detox](https://github.com/wix/detox) 進行整合測試，研究透過 JSC Context 在當機報告中獲取變數資訊。

### Shoutem

- 簡化使用 React Native 社群工具開發 Shoutem 應用的流程，未來可在 [Shoutem](https://shoutem.github.io/) 建立的應用中直接執行所有 React Native 指令。
- 研究 React Native 效能分析工具，團隊在設置過程中遇到許多問題，將分享相關經驗。
- 正在開發更簡便的 React Native 與現有原生應用整合方案，計劃公開內部開發的概念以獲取社群反饋。

### Wix

- 內部推動採用 [Detox](https://github.com/wix/detox)，目標將 Wix 應用程式的重大功能轉為「零人工 QA」。目前已有數十名開發者在生產環境中密集使用，加速 Detox 的成熟度。
- 為 [Metro Bundler](https://github.com/facebook/metro) 新增支援建置時覆寫任意副檔名的功能（不僅限 "ios" 和 "android"，還可支援 "e2e" 或 "detox" 等自訂副檔名），計劃用於 E2E 模擬測試。已發布 [react-native-repackager](https://github.com/wix/react-native-repackager) 函式庫，現正準備提交 PR。
- 開發自動化效能測試工具 [DetoxInstruments](https://github.com/wix/DetoxInstruments)，該專案以開源方式進行。
- 與 KPN 貢獻者合作開發 Detox 的 Android 版本與實體裝置支援。
- 構思「Detox 平台化」，讓其他工具能自動化模擬器/裝置操作，例如 [Storybook](https://github.com/storybooks/react-native-storybook) for React Native 或 Ram 提出的整合測試方案。

## 下次會議

Meetings will be held every four weeks. The next session is scheduled for July 12, 2017. As we just started with this meeting, we'd like to know how do these notes benefit the React Native community. Feel free to ping me [on Twitter](https://twitter.com/TomislavTenodi) if you have any suggestion on what we should cover in the following sessions, or how we should improve the output of the meeting.