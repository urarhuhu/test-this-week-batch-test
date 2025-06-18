---
id: direct-manipulation
title: Direct Manipulation
---

import NativeDeprecated from './the-new-architecture/\_markdown_native_deprecation.mdx'

<NativeDeprecated />

有時需要直接修改元件而不透過 state/props 觸發整個子樹重新渲染。例如在瀏覽器中使用 React 時，您可能需要直接操作 DOM 節點，這在行動應用程式的視圖中同樣適用。`setNativeProps` 就是 React Native 中直接設定 DOM 節點屬性的等效方法。

:::caution
請在頻繁重新渲染導致效能瓶頸時使用 `setNativeProps`！

直接操作不應是您頻繁使用的工具。通常僅在創建連續動畫以避免渲染元件層級和協調多個視圖的開銷時使用。
`setNativeProps` 是命令式的，它將狀態儲存在原生層（DOM、UIView 等）而非 React 元件中，這會使您的程式碼更難推斷。

在使用前，請先嘗試用 `setState` 和 [`shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#shouldcomponentupdate-in-action) 解決問題。
:::

## 在 TouchableOpacity 中使用 setNativeProps

[TouchableOpacity](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Components/Touchable/TouchableOpacity.js) 內部使用 `setNativeProps` 來更新其子元件的不透明度：

```jsx
const viewRef = useRef();
const setOpacityTo = useCallback(value => {
  // Redacted: animation related code
  viewRef.current.setNativeProps({
    opacity: value,
  });
}, []);
```

這使我們能編寫以下程式碼，並確知子元件會根據點擊更新不透明度，而無需子元件知曉此邏輯或修改其實作：

```jsx
<TouchableOpacity onPress={handlePress}>
  <View>
    <Text>Press me!</Text>
  </View>
</TouchableOpacity>
```

假設 `setNativeProps` 不可用，我們可能會將不透明度值儲存在 state 中，並在 `onPress` 觸發時更新該值：

```jsx
const [buttonOpacity, setButtonOpacity] = useState(1);
return (
  <TouchableOpacity
    onPressIn={() => setButtonOpacity(0.5)}
    onPressOut={() => setButtonOpacity(1)}>
    <View style={{opacity: buttonOpacity}}>
      <Text>Press me!</Text>
    </View>
  </TouchableOpacity>
);
```

與原始範例相比，這種做法計算量較大——即使視圖及其子元件的其他屬性未改變，React 仍需在每次不透明度變化時重新渲染元件層級。通常這種開銷無需擔心，但在執行連續動畫和響應手勢時，謹慎優化元件可提高動畫的流暢度。

若查看 [NativeMethodsMixin](https://github.com/facebook/react-native/blob/0.70-stable/Libraries/Renderer/implementations/ReactNativeRenderer-prod.js) 中 `setNativeProps` 的實作，會發現它其實是 `RCTUIManager.updateView` 的封裝——這與重新渲染產生的函數調用完全相同，參見 [ReactNativeBaseComponent 中的 receiveComponent](https://github.com/facebook/react-native/blob/fb2ec1ea47c53c2e7b873acb1cb46192ac74274e/Libraries/Renderer/oss/ReactNativeRenderer-prod.js#L5793-L5813)。

## 複合元件與 setNativeProps

複合元件沒有對應的原生視圖，因此無法對其調用 `setNativeProps`。參考以下範例：

```SnackPlayer name=setNativeProps%20with%20Composite%20Components
import React from "react";
import { Text, TouchableOpacity, View } from "react-native";

const MyButton = (props) => (
  <View style={{ marginTop: 50 }}>
    <Text>{props.label}</Text>
  </View>
);

export default App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);
```

運行時會立即看到錯誤：`Touchable 子元件必須是原生元件或將 setNativeProps 轉發給原生元件`。這是因為 `MyButton` 並非直接由可設置不透明度的原生視圖支持。可以這樣理解：如果用 `createReactClass` 定義元件，您不會期望直接設置 style prop 就能生效——除非包裝了原生元件，否則需要將 style prop 傳遞給子元件。同理，我們需要將 `setNativeProps` 轉發給原生支持的子元件。

#### 將 setNativeProps 轉發給子元件

由於 `setNativeProps` 方法存在於任何 `View` 元件的 ref 上，只需將自訂元件的 ref 轉發給其渲染的某個 `<View />` 元件即可。這意味著對自訂元件調用 `setNativeProps` 的效果，等同於直接對包裝的 `View` 元件調用該方法。

```SnackPlayer name=Forwarding%20setNativeProps
import React from "react";
import { Text, TouchableOpacity, View } from "react-native";

const MyButton = React.forwardRef((props, ref) => (
  <View {...props} ref={ref} style={{ marginTop: 50 }}>
    <Text>{props.label}</Text>
  </View>
));

