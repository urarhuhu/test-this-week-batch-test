---
id: network
title: Networking
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

許多行動應用程式需要從遠端URL加載資源。您可能需要向REST API發送POST請求，或是從其他伺服器獲取靜態內容片段。

## 使用Fetch

React Native提供[Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)來滿足網路請求需求。如果您曾使用過`XMLHttpRequest`或其他網路API，Fetch會讓您感到熟悉。更多資訊可參考MDN的[使用Fetch指南](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch)。

### 發送請求

要從任意URL獲取內容，只需將URL傳遞給fetch：

```jsx
fetch('https://mywebsite.com/mydata.json');
```

Fetch還接受可選的第二個參數，允許您自訂HTTP請求。您可以指定額外的標頭或發送POST請求：

```jsx
fetch('https://mywebsite.com/endpoint/', {
  method: 'POST',
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    firstParam: 'yourValue',
    secondParam: 'yourOtherValue',
  }),
});
```

完整屬性列表請參閱[Fetch Request文檔](https://developer.mozilla.org/en-US/docs/Web/API/Request)。

### 處理回應

上述範例展示了如何發送請求。多數情況下，您會需要對回應進行處理。

網路請求本質上是異步操作。Fetch方法會返回一個[Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)，讓異步程式碼的編寫更加直觀：

```jsx
const getMoviesFromApi = () => {
  return fetch('https://reactnative.dev/movies.json')
    .then(response => response.json())
    .then(json => {
      return json.movies;
    })
    .catch(error => {
      console.error(error);
    });
};
```

您也可以在React Native應用中使用`async`/`await`語法：

```jsx
const getMoviesFromApiAsync = async () => {
  try {
    const response = await fetch(
      'https://reactnative.dev/movies.json',
    );
    const json = await response.json();
    return json.movies;
  } catch (error) {
    console.error(error);
  }
};
```

請務必捕獲`fetch`可能拋出的錯誤，否則這些錯誤會被靜默忽略。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Fetch%20Example
import React, { useEffect, useState } from 'react';
import { ActivityIndicator, FlatList, Text, View } from 'react-native';

export default App = () => {
  const [isLoading, setLoading] = useState(true);
  const [data, setData] = useState([]);

  const getMovies = async () => {
     try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      setData(json.movies);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  }

  useEffect(() => {
    getMovies();
  }, []);

  return (
    <View style={{ flex: 1, padding: 24 }}>
      {isLoading ? <ActivityIndicator/> : (
        <FlatList
          data={data}
          keyExtractor={({ id }, index) => id}
          renderItem={({ item }) => (
            <Text>{item.title}, {item.releaseYear}</Text>
          )}
        />
      )}
    </View>
  );
};
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Fetch%20Example
import React, { Component } from 'react';
import { ActivityIndicator, FlatList, Text, View } from 'react-native';

export default class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      data: [],
      isLoading: true
    };
  }

  async getMovies() {
    try {
      const response = await fetch('https://reactnative.dev/movies.json');
      const json = await response.json();
      this.setState({ data: json.movies });
    } catch (error) {
      console.log(error);
    } finally {
      this.setState({ isLoading: false });
    }
  }

  componentDidMount() {
    this.getMovies();
  }

  render() {
    const { data, isLoading } = this.state;

    return (
      <View style={{ flex: 1, padding: 24 }}>
        {isLoading ? <ActivityIndicator/> : (
          <FlatList
            data={data}
            keyExtractor={({ id }, index) => id}
            renderItem={({ item }) => (
              <Text>{item.title}, {item.releaseYear}</Text>
            )}
          />
        )}
      </View>
    );
  }
};
```

</TabItem>
</Tabs>

> 預設情況下，iOS會封鎖所有未使用[SSL](https://hosting.review/web-hosting-glossary/#12)加密的請求。如果您需要從明文字URL（以`http`開頭）獲取資料，首先需要[添加App Transport Security例外](integration-with-existing-apps.md#test-your-integration)。若事先知道需要存取的網域，僅為這些網域添加例外會更安全；若網域在運行時才確定，您可以[完全禁用ATS](publishing-to-app-store.md#1-enable-app-transport-security)。但請注意，自2017年1月起，[Apple的App Store審核會要求提供禁用ATS的合理理由](https://forums.developer.apple.com/thread/48979)。詳見[Apple文檔](https://developer.apple.com/library/ios/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW33)。

> 在Android上，從API Level 28開始，預設也會封鎖明文字流量。可透過在應用清單文件中設置[`android:usesCleartextTraffic`](https://developer.android.com/guide/topics/manifest/application-element#usesCleartextTraffic)來覆寫此行為。

## 使用其他網路庫

[XMLHttpRequest API](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)已內建於React Native中。這意味著您可以使用依賴它的第三方庫，如[frisbee](https://github.com/niftylettuce/frisbee)或[axios](https://github.com/mzabriskie/axios)，也可以直接使用XMLHttpRequest API。

```jsx
var request = new XMLHttpRequest();
request.onreadystatechange = e => {
  if (request.readyState !== 4) {
    return;
  }

  if (request.status === 200) {
    console.log('success', request.responseText);
  } else {
    console.warn('error');
  }
};

