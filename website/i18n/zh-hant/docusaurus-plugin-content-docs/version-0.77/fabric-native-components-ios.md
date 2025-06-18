---
id: fabric-native-components-ios
title: 'Fabric Native Components: iOS'
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

現在是時候撰寫一些 iOS 平台代碼來渲染網頁視圖。您需要遵循的步驟如下：

- 執行 Codegen
- 撰寫 `RCTWebView` 的代碼
- 在應用程式中註冊 `RCTWebView`

### 1. 執行 Codegen

您可以[手動執行](the-new-architecture/codegen-cli) Codegen，但更簡單的方法是使用您將用來演示該元件的應用程式來完成這項工作。

```bash
cd ios
bundle install
bundle exec pod install
```

重要的是，您將看到來自 Codegen 的日誌輸出，我們將在 Xcode 中使用這些輸出來構建我們的 WebView 原生元件。

:::warning
您應該謹慎將生成的代碼提交到您的儲存庫。生成的代碼針對每個 React Native 版本都是特定的。請使用 npm 的 [peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies) 來限制與 React Native 版本的兼容性。
:::

### 3. 撰寫 `RCTWebView`

我們需要通過完成以下 **5 個步驟** 來準備您的 iOS 專案（使用 Xcode）：

1. 打開由 CocoPods 生成的 Xcode Workspace：

```bash
cd ios
open Demo.xcworkspace
```

<img class="half-size" alt="Open Xcode Workspace" src="/docs/assets/fabric-native-components/1.webp" />

2. 右鍵點擊應用程式並選擇 <code>New Group</code>，將新群組命名為 `WebView`。

<img class="half-size" alt="Right click on app and select New Group" src="/docs/assets/fabric-native-components/2.webp" />

3. 在 `WebView` 群組中，創建 <code>New</code>→<code>File from Template</code>。

<img class="half-size" alt="Create a new file using the Cocoa Touch Classs template" src="/docs/assets/fabric-native-components/3.webp" />

4. 使用 <code>Objective-C File</code> 模板，並將其命名為 <code>RCTWebView</code>。

<img class="half-size" alt="Create an Objective-C RCTWebView class" src="/docs/assets/fabric-native-components/4.webp" />

5. 重複步驟 4 並創建一個名為 `RCTWebView.h` 的標頭檔。

6. 將 <code>RCTWebView.m</code> 重新命名為 <code>RCTWebView.mm</code>，使其成為 Objective-C++ 檔案。

```text title="Demo/ios"
Podfile
...
Demo
├── AppDelegate.swift
...
// highlight-start
├── RCTWebView.h
└── RCTWebView.mm
// highlight-end
```

創建標頭檔和實作檔案後，您可以開始實作它們。

這是 `RCTWebView.h` 檔案的代碼，它宣告了元件的介面。

```objc title="Demo/RCTWebView/RCTWebView.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface RCTWebView : RCTViewComponentView

// You would declare native methods you'd want to access from the view here

@end

NS_ASSUME_NONNULL_END
```

這個類別定義了一個 `RCTWebView`，它繼承自 `RCTViewComponentView` 類別。這是所有原生元件的基礎類別，由 React Native 提供。

實作檔案（`RCTWebView.mm`）的代碼如下：

