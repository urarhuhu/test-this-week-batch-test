---
id: app-extensions
title: App Extensions
---

應用程式擴充功能讓您能在主應用程式之外提供自訂功能與內容。iOS上有不同類型的應用程式擴充功能，這些都在[《應用程式擴充功能程式設計指南》](https://developer.apple.com/library/content/documentation/General/Conceptual/ExtensibilityPG/index.html#//apple_ref/doc/uid/TP40014214-CH20-SW1)中有詳細說明。本指南將簡要介紹如何在iOS上善用應用程式擴充功能。

## 擴充功能中的記憶體使用

由於這些擴充功能是在常規應用程式沙盒之外載入的，很可能會同時載入多個應用程式擴充功能。如您所料，這些擴充功能有嚴格的記憶體使用限制。開發應用程式擴充功能時請務必注意這一點。我們強烈建議在實際裝置上測試您的應用程式，開發擴充功能時更是如此：開發者經常發現擴充功能在iOS模擬器上運作良好，但實際裝置上的使用者卻回報擴充功能無法載入。

我們強烈建議您觀看Conrad Kramer關於[《擴充功能中的記憶體使用》](https://www.youtube.com/watch?v=GqXMqn6MXrM)的演講，以深入了解此主題。

### 今日小工具

今日小工具的記憶體限制為16 MB。使用React Native實作的今日小工具可能會因記憶體使用量過高而運作不穩定。若您的今日小工具顯示「無法載入」訊息，即表示已超出記憶體限制：

![](/docs/assets/TodayWidgetUnableToLoad.jpg)

請務必在實際裝置上測試您的應用程式擴充功能，但請注意這可能仍不足夠，尤其是處理今日小工具時。除錯配置的建置版本較容易超出記憶體限制，而發布配置的建置版本不會立即失敗。我們強烈建議使用[Xcode的Instruments工具](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/InstrumentsUserGuide/index.html)分析實際記憶體使用情況，因為您的發布配置建置版本很可能已接近16 MB限制。在這種情況下，執行常見操作（例如從API獲取數據）可能會迅速超過16 MB限制。

若要實驗React Native今日小工具實作的限制，可嘗試擴充[react-native-today-widget](https://github.com/matejkriz/react-native-today-widget/)中的範例專案。

### 其他應用程式擴充功能

其他類型的應用程式擴充功能有比今日小工具更高的記憶體限制。例如，自訂鍵盤擴充功能限制為48 MB，分享擴充功能限制為120 MB。使用React Native實作這類應用程式擴充功能更為可行。[react-native-ios-share-extension](https://github.com/andrewsardone/react-native-ios-share-extension)即是一個概念驗證範例。