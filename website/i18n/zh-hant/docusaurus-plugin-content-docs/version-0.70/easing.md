---
id: easing
title: Easing
---

`Easing` 模組實現了常見的緩動函數。該模組被 [`Animated.timing()`](animated.md#timing) 用於在動畫中傳達物理可信的運動效果。

您可以在 http://easings.net/ 找到一些常見緩動函數的可視化示例。

### 預定義動畫

`Easing` 模組通過以下方法提供了幾種預定義的動畫效果：

- [`back`](easing.md#back) 提供基礎回彈動畫，物體會先略微後退再前進
- [`bounce`](easing.md#bounce) 提供彈跳動畫效果
- [`ease`](easing.md#ease) 提供基礎慣性動畫
- [`elastic`](easing.md#elastic) 提供基礎彈簧交互效果

### 標準函數

提供三種標準緩動函數：

- [`linear`](easing.md#linear)
- [`quad`](easing.md#quad)
- [`cubic`](easing.md#cubic)

[`poly`](easing.md#poly) 函數可用於實現四次方、五次方等高階冪函數。

### 附加函數

以下方法提供額外的數學函數：

- [`bezier`](easing.md#bezier) 提供三次貝塞爾曲線
- [`circle`](easing.md#circle) 提供圓形函數
- [`sin`](easing.md#sin) 提供正弦函數
- [`exp`](easing.md#exp) 提供指數函數

以下輔助函數用於修改其他緩動函數的行為：

- [`in`](easing.md#in) 正向執行緩動函數
- [`inOut`](easing.md#inout) 使任何緩動函數對稱執行
- [`out`](easing.md#out) 反向執行緩動函數

## 範例

```SnackPlayer name=Easing%20Demo&supportedPlatforms=ios,android
import React from "react";
import { Animated, Easing, SectionList, StatusBar, StyleSheet, Text, TouchableOpacity, View } from "react-native";

const App = () => {
  let opacity = new Animated.Value(0);

  const animate = easing => {
    opacity.setValue(0);
    Animated.timing(opacity, {
      toValue: 1,
      duration: 1200,
      easing
    }).start();
  };

  const size = opacity.interpolate({
    inputRange: [0, 1],
    outputRange: [0, 80]
  });

  const animatedStyles = [
    styles.box,
    {
      opacity,
      width: size,
      height: size
    }
  ];

  return (
    <View style={styles.container}>
      <StatusBar hidden={true} />
      <Text style={styles.title}>
        Press rows below to preview the Easing!
      </Text>
      <View style={styles.boxContainer}>
        <Animated.View style={animatedStyles} />
      </View>
      <SectionList
        style={styles.list}
        sections={SECTIONS}
        keyExtractor={(item) => item.title}
        renderItem={({ item }) => (
          <TouchableOpacity
            onPress={() => animate(item.easing)}
            style={styles.listRow}
          >
            <Text>{item.title}</Text>
          </TouchableOpacity>
        )}
        renderSectionHeader={({ section: { title } }) => (
          <Text style={styles.listHeader}>{title}</Text>
        )}
      />
    </View>
  );
};

const SECTIONS = [
  {
    title: "Predefined animations",
    data: [
      { title: "Bounce", easing: Easing.bounce },
      { title: "Ease", easing: Easing.ease },
      { title: "Elastic", easing: Easing.elastic(4) }
    ]
  },
  {
    title: "Standard functions",
    data: [
      { title: "Linear", easing: Easing.linear },
      { title: "Quad", easing: Easing.quad },
      { title: "Cubic", easing: Easing.cubic }
    ]
  },
  {
    title: "Additional functions",
    data: [
      {
        title: "Bezier",
        easing: Easing.bezier(0, 2, 1, -1)
      },
      { title: "Circle", easing: Easing.circle },
      { title: "Sin", easing: Easing.sin },
      { title: "Exp", easing: Easing.exp }
    ]
  },
  {
    title: "Combinations",
    data: [
      {
        title: "In + Bounce",
        easing: Easing.in(Easing.bounce)
      },
      {
        title: "Out + Exp",
        easing: Easing.out(Easing.exp)
      },
      {
        title: "InOut + Elastic",
        easing: Easing.inOut(Easing.elastic(1))
      }
    ]
  }
];

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#20232a"
  },
  title: {
    marginTop: 10,
    textAlign: "center",
    color: "#61dafb"
  },
  boxContainer: {
    height: 160,
    alignItems: "center"
  },
  box: {
    marginTop: 32,
    borderRadius: 4,
    backgroundColor: "#61dafb"
  },
  list: {
    backgroundColor: "#fff"
  },
  listHeader: {
    paddingHorizontal: 8,
    paddingVertical: 4,
    backgroundColor: "#f4f4f4",
    color: "#999",
    fontSize: 12,
    textTransform: "uppercase"
  },
  listRow: {
    padding: 8
  }
});

export default App;
```

---

# 參考文檔

## 方法

### `step0()`

```jsx
static step0(n)
```

階躍函數，對於任何正數 `n` 都返回 1。

---

### `step1()`

```jsx
static step1(n)
```

階躍函數，當 `n` 大於或等於 1 時返回 1。

---

### `linear()`

```jsx
static linear(t)
```

線性函數，`f(t) = t`。位置與經過時間呈一比一對應關係。

http://cubic-bezier.com/#0,0,1,1

---

### `ease()`

```jsx
static ease(t)
```

基礎慣性交互效果，類似物體緩慢加速至終速的過程。

http://cubic-bezier.com/#.42,0,1,1

---

### `quad()`

```jsx
static quad(t)
```

二次函數，`f(t) = t * t`。位置等於經過時間的平方。

http://easings.net/#easeInQuad

---

### `cubic()`

```jsx
static cubic(t)
```

三次函數，`f(t) = t * t * t`。位置等於經過時間的立方。

http://easings.net/#easeInCubic

---

### `poly()`

```jsx
static poly(n)
```

一個冪函數。位置等於經過時間的N次方。

n = 4: http://easings.net/#easeInQuart n = 5: http://easings.net/#easeInQuint

---

### `sin()`

```jsx
static sin(t)
```

一個正弦函數。

http://easings.net/#easeInSine

---

### `circle()`

```jsx
static circle(t)
```

一個圓形函數。

http://easings.net/#easeInCirc

---

### `exp()`

```jsx
static exp(t)
```

一個指數函數。

http://easings.net/#easeInExpo

---

### `elastic()`

```jsx
static elastic(bounciness)
```

一個基本的彈性交互，類似於彈簧來回振盪。

默認彈性係數為1，會稍微過衝一次。彈性係數為0則完全不會過衝，而彈性係數N > 1會過衝大約N次。

http://easings.net/#easeInElastic

---

### `back()`

```jsx
static back(s)
```

與`Animated.parallel()`一起使用，創建一個基本效果，對象在動畫開始時稍微向後移動。

---

### `bounce()`

```jsx
static bounce(t)
```

提供一個基本的彈跳效果。

http://easings.net/#easeInBounce

---

### `bezier()`

```jsx
static bezier(x1, y1, x2, y2)
```

提供一個立方貝茲曲線，等同於CSS過渡的`transition-timing-function`。

可視化立方貝茲曲線的有用工具可以在 http://cubic-bezier.com/ 找到。

---

### `in()`

<!-- prettier-ignore-start -->

```jsx
static in(easing);
```

<!-- prettier-ignore-end -->

向前運行一個緩動函數。

---

### `out()`

```jsx
static out(easing)
```

向後運行一個緩動函數。

---

### `inOut()`

```jsx
static inOut(easing)
```

使任何緩動函數對稱。緩動函數將在前半段時間向前運行，然後在剩餘時間向後運行。