---
id: animations
title: Animations
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

動畫對於創造出色的使用者體驗至關重要。靜止的物體開始移動時必須克服慣性，而運動中的物體具有動量，很少會立即停止。動畫能讓您在介面中傳達符合物理規律的運動效果。

React Native 提供兩套互補的動畫系統：[`Animated`](animations#animated-api) 用於對特定值進行細粒度互動控制，[`LayoutAnimation`](animations#layoutanimation-api) 則用於全局佈局變更的動畫處理。

## `Animated` API

[`Animated`](animated) API 的設計目標是以高效能的方式簡潔表達各種動畫與互動模式。其核心在於通過可配置的轉換過程，建立輸入與輸出之間的宣告式關係，並透過 `start`/`stop` 方法控制基於時間軸的動畫執行。

`Animated` 導出六種可動畫化的元件類型：`View`、`Text`、`Image`、`ScrollView`、`FlatList` 和 `SectionList`，您也可以使用 `Animated.createAnimatedComponent()` 創建自訂元件。

例如，一個在掛載時淡入的容器視圖可能如下所示：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer ext=js&supportedPlatforms=ios,android
import React, {useEffect} from 'react';
import {Animated, Text, View, useAnimatedValue} from 'react-native';

const FadeInView = props => {
  const fadeAnim = useAnimatedValue(0); // Initial value for opacity: 0

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 10000,
      useNativeDriver: true,
    }).start();
  }, [fadeAnim]);

  return (
    <Animated.View // Special animatable View
      style={{
        ...props.style,
        opacity: fadeAnim, // Bind opacity to animated value
      }}>
      {props.children}
    </Animated.View>
  );
};

// You can then use your `FadeInView` in place of a `View` in your components:
export default () => {
  return (
    <View
      style={{
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
      }}>
      <FadeInView
        style={{
          width: 250,
          height: 50,
          backgroundColor: 'powderblue',
        }}>
        <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>
          Fading in
        </Text>
      </FadeInView>
    </View>
  );
};
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer ext=tsx
import React, {useEffect} from 'react';
import {Animated, Text, View, useAnimatedValue} from 'react-native';
import type {PropsWithChildren} from 'react';
import type {ViewStyle} from 'react-native';

type FadeInViewProps = PropsWithChildren<{style: ViewStyle}>;

const FadeInView: React.FC<FadeInViewProps> = props => {
  const fadeAnim = useAnimatedValue(0); // Initial value for opacity: 0

  useEffect(() => {
    Animated.timing(fadeAnim, {
      toValue: 1,
      duration: 10000,
      useNativeDriver: true,
    }).start();
  }, [fadeAnim]);

  return (
    <Animated.View // Special animatable View
      style={{
        ...props.style,
        opacity: fadeAnim, // Bind opacity to animated value
      }}>
      {props.children}
    </Animated.View>
  );
};

