---
title: 'React Native Monthly #2'
author: Tomislav Tenodi
authorTitle: Product Manager at Shoutem
authorURL: 'https://github.com/tenodi'
authorImageURL: 'https://pbs.twimg.com/profile_images/877237660225609729/bKFDwfAq.jpg'
authorTwitter: TomislavTenodi
tags: [engineering]
---

React Native 每月會議持續進行中！本次會議我們邀請到 [Infinite Red](https://infinite.red/) 團隊參與，他們正是 [Chain React 研討會](https://infinite.red/ChainReactConf) 的幕後推手。由於多數與會者都在 Chain React 發表議程，我們將會議順延一週。大會演講影片已[上線發布](https://www.youtube.com/playlist?list=PLFHvL21g9bk3RxJ1Ut5nR_uTZFVOxu522)，誠摯推薦您觀看。現在，讓我們看看各團隊的最新動態。

## 與會團隊

第二次會議共有 9 個團隊參與：

- [Airbnb](https://github.com/airbnb)
- [Callstack](https://github.com/callstack-io)
- [Expo](https://github.com/expo)
- [Facebook](https://github.com/facebook)
- [GeekyAnts](https://github.com/GeekyAnts)
- [Infinite Red](https://github.com/infinitered)
- [Microsoft](https://github.com/microsoft)
- [Shoutem](https://github.com/shoutem)
- [Wix](https://github.com/wix)

## 會議紀要

以下是各團隊的分享內容：

### Airbnb

- 歡迎查看 [Airbnb 代碼庫](https://github.com/airbnb) 中的 React Native 相關專案。

### Callstack

- [Mike Grabowski](https://github.com/grabbou) 持續負責 React Native 每月版本發布，包括推送多個測試版。特別著重於發布 v0.43.5 版本至 npm，此版本將解決 Windows 用戶的阻塞問題！
- [Haul](https://github.com/callstack-io/haul) 專案進展雖緩慢但穩定。已有實現 HMR 功能的拉取請求提交，其他改進也已發布。近期獲得業界領導者採用，考慮啟動該領域的全職付費開發計劃。
- 來自 [Jest](https://github.com/facebook/jest) 團隊的 [Michał Pierzchała](https://twitter.com/thymikee) 本月加入 Callstack。他將協助維護 [Haul](https://github.com/callstack-io/haul)，並可能參與 [Metro Bundler](https://github.com/facebook/metro) 和 [Jest](https://github.com/facebook/jest) 的開發。
- [Satyajit Sahoo](https://twitter.com/satya164) 正式加入團隊！
- 開源部門即將推出多項新成果，特別著重於將 Material Palette API 引入 React Native。計劃最終發布我們的原生 iOS 套件，旨在提供與原生組件 1:1 的視覺與操作體驗。

### Expo

- Recently launched [Native Directory](https://native.directory) to help with discoverability and evaluation of libraries in React Native ecosystem. The problem: lots of libraries, hard to test, need to manually apply heuristics and not immediately obvious which ones are just the best ones that you should use. It's also hard to know if something is compatible with CRNA/Expo. So Native Directory tries to solve these problems. Check it out and [add your library](https://github.com/react-community/native-directory) to it. The list of libraries is in [here](https://github.com/react-community/native-directory/blob/master/react-native-libraries.json). This is just our first pass of it, and we want this to be owned and run by the community, not just Expo folks. So please pitch in if you think this is valuable and want to make it better!
- Added initial support for installing npm packages in [Snack](https://snack.expo.io/) with Expo SDK 19. Let us know if you run into any issues with it, we are still working through some bugs. Along with Native Directory, this should make it easy to test libraries that have only JS dependencies, or dependencies included in [Expo SDK](https://github.com/expo/expo-sdk). Try it out:
  - [react-native-modal](https://snack.expo.io/ByBCD_2r-)
  - [react-native-animatable](https://snack.expo.io/SJfJguhrW)
  - [react-native-calendars](https://snack.expo.io/HkoXUdhr-)
- [Released Expo SDK19](https://blog.expo.io/expo-sdk-v19-0-0-is-now-available-821a62b58d3d) with a bunch of improvements across the board, and we're now using the [updated Android JSC](https://github.com/SoftwareMansion/jsc-android-buildscripts).
- Working on a guide in docs with [Alexander Kotliarskyi](https://github.com/frantic) with a list of tips on how to improve the user experience of your app. Please join in and add to the list or help write some of it!
  - Issue: [#14979](https://github.com/facebook/react-native/issues/14979)
  - Initial pull request: [#14993](https://github.com/facebook/react-native/pull/14993)
- Continuing to work on: audio/video, camera, gestures (with Software Mansion, `react-native-gesture-handler`), GL camera integration and hoping to land some of these for the first time in SDK20 (August), and significant improvements to others by then as well. We're just getting started on building infrastructure into the Expo client for background work (geolocation, audio, handling notifications, etc.).
- [Adam Miskiewicz](https://twitter.com/skevy) has made some nice progress on imitating the transitions from [UINavigationController](https://developer.apple.com/documentation/uikit/uinavigationcontroller) in [react-navigation](https://github.com/react-community/react-navigation). Check out an earlier version of it in [his tweet](https://twitter.com/skevy/status/884932473070735361) - release coming with it soon. Also check out `MaskedViewIOS` which he [upstreamed](https://github.com/facebook/react-native/commit/8ea6cea39a3db6171dd74838a6eea4631cf42bba). If you have the skills and desire to implement `MaskedView` for Android that would be awesome!

### Facebook

- Facebook is internally exploring being able to embed native [ComponentKit](https://componentkit.org/) and [Litho](https://fblitho.com/) components inside of React Native.
- Contributions to React Native are very welcome! If you are wondering how you can contribute, the ["How to Contribute" guide](https://github.com/facebook/react-native-website/blob/master/CONTRIBUTING.md) describes our development process and lays out the steps to send your first pull request. There are other ways to contribute that do not require writing code, such as by triaging issues or updating the docs.
  - At the time of writing, React Native has **635** [open issues](https://github.com/facebook/react-native/issues) and **249** [open pull requests](https://github.com/facebook/react-native/pulls). This is overwhelming for our maintainers, and when things get fixed internally, it is difficult to ensure the relevant tasks are updated.
  - We are unsure what the best approach is to handle this while keeping the community satisfied. Some (but not all!) options include closing stale issues, giving significantly more people permissions to manage issues, and automatically closing issues that do not follow the issue template. We wrote a "What to Expect from Maintainers" guide to set expectations and avoid surprises. If you have ideas on how we can make this experience better for maintainers as well as ensuring people opening issues and pull requests feel heard and valued, please let us know!

### GeekyAnts

- We demoed the Designer Tool which works with React Native files on Chain React. Many attendees signed up for the waiting list.
- We are also looking at other cross-platform solutions like [Google Flutter](https://flutter.io/) (a major comparison coming along), [Kotlin Native](https://github.com/JetBrains/kotlin-native), and [Apache Weex](https://weex.incubator.apache.org/) to understand the architectural differences and what we can learn from them to improve the overall performance of React Native.
- Switched to [react-navigation](https://github.com/react-community/react-navigation) for most of our apps, which has improved the overall performance.
- Also, announced [NativeBase Market](https://market.nativebase.io/) - A marketplace for React Native components and apps (for and by the developers).

### Infinite Red

- We want to introduce the [Reactotron](https://github.com/infinitered/reactotron). Check out the [introductory video](https://www.youtube.com/watch?v=tPBRfxswDjA). We'll be adding more features very soon!
- Organised Chain React Conference. It was awesome, thanks all for coming! [The videos are now online!](https://www.youtube.com/playlist?list=PLFHvL21g9bk3RxJ1Ut5nR_uTZFVOxu522)

### Microsoft

- [CodePush](https://github.com/Microsoft/code-push) has now been integrated into [Mobile Center](https://mobile.azure.com/). Existing users will have no change in their workflow.
  - Some people have reported an issue with duplicate apps - they already had an app on Mobile Center. We are working on resolving them, but if you have two apps, let us know, and we can merge them for you.
- Mobile Center now supports Push Notifications for CodePush. We also showed how a combination of Notifications and CodePush could be used for A/B testing apps - something unique to the ReactNative architecture.
- [VS Code](https://github.com/Microsoft/vscode) has a known debugging issue with ReactNative - the next release of the extension in a couple of days will be fixing the issue.
- Since there are many other teams also working on React Native inside Microsoft, we will work on getting better representation from all the groups for the next meeting.

### Shoutem

- Finished the process of making the React Native development easier on [Shoutem](https://shoutem.github.io/). You can use all the standard `react-native` commands when developing apps on Shoutem.
- We did a lot of work trying to figure out how to best approach the profiling on React Native. A big chunk of [documentation](/docs/performance) is outdated, and we'll do our best to create a pull request on the official docs or at least write some of our conclusions in a blog post.
- Switching our navigation solution to [react-navigation](https://github.com/react-community/react-navigation), so we might have some feedback soon.
- We released [a new HTML component](https://github.com/shoutem/ui/tree/develop/html) in our toolkit which transforms the raw HTML to the React Native components tree.

### Wix

- We started working on a pull request to [Metro Bundler](https://github.com/facebook/metro) with [react-native-repackager](https://github.com/wix/react-native-repackager) capabilities. We updated react-native-repackager to support RN 44 (which we use in production). We are using it for our mocking infrastructure for [detox](https://github.com/wix/detox).
- We have been covering the Wix app in detox tests for the last three weeks. It's an amazing learning experience of how to reduce manual QA in an app of this scale (over 40 engineers). We have resolved several issues with detox as a result, a new version was just published. I am happy to report that we are living up to the "zero flakiness policy" and the tests are passing consistently so far.
- Detox for Android is moving forward nicely. We are getting significant help from the community. We are expecting an initial version in about two weeks.
- [DetoxInstruments](https://github.com/wix/detoxinstruments), our performance testing tool, is getting a little bigger than we originally intended. We are now planning to turn it into a standalone tool that will not be tightly coupled to detox. It will allow investigating the performance of iOS apps in general. It will also be integrated with detox so we can run automated tests on performance metrics.

## Next session

The next session is scheduled for August 16, 2017. As this was only our second meeting, we'd like to know how do these notes benefit the React Native community. Feel free to ping me [on Twitter](https://twitter.com/TomislavTenodi) if you have any suggestion on how we should improve the output of the meeting.