export default App = () => (
  <TouchableOpacity>
    <MyButton label="Press me!" />
  </TouchableOpacity>
);
```

現在你可以在 `TouchableOpacity` 中使用 `MyButton` 了！

你可能注意到我們使用 `{...props}` 將所有屬性傳遞給子視圖。這是因為 `TouchableOpacity` 實際上是一個複合元件，除了依賴子元件的 `setNativeProps` 外，還需要子元件處理觸控操作。為此，它會傳遞[各種屬性](view.md#onmoveshouldsetresponder)來回調 `TouchableOpacity` 元件。相比之下，`TouchableHighlight` 由原生視圖支持，僅需要我們實現 `setNativeProps`。

## 使用 setNativeProps 編輯 TextInput 值

`setNativeProps` 另一個常見用途是編輯 TextInput 的值。當 `bufferDelay` 較低且用戶輸入速度極快時，TextInput 的 `controlled` 屬性有時會丟失字符。部分開發者傾向完全跳過此屬性，改為在必要時直接使用 `setNativeProps` 操作 TextInput 值。例如，以下代碼展示點擊按鈕時編輯輸入框：

```SnackPlayer name=Clear%20text
import React from "react";
import { useCallback, useRef } from "react";
import { StyleSheet, TextInput, Text, TouchableOpacity, View } from "react-native";

const App = () => {
  const inputRef = useRef();
  const editText = useCallback(() => {
    inputRef.current.setNativeProps({ text: "Edited Text" });
  }, []);

  return (
    <View style={styles.container}>
      <TextInput ref={inputRef} style={styles.input} />
      <TouchableOpacity onPress={editText}>
        <Text>Edit text</Text>
      </TouchableOpacity>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  input: {
    height: 50,
    width: 200,
    marginHorizontal: 20,
    borderWidth: 1,
    borderColor: "#ccc",
  },
});

export default App;
```

你可以使用 [`clear`](textinput#clear) 方法清除 `TextInput`，該方法採用相同方式清除當前輸入文本。

## 避免與渲染函數衝突

若你更新的屬性同時由渲染函數管理，可能導致不可預測的錯誤，因為每當元件重新渲染且該屬性變化時，先前透過 `setNativeProps` 設定的值會被完全忽略並覆蓋。

## setNativeProps 與 shouldComponentUpdate

透過[智能應用 `shouldComponentUpdate`](https://reactjs.org/docs/optimizing-performance.html#avoid-reconciliation)，可避免協調未變更元件子樹的額外開銷，甚至可能使 `setState` 的性能足夠替代 `setNativeProps`。

## 其他原生方法

上述方法適用於 React Native 提供的大多數預設元件。但需注意，這些方法_不適用_於非直接由原生視圖支持的複合元件，這通常包含你在應用中自訂的多數元件。

### measure(callback)

測量指定視圖在屏幕上的位置、寬度和高度，並通過異步回調返回結果。成功時回調參數包括：

- x
- y
- width
- height
- pageX
- pageY

注意這些測量值需等待原生端渲染完成後才能取得。若需盡快獲取測量值且不需要 `pageX` 和 `pageY`，建議改用 [`onLayout`](view.md#onlayout) 屬性。

此外，`measure()` 返回的寬高是元件在視窗中的尺寸。若需要元件的實際尺寸，建議改用 [`onLayout`](view.md#onlayout) 屬性。

### measureInWindow(callback)

測量指定視圖在視窗中的絕對位置，並通過異步回調返回結果。若 React 根視圖嵌入其他原生視圖中，此方法將提供絕對座標。成功時回調參數包括：

- x
- y
- width
- height

### measureLayout(relativeToNativeComponentRef, onSuccess, onFail)

類似 `measure()`，但測量結果是相對於祖先視圖（透過 `relativeToNativeComponentRef` 引用指定）的座標系。返回的座標為相對於祖先視圖原點 `x`, `y` 的相對值。

:::note
此方法也可使用 `relativeToNativeNode` 處理程序（替代引用）調用，但此變體已棄用。
:::

```SnackPlayer name=measureLayout%20example&supportedPlatforms=android,ios
import React, { useEffect, useRef, useState } from "react";
import { Text, View, StyleSheet } from "react-native";

const App = () => {
  const textContainerRef = useRef(null);
  const textRef = useRef(null);
  const [measure, setMeasure] = useState(null);

  useEffect(() => {
    if (textRef.current && textContainerRef.current) {
      textRef.current.measureLayout(
        textContainerRef.current,
        (left, top, width, height) => {
          setMeasure({ left, top, width, height });
        }
      );
    }
  }, [measure]);

  return (
    <View style={styles.container}>
      <View
        ref={textContainerRef}
        style={styles.textContainer}
      >
        <Text ref={textRef}>
          Where am I? (relative to the text container)
        </Text>
      </View>
      <Text style={styles.measure}>
        {JSON.stringify(measure)}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
  },
  textContainer: {
    backgroundColor: "#61dafb",
    justifyContent: "center",
    alignItems: "center",
    padding: 12,
  },
  measure: {
    textAlign: "center",
    padding: 12,
  },
});

export default App;
```

### focus()

請求對指定的輸入框或視圖進行聚焦。觸發的具體行為將取決於平台和視圖類型。

### blur()

移除輸入框或視圖的聚焦狀態。此為 `focus()` 的反向操作。