```objc title="Demo/RCTWebView/RCTWebView.mm"
#import "RCTWebView.h"

#import <react/renderer/components/AppSpec/ComponentDescriptors.h>
#import <react/renderer/components/AppSpec/EventEmitters.h>
#import <react/renderer/components/AppSpec/Props.h>
#import <react/renderer/components/AppSpec/RCTComponentViewHelpers.h>
// highlight-next-line
#import <WebKit/WebKit.h>

using namespace facebook::react;

@interface RCTWebView () <RCTCustomWebViewViewProtocol, WKNavigationDelegate>
@end

@implementation RCTWebView {
  NSURL * _sourceURL;
  WKWebView * _webView;
}

-(instancetype)init
{
  if(self = [super init]) {
    // highlight-start
    _webView = [WKWebView new];
    _webView.navigationDelegate = self;
    [self addSubview:_webView];
    // highlight-end
  }
  return self;
}

- (void)updateProps:(Props::Shared const &)props oldProps:(Props::Shared const &)oldProps
{
  const auto &oldViewProps = *std::static_pointer_cast<CustomWebViewProps const>(_props);
  const auto &newViewProps = *std::static_pointer_cast<CustomWebViewProps const>(props);

  // Handle your props here
  if (oldViewProps.sourceURL != newViewProps.sourceURL) {
    NSString *urlString = [NSString stringWithCString:newViewProps.sourceURL.c_str() encoding:NSUTF8StringEncoding];
    _sourceURL = [NSURL URLWithString:urlString];
    // highlight-start
    if ([self urlIsValid:newViewProps.sourceURL]) {
      [_webView loadRequest:[NSURLRequest requestWithURL:_sourceURL]];
    }
    // highlight-end
  }

  [super updateProps:props oldProps:oldProps];
}

-(void)layoutSubviews
{
  [super layoutSubviews];
  _webView.frame = self.bounds;

}

#pragma mark - WKNavigationDelegate

// highlight-start
-(void)webView:(WKWebView *)webView didFinishNavigation:(WKNavigation *)navigation
{
  CustomWebViewEventEmitter::OnScriptLoaded result = CustomWebViewEventEmitter::OnScriptLoaded{CustomWebViewEventEmitter::OnScriptLoadedResult::Success};
  self.eventEmitter.onScriptLoaded(result);
}

- (BOOL)urlIsValid:(std::string)propString
{
  if (propString.length() > 0 && !_sourceURL) {
    CustomWebViewEventEmitter::OnScriptLoaded result = CustomWebViewEventEmitter::OnScriptLoaded{CustomWebViewEventEmitter::OnScriptLoadedResult::Error};

    self.eventEmitter.onScriptLoaded(result);
    return NO;
  }
  return YES;
}

// Event emitter convenience method
- (const CustomWebViewEventEmitter &)eventEmitter
{
  return static_cast<const CustomWebViewEventEmitter &>(*_eventEmitter);
}
// highlight-end

+ (ComponentDescriptorProvider)componentDescriptorProvider
{
  return concreteComponentDescriptorProvider<CustomWebViewComponentDescriptor>();
}

@end
```

這段代碼是用 Objective-C++ 撰寫的，包含以下細節：

- `@interface` 實作了兩個協議：
  - `RCTCustomWebViewViewProtocol`，由 Codegen 生成；
  - `WKNavigationDelegate`，由 WebKit 框架提供，用於處理網頁視圖的導航事件；
- `init` 方法，用於實例化 `WKWebView`，將其添加到子視圖中並設置 `navigationDelegate`；
- `updateProps` 方法，當元件的 props 發生變化時由 React Native 調用；
- `layoutSubviews` 方法，描述自訂視圖的佈局方式；
- `webView:didFinishNavigation:` 方法，讓您處理 `WKWebView` 完成頁面加載後的操作；
- `urlIsValid:(std::string)propString` 方法，檢查作為 prop 接收的 URL 是否有效；
- `eventEmitter` 方法，這是一個實用工具，用於檢索強型別的 `eventEmitter` 實例；
- `componentDescriptorProvider`，返回由 Codegen 生成的 `ComponentDescriptor`；

#### 添加 WebKit 框架

:::note
此步驟僅在創建 Web 視圖時需要。iOS 上的 Web 組件需要連結 Apple 提供的 WebKit 框架。若您的組件無需存取網路相關功能，可跳過此步驟。
:::

Web 視圖需透過 Xcode 和裝置內建的框架（WebKit）存取特定功能。您可在原生代碼中看到 `RCTWebView.mm` 檔案內加入的 `#import <WebKit/WebKit.h>` 語句。

連結 WebKit 框架至應用程式的步驟如下：

1. 在 Xcode 中點選您的專案
2. 選擇應用程式目標
3. 選擇 General 標籤頁
4. 向下滾動至「Frameworks, Libraries, and Embedded Contents」區段，點擊 `+` 按鈕

<img class="half-size" alt="Add webkit framework to your app 1" src="/docs/assets/AddWebKitFramework1.png" />

5. 在搜尋欄中過濾 WebKit
6. 選擇 WebKit 框架
7. 點擊 Add 按鈕

<img class="half-size" alt="Add webkit framework to your app 2" src="/docs/assets/AddWebKitFramework2.png" />