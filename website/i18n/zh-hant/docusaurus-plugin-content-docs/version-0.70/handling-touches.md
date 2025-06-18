---
id: handling-touches
title: Handling Touches
---

使用者主要透過觸控與行動應用程式互動。他們可以使用多種手勢組合，例如點擊按鈕、滾動列表或縮放地圖。React Native 提供了處理各種常見手勢的元件，以及一套全面的[手勢回應系統](gesture-responder-system.md)來實現更進階的手勢識別，但您最可能感興趣的元件是基本的 Button（按鈕）。

## 顯示基本按鈕

[Button](button.md) 提供了一個基礎按鈕元件，可在所有平台上美觀呈現。顯示按鈕的最簡單範例如下：

```jsx
<Button
  onPress={() => {
    alert('You tapped the button!');
  }}
  title="Press Me"
/>
```

在 iOS 上會渲染藍色標籤，在 Android 上則會渲染帶有淺色文字的藍色圓角矩形。按下按鈕會呼叫 "onPress" 函數，本例中會顯示一個警示彈窗。如果需要，您可以指定 "color" 屬性來改變按鈕顏色。

![](/docs/assets/Button.png)

您可以使用下方範例實際操作 `Button` 元件。透過點擊右下角的切換按鈕選擇預覽平台，然後點擊「Tap to Play」預覽應用程式。

```SnackPlayer name=Button%20Basics
import React, { Component } from 'react';
import { Button, StyleSheet, View } from 'react-native';

export default class ButtonBasics extends Component {
  _onPressButton() {
    alert('You tapped the button!')
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
          />
        </View>
        <View style={styles.buttonContainer}>
          <Button
            onPress={this._onPressButton}
            title="Press Me"
            color="#841584"
          />
        </View>
        <View style={styles.alternativeLayoutButtonContainer}>
          <Button
            onPress={this._onPressButton}
            title="This looks great!"
          />
          <Button
            onPress={this._onPressButton}
            title="OK!"
            color="#841584"
          />
        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
   flex: 1,
   justifyContent: 'center',
  },
  buttonContainer: {
    margin: 20
  },
  alternativeLayoutButtonContainer: {
    margin: 20,
    flexDirection: 'row',
    justifyContent: 'space-between'
  }
});
```

## 可觸控元件

如果基礎按鈕不符合您的應用程式風格，您可以使用 React Native 提供的任何「Touchable」元件來自訂按鈕。這些元件能捕捉點擊手勢，並在手勢被識別時顯示反饋效果。但請注意，這些元件不提供預設樣式，因此您需要自行設計以符合應用程式外觀。

根據您想提供的反饋類型，可選擇不同的「Touchable」元件：

- 通常，[**TouchableHighlight**](touchablehighlight.md) 適用於任何需要網頁按鈕或連結效果的場景。使用者按下時，視圖背景會變暗。
  
- 在 Android 平台上，可考慮使用 [**TouchableNativeFeedback**](touchablenativefeedback.md) 來顯示墨水漣漪效果，回應使用者的觸控操作。

- 使用 [**TouchableOpacity**](touchableopacity.md) 可透過降低按鈕不透明度來提供反饋，讓背景在使用者按壓時顯現。

- 若需要處理點擊手勢但不希望顯示任何視覺反饋，請使用 [**TouchableWithoutFeedback**](touchablewithoutfeedback.md)。

某些情況下，您可能需要檢測使用者長按視圖的行為。在任何「Touchable」元件的 `onLongPress` 屬性中傳遞函數，即可處理這類長按手勢。

以下是這些元件的實際範例：

```SnackPlayer name=Touchables
import React, { Component } from 'react';
import { Platform, StyleSheet, Text, TouchableHighlight, TouchableOpacity, TouchableNativeFeedback, TouchableWithoutFeedback, View } from 'react-native';

export default class Touchables extends Component {
  _onPressButton() {
    alert('You tapped the button!')
  }

  _onLongPressButton() {
    alert('You long-pressed the button!')
  }


  render() {
    return (
      <View style={styles.container}>
        <TouchableHighlight onPress={this._onPressButton} underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableHighlight</Text>
          </View>
        </TouchableHighlight>
        <TouchableOpacity onPress={this._onPressButton}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableOpacity</Text>
          </View>
        </TouchableOpacity>
        <TouchableNativeFeedback
            onPress={this._onPressButton}
            background={Platform.OS === 'android' ? TouchableNativeFeedback.SelectableBackground() : ''}>
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableNativeFeedback {Platform.OS !== 'android' ? '(Android only)' : ''}</Text>
          </View>
        </TouchableNativeFeedback>
        <TouchableWithoutFeedback
            onPress={this._onPressButton}
            >
          <View style={styles.button}>
            <Text style={styles.buttonText}>TouchableWithoutFeedback</Text>
          </View>
        </TouchableWithoutFeedback>
        <TouchableHighlight onPress={this._onPressButton} onLongPress={this._onLongPressButton} underlayColor="white">
          <View style={styles.button}>
            <Text style={styles.buttonText}>Touchable with Long Press</Text>
          </View>
        </TouchableHighlight>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    paddingTop: 60,
    alignItems: 'center'
  },
  button: {
    marginBottom: 30,
    width: 260,
    alignItems: 'center',
    backgroundColor: '#2196F3'
  },
  buttonText: {
    textAlign: 'center',
    padding: 20,
    color: 'white'
  }
});
```

## 滾動與滑動

觸控螢幕裝置上常見的手勢包括滑動（swipe）和拖曳（pan），這些手勢讓使用者能滾動項目列表或滑動瀏覽內容頁面。相關功能請參考核心元件 [ScrollView](scrollview.md)。

## 已知問題

- [react-native#29308](https://github.com/facebook/react-native/issues/29308#issuecomment-792864162)：觸控區域不會延伸至父視圖邊界之外，且 Android 不支援負邊距設定。