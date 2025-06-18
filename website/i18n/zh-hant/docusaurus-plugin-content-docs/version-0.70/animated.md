---
id: animated
title: Animated
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

`Animated` 函式庫專為打造流暢、強大且易於建置維護的動畫而設計。其核心在於宣告式的輸入輸出關係定義、可配置的轉換過程，以及透過 `start`/`stop` 方法控制基於時間軸的動畫執行。

建立動畫的標準流程是：創建一個 `Animated.Value`，將其綁定至動畫元件的樣式屬性，再透過 `Animated.timing()` 驅動數值更新。

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

> Don't modify the animated value directly. You can use the [`useRef` Hook](https://reactjs.org/docs/hooks-reference.html#useref) to return a mutable ref object. This ref object's `current` property is initialized as the given argument and persists throughout the component lifecycle.

</TabItem>
<TabItem value="classical">

> Don't modify the animated value directly. It is usually stored as a [state variable](intro-react#state) in class components.

</TabItem>
</Tabs>

## 範例

以下範例展示了一個 `View` 元件如何根據動畫數值 `fadeAnim` 實現淡入淡出效果：

<Tabs groupId="syntax" queryString defaultValue={constants.defaultSyntax} values={constants.syntax}>
<TabItem value="functional">

```SnackPlayer name=Animated
import React, { useRef } from "react";
import { Animated, Text, View, StyleSheet, Button, SafeAreaView } from "react-native";

const App = () => {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  const fadeAnim = useRef(new Animated.Value(0)).current;

  const fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 5000
    }).start();
  };

  const fadeOut = () => {
    // Will change fadeAnim value to 0 in 3 seconds
    Animated.timing(fadeAnim, {
      toValue: 0,
      duration: 3000
    }).start();
  };

  return (
    <SafeAreaView style={styles.container}>
      <Animated.View
        style={[
          styles.fadingContainer,
          {
            // Bind opacity to animated value
            opacity: fadeAnim
          }
        ]}
      >
        <Text style={styles.fadingText}>Fading View!</Text>
      </Animated.View>
      <View style={styles.buttonRow}>
        <Button title="Fade In View" onPress={fadeIn} />
        <Button title="Fade Out View" onPress={fadeOut} />
      </View>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  fadingContainer: {
    padding: 20,
    backgroundColor: "powderblue"
  },
  fadingText: {
    fontSize: 28
  },
  buttonRow: {
    flexBasis: 100,
    justifyContent: "space-evenly",
    marginVertical: 16
  }
});

export default App;
```

</TabItem>
<TabItem value="classical">

```SnackPlayer name=Animated
import React, { Component } from "react";
import { Animated, Text, View, StyleSheet, Button, SafeAreaView } from "react-native";

class App extends Component {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  state = {
    fadeAnim: new Animated.Value(0)
  };

  fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(this.state.fadeAnim, {
      toValue: 1,
      duration: 5000
    }).start();
  };

  fadeOut = () => {
    // Will change fadeAnim value to 0 in 3 seconds
    Animated.timing(this.state.fadeAnim, {
      toValue: 0,
      duration: 3000
    }).start();
  };

  render() {
    return (
      <SafeAreaView style={styles.container}>
        <Animated.View
          style={[
            styles.fadingContainer,
            {
              // Bind opacity to animated value
              opacity: this.state.fadeAnim
            }
          ]}
        >
          <Text style={styles.fadingText}>Fading View!</Text>
        </Animated.View>
        <View style={styles.buttonRow}>
          <Button title="Fade In View" onPress={this.fadeIn} />
          <Button title="Fade Out View" onPress={this.fadeOut} />
        </View>
      </SafeAreaView>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center"
  },
  fadingContainer: {
    padding: 20,
    backgroundColor: "powderblue"
  },
  fadingText: {
    fontSize: 28
  },
  buttonRow: {
    flexBasis: 100,
    justifyContent: "space-evenly",
    marginVertical: 16
  }
});

export default App;
```

</TabItem>
</Tabs>