request.open('GET', 'https://mywebsite.com/endpoint/');
request.send();
```

> XMLHttpRequest的安全模型與網頁不同，因為原生應用中沒有[CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing)的概念。

## WebSocket支援

React Native 也支援 [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)，這是一種透過單一 TCP 連線提供全雙工通訊通道的協定。

```jsx
var ws = new WebSocket('ws://host.com/path');

ws.onopen = () => {
  // connection opened
  ws.send('something'); // send a message
};

ws.onmessage = e => {
  // a message was received
  console.log(e.data);
};

ws.onerror = e => {
  // an error occurred
  console.log(e.message);
};

ws.onclose = e => {
  // connection closed
  console.log(e.code, e.reason);
};
```

## 已知的 `fetch` 與基於 cookie 驗證問題

以下選項目前在 `fetch` 中無法正常運作：

- `redirect:manual`
- `credentials:omit`

* 在 Android 上若有多個相同名稱的標頭，只有最後一個會生效。臨時解決方案可參考：https://github.com/facebook/react-native/issues/18837#issuecomment-398779994。
* 基於 cookie 的驗證目前不穩定。相關問題可參閱：https://github.com/facebook/react-native/issues/23185。
* 至少在 iOS 上，當透過 `302` 重新導向時，若存在 `Set-Cookie` 標頭，cookie 無法正確設定。由於無法手動處理重新導向，若重新導向是因 session 過期所致，可能導致無限請求的狀況。

## 在 iOS 上配置 NSURLSession

對於某些應用程式，可能需要為 iOS 上執行的 React Native 應用程式中的底層 `NSURLSession` 提供自訂的 `NSURLSessionConfiguration`。例如，可能需要為應用程式的所有網路請求設定自訂的使用者代理字串，或為 `NSURLSession` 提供暫時性的 `NSURLSessionConfiguration`。函式 `RCTSetCustomNSURLSessionConfigurationProvider` 允許進行此類自訂。請記得在呼叫 `RCTSetCustomNSURLSessionConfigurationProvider` 的檔案中加入以下導入：

```objectivec
#import <React/RCTHTTPRequestHandler.h>
```

`RCTSetCustomNSURLSessionConfigurationProvider` 應在應用程式生命週期的早期呼叫，以便 React 在需要時能立即使用，例如：

```objectivec
-(void)application:(__unused UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

  // set RCTSetCustomNSURLSessionConfigurationProvider
  RCTSetCustomNSURLSessionConfigurationProvider(^NSURLSessionConfiguration *{
     NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
     // configure the session
     return configuration;
  });

  // set up React
  _bridge = [[RCTBridge alloc] initWithDelegate:self launchOptions:launchOptions];
}
```