---
title: React Native Core Contributor Summit 2022
authors: [thymikee, cortinico]
tags: [announcement]
date: 2022-11-22
---

# React Native 核心貢獻者峰會 2022

歷經多年疫情與僅限線上的活動後，我們深感是時候讓 React Native 的核心貢獻者們齊聚一堂！

因此，我們在九月初召集了 React Native 活躍的核心貢獻者、函式庫維護者，以及 Meta 的 React Native 與 Metro 團隊成員，舉辦了 **核心貢獻者峰會 2022**。[Callstack](https://www.callstack.com/) 在其位於波蘭弗羅茨瓦夫的總部主辦了這場峰會，作為同期舉行的 [React Native EU](https://www.react-native.eu/) 大會的一部分。

我們與 React Native 核心團隊共同設計了一系列**工作坊**，供與會者參與。主題包括：

- React Native Codegen 與 TypeScript 支援
- React Native 新架構函式庫遷移
- React Native 單體倉庫
- Metro 網頁與生態系統對齊
- Metro 簡化發布流程

這兩天中，我們對知識分享與協作的深度感到驚艷。在這篇部落格文章中，我們想帶您一窺此次聚會的成果。

<!--truncate-->

### React Native Codegen 與 TypeScript 支援

[React Native 的 Codegen](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/codegen.md) 是 React Native 新架構的基礎部分。支援並改進它是我們對 React Native 未來的首要任務之一。例如，今年稍早，我們新增了從 TypeScript 規格而非 Flow 開始的泛型程式碼支援。

在這個工作坊中，我們藉此機會向新貢獻者介紹 Codegen，解釋其核心概念並描述其運作方式。接著我們聚焦於兩個主要領域：

#### 1. 支援 Codegen 目前**尚未支援的新類型**。其中一個高度需求的功能是 [TypeScript 中的字串聯合類型](https://github.com/Titozzz/react-native/tree/codegen-string-union)。

一小組人進入會議室專攻此任務。他們在過程中遇到並克服了一些困難，例如如何執行 Codegen 的單元測試。他們花了不少時間理解程式碼執行流程，才開始處理程式碼本身。經過數小時的協作後，他們完成了能識別字串聯合類型的第一個原型。這次經驗對於討論設計模式與未來理想架構極具價值。

#### 2. 改進 **[iOS 的自動連結功能](https://github.com/facebook/react-native/pull/34580)**，該功能原先缺少一個使用情境的支援。

具體而言，當函式庫與應用程式共存於單體倉庫時，自動連結無法順利運作。Android 已支援此情境，但 iOS 尚缺此功能。

與貢獻者共同處理 Codegen 的過程讓我們意識到，在其程式碼庫中工作並非易事。例如，新增對一種類型的支援需要在四個不同地方複製貼上相同程式碼：使用 Flow 編寫規格的模組、使用 TypeScript 編寫規格的模組、使用 Flow 編寫規格的元件，以及使用 TypeScript 編寫規格的元件。

這個發現促使我們建立了一個[總體任務](https://github.com/facebook/react-native/issues/34872)，向社群尋求協助以改善程式碼庫狀態，使其更易維護。

參與情況非常踴躍：我們在 **5 天內迅速分配了前 40 項任務**。截至十月底，社群已完成 **47 項任務**，還有許多其他任務已準備就緒等待合併。

這項計畫也為所有參與改進的貢獻者們的 [Hacktoberfest](https://hacktoberfest.com/) 活動添磚加瓦！

### React Native 新架構函式庫遷移

The hot topic in the React Native space is the New Architecture. Having **libraries** that support the New Architecture is a crucial point in the [migration for the whole ecosystem](/blog/2022/06/16/resources-migrating-your-react-native-library-to-the-new-architecture). Therefore, we want to support library maintainers in migrating to the New Architectures.

Initially, this session started as a brainstorming, where the core contributors had the opportunity to ask the React Native team all the questions they had related to the New Architecture. This in-person feedback loop was crucial for both the core contributors to bring clarity and for the React Native team to collect feedback. Some of the shared feedback and concerns will end up being implemented in React Native 0.71.

We then moved to practically migrating as many libraries to the new architecture as possible. During this session, we kicked off the migration process for several community packages, such as `react-native-document-picker`, `react-native-store-review`, and `react-native-orientation`.

As a reminder, if you’re also migrating a library and need support in doing so, please reach out to our [New Architecture Working Group](https://github.com/reactwg/react-native-new-architecture) on GitHub.

### ​​React Native Monorepo

Releasing a new version of React Native is not trivial today. React Native is one of the most downloaded packages on NPM, and we want to make sure that our release process is smooth.

That’s why we want to refactor the `react-native` repository and implement the **Monorepo RFC** ([#480](https://github.com/react-native-community/discussions-and-proposals/pull/480)).

In this session, we initially brainstormed and collected opinions from every contributor, as it’s crucial that we evolve our repository to reduce the breaking changes for our downstream dependencies.

We then started working on two fronts. First, we had to expand our Continuous Integration infrastructure to support our monorepo, adding [Verdaccio](https://verdaccio.org/) to our testing infrastructure. We then started renaming & adding scopes to several of our packages, resulting in 6 distinct contributions.

You can track the status of this effort under this [umbrella issue](https://github.com/facebook/react-native/issues/34692) and we hope to share more on this effort in the near future.

### Metro Web and Ecosystem Alignment

[Metro](https://github.com/facebook/metro), our JavaScript Bundler, is a foundational and integrated part of the React Native development experience and we want to make sure it works with the latest standards in the JS ecosystem.

The focus of this session was to discuss improving Metro's feature set to work better for web use cases and with the npm and bundler ecosystem. Two major areas of discussion:

#### 1. Adopting the `"exports"` ([package entry points](https://nodejs.org/api/packages.html#package-entry-points)) specification

From the [Node.js documentation](https://nodejs.org/api/packages.html#package-entry-points):

<!-- alex ignore clearly -->

:::info
The ["exports"](https://nodejs.org/api/packages.html#exports) provides a modern alternative to ["main"](https://nodejs.org/api/packages.html#main) allowing multiple entry points to be defined, conditional entry resolution support between environments, and **preventing any other entry points besides those defined in ["exports"](https://nodejs.org/api/packages.html#exports)**. This encapsulation allows module authors to define the public interface for their package clearly.
:::

Adopting the `"exports"` specification has a lot of potential. In this session, we debated on how to handle [Platform Specific Code](/docs/platform-specific-code#platform-specific-extensions) with `"exports"`. Considering many factors, we came up with a fairly non-breaking rollout plan for `"exports"`, by adding a `"strict"` and `"non-strict"` mode to Metro resolver. We discussed how leveraging [builder-bob](https://github.com/callstack/react-native-builder-bob) would help library creators adopt the strict mode without friction.

This discussion resulted in:

1. An [RFC](https://github.com/react-native-community/discussions-and-proposals/pull/534) for Metro on how package exports would work with React Native.
2. An [RFC](https://github.com/nodejs/node/pull/45367) for Node.js to include “react-native” as a Community Condition.

#### 2. Web and bundler ecosystem

The Metro team shared progress from their partnership with Expo and the intent to continue this working model for upcoming bundle splitting and tree-shaking support. We touched again on ES module support and considered potential future features such as Yarn PnP and output optimization on the web. We discussed how Metro’s core shares logic and data structures with Jest and opportunities for more reuse.

Developers surfaced insightful use cases for bundle splitting and interoperability with third-party tooling. This led us to discuss potential extension points in Metro and improving current documentation.

This discussion provided us with good grounding for the next day's session on simplifying the release workflow.

### Metro Simplified Release Workflow

As mentioned, releasing React Native is not trivial.

Things get harder as we need to release React Native, the React Native CLI, and Metro. Those tools are connected to each other as React Native and the CLI both depend on Metro. This creates some friction when any of the packages releases a new version.

Currently, we manage this through direct communication and synchronized releases, but there is space for improvement.

In this session, we reconsidered the **dependencies** between React Native, Metro, and the CLI. We uncovered how some design decisions during the [“Lean Core” effort](https://github.com/react-native-community/discussions-and-proposals/issues/6), when we extracted the CLI from React Native, made these two projects codependent with some functionalities duplicated among efforts. The decisions back then made sense and allowed the CLI team to iterate faster than ever.

It was about time to revisit them and take the experience of both teams to figure out the way through. As a result, the Metro team will take over the [`@react-native-community/cli-plugin-metro`](https://github.com/react-native-community/cli/tree/main/packages/cli-plugin-metro) development, temporarily moving it back to the core of React Native, and then most likely to the Metro monorepo.

![](/blog/assets/core-contributor-summit-2022.jpg)

The biggest takeaway, apart from three hours of drawing dependencies between the packages on the whiteboard, was for the CLI and Metro team to exchange their issues, experiences, and plans, resulting in a better understanding of each other.

We wouldn’t be able to achieve this level of cooperation without actually meeting each other.

---

We’re still impressed by how spending several hours together for a couple of days resulted in so much knowledge-sharing and cross-pollination of ideas. During this summit, we planted the seeds for initiatives that will help us improve and re-shape the React Native ecosystem.

We want to say thank you again to [Callstack](https://www.callstack.com/) for hosting us and to all the participants for joining us at the Core Contributor Summit 2022.

If you’re interested in joining the development of React Native, make sure you join our open initiatives and read the [contribution guide](https://reactnative.dev/contributing/overview) we have on our website. We hope to meet you in person as well in the future!