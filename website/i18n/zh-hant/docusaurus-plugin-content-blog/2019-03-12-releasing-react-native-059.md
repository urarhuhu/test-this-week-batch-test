---
title: Releasing React Native 0.59
author: Ryan Turner
authorTitle: Core Maintainer & React Native Developer
authorURL: 'https://twitter.com/turnrye'
authorImageURL: 'https://avatars0.githubusercontent.com/u/701035?s=460&v=4'
authorTwitter: turnrye
tags: [announcement, release]
---

Welcome to the 0.59 release of React Native! This is another big release with 644 commits by 88 contributors. Contributions also come in other forms, so _thank you_ for maintaining issues, fostering communities, and teaching people about React Native. This month brings a number of highly anticipated changes, and we hope you enjoy them.

## 🎣 Hooks are here

React Hooks are part of this release, which let you reuse stateful logic across components. There is a lot of buzz about hooks, but if you haven't heard, take a look at some of the wonderful resources below:

> - [Introducing Hooks](https://reactjs.org/docs/hooks-intro.html) explains why we’re adding Hooks to React.
> - [Hooks at a Glance](https://reactjs.org/docs/hooks-overview.html) is a fast-paced overview of the built-in Hooks.
> - [Building Your Own Hooks](https://reactjs.org/docs/hooks-custom.html) demonstrates code reuse with custom Hooks.
> - [Making Sense of React Hooks](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889) explores the new possibilities unlocked by Hooks.
> - [useHooks.com](https://usehooks.com/) showcases community-maintained Hooks recipes and demos.

Be sure to give this a try in your apps. We hope that you find the reuse as exciting as we do.

## 📱 Updated JSC means performance gains and 64-bit support on Android

React Native uses JSC ([JavaScriptCore](https://webkit.org/)) to power your application. JSC on Android was a few years old, which meant that a lot of modern JavaScript features weren't supported. Even worse, it performed poorly compared iOS's modern JSC. With this release, that all changes.

Thanks to some awesome work by [@DanielZlotin](https://github.com/danielzlotin), [@dulmandakh](https://github.com/dulmandakh), [@gengjiawen](https://github.com/gengjiawen), [@kmagiera](https://github.com/kmagiera), and [@kudo](https://github.com/kudo) JSC has caught up with the past few years. This brings with it 64-bit support, modern JavaScript support, and [big performance improvements](https://github.com/react-native-community/jsc-android-buildscripts/tree/master/measure). Kudos for also making this a maintainable process now so that we can take advantage of future WebKit improvements without so much legwork, and thank you Software Mansion and Expo for making this work possible.

## 💨 Faster app launches with inline requires

We want to help people have performant React Native apps by default and are working to bring Facebook's optimizations to the community. Applications load resources as needed rather than slowing down launch. This feature is called "inline requires", as it lets Metro identify components to be lazy loaded. Apps with a deep and varied component architecture will see the most improvement.

![source of the `metro.config.js` file in the 0.59 template, demonstrating where to enable `inlineRequires`](/blog/assets/inline-requires.png)

We need the community to let us know how it works before we turn it on by default. When you upgrade to 0.59, there will be a new `metro.config.js` file; flip the options to true and give us [your feedback](https://twitter.com/hashtag/inline-requires)! Read more about inline requires [in the performance docs](/docs/performance#inline-requires) to benchmark your app.

## 🚅 Lean core is underway

React Native is a large and complex project with a complicated repository. This makes the codebase less approachable to contributors, difficult to test, and bloated as a dev dependency. [Lean Core](https://github.com/react-native-community/discussions-and-proposals/issues/6) is our effort to address these issues by migrating code to separate libraries for better management. The past few releases have seen the first steps of this, but [let's get serious](https://www.youtube.com/watch?v=FMLKb4or8yg).

您可能會注意到，現在有額外的元件已被正式棄用。這是個好消息，因為這些功能現在有維護者積極維護它們。請注意警告訊息，並將這些功能遷移到新的函式庫，因為它們將在未來的版本中被移除。下方表格列出了元件、其狀態以及您可以遷移到的替代方案。

| Component            | Deprecated? | New home                                                                                                                                                 |
| -------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AsyncStorage**     | 0.59        | [@react-native-community/react-native-async-storage](https://github.com/react-native-community/react-native-async-storage)                               |
| **ImageStore**       | 0.59        | [expo-file-system](https://github.com/expo/expo/tree/master/packages/expo-file-system) or [react-native-fs](https://github.com/itinance/react-native-fs) |
| **MaskedViewIOS**    | 0.59        | [@react-native-community/react-native-masked-view](https://github.com/react-native-community/react-native-masked-view)                                   |
| **NetInfo**          | 0.59        | [@react-native-community/react-native-netinfo](https://github.com/react-native-community/react-native-netinfo)                                           |
| **Slider**           | 0.59        | [@react-native-community/react-native-slider](https://github.com/react-native-community/react-native-slider)                                             |
| **ViewPagerAndroid** | 0.59        | [@react-native-community/react-native-viewpager](https://github.com/react-native-community/react-native-viewpager)                                       |

在接下來的幾個月裡，將有更多元件遵循這條精簡核心的路徑。我們正在尋求協助 — 請前往 [lean core umbrella](https://github.com/facebook/react-native/issues/23313) 參與貢獻。

## 👩🏽‍💻 CLI 改進

React Native 的命令行工具是開發者進入生態系統的入口，但它們長期存在問題且缺乏官方支援。CLI 工具已被移至 [新的儲存庫](https://github.com/react-native-community/react-native-cli)，並且由 [專職的維護者團隊](https://blog.callstack.io/the-react-native-cli-has-a-new-home-79b63838f0e6) 已經做出了一些令人興奮的改進。

日誌的格式現在更加美觀。命令現在幾乎瞬間執行 — 您會立即注意到差異：

![0.58 的 CLI 啟動速度較慢](/blog/assets/0.58-cli-speed.png)![0.59 的 CLI 幾乎瞬間啟動](/blog/assets/0.59-cli-speed.png)

## 🚀 升級至 0.59

我們聽取了您關於 [React Native 升級流程](https://github.com/react-native-community/discussions-and-proposals/issues/68) 的反饋，並正在採取措施在 [未來的版本](https://github.com/react-native-community/discussions-and-proposals/issues/64#issuecomment-444775432) 中改善體驗。要升級至 0.59，我們建議使用 [`rn-diff-purge`](https://github.com/react-native-community/rn-diff-purge) 來確定您當前的 React Native 版本與 0.59 之間的變更，然後手動應用這些變更。一旦您將專案升級至 0.59，您將能夠使用新改進的 `react-native upgrade` 命令（基於 `rn-diff-purge`！）來升級至 0.60 及更高版本。

## 🔨 重大變更

0.59 中的 Android 支援已根據 Google 的最新建議進行了清理，這可能會導致現有應用程式出現潛在的中斷。此問題可能表現為運行時崩潰和訊息「You need to use a Theme.AppCompat theme (or descendant) with this activity」。我們建議更新專案的 `AndroidManifest.xml` 文件，確保 `android:theme` 值是一個 `AppCompat` 主題（例如 `@style/Theme.AppCompat.Light.NoActionBar`）。

`react-native-git-upgrade` 命令已在 0.59 中被移除，取而代之的是新改進的 `react-native upgrade` 命令。

## 🤗 感謝

許多新貢獻者協助 [啟用從 Flow 類型生成原生代碼](https://github.com/facebook/react-native/issues/22990) 和 [解決 Xcode 警告](https://github.com/facebook/react-native/issues/22609) — 這些是學習 React Native 工作原理並為公共利益做出貢獻的好方法。謝謝！請關注未來類似的問題。

雖然這些是我們注意到的亮點，但還有許多其他令人興奮的更新。要查看所有更新，請參閱 [變更日誌](https://github.com/react-native-community/react-native-releases/blob/master/CHANGELOG.md)。0.59 是一個巨大的版本 — 我們迫不及待想讓您試用。

我們在今年剩餘時間裡還會有更多改進。敬請期待！

[Ryan](https://github.com/turnrye) 和整個 [React Native 核心團隊](https://twitter.com/reactnative)