// You can then use your `FadeInView` in place of a `View` in your components:
export default () => {
  return (
    <View
      style={{
        flex: 1,
        alignItems: 'center',
        justifyContent: 'center',
      }}>
      <FadeInView
        style={{
          width: 250,
          height: 50,
          backgroundColor: 'powderblue',
        }}>
        <Text style={{fontSize: 28, textAlign: 'center', margin: 10}}>
          Fading in
        </Text>
      </FadeInView>
    </View>
  );
};
```

</TabItem>
</Tabs>

讓我們解析這段代碼。在 `FadeInView` 建構函式中，初始化了一個名為 `fadeAnim` 的 `Animated.Value` 並存入 `state`。`View` 的透明度屬性被映射到這個動畫值，底層會提取數值並用於設置透明度。

元件掛載時，透明度被設為 0。接著在 `fadeAnim` 上啟動一個緩動動畫，該值會逐幀過渡到最終值 1，同時更新所有依賴映射（本例中僅透明度屬性）。

這種實現方式經過優化，比呼叫 `setState` 重新渲染更高效。由於整個配置是宣告式的，未來還能實現將配置序列化並在高優先級線程運行動畫的進一步優化。

### 配置動畫

動畫具有高度可配置性。根據動畫類型的不同，可調整自訂/預設緩動函數、延遲時間、持續時長、衰減因子、彈簧係數等參數。

`Animated` 提供多種動畫類型，最常用的是 [`Animated.timing()`](animated#timing)。它支持使用預設緩動函數或自訂函數隨時間變化數值。緩動函數通常用於表現物體逐漸加速或減速的運動效果。

預設情況下，`timing` 採用 easeInOut 曲線，表現為先漸加速至全速，結束時漸減速至停止。您可通過 `easing` 參數指定其他緩動函數，也支持設置自訂 `duration` 或動畫開始前的 `delay`。

例如要創建一個 2 秒長的動畫，讓物體在到達最終位置前先輕微後退：

```tsx
Animated.timing(this.state.xPosition, {
  toValue: 100,
  easing: Easing.back(),
  duration: 2000,
  useNativeDriver: true,
}).start();
```

查閱 `Animated` API 參考文件的[配置動畫](animated#configuring-animations)章節，了解內建動畫支持的所有配置參數。

### 組合動畫

動畫可以組合並按順序或並行播放。順序動畫可在前一個動畫結束後立即播放，或延遲指定時間啟動。`Animated` API 提供 `sequence()` 和 `delay()` 等方法，這些方法接收動畫陣列並自動按需呼叫 `start()`/`stop()`。

例如以下動畫會先滑行停止，然後同時彈回並旋轉：

```tsx
Animated.sequence([
  // decay, then spring to start and twirl
  Animated.decay(position, {
    // coast to a stop
    velocity: {x: gestureState.vx, y: gestureState.vy}, // velocity from gesture release
    deceleration: 0.997,
    useNativeDriver: true,
  }),
  Animated.parallel([
    // after decay, in parallel:
    Animated.spring(position, {
      toValue: {x: 0, y: 0}, // return to start
      useNativeDriver: true,
    }),
    Animated.timing(twirl, {
      // and twirl
      toValue: 360,
      useNativeDriver: true,
    }),
  ]),
]).start(); // start the sequence group
```

如果一個動畫被停止或中斷，那麼群組中的所有其他動畫也會被停止。`Animated.parallel` 提供了一個 `stopTogether` 選項，可設為 `false` 來禁用此行為。

您可以在 `Animated` API 參考文件的[組合動畫](animated#composing-animations)章節中找到完整的組合方法列表。

### 合併動畫數值

您可以通過加法、乘法、除法或取模來[合併兩個動畫數值](animated#combining-animated-values)，從而創建一個新的動畫數值。

在某些情況下，一個動畫數值需要反轉另一個動畫數值進行計算。例如反轉縮放比例（2倍 --> 0.5倍）：

```tsx
const a = new Animated.Value(1);
const b = Animated.divide(1, a);

Animated.spring(a, {
  toValue: 2,
  useNativeDriver: true,
}).start();
```

### 插值

每個屬性都可以先經過插值處理。插值將輸入範圍映射到輸出範圍，通常使用線性插值，但也支援緩動函數。默認情況下，它會外推超出給定範圍的曲線，但您也可以設置使其限制輸出值。

將 0-1 範圍轉換為 0-100 範圍的基本映射如下：

```tsx
value.interpolate({
  inputRange: [0, 1],
  outputRange: [0, 100],
});
```

例如，您可能希望將 `Animated.Value` 視為從 0 到 1，但將位置從 150px 動畫到 0px，並將透明度從 0 動畫到 1。可以通過修改上述示例中的 `style` 來實現：

```tsx
  style={{
    opacity: this.state.fadeAnim, // Binds directly
    transform: [{
      translateY: this.state.fadeAnim.interpolate({
        inputRange: [0, 1],
        outputRange: [150, 0]  // 0 : 150, 0.5 : 75, 1 : 0
      }),
    }],
  }}
