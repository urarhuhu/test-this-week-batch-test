---
id: custom-webview-ios
title: Custom WebView
---

雖然內建的 WebView 具備許多功能，但 React Native 無法涵蓋所有使用情境。不過，您可以透過原生程式碼擴展 WebView，而無需分叉 React Native 或複製現有的 WebView 程式碼。

進行此操作前，您應熟悉[原生 UI 元件](native-components-ios)的概念。同時也建議您了解[WebView 的原生程式碼](https://github.com/react-native-webview/react-native-webview/blob/master/apple/RNCWebViewManager.mm)，因為實作新功能時需以此為參考——儘管不需要深入理解。

## 原生程式碼

與常規原生元件相同，您需要一個視圖管理器和一個 WebView。

對於視圖，您需要建立 `RCTWebView` 的子類別。

```objc
// RCTCustomWebView.h
#import <React/RCTWebView.h>

@interface RCTCustomWebView : RCTWebView

@end

// RCTCustomWebView.m
#import "RCTCustomWebView.h"

@interface RCTCustomWebView ()

@end

@implementation RCTCustomWebView { }

@end
```

對於視圖管理器，您需要建立 `RCTWebViewManager` 的子類別。您仍需包含：

- 回傳自訂視圖的 `(UIView *)view`
- `RCT_EXPORT_MODULE()` 標記

```objc
// RCTCustomWebViewManager.h
#import <React/RCTWebViewManager.h>

@interface RCTCustomWebViewManager : RCTWebViewManager

@end

// RCTCustomWebViewManager.m
#import "RCTCustomWebViewManager.h"
#import "RCTCustomWebView.h"

@interface RCTCustomWebViewManager () <RCTWebViewDelegate>

@end

@implementation RCTCustomWebViewManager { }

RCT_EXPORT_MODULE()

- (UIView *)view
{
  RCTCustomWebView *webView = [RCTCustomWebView new];
  webView.delegate = self;
  return webView;
}

@end
```

### 新增事件與屬性

新增屬性和事件的方式與常規 UI 元件相同。對於屬性，您需在標頭檔中定義 `@property`；對於事件，則需在視圖的 `@interface` 中定義 `RCTDirectEventBlock`。

```objc
// RCTCustomWebView.h
@property (nonatomic, copy) NSString *finalUrl;

// RCTCustomWebView.m
@interface RCTCustomWebView ()

@property (nonatomic, copy) RCTDirectEventBlock onNavigationCompleted;

@end
```

接著在視圖管理器的 `@implementation` 中公開這些定義。

```objc
// RCTCustomWebViewManager.m
RCT_EXPORT_VIEW_PROPERTY(onNavigationCompleted, RCTDirectEventBlock)
RCT_EXPORT_VIEW_PROPERTY(finalUrl, NSString)
```

### 擴展現有事件

您應參考 React Native WebView 程式庫中的 [RCTWebView.m](https://github.com/react-native-webview/react-native-webview/blob/master/apple/RNCWebView.m)，了解現有處理程序的實作方式。您可以擴展此處的任何方法來提供額外功能。

預設情況下，RCTWebView 並未公開大多數方法。若需公開，您需要建立 [Objective C 類別擴展](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/CustomizingExistingClasses/CustomizingExistingClasses.html)，然後公開所有需要使用的的方法。

```objc
// RCTWebView+Custom.h
#import <React/RCTWebView.h>

@interface RCTWebView (Custom)
- (BOOL)webView:(__unused UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType;
- (NSMutableDictionary<NSString *, id> *)baseEvent;
@end
```

公開這些方法後，您即可在自訂 WebView 類別中引用它們。

```objc
// RCTCustomWebView.m

// Remember to import the category file.
#import "RCTWebView+Custom.h"

- (BOOL)webView:(__unused UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request
 navigationType:(UIWebViewNavigationType)navigationType
{
  BOOL allowed = [super webView:webView shouldStartLoadWithRequest:request navigationType:navigationType];

  if (allowed) {
    NSString* url = request.URL.absoluteString;
    if (url && [url isEqualToString:_finalUrl]) {
      if (_onNavigationCompleted) {
        NSMutableDictionary<NSString *, id> *event = [self baseEvent];
        _onNavigationCompleted(event);
      }
    }
  }

  return allowed;
}
```

## JavaScript 介面

要使用自訂 WebView，您需要為其建立一個類別。您的類別必須：

- 從 `WebView.propTypes` 匯出所有屬性類型
- 回傳一個 `WebView` 元件，並將 `nativeConfig.component` 屬性設為您的原生元件（如下所示）

與常規自訂元件相同，您需使用 `requireNativeComponent` 來取得原生元件。但需額外傳入第三個參數 `WebView.extraNativeComponentConfig`，此參數包含僅供原生程式碼使用的屬性類型。

```jsx
import React, {Component, PropTypes} from 'react';
import {
  WebView,
  requireNativeComponent,
  NativeModules,
} from 'react-native';
const {CustomWebViewManager} = NativeModules;

export default class CustomWebView extends Component {
  static propTypes = WebView.propTypes;

  render() {
    return (
      <WebView
        {...this.props}
        nativeConfig={{
          component: RCTCustomWebView,
          viewManager: CustomWebViewManager,
        }}
      />
    );
  }
}

const RCTCustomWebView = requireNativeComponent(
  'RCTCustomWebView',
  CustomWebView,
  WebView.extraNativeComponentConfig,
);
```

若要為原生元件新增自訂屬性，可在 WebView 上使用 `nativeConfig.props`。對於 iOS 平台，您還應如上述範例所示，透過 `nativeConfig.viewManager` 屬性設定自訂的 WebView 視圖管理器。

對於事件處理程序，必須始終將其設為函式。這意味著直接從 `this.props` 使用事件處理程序並不安全，因為使用者可能未提供。標準做法是在類別中建立事件處理程序，然後在 `this.props` 提供時調用該處理程序。

若不確定如何從 JavaScript 端實作某些功能，請參考 React Native 原始碼中的 [WebView.ios.tsx](https://github.com/react-native-webview/react-native-webview/blob/master/src/WebView.ios.tsx)。

```jsx
export default class CustomWebView extends Component {
  static propTypes = {
    ...WebView.propTypes,
    finalUrl: PropTypes.string,
    onNavigationCompleted: PropTypes.func,
  };

  static defaultProps = {
    finalUrl: 'about:blank',
  };

  _onNavigationCompleted = event => {
    const {onNavigationCompleted} = this.props;
    onNavigationCompleted && onNavigationCompleted(event);
  };

  render() {
    return (
      <WebView
        {...this.props}
        nativeConfig={{
          component: RCTCustomWebView,
          props: {
            finalUrl: this.props.finalUrl,
            onNavigationCompleted: this._onNavigationCompleted,
          },
          viewManager: CustomWebViewManager,
        }}
      />
    );
  }
}
```

與常規原生元件類似，您必須在元件中提供所有的 prop 類型，才能將它們轉發到原生元件。然而，如果您有一些僅在元件內部使用的 prop 類型，可以將它們添加到前面提到的第三個參數的 `nativeOnly` 屬性中。對於事件處理程序，您必須使用 `true` 值，而不是常規的 prop 類型。

例如，如果您想添加一個名為 `onScrollToBottom` 的內部事件處理程序，您會使用：

```jsx
const RCTCustomWebView = requireNativeComponent(
  'RCTCustomWebView',
  CustomWebView,
  {
    ...WebView.extraNativeComponentConfig,
    nativeOnly: {
      ...WebView.extraNativeComponentConfig.nativeOnly,
      onScrollToBottom: true,
    },
  },
);
```