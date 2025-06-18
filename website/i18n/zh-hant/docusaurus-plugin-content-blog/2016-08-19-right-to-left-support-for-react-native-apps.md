---
title: Right-to-Left Layout Support For React Native Apps
author: Mengjue (Mandy) Wang
authorTitle: Software Engineer Intern at Facebook
authorURL: 'https://github.com/MengjueW'
authorImageURL: 'https://avatars0.githubusercontent.com/u/13987140?v=3&s=128'
tags: [engineering]
---

在將應用程式發布至應用商店後，國際化是擴大受眾範圍的下一步。全球有超過20個國家和無數人口使用從右至左（RTL）的語言。因此，讓您的應用程式支援RTL對他們來說是必要的。

我們很高興地宣布，React Native已經改進以支援RTL佈局。這項功能現在已經可以在[react-native](https://github.com/facebook/react-native)的主分支中使用，並且將在下一個RC版本：[`v0.33.0-rc`](https://github.com/facebook/react-native/releases)中提供。

這涉及更改[css-layout](https://github.com/facebook/css-layout)——RN使用的核心佈局引擎，以及RN核心實現，還有特定的開源JS元件以支援RTL。

為了在生產環境中測試RTL支援，最新版本的**Facebook廣告管理員**應用（第一個跨平台100% RN應用）現在已經在阿拉伯語和希伯來語中提供RTL佈局，適用於[iOS](https://itunes.apple.com/app/id964397083)和[Android](https://play.google.com/store/apps/details?id=com.facebook.adsmanager)。以下是這些RTL語言中的外觀：

<>
<img src="/blog/assets/rtl-ama-ios-arabic.png" width={280} style={{ margin: 10 }} />
<img src="/blog/assets/rtl-ama-android-hebrew.png" width={280} style={{ margin: 10 }} />
</>

## RN中RTL支援的概述變更

[css-layout](https://github.com/facebook/css-layout)已經有`start`和`end`的佈局概念。在從左至右（LTR）佈局中，`start`意味著`left`，而`end`意味著`right`。但在RTL中，`start`意味著`right`，而`end`意味著`left`。這意味著我們可以讓RN依賴於`start`和`end`的計算來得出正確的佈局，這包括`position`、`padding`和`margin`。

此外，[css-layout](https://github.com/facebook/css-layout)已經讓每個元件的方向繼承自其父元件。這意味著，我們只需將根元件的方向設置為RTL，整個應用就會翻轉。

下圖從高層次描述了這些變更：

![](/blog/assets/rtl-rn-core-updates.png)

這些包括：

- [css-layout對絕對定位的RTL支援](https://github.com/facebook/css-layout/commit/46c842c71a1232c3c78c4215275d104a389a9a0f)
- 在RN核心實現中將`left`和`right`映射到`start`和`end`以用於陰影節點
- 並公開一個[橋接的實用模組](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js)來幫助控制RTL佈局

通過這次更新，當您允許應用程式使用RTL佈局時：

- 每個元件的佈局將水平翻轉
- 如果您使用RTL-ready的開源元件，某些手勢和動畫將自動具有RTL佈局
- 可能需要最少的額外努力來讓您的應用完全支援RTL

## 讓應用程式支援RTL

1. To support RTL, you should first add the RTL language bundles to your app.

   - See the general guides from [iOS](https://developer.apple.com/library/ios/documentation/MacOSX/Conceptual/BPInternational/LocalizingYourApp/LocalizingYourApp.html#//apple_ref/doc/uid/10000171i-CH5-SW1) and [Android](https://developer.android.com/training/basics/supporting-devices/languages.html).

2. Allow RTL layout for your app by calling the `allowRTL()` function at the beginning of native code. We provided this utility to only apply to an RTL layout when your app is ready. Here is an example:

   iOS:

   ```objc
   // in AppDelegate.m
     [[RCTI18nUtil sharedInstance] allowRTL:YES];
   ```

   Android:

   ```java
   // in MainActivity.java
     I18nUtil sharedI18nUtilInstance = I18nUtil.getInstance();
     sharedI18nUtilInstance.allowRTL(context, true);
   ```

3. For Android, you need add `android:supportsRtl="true"` to the [`<application>`](https://developer.android.com/guide/topics/manifest/application-element.html) element in `AndroidManifest.xml` file.

Now, when you recompile your app and change the device language to an RTL language (e.g. Arabic or Hebrew), your app layout should change to RTL automatically.

## Writing RTL-ready Components

In general, most components are already RTL-ready, for example:

- Left-to-Right Layout

<img src="/blog/assets/rtl-demo-listitem-ltr.png" width="300" />

- Right-to-Left Layout

<img src="/blog/assets/rtl-demo-listitem-rtl.png" width="300" />

However, there are several cases to be aware of, for which you will need the [`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js). In [`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js), there is a constant `isRTL` to tell if layout of app is RTL or not, so that you can make the necessary changes according to the layout.

#### Icons with Directional Meaning

If your component has icons or images, they will be displayed the same way in LTR and RTL layout, because RN will not flip your source image. Therefore, you should flip them according to the layout style.

- Left-to-Right Layout

<img src="/blog/assets/rtl-demo-icon-ltr.png" width="300" />

- Right-to-Left Layout

<img src="/blog/assets/rtl-demo-icon-rtl.png" width="300" />

Here are two ways to flip the icon according to the direction:

- Adding a `transform` style to the image component:

  ```jsx
  <Image
    source={...}
    style={{transform: [{scaleX: I18nManager.isRTL ? -1 : 1}]}}
  />
  ```

- Or, changing the image source according to the direction:

  ```jsx
  let imageSource = require('./back.png');
  if (I18nManager.isRTL) {
    imageSource = require('./forward.png');
  }
  return <Image source={imageSource} />;
  ```

#### Gestures and Animations

In Android and iOS development, when you change to RTL layout, the gestures and animations are the opposite of LTR layout. Currently, in RN, gestures and animations are not supported on RN core code level, but on components level. The good news is, some of these components already support RTL today, such as [`SwipeableRow`](https://github.com/facebook/react-native/blob/38a6eec0db85a5204e85a9a92b4dee2db9641671/Libraries/Experimental/SwipeableRow/SwipeableRow.js) and [`NavigationExperimental`](https://github.com/facebook/react-native/tree/master/Libraries/NavigationExperimental). However, other components with gestures will need to support RTL manually.

A good example to illustrate gesture RTL support is [`SwipeableRow`](https://github.com/facebook/react-native/blob/38a6eec0db85a5204e85a9a92b4dee2db9641671/Libraries/Experimental/SwipeableRow/SwipeableRow.js).

<p align="center">
  <img src="/blog/assets/rtl-demo-swipe-ltr.png" width={280} style={{margin: 10}} />
  <img src="/blog/assets/rtl-demo-swipe-rtl.png" width={280} style={{margin: 10}} />
</p>

##### Gestures Example

```js
// SwipeableRow.js
_isSwipingExcessivelyRightFromClosedPosition(gestureState: Object): boolean {
  // ...
  const gestureStateDx = IS_RTL ? -gestureState.dx : gestureState.dx;
  return (
    this._isSwipingRightFromClosed(gestureState) &&
    gestureStateDx > RIGHT_SWIPE_THRESHOLD
  );
},
```

##### Animation Example

```js
// SwipeableRow.js
_animateBounceBack(duration: number): void {
  // ...
  const swipeBounceBackDistance = IS_RTL ?
    -RIGHT_SWIPE_BOUNCE_BACK_DISTANCE :
    RIGHT_SWIPE_BOUNCE_BACK_DISTANCE;
  this._animateTo(
    -swipeBounceBackDistance,
    duration,
    this._animateToClosedPositionDuringBounce,
  );
},
```

## Maintaining Your RTL-ready App

Even after the initial RTL-compatible app release, you will likely need to iterate on new features. To improve development efficiency, [`I18nManager`](https://github.com/facebook/react-native/blob/f0fb228ec76ed49e6ed6d786d888e8113b8959a2/Libraries/Utilities/I18nManager.js) provides the `forceRTL()` function for faster RTL testing without changing the test device language. You might want to provide a simple switch for this in your app. Here's an example from the RTL example in the RNTester:

<p align="center">
  <img src="/blog/assets/rtl-demo-forcertl.png" width="300" />
</p>

```js
<RNTesterBlock title={'Quickly Test RTL Layout'}>
  <View style={styles.flexDirectionRow}>
    <Text style={styles.switchRowTextView}>forceRTL</Text>
    <View style={styles.switchRowSwitchView}>
      <Switch
        onValueChange={this._onDirectionChange}
        style={styles.rightAlignStyle}
        value={this.state.isRTL}
      />
    </View>
  </View>
</RNTesterBlock>;

_onDirectionChange = () => {
  I18nManager.forceRTL(!this.state.isRTL);
  this.setState({isRTL: !this.state.isRTL});
  Alert.alert(
    'Reload this page',
    'Please reload this page to change the UI direction! ' +
      'All examples in this app will be affected. ' +
      'Check them out to see what they look like in RTL layout.',
  );
};
```

When working on a new feature, you can easily toggle this button and reload the app to see RTL layout. The benefit is you won't need to change the language setting to test, however some text alignment won't change, as explained in the next section. Therefore, it's always a good idea to test your app in the RTL language before launching.

## Limitations and Future Plan

The RTL support should cover most of the UX in your app; however, there are some limitations for now:

- Text alignment behaviors differ in Android and iOS
  - In iOS, the default text alignment depends on the active language bundle, they are consistently on one side. In Android, the default text alignment depends on the language of the text content, i.e. English will be left-aligned and Arabic will be right-aligned.
  - In theory, this should be made consistent across platform, but some people may prefer one behavior to another when using an app. More user experience research may be needed to find out the best practice for text alignment.

* There is no "true" left/right

  As discussed before, we map the `left`/`right` styles from the JS side to `start`/`end`, all `left` in code for RTL layout becomes "right" on screen, and `right` in code becomes "left" on screen. This is convenient because you don't need to change your product code too much, but it means there is no way to specify "true left" or "true right" in the code. In the future, allowing a component to control its direction regardless of the language may be necessary.

* Make RTL support for gestures and animations more developer friendly

  Currently, there is still some programming effort required to make gestures and animations RTL compatible. In the future, it would be ideal to find a way to make gestures and animations RTL support more developer friendly.

## Try it Out!

Check out the [`RTLExample`](https://github.com/facebook/react-native/blob/master/packages/rn-tester/js/examples/RTL/RTLExample.js) in the `RNTester` to understand more about RTL support, and let us know how it works for you!

Finally, thank you for reading! We hope that the RTL support for React Native helps you grow your apps for international audience!