更多實際應用範例可參閱[動畫指南](animations#animated-api)。

## 概覽

`Animated` 支援兩種數值類型：

- [`Animated.Value()`](animated#value) 用於單一數值
- [`Animated.ValueXY()`](animated#valuexy) 用於向量

`Animated.Value` 可綁定至樣式屬性或其他參數，並支援插值運算。單一 `Animated.Value` 可驅動多個屬性。

### 動畫配置

`Animated` 提供三種動畫類型，每種都具備特定的動畫曲線來控制數值從初始狀態到最終狀態的變化過程：

- [`Animated.decay()`](animated#decay) 以初始速度開始並逐漸減速至停止
- [`Animated.spring()`](animated#spring) 提供基礎彈簧物理模型
- [`Animated.timing()`](animated#timing) 透過[緩動函式](easing)隨時間變化數值

多數情況下建議使用 `timing()`。預設採用對稱的 easeInOut 曲線，模擬物體加速至全速後再減速停止的過程。

### 動畫控制

透過呼叫動畫實例的 `start()` 啟動動畫，該方法接受一個在動畫完成時執行的回調函式。若動畫正常完成，回調將收到 `{finished: true}`；若因中途呼叫 `stop()` 而中止（例如被手勢或其他動畫打斷），則回傳 `{finished: false}`。

```jsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

### 使用原生驅動

啟用原生驅動後，動畫參數會預先傳送至原生端，由 UI 線程直接執行動畫，避免每幀都需跨橋接通訊。動畫啟動後，即便 JavaScript 線程阻塞也不影響動畫執行。

在動畫配置中設定 `useNativeDriver: true` 即可啟用，詳見[動畫指南](animations#using-the-native-driver)。

### 可動畫元件

僅特定元件支援動畫效果。這類元件會將動畫數值綁定至屬性，並直接更新原生層以避免每幀的 React 渲染開銷，同時自動處理卸載時的資源清理。

- 透過 [`createAnimatedComponent()`](animated#createanimatedcomponent) 可將普通元件轉換為可動畫元件

`Animated` 預置以下已封裝的可動畫元件：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`
- `Animated.FlatList`
- `Animated.SectionList`

### 動畫組合

透過組合函式可實現複雜的動畫效果：

- [`Animated.delay()`](animated#delay) 在指定延遲後開始動畫。
- [`Animated.parallel()`](animated#parallel) 同時啟動多個動畫。
- [`Animated.sequence()`](animated#sequence) 按順序啟動動畫，等待前一個動畫完成後才開始下一個。
- [`Animated.stagger()`](animated#stagger) 按順序且並行啟動動畫，但會連續延遲。

動畫也可以通過將一個動畫的 `toValue` 設置為另一個 `Animated.Value` 來鏈接在一起。詳見動畫指南中的[追蹤動態值](animations#tracking-dynamic-values)。

預設情況下，如果一個動畫被停止或中斷，則組中的所有其他動畫也會停止。

### 合併動畫值

您可以通過加法、減法、乘法、除法或取模來合併兩個動畫值，從而創建一個新的動畫值：

- [`Animated.add()`](animated#add)
- [`Animated.subtract()`](animated#subtract)
- [`Animated.divide()`](animated#divide)
- [`Animated.modulo()`](animated#modulo)
- [`Animated.multiply()`](animated#multiply)

### 插值

`interpolate()` 函數允許輸入範圍映射到不同的輸出範圍。預設情況下，它會將曲線外推到給定範圍之外，但您也可以讓它限制輸出值。它預設使用線性插值，但也支持緩動函數。

- [`interpolate()`](animated#interpolate)

更多關於插值的內容，請參閱[動畫](animations#interpolation)指南。

### 處理手勢和其他事件

手勢（如平移或滾動）和其他事件可以使用 `Animated.event()` 直接映射到動畫值。這是通過結構化映射語法實現的，以便可以從複雜的事件對象中提取值。第一層是一個數組，允許跨多個參數進行映射，該數組包含嵌套對象。

- [`Animated.event()`](animated#event)

例如，在處理水平滾動手勢時，您可以執行以下操作，將 `event.nativeEvent.contentOffset.x` 映射到 `scrollX`（一個 `Animated.Value`）：

```jsx
 onScroll={Animated.event(
   // scrollX = e.nativeEvent.contentOffset.x
   [{ nativeEvent: {
        contentOffset: {
          x: scrollX
        }
      }
    }]
 )}
```

---

# 參考

## 方法

當給定的值是 ValueXY 而不是 Value 時，每個配置選項可以是 `{x: ..., y: ...}` 形式的向量，而不是標量。

### `decay()`

```jsx
static decay(value, config)
```

根據衰減係數，從初始速度到零動畫化一個值。

配置是一個對象，可能包含以下選項：

- `velocity`: 初始速度。必需。
- `deceleration`: 衰減率。預設 0.997。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「交互句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。預設 false。

---

### `timing()`

```jsx
static timing(value, config)
```

沿著定時緩動曲線動畫化一個值。[`Easing`](easing) 模塊有大量預定義曲線，或者您可以使用自己的函數。

配置是一個對象，可能包含以下選項：

- `duration`: 動畫長度（毫秒）。預設 500。
- `easing`: 定義曲線的緩動函數。預設為 `Easing.inOut(Easing.ease)`。
- `delay`: 在延遲後開始動畫（毫秒）。預設 0。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「交互句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。預設 false。

---

### `spring()`

```jsx
static spring(value, config)
```

根據[阻尼諧振子](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型動畫化數值。會追蹤速度狀態以在`toValue`更新時創造流暢運動，並可被串聯使用。

配置為一個可能包含以下選項的物件：

請注意，您只能定義bounciness/speed、tension/friction或stiffness/damping/mass中的一組參數，不可同時定義多組：

friction/tension或bounciness/speed選項與[`Facebook Pop`](https://github.com/facebook/pop)、[Rebound](https://github.com/facebookarchive/rebound)和[Origami](http://origami.design/)中的彈簧模型相匹配。

- `friction`: 控制「彈性」/過衝。預設值7。
- `tension`: 控制速度。預設值40。
- `speed`: 控制動畫速度。預設值12。
- `bounciness`: 控制彈性。預設值8。

指定stiffness/damping/mass作為參數會使`Animated.spring`使用基於[阻尼諧振子](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)運動方程的解析彈簧模型。此行為在物理上更精確且貼近彈簧動力學，並密切模仿iOS中CASpringAnimation的實現。

- `stiffness`: 彈簧剛度係數。預設值100。
- `damping`: 定義彈簧運動應如何因摩擦力而衰減。預設值10。
- `mass`: 附加在彈簧末端的物體質量。預設值1。

其他配置選項如下：

- `velocity`: 附加在彈簧上物體的初始速度。預設0（物體處於靜止狀態）。
- `overshootClamping`: 布林值，指示彈簧是否應被鉗制而不反彈。預設false。
- `restDisplacementThreshold`: 從靜止位置的位移閾值，低於此值彈簧將被視為靜止。預設0.001。
- `restSpeedThreshold`: 彈簧被視為靜止的速度閾值（像素/秒）。預設0.001。
- `delay`: 延遲後開始動畫（毫秒）。預設0。
- `isInteraction`: 此動畫是否在`InteractionManager`上建立「互動句柄」。預設true。
- `useNativeDriver`: 為true時使用原生驅動。預設false。

---

### `add()`

```jsx
static add(a, b)
```

創建一個由兩個Animated值相加組成的新Animated值。

---

### `subtract()`

```jsx
static subtract(a, b)
```

創建一個由第一個Animated值減去第二個Animated值組成的新Animated值。

---

### `divide()`

```jsx
static divide(a, b)
```

創建一個由第一個Animated值除以第二個Animated值組成的新Animated值。

---

### `multiply()`

```jsx
static multiply(a, b)
```

創建一個由兩個Animated值相乘組成的新Animated值。

---

### `modulo()`

```jsx
static modulo(a, modulus)
```

創建一個新Animated值，該值是所提供Animated值的（非負）模數

---

### `diffClamp()`

```jsx
static diffClamp(a, min, max)
```

建立一個新的動畫值，其值被限制在兩個值之間。它使用最後一個值的差異，因此即使當前值遠離邊界，當值開始再次接近時，它也會開始變化（`value = clamp(value + diff, min, max)`）。

這在滾動事件中非常有用，例如在向上滾動時顯示導航欄，向下滾動時隱藏它。

---

### `delay()`

```jsx
static delay(time)
```

在給定的延遲後開始動畫。

---

### `sequence()`

```jsx
static sequence(animations)
```

按順序開始一系列動畫，等待每個動畫完成後再開始下一個。如果當前運行的動畫被停止，後續的動畫將不會開始。

---

### `parallel()`

```jsx
static parallel(animations, config?)
```

同時開始一系列動畫。默認情況下，如果其中一個動畫被停止，所有動畫都會停止。你可以通過`stopTogether`標誌來覆蓋此行為。

---

### `stagger()`

```jsx
static stagger(time, animations)
```

動畫陣列可以並行運行（重疊），但會以連續的延遲依次開始。適合製作拖尾效果。

---

### `loop()`

```jsx
static loop(animation, config?)
```

循環播放給定的動畫，每次到達結尾時重置並從頭開始。如果子動畫設置為`useNativeDriver: true`，則不會阻塞JS線程。此外，循環可能會阻止基於`VirtualizedList`的組件在動畫運行時渲染更多行。你可以在子動畫配置中傳遞`isInteraction: false`來解決這個問題。

配置是一個可能包含以下選項的對象：

- `iterations`: 動畫應該循環的次數。默認為`-1`（無限）。

---

### `event()`

```jsx
static event(argMapping, config?)
```

接受一個映射陣列，並從每個參數中提取相應的值，然後在映射的輸出上調用`setValue`。例如：

```jsx
 onScroll={Animated.event(
   [{nativeEvent: {contentOffset: {x: this._scrollX}}}],
   {listener: (event) => console.log(event)}, // Optional async listener
 )}
 ...
 onPanResponderMove: Animated.event([
   null,                // raw event arg ignored
   {dx: this._panX}],    // gestureState arg
{listener: (event, gestureState) => console.log(event, gestureState)}, // Optional async listener
 ),
```

配置是一個可能包含以下選項的對象：

- `listener`: 可選的異步監聽器。
- `useNativeDriver`: 為true時使用原生驅動。默認為false。

---

### `forkEvent()`

```jsx
static forkEvent(event, listener)
```

用於監聽通過props傳遞的動畫事件的高級命令式API。它允許向現有的`AnimatedEvent`添加一個新的javascript監聽器。如果`animatedEvent`是一個javascript監聽器，它會將兩個監聽器合併為一個；如果`animatedEvent`為null/undefined，它會直接分配javascript監聽器。盡可能直接使用值。

---

### `unforkEvent()`

```jsx
static unforkEvent(event, listener)
```

---

### `start()`

```jsx
static start([callback]: ?(result?: {finished: boolean}) => void)
```

通過在動畫上調用start()來開始動畫。start()接受一個完成回調，該回調將在動畫完成時或在動畫完成前調用stop()時被調用。

**參數：**

| Name     | Type                              | Required | Description                                                                                                                                                     |
| -------- | --------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback | `?(result?: {finished: boolean})` | No       | Function that will be called after the animation finished running normally or when the animation is done because stop() was called on it before it could finish |

帶有回調的開始示例：

```jsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

---

### `stop()`

```jsx
static stop()
```

停止所有正在執行的動畫。

---

### `reset()`

```jsx
static reset()
```

停止所有正在執行的動畫並將數值重置為原始值。

## 屬性

### `Value`

驅動動畫的標準數值類別。通常初始化為 `new Animated.Value(0);`

您可以在獨立的[頁面](animatedvalue)上閱讀更多關於 `Animated.Value` API 的資訊。

---

### `ValueXY`

用於驅動二維動畫（例如平移手勢）的 2D 數值類別。

您可以在獨立的[頁面](animatedvaluexy)上閱讀更多關於 `Animated.ValueXY` API 的資訊。

---

### `Interpolation`

導出以在流程中使用 Interpolation 類型。

---

### `Node`

導出以便於類型檢查。所有動畫數值均派生自此類別。

---

### `createAnimatedComponent`

使任何 React 組件可動畫化。用於創建 `Animated.View` 等。

---

### `attachNativeEvent`

將動畫數值附加到視圖事件的命令式 API。如果可能，建議優先使用帶有 `useNativeDrive: true` 的 `Animated.event`。