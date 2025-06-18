---
id: animated
title: Animated
---

`Animated` 函式庫專為打造流暢、強大且易於建構維護的動畫而設計。其核心在於宣告式的輸入輸出關係、可配置的轉換過程，以及透過 `start`/`stop` 方法控制基於時間軸的動畫執行。

建立動畫的核心流程是：創建一個 `Animated.Value`，將其綁定至動畫元件的樣式屬性，再透過 `Animated.timing()` 驅動數值更新。

> 請勿直接修改動畫數值。建議使用 [`useRef` Hook](https://reactjs.org/docs/hooks-reference.html#useref) 回傳可變的 ref 物件，該物件的 `current` 屬性會在元件生命週期中持續保存初始值。

## 範例

以下範例展示了一個 `View` 元件如何根據動畫數值 `fadeAnim` 實現淡入淡出效果：

```SnackPlayer name=Animated%20Example&supportedPlatforms=ios,android
import React from 'react';
import {Animated, Text, View, StyleSheet, Button, useAnimatedValue} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const App = () => {
  // fadeAnim will be used as the value for opacity. Initial Value: 0
  const fadeAnim = useAnimatedValue(0);

  const fadeIn = () => {
    // Will change fadeAnim value to 1 in 5 seconds
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 5000,
      useNativeDriver: true,
    }).start();
  };

  const fadeOut = () => {
    // Will change fadeAnim value to 0 in 3 seconds
    Animated.timing(fadeAnim, {
      toValue: 0,
      duration: 3000,
      useNativeDriver: true,
    }).start();
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Animated.View
          style={[
            styles.fadingContainer,
            {
              // Bind opacity to animated value
              opacity: fadeAnim,
            },
          ]}>
          <Text style={styles.fadingText}>Fading View!</Text>
        </Animated.View>
        <View style={styles.buttonRow}>
          <Button title="Fade In View" onPress={fadeIn} />
          <Button title="Fade Out View" onPress={fadeOut} />
        </View>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  fadingContainer: {
    padding: 20,
    backgroundColor: 'powderblue',
  },
  fadingText: {
    fontSize: 28,
  },
  buttonRow: {
    flexBasis: 100,
    justifyContent: 'space-evenly',
    marginVertical: 16,
  },
});

export default App;
```

更多實際應用範例請參閱 [動畫指南](animations#animated-api)。

## 概覽

`Animated` 支援兩種數值類型：

- [`Animated.Value()`](animated#value) 用於單一數值
- [`Animated.ValueXY()`](animated#valuexy) 用於向量

`Animated.Value` 可綁定至樣式屬性或其他參數，並支援插值運算。單一 `Animated.Value` 可驅動多個屬性。

### 動畫配置

`Animated` 提供三種動畫類型，每種都具備特定的動畫曲線來控制數值從初始狀態到最終狀態的變化過程：

- [`Animated.decay()`](animated#decay) 以初始速度開始並逐漸減速至停止
- [`Animated.spring()`](animated#spring) 提供基礎彈簧物理模型
- [`Animated.timing()`](animated#timing) 透過[緩動函式](easing)實現時序動畫

多數情況下會使用 `timing()`。預設採用對稱的 easeInOut 曲線，模擬物體加速至全速後再減速停止的過程。

### 動畫操作

呼叫動畫實例的 `start()` 方法可啟動動畫，該方法接受一個在動畫完成時執行的回調函式。若動畫正常完成，回調函式會收到 `{finished: true}`；若因呼叫 `stop()` 中斷（例如被手勢或其他動畫打斷），則會收到 `{finished: false}`。

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

### 使用原生驅動

啟用原生驅動後，系統會在動畫開始前將所有參數傳送至原生端，讓 UI 執行緒直接處理動畫，避免每幀都需跨橋接通訊。動畫啟動後，即使 JavaScript 執行緒阻塞也不影響動畫運行。

在動畫配置中加入 `useNativeDriver: true` 即可啟用原生驅動，詳見[動畫指南](animations#using-the-native-driver)。

### 可動畫化元件

僅有特殊封裝的可動畫化元件能綁定動畫數值。這些元件會將動畫值映射至屬性，並透過原生端定向更新來避免每幀的 React 渲染與協調開銷，同時自動處理卸載時的資源清理。

- 使用 [`createAnimatedComponent()`](animated#createanimatedcomponent) 可將普通元件轉化為可動畫化元件

`Animated` 內建以下已封裝的可動畫化元件：

- `Animated.Image`
- `Animated.ScrollView`
- `Animated.Text`
- `Animated.View`
- `Animated.FlatList`
- `Animated.SectionList`

### 動畫組合

動畫也可以透過組合函數以複雜的方式結合：

- [`Animated.delay()`](animated#delay) 在指定延遲後開始動畫。
- [`Animated.parallel()`](animated#parallel) 同時啟動多個動畫。
- [`Animated.sequence()`](animated#sequence) 按順序啟動動畫，等待前一個完成後才開始下一個。
- [`Animated.stagger()`](animated#stagger) 按順序且並行啟動動畫，但具有連續的延遲。

動畫也可以透過將一個動畫的 `toValue` 設為另一個 `Animated.Value` 來串聯。請參閱動畫指南中的[追蹤動態值](animations#tracking-dynamic-values)。

預設情況下，如果一個動畫被停止或中斷，則群組中的所有其他動畫也會停止。

### 結合動畫值

您可以透過加法、減法、乘法、除法或模運算來結合兩個動畫值，以創建新的動畫值：

- [`Animated.add()`](animated#add)
- [`Animated.subtract()`](animated#subtract)
- [`Animated.divide()`](animated#divide)
- [`Animated.modulo()`](animated#modulo)
- [`Animated.multiply()`](animated#multiply)

### 插值

`interpolate()` 函數允許將輸入範圍映射到不同的輸出範圍。預設情況下，它會將曲線外推到給定範圍之外，但您也可以讓它限制輸出值。它預設使用線性插值，但也支援緩動函數。

- [`interpolate()`](animatedvalue#interpolate)

請在[動畫](animations#interpolation)指南中閱讀更多關於插值的資訊。

### 處理手勢和其他事件

手勢（如平移或滾動）和其他事件可以使用 `Animated.event()` 直接映射到動畫值。這是透過結構化映射語法完成的，以便可以從複雜的事件物件中提取值。第一層是一個陣列，允許跨多個參數進行映射，該陣列包含嵌套物件。

- [`Animated.event()`](animated#event)

例如，在處理水平滾動手勢時，您可以執行以下操作，將 `event.nativeEvent.contentOffset.x` 映射到 `scrollX`（一個 `Animated.Value`）：

```tsx
 onScroll={Animated.event(
   // scrollX = e.nativeEvent.contentOffset.x
   [{nativeEvent: {
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

當給定的值是 ValueXY 而不是 Value 時，每個配置選項可以是 `{x: ..., y: ...}` 形式的向量，而不是純量。

### `decay()`

```tsx
static decay(value, config): CompositeAnimation;
```

根據衰減係數，將值從初始速度動畫化為零。

配置是一個物件，可能具有以下選項：

- `velocity`: 初始速度。必填。
- `deceleration`: 衰減率。預設 0.997。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「互動句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必填。

---

### `timing()`

```tsx
static timing(value, config): CompositeAnimation;
```

沿著定時緩動曲線動畫化一個值。[`Easing`](easing) 模組有大量預定義曲線，或者您可以使用自己的函數。

配置是一個物件，可能具有以下選項：

- `duration`: 動畫長度（毫秒）。預設 500。
- `easing`: 定義曲線的緩動函數。預設為 `Easing.inOut(Easing.ease)`。
- `delay`: 在延遲後開始動畫（毫秒）。預設 0。
- `isInteraction`: 此動畫是否在 `InteractionManager` 上創建「互動句柄」。預設 true。
- `useNativeDriver`: 為 true 時使用原生驅動。必填。

---

### `spring()`

```tsx
static spring(value, config): CompositeAnimation;
```

根據[阻尼諧振子模型](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧系統動畫化數值。通過追蹤速度狀態來在`toValue`更新時創建流暢運動，並可進行鏈式組合。

配置為一個可包含以下選項的物件：

注意：只能選擇定義以下一組參數，不可同時定義多組：

friction/tension或bounciness/speed選項與[Facebook Pop](https://github.com/facebook/pop)、[Rebound](https://github.com/facebookarchive/rebound)及[Origami](https://origami.design/)中的彈簧模型匹配。

- `friction`: 控制「彈性」/過衝。預設值7。
- `tension`: 控制速度。預設值40。
- `speed`: 控制動畫速度。預設值12。
- `bounciness`: 控制彈性。預設值8。

指定stiffness/damping/mass參數會使`Animated.spring`採用基於[阻尼諧振子運動方程](https://en.wikipedia.org/wiki/Harmonic_oscillator#Damped_harmonic_oscillator)的解析彈簧模型。此行為更精確地模擬彈簧動力學的物理特性，並高度還原iOS中CASpringAnimation的實現。

- `stiffness`: 彈簧剛度係數。預設值100。
- `damping`: 定義因摩擦力導致的彈簧運動衰減程度。預設值10。
- `mass`: 附著於彈簧末端的物體質量。預設值1。

其他配置選項如下：

- `velocity`: 附著於彈簧的物體初始速度。預設0（物體靜止）。
- `overshootClamping`: 布林值，指示彈簧是否應箝位而不反彈。預設false。
- `restDisplacementThreshold`: 靜止位移閾值，低於此值時彈簧視為靜止。預設0.001。
- `restSpeedThreshold`: 彈簧視為靜止的速度閾值（像素/秒）。預設0.001。
- `delay`: 延遲後開始動畫（毫秒）。預設0。
- `isInteraction`: 是否在`InteractionManager`上建立「互動句柄」。預設true。
- `useNativeDriver`: 為true時使用原生驅動。必需。

---

### `add()`

```tsx
static add(a: Animated, b: Animated): AnimatedAddition;
```

創建由兩個Animated值相加組成的新Animated值。

---

### `subtract()`

```tsx
static subtract(a: Animated, b: Animated): AnimatedSubtraction;
```

創建由第一個Animated值減去第二個Animated值組成的新Animated值。

---

### `divide()`

```tsx
static divide(a: Animated, b: Animated): AnimatedDivision;
```

創建由第一個Animated值除以第二個Animated值組成的新Animated值。

---

### `multiply()`

```tsx
static multiply(a: Animated, b: Animated): AnimatedMultiplication;
```

創建由兩個Animated值相乘組成的新Animated值。

---

### `modulo()`

```tsx
static modulo(a: Animated, modulus: number): AnimatedModulo;
```

創建提供Animated值之非負模數的新Animated值。

---

### `diffClamp()`

```tsx
static diffClamp(a: Animated, min: number, max: number): AnimatedDiffClamp;
```

建立一個新的動畫數值，該數值會被限制在兩個值之間。它會根據前後數值的差異來調整，因此即使當前數值遠離邊界，只要數值開始接近邊界時就會開始變化（計算方式為 `value = clamp(value + diff, min, max)`）。

這在處理滾動事件時特別有用，例如在向上滾動時顯示導航欄，向下滾動時隱藏導航欄。

---

### `delay()`

```tsx
static delay(time: number): CompositeAnimation;
```

在指定的延遲時間後開始動畫。

---

### `sequence()`

```tsx
static sequence(animations: CompositeAnimation[]): CompositeAnimation;
```

按順序開始一系列動畫，等待前一個動畫完成後才開始下一個。如果當前運行的動畫被停止，後續的動畫將不會開始。

---

### `parallel()`

```tsx
static parallel(
  animations: CompositeAnimation[],
  config?: ParallelConfig
): CompositeAnimation;
```

同時開始一系列動畫。默認情況下，如果其中一個動畫被停止，所有動畫都會停止。可以通過設置 `stopTogether` 標誌來覆蓋此行為。

---

### `stagger()`

```tsx
static stagger(
  time: number,
  animations: CompositeAnimation[]
): CompositeAnimation;
```

一系列動畫可以並行運行（重疊），但會按順序以連續的延遲時間開始。適合用於實現拖尾效果。

---

### `loop()`

```tsx
static loop(
  animation: CompositeAnimation[],
  config?: LoopAnimationConfig
): CompositeAnimation;
```

循環播放指定的動畫，每次到達結尾時會重置並從頭開始。如果子動畫設置為 `useNativeDriver: true`，則可以在不阻塞 JS 線程的情況下循環播放。此外，循環動畫可能會阻止基於 `VirtualizedList` 的組件在動畫運行時渲染更多行。可以在子動畫配置中傳遞 `isInteraction: false` 來解決此問題。

配置是一個對象，可以包含以下選項：

- `iterations`: 動畫循環的次數。默認為 `-1`（無限循環）。

---

### `event()`

```tsx
static event(
  argMapping: Mapping[],
  config?: EventConfig
): (...args: any[]) => void;
```

接收一個映射數組，並從每個參數中提取相應的值，然後對映射後的輸出調用 `setValue`。例如：

```tsx
onScroll={Animated.event(
  [{nativeEvent: {contentOffset: {x: this._scrollX}}}],
  {listener: (event: ScrollEvent) => console.log(event)}, // Optional async listener
)}
 ...
onPanResponderMove: Animated.event(
  [
    null, // raw event arg ignored
    {dx: this._panX},
  ], // gestureState arg
  {
    listener: (
      event: GestureResponderEvent,
      gestureState: PanResponderGestureState
    ) => console.log(event, gestureState),
  } // Optional async listener
);
```

配置是一個對象，可以包含以下選項：

- `listener`: 可選的異步監聽器。
- `useNativeDriver`: 為 true 時使用原生驅動。必需。

---

### `forkEvent()`

```jsx
static forkEvent(event: AnimatedEvent, listener: Function): AnimatedEvent;
```

這是一個高級的命令式 API，用於監聽通過 props 傳遞的動畫事件。它允許在現有的 `AnimatedEvent` 上添加一個新的 JavaScript 監聽器。如果 `animatedEvent` 是一個 JavaScript 監聽器，它會將兩個監聽器合併為一個；如果 `animatedEvent` 為 null/undefined，則會直接分配 JavaScript 監聽器。盡可能直接使用數值。

---

### `unforkEvent()`

```jsx
static unforkEvent(event: AnimatedEvent, listener: Function);
```

---

### `start()`

```tsx
static start(callback?: (result: {finished: boolean}) => void);
```

通過調用動畫的 start() 方法來啟動動畫。start() 接受一個完成回調函數，該函數會在動畫完成時或在動畫完成前因調用 stop() 而被停止時調用。

**參數：**

| Name     | Type                                    | Required | Description                                                                                                                                                     |
| -------- | --------------------------------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| callback | `(result: {finished: boolean}) => void` | No       | Function that will be called after the animation finished running normally or when the animation is done because stop() was called on it before it could finish |

帶回調的啟動示例：

```tsx
Animated.timing({}).start(({finished}) => {
  /* completion callback */
});
```

---

### `stop()`

```tsx
static stop();
```

停止所有正在執行的動畫。

---

### `reset()`

```tsx
static reset();
```

停止所有正在執行的動畫並將數值重置為原始狀態。

## 屬性

### `Value`

驅動動畫的標準數值類別。通常透過 `useAnimatedValue(0);` 或在類別元件中使用 `new Animated.Value(0);` 初始化。

您可以在專屬的 [頁面](animatedvalue) 上閱讀更多關於 `Animated.Value` API 的資訊。

---

### `ValueXY`

用於驅動二維動畫（例如平移手勢）的 2D 數值類別。

您可以在專屬的 [頁面](animatedvaluexy) 上閱讀更多關於 `Animated.ValueXY` API 的資訊。

---

### `Interpolation`

導出用於在流程中使用 Interpolation 類型。

---

### `Node`

為方便類型檢查而導出。所有動畫數值均派生自此類別。

---

### `createAnimatedComponent`

使任何 React 元件可動畫化。用於創建 `Animated.View` 等元件。

---

### `attachNativeEvent`

將動畫數值附加到視圖事件的命令式 API。如果可能，建議優先使用 `Animated.event` 並設定 `useNativeDriver: true`。