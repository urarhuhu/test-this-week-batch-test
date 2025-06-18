---
title: 'React Native Monthly #5'
author: Tomislav Tenodi
authorTitle: Founder at Speck
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

React Native 月度會議持續進行中！來看看我們的團隊最近在忙些什麼。

### Callstack

- 我們一直在改進 React Native 的持續整合系統。最重要的是，我們已從 Travis 遷移至 CircleCI，讓 React Native 擁有統一的工作流程。
- 我們舉辦了 [Hacktoberfest - React Native 特別版](https://blog.callstack.io/announcing-hacktoberfest-7313ea5ccf4f)，與參與者共同向開源專案提交了大量 Pull Request。
- 我們持續開發 [Haul](https://github.com/callstack/haul)。上個月發布了兩個新版本，包含 webpack 3 支援。未來計劃增加 [CRNA](https://github.com/react-community/create-react-native-app) 和 [Expo](https://github.com/expo/expo) 支援，並改進熱模組替換功能。開發路線圖已公開在 issue 追蹤系統中，歡迎提供建議或反饋！

### Expo

- 發布了 [Expo SDK 22](https://blog.expo.io/expo-sdk-v22-0-0-is-now-available-7745bfe97fc6)（基於 React Native 0.49），並更新了對應的 [CRNA](https://github.com/react-community/create-react-native-app)。
  - 包含改進的啟動畫面 API、基礎 ARKit 支援、「DeviceMotion」API、iOS11 的 SFAuthenticationSession 支援等[更多功能](https://blog.expo.io/expo-sdk-v22-0-0-is-now-available-7745bfe97fc6)。
- [Snack](https://snack.expo.io) 現在支援多個 JavaScript 檔案，並可透過拖放方式上傳圖片等資源。
- 參與 [react-navigation](https://github.com/react-community/react-navigation) 開發，新增 iPhone X 支援。
- 聚焦於改善使用 Expo 開發大型應用時的痛點，例如：
  - 強化多環境部署支援（測試、生產與自訂頻道），頻道將支援版本回滾與指定發布版本。歡迎透過 [@expo_io](https://twitter.com/expo_io) 申請成為早期測試者。
  - 改進獨立應用建置系統，新增非程式碼資源（如圖片）的打包功能，同時保持空中更新能力。

### Facebook

- 強化 RTL 支援：
  - 新增多種方向感知樣式：
    - 定位：
      - (left|right) → (start|end)
    - 邊距：
      - margin(Left|Right) → margin(Start|End)
    - 內距：
      - padding(Left|Right) → padding(Start|End)
    - 邊框：
      - borderTop(Left|Right)Radius → borderTop(Start|End)Radius
      - borderBottom(Left|Right)Radius → borderBottom(Start|End)Radius
      - border(Left|Right)Width → border(Start|End)Width
      - border(Left|Right)Color → border(Start|End)Color
  - 在 RTL 模式下，「left」與「right」的定位、邊距、內距和邊框樣式意義將被修正。未來幾個月內，我們會移除舊有行為，使「left」始終表示左側，「right」始終表示右側。此變更可透過 `I18nManager.swapLeftAndRightInRTL(false)` 提前啟用。
- 使用 [Flow](https://github.com/facebook/flow) 為內部原生模組建立型別定義，並據此生成 Java 介面與 Objective-C 協議。預計明年開源此程式碼生成工具。

### Infinite Red

- 發布新開源工具 [Solidarity](https://shift.infinite.red/solidarity-the-cli-for-developer-sanity-672fa81b98e9)，協助 React Native 等專案的開發者。
- 正在重構 [Ignite](https://github.com/infinitered/ignite) 以發布新版樣板（代號：Bowser）。

### Shoutem

- Improving the development flow on Shoutem. We want to streamline the process from creating an app to first custom screen and make it really easy, thus lowering the barrier for new React Native developers. Prepared a few workshops to test out new features. We also improved [Shoutem CLI](https://github.com/shoutem/cli) to support new flows.
- [Shoutem UI](https://github.com/shoutem/ui) received a few component improvements and bugfixes. We also checked compatibility with latest React Native versions.
- Shoutem platform received a few notable updates, new integrations are available as part of the [open-source extensions project](https://github.com/shoutem/extensions). We are really excited to see active development on Shoutem extensions from other developers. We actively contact and offer advice and guidance about their extensions.

## Next session

The next session is scheduled for Wednesday 6, December 2017. Feel free to ping me [on Twitter](https://twitter.com/TomislavTenodi) if you have any suggestion on how we should improve the output of the meeting.