```

[`interpolate()`](animated#interpolate) 還支援多個範圍段，這對於定義死區和其他技巧非常有用。例如，要在 -300 處獲得否定關係，在 -100 處變為 0，然後在 0 處回到 1，接著在 100 處再次降為 0，之後的死區保持為 0，您可以這樣做：

```tsx
value.interpolate({
  inputRange: [-300, -100, 0, 100, 101],
  outputRange: [300, 0, 1, 0, 0],
});
```

映射結果如下：

```
Input | Output
------|-------
  -400|    450
  -300|    300
  -200|    150
  -100|      0
   -50|    0.5
     0|      1
    50|    0.5
   100|      0
   101|      0
   200|      0
```

`interpolate()` 還支援映射到字符串，允許您動畫顏色以及帶單位的值。例如，如果要動畫旋轉，可以這樣做：

```tsx
value.interpolate({
  inputRange: [0, 360],
  outputRange: ['0deg', '360deg'],
});
```

`interpolate()` 還支援任意緩動函數，其中許多已在 [`Easing`](easing) 模塊中實現。`interpolate()` 還可配置外推 `outputRange` 的行為。您可以通過設置 `extrapolate`、`extrapolateLeft` 或 `extrapolateRight` 選項來配置外推。默認值為 `extend`，但您可以使用 `clamp` 來防止輸出值超出 `outputRange`。

### 追蹤動態數值

動畫數值還可以通過將動畫的 `toValue` 設置為另一個動畫數值而非普通數字來追蹤其他數值。例如，Android 版 Messenger 使用的「聊天頭像」動畫可以通過固定在另一個動畫數值上的 `spring()` 來實現，或者使用 `timing()` 並將 `duration` 設為 0 以進行嚴格追蹤。它們還可以與插值組合使用：

```tsx
Animated.spring(follower, {toValue: leader}).start();
Animated.timing(opacity, {
  toValue: pan.x.interpolate({
    inputRange: [0, 300],
    outputRange: [1, 0],
    useNativeDriver: true,
  }),
}).start();
```

`leader` 和 `follower` 動畫數值將使用 `Animated.ValueXY()` 實現。`ValueXY` 是處理 2D 交互（如平移或拖動）的便捷方式。它是一個基本包裝器，包含兩個 `Animated.Value` 實例和一些調用它們的輔助函數，使 `ValueXY` 在許多情況下可以作為 `Value` 的直接替代品。它允許我們在上面的示例中同時追蹤 x 和 y 值。

### 追蹤手勢

手勢（如平移或滾動）和其他事件可以使用 [`Animated.event`](animated#event) 直接映射到動畫數值。這是通過結構化映射語法實現的，以便可以從複雜的事件對象中提取數值。第一層是一個數組，允許跨多個參數進行映射，該數組包含嵌套對象。

舉例來說，當處理水平滾動手勢時，您可以透過以下方式將 `event.nativeEvent.contentOffset.x` 映射到 `scrollX`（一個 `Animated.Value`）：

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

以下範例實現了一個水平滾動的輪播組件，其中滾動位置指示器透過在 `ScrollView` 中使用的 `Animated.event` 實現動畫效果：

#### 使用動畫事件的 ScrollView 範例

```SnackPlayer name=Animated&supportedPlatforms=ios,android
import React from 'react';
import {
  SafeAreaView,
  ScrollView,
  Text,
  StyleSheet,
  View,
  ImageBackground,
  Animated,
  useWindowDimensions,
  useAnimatedValue,
} from 'react-native';

const images = new Array(6).fill(
  'https://images.unsplash.com/photo-1556740749-887f6717d7e4',
);

