---
id: improvingux
title: Improving User Experience
---

## 設定文字輸入框

在觸控手機上輸入文字是一項挑戰——螢幕小、軟體鍵盤佔空間。但根據您需要的資料類型，您可以透過適當設定文字輸入框來簡化操作：

- 自動聚焦第一個欄位
- 使用預留文字作為預期資料格式的範例
- 啟用或停用自動大寫與自動校正
- 選擇鍵盤類型（例如電子郵件、數字鍵盤）
- 確保返回按鈕能聚焦下一個欄位或提交表單

更多設定選項請參閱[`TextInput` 文件](textinput.md)。

```SnackPlayer name=TextInput%20form%20example
import React, { useState, useRef } from 'react';
import { Text, StatusBar, TextInput, View, StyleSheet } from 'react-native';
import { Constants } from 'expo';

const App = () => {
  const emailInput = useRef(null);
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');

  const submit = () => {
    alert(`Welcome, ${name}! Confirmation email has been sent to ${email}`);
  };

  return (
    <View style={styles.container}>
      <StatusBar barStyle="light-content" />
      <View style={styles.header}>
        <Text style={styles.description}>
          This demo shows how using available TextInput customizations can make
          forms much easier to use. Try completing the form and notice that
          different fields have specific optimizations and the return key
          changes from focusing next input to submitting the form.
        </Text>
      </View>
      <TextInput
        style={styles.input}
        value={name}
        onChangeText={(name) => setName(name)}
        placeholder="Full Name"
        autoFocus={true}
        autoCapitalize="words"
        autoCorrect={true}
        keyboardType="default"
        returnKeyType="next"
        onSubmitEditing={() => emailInput.current.focus()}
        blurOnSubmit={false}
      />
      <TextInput
        style={styles.input}
        value={email}
        onChangeText={(email) => setEmail(email)}
        ref={emailInput}
        placeholder="email@example.com"
        autoCapitalize="none"
        autoCorrect={false}
        keyboardType="email-address"
        returnKeyType="send"
        onSubmitEditing={submit}
        blurOnSubmit={true}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  header: {
    paddingTop: 64,
    padding: 20,
    backgroundColor: '#282c34',
  },
  description: {
    fontSize: 14,
    color: 'white',
  },
  input: {
    margin: 20,
    marginBottom: 0,
    height: 34,
    paddingHorizontal: 10,
    borderRadius: 4,
    borderColor: '#ccc',
    borderWidth: 1,
    fontSize: 16,
  },
});

export default App;
```

## 管理鍵盤顯示時的版面配置

軟體鍵盤會佔用近半個螢幕。若有可互動元素可能被鍵盤遮蓋，請使用[`KeyboardAvoidingView` 元件](keyboardavoidingview.md)確保這些元素仍可觸及。

```SnackPlayer name=KeyboardAvoidingView%20example
import React, { useState, useRef } from 'react';
import {
  Text,
  Button,
  StatusBar,
  TextInput,
  KeyboardAvoidingView,
  View,
  StyleSheet,
} from 'react-native';

const App = () => {
  const emailInput = useRef(null);
  const [email, setEmail] = useState('');

  const submit = () => {
    emailInput.current.blur();
    alert(`Confirmation email has been sent to ${email}`);
  };

  return (
    <View style={styles.container}>
      <StatusBar barStyle="light-content" />
      <View style={styles.header}>
        <Text style={styles.description}>
          This demo shows how to avoid covering important UI elements with the
          software keyboard. Focus the email input below and notice that the
          Sign Up button and the text adjusted positions to make sure they are
          not hidden under the keyboard.
        </Text>
      </View>
      <KeyboardAvoidingView behavior="padding" style={styles.form}>
        <TextInput
          style={styles.input}
          value={email}
          onChangeText={(email) => setEmail(email)}
          ref={emailInput}
          placeholder="email@example.com"
          autoCapitalize="none"
          autoCorrect={false}
          keyboardType="email-address"
          returnKeyType="send"
          onSubmitEditing={submit}
          blurOnSubmit={true}
        />
        <View>
          <Button title="Sign Up" onPress={submit} />
          <Text style={styles.legal}>Some important legal fine print here</Text>
        </View>
      </KeyboardAvoidingView>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  header: {
    paddingTop: 64,
    padding: 20,
    backgroundColor: '#282c34',
  },
  description: {
    fontSize: 14,
    color: 'white',
  },
  input: {
    margin: 20,
    marginBottom: 0,
    height: 34,
    paddingHorizontal: 10,
    borderRadius: 4,
    borderColor: '#ccc',
    borderWidth: 1,
    fontSize: 16,
  },
  legal: {
    margin: 10,
    color: '#333',
    fontSize: 12,
    textAlign: 'center',
  },
  form: {
    flex: 1,
    justifyContent: 'space-between',
  },
});

export default App;
```

## 擴大可點擊區域

