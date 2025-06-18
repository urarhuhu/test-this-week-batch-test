---
title: Toward Hermes being the Default
authors: [huxpro]
tags: [announcement]
---

Since [we announced Hermes in 2019](https://engineering.fb.com/2019/07/12/android/hermes/), it has been increasingly gaining adoption in the community. The team at [Expo](https://expo.dev/), who maintain a popular meta-framework for React Native apps, recently [announced experimental](https://blog.expo.dev/expo-sdk-42-579aee2348b6) [support](https://blog.expo.dev/expo-sdk-43-beta-is-now-available-47dc54a8d29f) for Hermes after being [one of the most requested features of Expo](https://expo.canny.io/feature-requests/p/enabling-hermes). The team at [Realm](https://realm.io/), a popular mobile database, also recently shipped its [alpha support](https://github.com/realm/realm-js/issues/3940) for Hermes. In this post, we want to highlight some of the most exciting progress we've made over the past two years to push Hermes towards being _the best_ JavaScript engine for React Native. Looking forward, we are confident that with these improvements and more to come, we can make Hermes the default JavaScript engine for React Native across all platforms.

<!--truncate-->

## Optimizing for React Native

Hermes’s defining feature is how it performs compilation work ahead-of-time, meaning that React Native apps with Hermes enabled ship with precompiled optimized bytecode instead of plain JavaScript source. This drastically reduces the amount of work needed to start up your product for users. Measurements from both Facebook and community apps have suggested that enabling Hermes often cut a product’s TTI (or [Time-To-Interactive](https://web.dev/interactive/)) metric by nearly half.

That being said, we’ve been working on improving Hermes in many other aspects to make it even better as a JavaScript engine specialized for React Native.

### Building a New Garbage Collector for Fabric

With the upcoming [Fabric](https://github.com/react-native-community/discussions-and-proposals/issues/4) renderer in the new React Native architecture, it will be possible to synchronously call JavaScript on the UI thread. However, this means if the JavaScript thread takes too long to execute, it can cause noticeable UI frame drops and block user inputs. The [concurrent rendering](https://reactjs.org/blog/2021/06/08/the-plan-for-react-18.html) enabled by React [Fiber](https://reactjs.org/docs/faq-internals.html#what-is-react-fiber) will avoid scheduling long JavaScript tasks by splitting rendering work into chunks. However, there is another common source of latency from the JavaScript thread — when the JavaScript engine has to “stop the world” to perform a garbage collection (GC).

The previous default garbage collector in Hermes, [GenGC](https://hermesengine.dev/docs/gengc/), was a single-threaded generational garbage collector. The new generations uses a typical semi-space copying strategy, and the old generations uses a mark-compact strategy to make it really good at aggressively returning memory to the operating system. Due to its single-thread, GenGC has the downside of causing long GC pauses. On apps that are as complicated as Facebook for Android, we observed an average pause of 200ms, or 1.4s at p99. We have even seen it be as long as 7 seconds, considering the large and diverse user base of Facebook for Android.

為了解決這個問題，我們實現了一個全新的「大部分並行」垃圾回收器，名為 [Hades](https://hermesengine.dev/docs/hades)。Hades 的年輕代回收方式與 GenGC 完全相同，但其老年代則採用了一種「開始時快照」風格的標記-清除回收器。這種設計能大幅減少 GC 暫停時間，因為大部分工作會在背景執行緒中完成，不會阻擋引擎主執行緒執行 JavaScript 程式碼。**我們的統計數據顯示，Hades 在 64 位元裝置上的 p99.9 暫停時間僅為 48ms（比 GenGC 快 34 倍！）**，而在 32 位元裝置上（此時它作為單執行緒的「增量式」GC 運作）約為 88ms。這些暫停時間的改善可能會以整體吞吐量為代價，因為需要更昂貴的寫入屏障、較慢的基於空閒列表的記憶體配置（相對於指標碰撞配置器），以及增加的堆碎片化。我們認為這些是合理的權衡，並且我們能夠通過合併和其他將要討論的記憶體優化來實現整體更低的記憶體消耗。

### 針對效能痛點出擊

應用程式的啟動時間對許多應用的成功至關重要，我們持續在為 React Native 突破界限。對於 Hermes 中實現的任何新 JavaScript 功能，我們都會仔細監控它們對生產效能的影響，確保不會導致指標倒退。在 Facebook，我們目前正在實驗 [Metro 中專為 Hermes 設計的 Babel 轉換配置](https://github.com/facebook/react-native/blob/main/packages/react-native-babel-preset/src/configs/main.js#L41)，用 Hermes 的原生 ESNext 實現取代十幾個 Babel 轉換。我們在許多介面上觀察到 **18-25% 的 TTI 改善** 以及 **整體位元組碼大小減少**，並期待在開源社群中看到類似結果。

除了啟動效能，我們還發現記憶體佔用是 React Native 應用的一個改進機會，尤其是對於 [虛擬實境](https://reactnative.dev/blog/2021/08/26/many-platform-vision#expanding-to-new-platforms)。得益於我們作為 JavaScript 引擎的低階控制能力，我們能夠通過精打細算地優化位元和位元組來實現多輪記憶體優化：

1. 以前，所有 JavaScript 值都以 64 位元的 NaN-boxing 編碼標籤值表示，用於在 64 位元架構上表示浮點雙精度數和指標。然而，這在實際上是浪費的，因為大多數數字是小整數（SMI），且客戶端應用的 JavaScript 堆通常不會超過 4GiB。為了解決這個問題，我們引入了一種新的 32 位元編碼，其中 SMI 和指標以 29 位元編碼（因為指標是 8 位元組對齊的，我們可以假設最低 3 位元始終為零），其餘的 JS 數字則被封裝到堆上。**這使得 JavaScript 堆大小減少了約 30%。**
2. 不同種類的 JavaScript 物件在 JavaScript 堆中以不同種類的 GC 管理單元表示。通過積極優化這些單元的標頭記憶體佈局，**我們能夠再減少約 15% 的記憶體使用量**。

我們在 Hermes 上的一個關鍵決策是不實現 [即時編譯器（JIT）](https://en.wikipedia.org/wiki/Just-in-time_compilation)，因為我們認為對於大多數 React Native 應用來說，額外的暖機成本和二進位及記憶體上的額外佔用並不值得。多年來，我們投入了大量精力優化直譯器效能和編譯器優化，使 Hermes 的吞吐量在 React Native 工作負載上能與其他引擎競爭。我們將繼續通過識別各處的效能瓶頸（直譯器分派循環、堆疊佈局、物件模型、GC 等）來改善吞吐量。期待在未來的版本中看到更多數字！

### 垂直整合的先鋒

在 Facebook，我們傾向於將專案集中在一個大型 [單一儲存庫](https://en.wikipedia.org/wiki/Monorepo) 中。通過讓引擎（Hermes）和宿主（React Native）緊密協同迭代，我們開闢了許多垂直整合的空間。舉幾個例子：

- Hermes supports [on-device JavaScript debugging with the Chrome debugger](https://reactnative.dev/docs/hermes#debugging-js-on-hermes-using-google-chromes-devtools) by speaking the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/). It’s better than the legacy “[Remote JS Debugging](https://reactnative.dev/docs/debugging#chrome-developer-tools)” (which uses an in-app proxy to run JS in desktop Chrome) because it supports debugging synchronous native calls and guaranteed a consistent runtime environment. Together with React DevTools, Metro, Inspector, and so on, Hermes debugger is now part of [Flipper](https://reactnative.dev/blog/2020/03/26/version-0.62) to provide a one-stop developer experience.
- Objects allocated during the initialization path of React Native apps are often long-lived and don’t follow the _generational_ _hypothesis_ leveraged by generational GCs. Therefore, we [configured Hermes in React Native](https://github.com/facebook/react-native/blob/main/ReactAndroid/src/main/java/com/facebook/hermes/reactexecutor/OnLoad.cpp#L37-L42) to allocate the first 32MiB directly into old generations (known as _pre-tenuring_) to avoid triggering GC pauses and delaying TTI.
- The new React Native architecture is heavily based on [JSI (or JavaScript Interface)](https://github.com/react-native-community/discussions-and-proposals/issues/91), a lightweight, general-purposed API for embedding a JavaScript engine into a C++ program. By having the team maintaining the JS engine also maintains the JSI API implementation, we are confident in providing the best possible integration that is reliable, performant and battle-tested at the Facebook’s scale.
- Getting JavaScript concurrency primitives (e.g. [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)) and platform concurrency primitives (e.g. [microtasks](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide)) both semantically correct and performant are critical to React concurrent rendering and the future of React Native apps. Historically, promises in React Native were [polyfilled](https://github.com/facebook/react-native/blob/main/Libraries/Core/polyfillPromise.js#L37) using non-standardized [`setImmediate`](https://developer.mozilla.org/en-US/docs/Web/API/Window/setImmediate) APIs. We are working on making native promises and microtasks from JS engines available via JSI, and introducing [`queueMicrotask`](https://developer.mozilla.org/en-US/docs/Web/API/queueMicrotask), a recent addition to the web standard, to the platform, to better support modern asynchronous JavaScript code.

## Bringing Along the Whole Community

Hermes has been really great for us at Facebook. But our work is not done until our community can use Hermes to power experiences throughout the ecosystem, so that everyone leverage all of its features and to embrace its full potential.

### Expanding to New Platforms

Hermes was initially open sourced only for React Native on Android. Since then, we have been thrilled to see our members of the community expanding Hermes support into [many other platforms that React Native’s ecosystem has expanded](https://reactnative.dev/blog/2021/08/26/many-platform-vision).

[Callstack](https://callstack.com/) led the effort of bringing [Hermes to iOS in React Native 0.64](https://reactnative.dev/blog/2021/03/12/version-0.64). They wrote a [series of articles](https://callstack.com/blog/bringing-hermes-to-ios-in-react-native/) and hosted a [podcast](https://callstack.com/podcasts/react-native-0-64-with-hermes-for-ios-ep-5) on how they achieved it. According to their benchmarks, Hermes was able to **consistently deliver ~40% improvement to startup and ~18% reduced memory on iOS** compared to JSC for the Mattermost app, with only 2.4 MiB of app size overhead. I encourage you to [see it live with your own eyes](https://callstack.com/blog/hermes-performance-on-ios/).

Microsoft has been bringing [Hermes to React Native for Windows and macOS](https://microsoft.github.io/react-native-windows/docs/hermes). [At Microsoft Build 2020](https://youtu.be/QMFbrHZnvvw?t=389), Microsoft shared that Hermes’s memory impact ([working set](https://en.wikipedia.org/wiki/Working_set)) is 13% lower than the Chakra engine on React Native for Windows. Recently, on some synthetic benchmarks, they’ve found Hermes 0.8 (shipped with Hades and aforementioned SMI and pointer compression optimization) **uses 30%-40% less memory than other engines**. Not surprisingly, the [desktop Messenger](https://www.messenger.com/desktop) video calling experience built on React Native, is also powered by Hermes.

Last but not least, Hermes has also been powering all virtual reality experiences built with the React family of technologies on Oculus, including Oculus Home.

### Supporting our Community

We acknowledge there are still blockers that prevent parts of the community from adopting Hermes and we are committed to building support for these missing features. Our goal is to be fully featured so that Hermes is the right choice for most React Native apps. Here is how the community has already shaped the Hermes roadmap:

<!-- alex ignore just fellowship -->

- [`Proxy` and `Reflect`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Meta_programming) were originally excluded from Hermes because Facebook does not use them. We were also concerned that adding Proxy would hurt property lookup performance even when Proxy is not used. But Proxy quickly become [the most requested feature](https://github.com/facebook/hermes/issues/33) of Hermes due to popular libraries such as [MobX](https://mobx.js.org/README.html) and [Immer](https://immerjs.github.io/immer/). We carefully evaluated and decided to build it just for the community, and we managed to implement it with very low cost. Since this is a feature we don’t use, we relied on our community to prove its stability. We started by testing Proxy behind a flag and created opt-in npm packages for [release v0.4](https://github.com/facebook/hermes/issues/33#issuecomment-668374607) and [v0.5](https://github.com/facebook/hermes/issues/33#issuecomment-668374607), and it’s [enabled by default starting from v0.7](https://github.com/facebook/hermes/releases/tag/v0.7.0).
- [ECMAScript Internationalization API Specification (ECMA-402, or `Intl`)](https://hermesengine.dev/docs/intl) was [the second most requested feature](https://github.com/facebook/hermes/issues/23). `Intl` is a huge set of APIs and often requires the implementation to include **6MB worth** of [Unicode CLDR](https://cldr.unicode.org/index) data. This is why polyfills like [FormatJS (a.k.a. `react-intl`)](https://github.com/formatjs/formatjs) and JS engines like the [international variant build of community JSC](https://github.com/react-native-community/jsc-android-buildscripts#international-variant) are so huge. To avoid substantially increasing the binary size of Hermes, we decided to implement it with another strategy by consuming and mapping the ICU facilities provided by the libraries included in the operating systems, at the cost of some (often minor) variance in behaviors across platforms.
  - Microsoft collaborated to build support on Android. It covers almost everything from ECMA-402 up to ES2020, **with only a size impact as small as 3% (57-62K per ABI)**. We ran [a poll on Twitter](https://twitter.com/tmikov/status/1336442786694893568) and the results were strongly in favor of including `Intl` by default, so that’s what we did and it’s available starting from [release v0.8](https://github.com/facebook/hermes/releases/tag/v0.8.0).
  - Facebook has sponsored [Major League Hacking](https://mlh.io/) to launch a [remote open source fellowship program](https://news.mlh.io/welcoming-facebook-back-as-a-sponsor-of-the-2020-2021-mlh-fellowship-08-12-2020). Last year, we launched the [Hermes sampling profiler](https://reactnative.dev/docs/profile-hermes). This year, our fellows will be working with members from Hermes, React Native, and Callstack, to add support for Hermes `Intl` on iOS. Stayed tuned!
- We appreciate that people have been working with us to discover issues affecting the community.
  - People have helped us identify critical spec divergence such as [stability of `Array.prototype.sort`](https://github.com/facebook/hermes/issues/212) amended in [ES2019](https://github.com/tc39/ecma262/pull/1340). This has been fixed and will be available in the next release.
  - People found out that our default heap size limit was too small and caused [unnecessary GC pressure](https://github.com/facebook/hermes/issues/295) and [OOM crashes](https://github.com/facebook/hermes/issues/511) for many users who are not familiar with customizing Hermes’s GC configs. So we increased it from 512MiB to 3GiB to be more than sufficient for most users by default.
  - People also reported that our specialized `Function.prototype.toString` implementation [caused performance to drop in libraries doing improper feature detection](https://github.com/facebook/hermes/issues/471#issuecomment-820123463) and [blocked users from doing source code injecting](https://github.com/facebook/hermes/issues/114#issuecomment-887106990). This helped us strengthen our stance that Hermes, whenever possible, should not get in the way of developers and to respect de-facto practices.

## Summary

In summary, our vision is to make Hermes ready to be the default JavaScript engine across all React Native platforms. We’ve already started working towards it, and we want to hear from all of you about this direction.

It’s extremely important for us to prepare the ecosystem for a smooth adoption. We encourage you to try out Hermes, and file issues on our [GitHub repository](https://github.com/facebook/hermes) for any feedbacks, questions, feature requests and incompatibilities.

## Thanks

We’d love to thank the Hermes team, the React Native team, and the many contributors from the React Native community for their work to improve Hermes.

<!-- alex ignore white -->

I’d also love to personally thank (in alphabetic order) Eli White, Luna Wei, Neil Dhar, Tim Yung, Tzvetan Mikov, and many others for their help during the writing.