const App = () => {
  const scrollX = useAnimatedValue(0);

  const {width: windowWidth} = useWindowDimensions();

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.scrollContainer}>
        <ScrollView
          horizontal={true}
          pagingEnabled
          showsHorizontalScrollIndicator={false}
          onScroll={Animated.event([
            {
              nativeEvent: {
                contentOffset: {
                  x: scrollX,
                },
              },
            },
          ])}
          scrollEventThrottle={1}>
          {images.map((image, imageIndex) => {
            return (
              <View style={{width: windowWidth, height: 250}} key={imageIndex}>
                <ImageBackground source={{uri: image}} style={styles.card}>
                  <View style={styles.textContainer}>
                    <Text style={styles.infoText}>
                      {'Image - ' + imageIndex}
                    </Text>
                  </View>
                </ImageBackground>
              </View>
            );
          })}
        </ScrollView>
        <View style={styles.indicatorContainer}>
          {images.map((image, imageIndex) => {
            const width = scrollX.interpolate({
              inputRange: [
                windowWidth * (imageIndex - 1),
                windowWidth * imageIndex,
                windowWidth * (imageIndex + 1),
              ],
              outputRange: [8, 16, 8],
              extrapolate: 'clamp',
            });
            return (
              <Animated.View
                key={imageIndex}
                style={[styles.normalDot, {width}]}
              />
            );
          })}
        </View>
      </View>
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  scrollContainer: {
    height: 300,
    alignItems: 'center',
    justifyContent: 'center',
  },
  card: {
    flex: 1,
    marginVertical: 4,
    marginHorizontal: 16,
    borderRadius: 5,
    overflow: 'hidden',
    alignItems: 'center',
    justifyContent: 'center',
  },
  textContainer: {
    backgroundColor: 'rgba(0,0,0, 0.7)',
    paddingHorizontal: 24,
    paddingVertical: 8,
    borderRadius: 5,
  },
  infoText: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  },
  normalDot: {
    height: 8,
    width: 8,
    borderRadius: 4,
    backgroundColor: 'silver',
    marginHorizontal: 4,
  },
  indicatorContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
  },
});

export default App;
```

當使用 `PanResponder` 時，您可以使用以下程式碼從 `gestureState.dx` 和 `gestureState.dy` 提取 x 和 y 座標。我們在陣列的第一個位置使用 `null`，因為我們只對傳遞給 `PanResponder` 處理器的第二個參數（即 `gestureState`）感興趣：

```tsx
onPanResponderMove={Animated.event(
  [null, // ignore the native event
  // extract dx and dy from gestureState
  // like 'pan.x = gestureState.dx, pan.y = gestureState.dy'
  {dx: pan.x, dy: pan.y}
])}
```

#### 使用動畫事件的 PanResponder 範例

```SnackPlayer name=Animated
import React, {useRef} from 'react';
import {Animated, View, StyleSheet, PanResponder, Text} from 'react-native';

const App = () => {
  const pan = useRef(new Animated.ValueXY()).current;
  const panResponder = useRef(
    PanResponder.create({
      onMoveShouldSetPanResponder: () => true,
      onPanResponderMove: Animated.event([null, {dx: pan.x, dy: pan.y}]),
      onPanResponderRelease: () => {
        Animated.spring(pan, {
          toValue: {x: 0, y: 0},
          useNativeDriver: true,
        }).start();
      },
    }),
  ).current;

  return (
    <View style={styles.container}>
      <Text style={styles.titleText}>Drag & Release this box!</Text>
      <Animated.View
        style={{
          transform: [{translateX: pan.x}, {translateY: pan.y}],
        }}
        {...panResponder.panHandlers}>
        <View style={styles.box} />
      </Animated.View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  titleText: {
    fontSize: 14,
    lineHeight: 24,
    fontWeight: 'bold',
  },
  box: {
    height: 150,
    width: 150,
    backgroundColor: 'blue',
    borderRadius: 5,
  },
});

