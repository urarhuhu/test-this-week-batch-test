---
id: fabric-native-components-ios
title: 'Fabric Native Components: iOS'
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

現在是時候編寫一些 iOS 平台代碼來渲染網頁視圖了。你需要遵循以下步驟：

- 執行 Codegen
- 編寫 `RCTWebView` 的代碼
- 在應用中註冊 `RCTWebView`

### 1. 執行 Codegen

你可以[手動執行](the-new-architecture/codegen-cli) Codegen，但更簡單的方法是使用你將用來演示該組件的應用來完成這一步驟。

```bash
cd ios
bundle install
bundle exec pods install
```

重要的是，你將看到 Codegen 的日誌輸出，我們將在 Xcode 中使用這些日誌來構建我們的 WebView 原生組件。

:::warning
你應該謹慎將生成的代碼提交到代碼庫中。生成的代碼針對每個 React Native 版本都是特定的。使用 npm 的 [peerDependencies](https://nodejs.org/en/blog/npm/peer-dependencies) 來限制與 React Native 版本的兼容性。
:::

### 3. 編寫 `RCTWebView`

我們需要通過完成以下 **5 個步驟** 來準備你的 iOS 項目（使用 Xcode）：

1. 打開由 CocoPods 生成的 Xcode Workspace：

```bash
cd ios
open Demo.xcworkspace
```

<img class="half-size" alt="Open Xcode Workspace" src="/docs/assets/fabric-native-components/1.webp" />

2. 右鍵點擊應用並選擇 <code>New Group</code>，將新組命名為 `WebView`。

<img class="half-size" alt="Right click on app and select New Group" src="/docs/assets/fabric-native-components/2.webp" />

3. 在 `WebView` 組中，創建 <code>New</code>→<code>File from Template</code>。

<img class="half-size" alt="Create a new file using the Cocoa Touch Classs template" src="/docs/assets/fabric-native-components/3.webp" />

4. 使用 <code>Objective-C File</code> 模板，並將其命名為 <code>RCTWebView</code>。

<img class="half-size" alt="Create an Objective-C RCTWebView class" src="/docs/assets/fabric-native-components/4.webp" />

5. 將 <code>RCTWebView.m</code> 重命名為 <code>RCTWebView.mm</code>，使其成為 Objective-C++ 文件

```text title="Demo/ios"
Podfile
...
Demo
├── AppDelegate.h
├── AppDelegate.mm
...
// highlight-start
├── RCTWebView.h
├── RCTWebView.mm
// highlight-end
└── main.m
```

創建頭文件和實現文件後，你可以開始實現它們。

這是 `RCTWebView.h` 文件的代碼，它聲明了組件的接口。

```objc title="Demo/RCTWebView/RTNWebView.h"
#import <React/RCTViewComponentView.h>
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface RCTWebView : RCTViewComponentView

// You would declare native methods you'd want to access from the view here

@end

NS_ASSUME_NONNULL_END
```

該類定義了一個 `RCTWebView`，它繼承自 `RCTViewComponentView` 類。這是所有原生組件的基類，由 React Native 提供。

實現文件 (`RCTWebView.mm`) 的代碼如下：

```objc title="Demo/RCTWebView/RCTWebView.mm"
#import "RCTWebView.h"

#import <react/renderer/components/AppSpecs/ComponentDescriptors.h>
#import <react/renderer/components/AppSpecs/EventEmitters.h>
#import <react/renderer/components/AppSpecs/Props.h>
#import <react/renderer/components/AppSpecs/RCTComponentViewHelpers.h>
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

Class<RCTComponentViewProtocol> WebViewCls(void)
{
  return RCTWebView.class;
}

@end
```

這段代碼是用 Objective-C++ 編寫的，包含以下細節：

- `@interface` 實現了兩個協議：
  - `RCTCustomWebViewViewProtocol`，由 Codegen 生成；
  - `WKNavigationDelegate`，由 WebKit 框架提供，用於處理網頁視圖的導航事件；
- `init` 方法，實例化 `WKWebView`，將其添加到子視圖中並設置 `navigationDelegate`；
- `updateProps` 方法，當組件的 props 發生變化時由 React Native 調用；
- `layoutSubviews` 方法，描述如何佈局自定義視圖；
- `webView:didFinishNavigation:` 方法，讓你可以處理 `WKWebView` 完成頁面加載後的操作；
- `urlIsValid:(std::string)propString` 方法，檢查作為 prop 接收的 URL 是否有效；
- `eventEmitter` 方法，這是一個工具方法，用於獲取強類型的 `eventEmitter` 實例；
- `componentDescriptorProvider`，返回由 Codegen 生成的 `ComponentDescriptor`；
- `WebViewCls`，這是一個輔助方法，用於在應用中註冊 `RCTWebView`。

#### AppDelegate.mm

最後，你可以在應用中註冊該組件。
更新 `AppDelegate.mm`，讓你的應用知道我們的自定義 WebView 組件：

```objc title="Demo/ios/Demo/AppDelegate.mm"
#import "AppDelegate.h"

#import <React/RCTBundleURLProvider.h>
#import <React/RCTBridge+Private.h>
// highlight-next-line
#import "RCTWebView.h"

@implementation AppDelegate
// ...
// highlight-start
- (NSDictionary<NSString *,Class<RCTComponentViewProtocol>> *)thirdPartyFabricComponents
{
  NSMutableDictionary * dictionary = [super thirdPartyFabricComponents].mutableCopy;
  dictionary[@"CustomWebView"] = [RCTWebView class];
  return dictionary;
}
// highlight-end

@end
```

這段程式碼覆寫了 `thirdPartyFabricComponents` 方法，透過取得來自其他來源（如第三方函式庫）的第三方元件字典的可變動副本。

接著在字典中加入一個條目，使用 Codegen 規格檔案中定義的名稱。如此一來，當 React 需要載入名為 `CustomWebView` 的元件時，React Native 就會實例化一個 `RCTWebView`。

最後，它會回傳這個新的字典。

#### 添加 WebKit 框架

:::note
此步驟僅因我們要建立網頁視圖而需要。iOS 上的網頁元件需連結至 Apple 提供的 WebKit 框架。若您的元件不需存取網頁相關功能，可跳過此步驟。
:::

網頁視圖需要存取 Apple 透過 Xcode 和裝置內建的框架之一 WebKit 提供的某些功能。
您可以在原生程式碼中看到 `RCTWebView.mm` 裡加入的 `#import <WebKit/WebKit.h>` 這行。

要在您的應用程式中連結 WebKit 框架，請遵循以下步驟：

1. 在 Xcode 中，點擊您的專案
2. 選擇應用程式目標
3. 選擇 General 標籤頁
4. 向下滾動直到找到「Frameworks, Libraries, and Embedded Contents」區段，然後點擊 `+` 按鈕

<img class="half-size" alt="Add webkit framework to your app 1" src="/docs/assets/AddWebKitFramework1.png" />

5. 在搜尋欄中，篩選 WebKit
6. 選擇 WebKit 框架
7. 點擊 Add。

<img class="half-size" alt="Add webkit framework to your app 2" src="/docs/assets/AddWebKitFramework2.png" />