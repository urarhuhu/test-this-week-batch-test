---
title: Making React Native apps accessible
author: Georgiy Kassabli
authorTitle: Software Engineer at Facebook
authorURL: 'https://www.facebook.com/georgiy.kassabli'
authorImageURL: 'https://scontent-sea1-1.xx.fbcdn.net/v/t1.0-1/c0.0.160.160/p160x160/1978838_795592927136196_1205041943_n.jpg?_nc_log=1&oh=d7a500fdece1250955a4d27b0a80fee2&oe=59E8165A'
hero: '/blog/assets/blue-hero.png'
tags: [engineering]
---

隨著 React 在網頁端和 React Native 在行動端的推出，我們為開發者提供了一個建構產品的新前端框架。打造穩健產品的關鍵環節之一，是確保包含視障或其他障礙人士在內的所有使用者都能使用。React 和 React Native 的無障礙功能 API（Accessibility API）能讓您開發的 React 應用程式，兼容盲人與視障者使用的螢幕閱讀器等輔助技術。

本文將聚焦於 React Native 應用程式。我們將 React 無障礙 API 的設計理念與 Android、iOS API 保持一致。若您曾為 Android、iOS 或網頁開發過無障礙應用程式，應該會對 React AX API 的架構和術語感到熟悉。例如，您可以將 UI 元素標記為 _accessible_（從而讓輔助技術偵測），並使用 _accessibilityLabel_ 為元素提供文字描述：

```
<View accessible={true} accessibilityLabel=”This is simple view”>
```

讓我們透過 Facebook 自家的 React 產品 **廣告管理員應用程式**，深入探討 React AX API 的進階應用實例：

<footer>
  <a
    href="https://code.facebook.com/posts/435862739941212/making-react-native-apps-accessible/"
    className="btn">Read more</a>
</footer>

> 此為節錄內容，完整文章請見 [Facebook Code](https://code.facebook.com/posts/435862739941212/making-react-native-apps-accessible/)。