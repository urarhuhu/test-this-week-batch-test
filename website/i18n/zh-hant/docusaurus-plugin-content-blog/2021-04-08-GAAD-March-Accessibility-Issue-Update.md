---
title: The GAAD Pledge - March Accessibility Issues Update
authors: [alexmarlette]
tags: [announcement]
---

自從我們向 GitHub 社群發布經過徹底審查的無障礙功能缺口分析與改進清單以來，已經過了四個星期。在 React Native 社群的努力下，我們在改善無障礙功能方面已取得顯著進展。社群成員持續協助貢獻者、審查測試案例，並喚起對既有無障礙問題的關注。自 3 月 8 日起，社群已關閉六個議題，合併四個拉取請求，另有七個拉取請求正在審查流程中。

在這項工作持續進行的同時，Facebook 的 React Native 團隊與無障礙團隊正在評估在此計劃前提交的無障礙錯誤與議題，以確認它們是否已被現有缺口分析涵蓋，或是否有其他需要納入專案的新問題。目前已發現一個新議題並納入專案，四個議題直接對應到現有問題，另有兩個議題預計將透過解決根本原因的既有議題來關閉。

感謝所有參與的社群成員，你們確實推動了 React Native 變得更具無障礙性，造福所有人！

<!--truncate-->

## 已合併的拉取請求 🎉

- [為按鈕無障礙功能新增 TalkBack 支援：disabled 屬性 #31001](https://github.com/facebook/react-native/pull/31001) - 由 [@huzaifaaak ](https://twitter.com/huzaifaaak) 關閉

- [功能：當 TouchableHighlight 禁用時設置無障礙狀態為 disabled #31135](https://github.com/facebook/react-native/pull/31135) 由 [@natural_clar](https://twitter.com/natural_clar) 關閉

- [[Android] 當 TextInput 元件被選中時未播報選中狀態 #31144](https://github.com/facebook/react-native/pull/31144) 由 [fabriziobertoglio1987](https://fabriziobertoglio.xyz/) 關閉

- [為 TouchableNativeFeedback 無障礙功能新增 TalkBack 支援：disabled 屬性 #31224](https://github.com/facebook/react-native/pull/31224) 由 [@kyamashiro73](https://twitter.com/kyamashiro73) 關閉

- [無障礙功能/按鈕測試 #31189](https://github.com/facebook/react-native/pull/31189) 由 [@huzaifaaak ](https://twitter.com/huzaifaaak) 關閉

  - 新增針對按鈕 accessibilityState 的測試案例

## 修復項目

- `Button` 元件（由 [#31001](https://github.com/facebook/react-native/pull/31001) 修復）：

  - 現在會播報禁用狀態

  - 當按鈕禁用時，會對螢幕閱讀器禁用點擊功能

  - 會播報按鈕的選中狀態

- `TextInput` 元件（由 [#31144](https://github.com/facebook/react-native/pull/31144) 修復）：

  - 當 accessibilityState 的 "selected" 設為 true 且元素獲得焦點時，會播報「已選中」

- `TouchableHighlight` 元件（由 [#31135](https://github.com/facebook/react-native/pull/31135) 修復）：

  - 當元件禁用時，會對螢幕閱讀器禁用點擊功能

- `TouchableNativeFeedback` 元件（由 [#31224](https://github.com/facebook/react-native/pull/31224) 修復）：

  - 當元件禁用時，會對螢幕閱讀器禁用點擊功能

## 其他進展

| Status                                  | Number of Issues |
| --------------------------------------- | :--------------: |
| Issues To Do                            |        53        |
| In Progress Issues by the Community     |        8         |
| In Progress Issues by React Native Team |        5         |
| Pull Request in Progress                |        3         |
| Pull Request in Reviews                 |        4         |

## 參與貢獻！

- New contributors should read the [contribution guide](https://github.com/facebook/react-native/blob/master/CONTRIBUTING.md) and browse the list of 37 [good first issues](https://github.com/facebook/react-native/issues?q=is%3Aopen+is%3Aissue+label%3A%22Good+first+issue%22+label%3AAccessibility) in the React Native GitHub.

- Contributors interested in issues requiring a bit more effort should visit [the project page for Improved React Native Accessibility](https://github.com/facebook/react-native/projects/15) to see the GitHub issues that need their knowledge of React Native.

- Technical writers interested in updating React Native's documentation to reflect the accessibility gaps being closed should visit the [React Native Docs](https://github.com/facebook/react-native-website#-overview).

- Share this initiative with anyone who may be able to help!

- Follow the GAAD Pledge Open Source Accessibility Community Manager for React Native on [Twitter](https://twitter.com/alexmarlette) or [Facebook](https://www.facebook.com/React-Native-Open-Source-Accessibility-Community-Manager-102732258549941) to keep up to date on progress.