export default App;
```

### 響應當前的動畫值

您可能會注意到，在動畫過程中沒有明確的方法可以讀取當前值。這是因為由於優化，該值可能僅在原生運行時中已知。如果您需要根據當前值執行 JavaScript，有兩種方法：

- `spring.stopAnimation(callback)` 會停止動畫並以最終值調用 `callback`。這在製作手勢過渡時非常有用。
- `spring.addListener(callback)` 會在動畫運行時異步調用 `callback`，提供最近的值。這對於觸發狀態變化非常有用，例如當用戶拖動時將一個泡泡吸附到新選項，因為這些較大的狀態變化對幾幀的延遲不太敏感，而像平移這樣的連續手勢需要以 60 fps 運行。

`Animated` 設計為完全可序列化，以便動畫可以以高性能的方式運行，獨立於正常的 JavaScript 事件循環。這會影響 API，因此當某些操作比完全同步系統更複雜時，請記住這一點。可以查閱 `Animated.Value.addListener` 作為解決這些限制的一種方法，但請謹慎使用，因為它可能會在未來影響性能。

### 使用原生驅動

`Animated` API 設計為可序列化。通過使用[原生驅動](/blog/2017/02/14/using-native-driver-for-animated)，我們在開始動畫之前將所有關於動畫的信息發送到原生端，允許原生代碼在 UI 線程上執行動畫，而無需在每一幀都通過橋接。一旦動畫開始，JS 線程可以被阻塞而不影響動畫。

為普通動畫使用原生驅動可以通過在啟動動畫時在動畫配置中設置 `useNativeDriver: true` 來實現。沒有 `useNativeDriver` 屬性的動畫將默認為 false（出於兼容性考慮），但會發出警告（在 TypeScript 中會出現類型檢查錯誤）。

```tsx
Animated.timing(this.state.animatedValue, {
  toValue: 1,
  duration: 500,
  useNativeDriver: true, // <-- Set this to true
}).start();
```

動畫值僅與一個驅動兼容，因此如果您在啟動動畫時使用原生驅動，請確保對該值的所有動畫也都使用原生驅動。

原生驅動也可以與 `Animated.event` 一起使用。這對於跟隨滾動位置的動畫特別有用，因為如果沒有原生驅動，由於 React Native 的異步特性，動畫總是會比手勢慢一幀。

```tsx
<Animated.ScrollView // <-- Use the Animated ScrollView wrapper
  onScroll={Animated.event(
    [
      {
        nativeEvent: {
          contentOffset: {y: this.state.animatedValue},
        },
      },
    ],
    {useNativeDriver: true}, // <-- Set this to true
  )}>
  {content}
</Animated.ScrollView>
```

您可以通過運行 [RNTester 應用](https://github.com/facebook/react-native/blob/main/packages/rn-tester/)，然後加載原生動畫範例來查看原生驅動的實際效果。您也可以查看[源代碼](https://github.com/facebook/react-native/blob/master/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)以了解這些範例是如何實現的。

#### 注意事項

並非所有使用 `Animated` 的功能都能被原生驅動器支援。主要的限制在於你只能對非佈局屬性進行動畫處理：例如 `transform` 和 `opacity` 會起作用，但 Flexbox 和位置屬性則不行。當使用 `Animated.event` 時，它僅適用於直接事件而不支援冒泡事件。這意味著它無法與 `PanResponder` 配合使用，但可以與 `ScrollView#onScroll` 這類事件協同工作。

當動畫正在執行時，它可能會阻止 `VirtualizedList` 元件渲染更多行。如果你需要在用戶滾動列表時執行長時間或循環動畫，可以在動畫配置中使用 `isInteraction: false` 來避免此問題。

### 注意事項

使用如 `rotateY`、`rotateX` 等變形樣式時，請確保同時設置 `perspective` 變形樣式。目前某些動畫若缺少此設定，在 Android 上可能無法正常渲染。下方提供範例。

```tsx
<Animated.View
  style={{
    transform: [
      {scale: this.state.scale},
      {rotateY: this.state.rotateY},
      {perspective: 1000}, // without this line this Animation will not render on Android while working fine on iOS
    ],
  }}
/>
```

### 其他範例

RNTester 應用程式內含多種 `Animated` 的使用範例：

