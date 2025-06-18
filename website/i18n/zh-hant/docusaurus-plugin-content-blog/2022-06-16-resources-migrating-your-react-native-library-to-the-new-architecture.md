---
title: Helping migrate React Native libraries to the New Architecture
authors: [cipolleschi]
tags: [announcement]
date: 2022-06-16
---

**tl; dr**: We are working on improving the resources supporting the React Native New Architecture. We have already released a repository to help migrate your app ([RNNewArchitectureApp](https://github.com/react-native-community/RNNewArchitectureApp)) and one for your libraries ([RNNewArchitectureLibraries](https://github.com/react-native-community/RNNewArchitectureLibraries)). We are also revamping the [New Architecture guide](https://github.com/facebook/react-native-website/pull/3037) on the Website and we created a [GitHub Working Group](https://github.com/reactwg/react-native-new-architecture/discussions) to answer questions related to the New Architecture.

<!--truncate-->

## Introduction

In this post we are sharing an update on tools and resources to help you migrate your **Native Modules** and **Native Components** to their **New Architecture** equivalents, **TurboModule** and **Fabric Components**.

React Native users leverage vast number of open source libraries for building apps. For a complete and consistent ecosystem, it is necessary that these libraries migrate such that everyone can benefit from the unlocked capabilities and performance improvements of the New Architecture.

Here is what we’re working on to support library developers in migrating to the New Architecture:

- **Documentation:** We are expanding the [New Architecture guide](https://github.com/facebook/react-native-website/pull/3037) on the website to cover more concepts of the New Architecture and how to develop your components.
- **Example Migrations:** We’ve set up two repositories to demonstrate how to migrate a React Native app to the New Architecture ([RNNewArchitectureApp](https://github.com/react-native-community/RNNewArchitectureApp)) and how to create a **Fabric Component** and a **TurboModule** that work with both architectures ([RNNewArchitectureLibraries](https://github.com/react-native-community/RNNewArchitectureLibraries)).
- **Support:** Earlier this year, we created a [GitHub Working Group](https://github.com/reactwg/react-native-new-architecture/discussions) dedicated to discussion and questions around the New Architecture.

In this post, we will dig deeper into these resources and explain in more detail how you can use them most efficiently. Finally, we will provide a snapshot of the current migration state for the most used React Native libraries.

### Documentation

In the past 6 months, we’ve added a [guide on adopting the New Architecture](https://github.com/reactwg/react-native-new-architecture#guides) and an [architecture deep-dive](/architecture/overview) on Fabric. We plan to expand this to include more guides and documentation around creating TurboModules, understanding CodeGen, and more. We plan to have updates to share by the 0.70 release.

Currently, the **New Architecture** guide covers how to [migrate your app](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/enable-apps.md) and [your libraries](https://github.com/reactwg/react-native-new-architecture/blob/main/docs/enable-libraries-prerequisites.md) to support the New Architecture properly.

If you are interested in the evolution of this guide, or have feedback, you can follow along on [this](https://github.com/facebook/react-native-website/pull/3037) pull request.

### Example Migrations

For developers who may want to follow along in code, we’ve prepared two example repositories.

#### RNNewArchitectureApp

[This repo](https://github.com/react-native-community/RNNewArchitectureApp) was created to demonstrate how to migrate an app, the native modules and the native components from the legacy architecture on the React Native version 0.67 to the New Architecture and the most recent version of React Native. Each commit corresponds to an isolated migration step.

<figure>
    <img src="/blog/assets/new-arch-example-steps-to-migrate-an-app.png" alt="Example steps to migrate an app" />
    <figcaption>Commit list for a migration in the RNNewArchitectureApp repository</figcaption>
</figure>

The repo is organized as follows:

- A **main** branch has no code but a README.md which advertises other branches.
- Several migration branches which show a migration from a specific version of RN to another.

部分遷移分支還包含一個 **RUN.md** 文件，以更人性化的方式描述每個提交中所應用的具體步驟。

我們計劃讓此範例與最新的穩定版本保持同步，針對每個即將發布的 React Native 次要版本新增遷移步驟。若您發現任何步驟存在問題，請在該存儲庫中提交 issue。此做法將持續至我們合理認為多數 React Native 用戶已完成新架構遷移為止。

#### RNNewArchitectureLibraries

類似地，[此存儲庫](https://github.com/react-native-community/RNNewArchitectureLibraries)提供了逐步指南，說明如何建立 **TurboModule** 和 **Fabric 元件**，特別著重於確保新架構與舊架構之間的向後兼容性。

該存儲庫的組織方式與前一個類似：

- **main** 分支不含程式碼，僅包含 README.md 文件以介紹其他分支。
- 其他分支則展示如何開發 **TurboModules** 和 **Fabric 元件**。

我們計劃讓此範例隨 React Native 的新版本更新，尤其是影響函式庫開發的版本，並新增更多關於使用進階功能（例如：實作指令、事件發射器、自訂狀態）的範例。若您發現錯誤，請在範例存儲庫中提交 issue。

### 支援

我們建立了一個專屬的[工作小組](https://github.com/reactwg/react-native-new-architecture)，讓社群能提問並獲取新架構的最新資訊。若您是函式庫維護者，此資源能協助您找到問題的解答，並讓我們了解您的需求。欲加入，請遵循[這些指示](https://github.com/reactwg/react-native-new-architecture#how-to-join-the-working-group)。歡迎所有人參與。

工作小組分為以下幾個部分：

- [公告](https://github.com/reactwg/react-native-new-architecture/discussions/categories/announcements)：分享 RN 新架構推展的重要里程碑與更新
- [深度探討](https://github.com/reactwg/react-native-new-architecture/discussions/categories/deep-dive)：討論技術性主題與深度解析
- [文件](https://github.com/reactwg/react-native-new-architecture/discussions/categories/documentation)：討論新架構文件與遷移資料
- [函式庫](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)：討論第三方函式庫及其遷移至新架構的過程
- [問答](https://github.com/reactwg/react-native-new-architecture/discussions/categories/q-a)：向社群尋求新架構相關問題的協助
- [版本發布](https://github.com/reactwg/react-native-new-architecture/discussions/categories/releases)：討論版本特定的錯誤與建置問題

有效使用此小組的方法：

- **確保您的函式庫列於[函式庫](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)區段中**。這有助於我們分享您的函式庫遷移狀態，並了解函式庫作者面臨的困難以提供更好的支援。
- **若遇到阻礙需協助，請利用[問答區](https://github.com/reactwg/react-native-new-architecture/discussions/categories/q-a)**。我們的團隊與社群專家會盡力提供支援。
- **關注其他區段中可能影響您的主題**。新版本可能正好包含您需要的 API。您可透過 GitHub 訂閱特定討論。

我們計劃持續支援此小組，直至**新架構**預設啟用且所有主要函式庫完成遷移為止。

### 熱門函式庫的遷移狀態

函式庫維護者已透過[工作小組](https://github.com/reactwg/react-native-new-architecture/discussions/categories/libraries)與我們分享其遷移進度，以下為簡要概述：

- [react-native-gesture-handler](https://github.com/reactwg/react-native-new-architecture/discussions/15): ✅ Migrated
- [react-native-navigation](https://github.com/reactwg/react-native-new-architecture/discussions/17): 🏃‍♂️ Ongoing
- [react-native-pager-view](https://github.com/reactwg/react-native-new-architecture/discussions/16): 🏃‍♂️ Ongoing
- [react-native-reanimated](https://github.com/reactwg/react-native-new-architecture/discussions/14): ✅ Migrated. In the process of testing and profiling for performances
- [react-native-screens](https://github.com/reactwg/react-native-new-architecture/discussions/13): 🏃‍♂️ Ongoing
- [react-native-slider](https://github.com/reactwg/react-native-new-architecture/discussions/38): 🎬 Started
- [react-native-template-new-architecture](https://github.com/reactwg/react-native-new-architecture/discussions/21): ✅ Migrated. Gradually adopting/testing more companion Libraries
- [react-native-template-typescript](https://github.com/reactwg/react-native-new-architecture/discussions/22): ✅ Migrated
- [react-native-webview](https://github.com/reactwg/react-native-new-architecture/discussions/19): 🎬 Started

## Next Steps

We are invested in supporting the React Native community’s adoption of the New Architecture. Concretely, we will continue to:

- Offer best-effort support in the **Working Group.**
- Provide more examples about how to achieve amazing results with the New Architecture in the **RNNewArchitecture** repositories.
- Provide clear and up-to-date documentation on the **New Architecture**.
- Track the migration status of essential React Native libraries in the **Working Group**.
- Simplify the migration path for developers

In addition, React Native 0.69 will ship with improved devX for app and library developers for New Architecture adoption. You can find more information about the 0.69.0 release [here](https://github.com/reactwg/react-native-releases/discussions/21).

We are excited about what we will build together with the **New Architecture**!