---
title: 'Implementing Twitter’s App Loading Animation in React Native'
authors: [Eli White]
tags: [engineering]
---

Twitter’s iOS app has a loading animation I quite enjoy.

<img src="/blog/assets/loading-screen-01.gif" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

Once the app is ready, the Twitter logo delightfully expands, revealing the app.

I wanted to figure out how to recreate this loading animation with React Native.

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

To understand _how_ to build it, I first had to understand the difference pieces of the loading animation. The easiest way to see the subtlety is to slow it down.

<img src="/blog/assets/loading-screen-02.gif" style={{marginTop: 20, float: 'left', paddingRight: 80, paddingBottom: 20}} />

There are a few major pieces in this that we will need to figure out how to build.

1. Scaling the bird.
1. As the bird grows, showing the app underneath
1. Scaling the app down slightly at the end

It took me quite a while to figure out how to make this animation.

I started with an _incorrect_ assumption that the blue background and Twitter bird were a layer on _top_ of the app and that as the bird grew, it became transparent which revealed the app underneath. This approach doesn’t work because the Twitter bird becoming transparent would show the blue layer, not the app underneath!

Luckily for you, dear reader, you don’t have to go through the same frustration I did. You get this nice tutorial skipping to the good stuff!

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

## The right way

Before we get to code, it is important to understand how to break this down. To help visualize this effect, I recreated it in [CodePen](https://codepen.io/TheSavior/pen/NXNoJM) (embedded in a few paragraphs) so you can interactively see the different layers.

<img src="/blog/assets/loading-screen-03.png" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

There are three main layers to this effect. The first is the blue background layer. Even though this seems to appear on top of the app, it is actually in the back.

We then have a plain white layer. And then lastly, in the very front, is our app.

<hr style={{clear: 'both', marginBottom: 40, width: 80}} />

<img src="/blog/assets/loading-screen-04.png" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

The main trick to this animation is using the Twitter logo as a `mask` and masking both the app, and the white layer. I won’t go too deep on the details of masking, there are [plenty](https://www.html5rocks.com/en/tutorials/masking/adobe/) of [resources](https://designshack.net/articles/graphics/a-complete-beginners-guide-to-masking-in-photoshop/) [online](https://www.sketchapp.com/docs/shapes/masking/) for that.

The basics of masking in this context are having images where opaque pixels of the mask show the content they are masking whereas transparent pixels of the mask hide the content they are masking.

We use the Twitter logo as a mask, and having it mask two layers; the solid white layer, and the app layer.

To reveal the app, we scale the mask up until it is larger than the entire screen.

While the mask is scaling up, we fade in the opacity of the app layer, showing the app and hiding the solid white layer behind it. To finish the effect, we start the app layer at a scale > 1, and scale it down to 1 as the animation is ending. We then hide the non-app layers as they will never be seen again.

They say a picture is worth 1,000 words. How many words is an interactive visualization worth? Click through the animation with the “Next Step” button. Showing the layers gives you a side view perspective. The grid is there to help visualize the transparent layers.

<iframe
  height="750"
  scrolling="no"
  title="Loading Screen Animation Steps"
  src="//codepen.io/TheSavior/embed/NXNoJM/?height=265&theme-id=0&default-tab=result&embed-version=2"
  frameborder="no"
  allowFullScreen={true}
  className="codepen">
  See the Pen{' '}
  <a href="https://codepen.io/TheSavior/pen/NXNoJM/">
    Loading Screen Animation Steps
  </a>
  {' '}by Eli White (
  <a href="https://codepen.io/TheSavior">@TheSavior</a>) on{' '}
  <a href="https://codepen.io">CodePen</a>.
</iframe>

## Now, for the React Native

Alrighty. Now that we know what we are building and how the animation works, we can get down to the code — the reason you are really here.

The main piece of this puzzle is [MaskedViewIOS](https://reactnative.dev/docs/0.63/maskedviewios), a core React Native component.

```jsx
import {MaskedViewIOS} from 'react-native';

<MaskedViewIOS maskElement={<Text>Basic Mask</Text>}>
  <View style={{backgroundColor: 'blue'}} />
</MaskedViewIOS>;
```

`MaskedViewIOS` takes props `maskElement` and `children`. The children are masked by the `maskElement`. Note that the mask doesn’t need to be an image, it can be any arbitrary view. The behavior of the above example would be to render the blue view, but for it to be visible only where the words “Basic Mask” are from the `maskElement`. We just made complicated blue text.

What we want to do is render our blue layer, and then on top render our masked app and white layers with the Twitter logo.

```jsx
{
  fullScreenBlueLayer;
}
<MaskedViewIOS
  style={{flex: 1}}
  maskElement={
    <View style={styles.centeredFullScreen}>
      <Image source={twitterLogo} />
    </View>
  }>
  {fullScreenWhiteLayer}
  <View style={{flex: 1}}>
    <MyApp />
  </View>
</MaskedViewIOS>;
```

This will give us the layers we see below.

<img src="/blog/assets/loading-screen-04.png" style={{marginLeft: 'auto', marginRight: 'auto', display: 'block'}} />

## Now for the Animated part

We have all the pieces we need to make this work, the next step is animating them. To make this animation feel good, we will be utilizing React Native’s [Animated](/docs/animated) API.

Animated lets us define our animations declaratively in JavaScript. By default, these animations run in JavaScript and tell the native layer what changes to make on every frame. Even though JavaScript will try to update the animation every frame, it will likely not be able to do that fast enough and will cause dropped frames (jank) to occur. Not what we want!

Animated has special behavior to allow you to get animations without this jank. Animated has a flag called `useNativeDriver` which sends your animation definition from JavaScript to native at the beginning of your animation, allowing the native side to process the updates to your animation without having to go back and forth to JavaScript every frame. The downside of `useNativeDriver` is you can only update a specific set of properties, mostly `transform` and `opacity`. You can’t animate things like background color with `useNativeDriver`, at least not yet — we will add more over time, and of course you can always submit a PR for properties you need for your project, benefitting the whole community 😀.

Since we want this animation to be smooth, we will work within these constraints. For a more in depth look at how `useNativeDriver` works under the hood, check out our [blog post announcing it](/blog/2017/02/14/using-native-driver-for-animated).

## Breaking down our animation

There are 4 components to our animation:

1. Enlarge the bird, revealing the app and the solid white layer
1. Fade in the app
1. Scale down the app
1. Hide the white layer and blue layer when it is done

With Animated, there are two main ways to define your animation. The first is by using `Animated.timing` which lets you say exactly how long your animation will run for, along with an easing curve to smooth out the motion. The other approach is by using the physics based apis such as `Animated.spring`. With `Animated.spring`, you specify parameters like the amount of friction and tension in the spring, and let physics run your animation.

We have multiple animations we want to be running at the same time which are all closely related to each other. For example, we want the app to start fading in while the mask is mid-reveal. Because these animations are closely related, we will use `Animated.timing` with a single `Animated.Value`.

`Animated.Value` is a wrapper around a native value that Animated uses to know the state of an animation. You typically want to only have one of these for a complete animation. Most components that use Animated will store the value in state.

Since I’m thinking about this animation as steps occurring at different points in time along the complete animation, we will start our `Animated.Value` at 0, representing 0% complete, and end our value at 100, representing 100% complete.

Our initial component state will be the following.

```jsx
state = {
  loadingProgress: new Animated.Value(0),
};
```

當我們準備開始動畫時，我們會告訴 Animated 將這個值動畫化到 100。

```jsx
Animated.timing(this.state.loadingProgress, {
  toValue: 100,
  duration: 1000,
  useNativeDriver: true, // This is important!
}).start();
```

接著我嘗試估算動畫的各個部分，以及在不同階段我希望它們達到的數值。以下是一個表格，顯示了動畫的各個部分，以及我認為它們在動畫進展的不同時間點應該具有的數值。

![](/blog/assets/loading-screen-05.png)

Twitter 鳥形遮罩應該從縮放比例 1 開始，在放大之前會先變小。因此，在動畫進行到 10% 時，它的縮放值應該是 0.8，然後在結束時放大到 70。選擇 70 其實相當隨意，坦白說，它需要足夠大才能讓鳥完全顯示畫面，而 60 還不夠大 😀。有趣的是，數字越大，它看起來增長得越快，因為它必須在相同的時間內達到目標。這個數字經過了一些嘗試和錯誤，才能讓這個標誌看起來效果良好。不同大小的標誌或設備將需要不同的最終縮放比例，以確保整個畫面都能顯示出來。

應用程式應該保持不透明一段時間，至少在 Twitter 標誌變小的過程中。根據官方動畫，我希望在鳥標誌放大到一半時開始顯示它，並在整體動畫進行到 30% 時完全顯示。

應用程式的縮放比例從 1.1 開始，並在動畫結束時縮小到正常比例。

## 現在，用代碼實現。

我們上面所做的本質上是將動畫進度百分比的值映射到各個部分的數值。我們使用 Animated 的 `.interpolate` 方法來實現這一點。我們創建了三個不同的樣式對象，每個對應動畫的一個部分，使用基於 `this.state.loadingProgress` 的插值數值。

```jsx
const loadingProgress = this.state.loadingProgress;

const opacityClearToVisible = {
  opacity: loadingProgress.interpolate({
    inputRange: [0, 15, 30],
    outputRange: [0, 0, 1],
    extrapolate: 'clamp',
    // clamp means when the input is 30-100, output should stay at 1
  }),
};

const imageScale = {
  transform: [
    {
      scale: loadingProgress.interpolate({
        inputRange: [0, 10, 100],
        outputRange: [1, 0.8, 70],
      }),
    },
  ],
};

const appScale = {
  transform: [
    {
      scale: loadingProgress.interpolate({
        inputRange: [0, 100],
        outputRange: [1.1, 1],
      }),
    },
  ],
};
```

現在我們有了這些樣式對象，可以在渲染文章前面提到的視圖片段時使用它們。注意，只有 `Animated.View`、`Animated.Text` 和 `Animated.Image` 能夠使用包含 `Animated.Value` 的樣式對象。

```jsx
const fullScreenBlueLayer = (
  <View style={styles.fullScreenBlueLayer} />
);
const fullScreenWhiteLayer = (
  <View style={styles.fullScreenWhiteLayer} />
);

return (
  <View style={styles.fullScreen}>
    {fullScreenBlueLayer}
    <MaskedViewIOS
      style={{flex: 1}}
      maskElement={
        <View style={styles.centeredFullScreen}>
          <Animated.Image
            style={[styles.maskImageStyle, imageScale]}
            source={twitterLogo}
          />
        </View>
      }>
      {fullScreenWhiteLayer}
      <Animated.View
        style={[opacityClearToVisible, appScale, {flex: 1}]}>
        {this.props.children}
      </Animated.View>
    </MaskedViewIOS>
  </View>
);
```

<img src="/blog/assets/loading-screen-06.gif" style={{float: 'left', paddingRight: 80, paddingBottom: 20}} />

太好了！現在我們的動畫部分看起來符合預期。接下來只需要清理那些永遠不會再看到的藍色和白色圖層。

要知道何時可以清理它們，我們需要知道動畫何時完成。幸運的是，在我們調用 `Animated.timing` 的地方，`.start` 方法接受一個可選的回調函數，該函數會在動畫完成時運行。

```jsx
Animated.timing(this.state.loadingProgress, {
  toValue: 100,
  duration: 1000,
  useNativeDriver: true,
}).start(() => {
  this.setState({
    animationDone: true,
  });
});
```

現在我們有了一個 `state` 中的值來判斷動畫是否完成，可以修改我們的藍色和白色圖層來使用它。

```jsx
const fullScreenBlueLayer = this.state.animationDone ? null : (
  <View style={[styles.fullScreenBlueLayer]} />
);
const fullScreenWhiteLayer = this.state.animationDone ? null : (
  <View style={[styles.fullScreenWhiteLayer]} />
);
```

瞧！我們的動畫現在可以正常運作，並且在動畫完成後清理了未使用的圖層。我們已經成功構建了 Twitter 應用程式的加載動畫！

## 但是等等，我的動畫無法運作！

別擔心，親愛的讀者。我也討厭那些只提供代碼片段而不提供完整源碼的指南。

這個組件已經發布到 npm，並在 GitHub 上名為 [react-native-mask-loader](https://github.com/TheSavior/react-native-mask-loader)。如果你想在手機上試試看，可以在 [Expo](https://expo.io/@eliwhite/react-native-mask-loader-example) 上找到它：

<img src="/blog/assets/loading-screen-07.png" style={{marginLeft: 'auto', marginRight: 'auto', display: 'block'}} />

## 更多閱讀 / 額外練習

1. [這本 gitbook](https://browniefed.com/react-native-animation-book/) 是閱讀完 React Native 文檔後，深入學習 Animated 的絕佳資源。
1. 實際的 Twitter 動畫似乎會在接近尾聲時加速遮罩顯現。試著修改加載器，使用不同的緩動函數（或彈簧效果！）來更貼近該行為。
1. 目前遮罩的終止縮放比例是硬編碼的，在平板上可能無法完整顯現整個應用程式。根據螢幕尺寸和圖像大小計算終止縮放比例，會是一個很棒的 Pull Request。