- [AnimatedGratuitousApp](https://github.com/facebook/react-native/tree/main/packages/rn-tester/js/examples/AnimatedGratuitousApp)
- [NativeAnimationsExample](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/NativeAnimation/NativeAnimationsExample.js)

## `LayoutAnimation` API

`LayoutAnimation` 可讓你全域配置「創建」和「更新」動畫，這些動畫將在下一個渲染/佈局週期中用於所有視圖。這對於進行 Flexbox 佈局更新非常有用，無需費心測量或計算特定屬性來直接對其進行動畫處理，特別是在佈局變更可能影響上層元素時（例如「查看更多」展開同時增大父容器尺寸並下推下方列的情況），否則需要元件間明確協調才能同步動畫。

請注意，儘管 `LayoutAnimation` 功能強大且相當實用，但它提供的控制力遠不如 `Animated` 和其他動畫函式庫，因此若無法透過 `LayoutAnimation` 實現需求，可能需要改用其他方法。

請注意，在 **Android** 上要使此功能正常運作，需透過 `UIManager` 設定以下標記：

```tsx
UIManager.setLayoutAnimationEnabledExperimental(true);
```

```SnackPlayer name=LayoutAnimations&supportedPlatforms=ios,android
import React from 'react';
import {
  NativeModules,
  LayoutAnimation,
  Text,
  TouchableOpacity,
  StyleSheet,
  View,
} from 'react-native';

const {UIManager} = NativeModules;

UIManager.setLayoutAnimationEnabledExperimental &&
  UIManager.setLayoutAnimationEnabledExperimental(true);

export default class App extends React.Component {
  state = {
    w: 100,
    h: 100,
  };

  _onPress = () => {
    // Animate the update
    LayoutAnimation.spring();
    this.setState({w: this.state.w + 15, h: this.state.h + 15});
  };

  render() {
    return (
      <View style={styles.container}>
        <View
          style={[styles.box, {width: this.state.w, height: this.state.h}]}
        />
        <TouchableOpacity onPress={this._onPress}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>Press me!</Text>
          </View>
        </TouchableOpacity>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  box: {
    width: 200,
    height: 200,
    backgroundColor: 'red',
  },
  button: {
    backgroundColor: 'black',
    paddingHorizontal: 20,
    paddingVertical: 15,
    marginTop: 15,
  },
  buttonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});
```

此範例使用預設值，你可根據需求自訂動畫，詳見 [LayoutAnimation.js](https://github.com/facebook/react-native/blob/main/packages/react-native/Libraries/LayoutAnimation/LayoutAnimation.js) 以獲取更多資訊。

## 補充說明

### `requestAnimationFrame`

`requestAnimationFrame` 是來自瀏覽器的 polyfill，你可能對它很熟悉。它接受一個函式作為唯一參數，並在下一次重繪前調用該函式。它是所有基於 JavaScript 的動畫 API 的基礎建構塊。通常你不需要自行調用它——動畫 API 會為你管理影格更新。

### `setNativeProps`

如[直接操作章節](direct-manipulation)所述，`setNativeProps` 允許我們直接修改原生支援元件（實際由原生視圖支援的元件，而非複合元件）的屬性，而無需透過 `setState` 和重新渲染元件樹。

我們可以在 Rebound 範例中使用此方法來更新縮放比例——當更新的元件層級很深且未經 `shouldComponentUpdate` 優化時，這會特別有用。

如果您發現動畫出現掉幀（低於每秒60幀）的情況，可以考慮使用 `setNativeProps` 或 `shouldComponentUpdate` 來優化。或者，您也可以選擇在UI執行緒而非JavaScript執行緒上運行動畫（透過[useNativeDriver選項](/blog/2017/02/14/using-native-driver-for-animated)）。此外，建議將任何計算密集型的工作延遲到動畫完成後再進行，可使用[InteractionManager](interactionmanager)來實現。您還可以使用應用內開發選單中的「FPS監測」工具來監控幀率。