在手機上精準點擊按鈕相當困難。請確保所有互動元素的尺寸至少為44x44像素。可透過保留足夠空間、使用`padding`、`minWidth`和`minHeight`樣式值達成，或使用[`hitSlop`屬性](touchablewithoutfeedback.md#hitslop)在不影響版面的情況下擴大觸控範圍。範例如下：

```SnackPlayer name=HitSlop%20example
import React, { Component } from 'react';
import {
  Text,
  StatusBar,
  TouchableOpacity,
  View,
  StyleSheet,
} from 'react-native';

const App = () => {
  return (
    <View style={styles.container}>
      <StatusBar barStyle="light-content" />
      <View style={styles.header}>
        <Text style={styles.description}>
          This demo shows how using hitSlop can make interactive elements much
          easier to tap without changing their layout and size. Try pressing
          each button quickly multiple times and notice which one is easier to
          hit.
        </Text>
      </View>
      <View style={styles.content}>
        <TouchableOpacity>
          <Text style={styles.label}>Without hitSlop</Text>
        </TouchableOpacity>
        <View style={styles.separator} />
        <View style={styles.preview}>
          <TouchableOpacity
            hitSlop={{ top: 20, left: 20, bottom: 20, right: 20 }}>
            <Text style={styles.label}>With hitSlop</Text>
          </TouchableOpacity>
        </View>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  header: {
    paddingTop: 64,
    padding: 20,
    backgroundColor: '#282c34',
  },
  description: {
    fontSize: 14,
    color: 'white',
  },
  content: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  label: {
    fontSize: 18,
    color: '#336699',
    textAlign: 'center',
    borderColor: '#ddd',
    borderWidth: 1,
  },
  separator: {
    height: 50,
  },
  preview: {
    padding: 20,
    backgroundColor: '#f6f6f6',
  },
});

export default App;
```

## 使用Android漣漪效果

Android API 21+ 採用材質設計的漣漪效果，在使用者觸碰螢幕互動區域時提供視覺回饋。React Native透過[`TouchableNativeFeedback`元件](touchablenativefeedback.md)實現此效果。相較於透明度或高亮效果，使用此觸控效果能讓應用更符合平台特性。但需注意此元件不支援iOS或Android API < 21，因此需在iOS上改用其他Touchable元件。您可使用[react-native-platform-touchable](https://github.com/react-community/react-native-platform-touchable)等函式庫自動處理平台差異。

```SnackPlayer name=Android%20Ripple%20example&supportedPlatforms=android
import React from 'react';
import {
  TouchableNativeFeedback,
  TouchableOpacity,
  TouchableHighlight,
  Platform,
  Text,
  View,
  StyleSheet,
} from 'react-native';

const SUPPORTS_NATIVE_FEEDBACK =
  Platform.OS === 'android' && Platform.Version >= 21;

const noop = () => {};
const defaultHitSlop = { top: 15, bottom: 15, right: 15, left: 15 };

const ButtonsWithNativeFeedback = () => (
  <View style={styles.buttonContainer}>
    <TouchableNativeFeedback
      onPress={noop}
      background={TouchableNativeFeedback.Ripple('#06bcee', false)}
      hitSlop={defaultHitSlop}>
      <View style={styles.button}>
        <Text style={styles.text}>This is a ripple respecting borders</Text>
      </View>
    </TouchableNativeFeedback>
    <TouchableNativeFeedback
      onPress={noop}
      background={TouchableNativeFeedback.Ripple('#06bcee', true)}
      hitSlop={defaultHitSlop}>
      <View style={styles.button}>
        <Text style={styles.text}>
          This is ripple without borders, this is more useful for icons, eg: in
          tab bar
        </Text>
      </View>
    </TouchableNativeFeedback>
  </View>
);

const Buttons = () => (
  <View style={styles.buttonContainer}>
    <TouchableOpacity
      style={styles.button}
      onPress={noop}
      hitSlop={defaultHitSlop}>
      <Text style={styles.text}>This is opacity</Text>
    </TouchableOpacity>
    <TouchableHighlight
      style={styles.button}
      onPress={noop}
      hitSlop={defaultHitSlop}
      underlayColor="#06bcee">
      <Text style={styles.text}>This is highlight</Text>
    </TouchableHighlight>
  </View>
);

const App = () => (
  <View style={styles.container}>
    {SUPPORTS_NATIVE_FEEDBACK ? <ButtonsWithNativeFeedback /> : <Buttons />}
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    paddingTop: 64,
    backgroundColor: '#fff',
  },
  buttonContainer: {
    margin: 24,
  },
  text: {
    fontSize: 20,
    color: '#fff',
    fontWeight: 'bold',
  },
  button: {
    padding: 25,
    borderRadius: 5,
    backgroundColor: '#000',
    marginBottom: 30,
  },
});

export default App;
```

## 螢幕方向鎖定

除非使用`Dimensions` API且未處理方向變更，否則多種螢幕方向應能正常運作。若不想支援多方向顯示，可將螢幕鎖定為直向或橫向。

在iOS上，於Xcode的General標籤頁中Deployment Info區段啟用欲支援的Device Orientation（需先從Devices選單選擇iPhone）。Android則需在AndroidManifest.xml檔案的activity元素內添加`'android:screenOrientation="portrait"'`鎖定直向，或`'android:screenOrientation="landscape"'`鎖定橫向。

# 延伸學習

[Material Design](https://material.io/)與[Human Interface Guidelines](https://developer.apple.com/ios/human-interface-guidelines/overview/design-principles/)是學習行動平台設計的絕佳資源。