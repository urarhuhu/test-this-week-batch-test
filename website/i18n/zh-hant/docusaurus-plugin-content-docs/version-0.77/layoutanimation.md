---
id: layoutanimation
title: LayoutAnimation
---

當下一次佈局發生時，自動將視圖動畫過渡到新位置。

常見的使用方式是在函數式組件中更新狀態鉤子前調用此API，或在類組件中調用`setState`前使用。

請注意，在**Android**平台上需通過`UIManager`設置以下標誌才能生效：

```js
if (Platform.OS === 'android') {
  if (UIManager.setLayoutAnimationEnabledExperimental) {
    UIManager.setLayoutAnimationEnabledExperimental(true);
  }
}
```

## 範例

```SnackPlayer name=LayoutAnimation%20Example&supportedPlatforms=android,ios
import React, {useState} from 'react';
import {
  LayoutAnimation,
  Platform,
  StyleSheet,
  Text,
  TouchableOpacity,
  UIManager,
  View,
} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

if (
  Platform.OS === 'android' &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}
const App = () => {
  const [expanded, setExpanded] = useState(false);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={style.container}>
        <TouchableOpacity
          onPress={() => {
            LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
            setExpanded(!expanded);
          }}>
          <Text>Press me to {expanded ? 'collapse' : 'expand'}!</Text>
        </TouchableOpacity>
        {expanded && (
          <View style={style.tile}>
            <Text>I disappear sometimes!</Text>
          </View>
        )}
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const style = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    gap: 16,
  },
  tile: {
    backgroundColor: 'lightgrey',
    borderWidth: 0.5,
    borderColor: '#d6d7da',
    padding: 4,
  },
});

export default App;
```

---

# 參考文檔

## 方法

### `configureNext()`

```tsx
static configureNext(
  config: LayoutAnimationConfig,
  onAnimationDidEnd?: () => void,
  onAnimationDidFail?: () => void,
);
```

安排在下次佈局時執行動畫。

#### 參數：

| Name               | Type     | Required | Description                         |
| ------------------ | -------- | -------- | ----------------------------------- |
| config             | object   | Yes      | See config description below.       |
| onAnimationDidEnd  | function | No       | Called when the animation finished. |
| onAnimationDidFail | function | No       | Called when the animation failed.   |

`config`參數為包含下列鍵值的物件。[`create`](layoutanimation.md#create)會返回符合`config`格式的有效物件，[`Presets`](layoutanimation.md#presets)物件也可直接作為`config`傳入。

- `duration` 單位為毫秒
- `create` 可選，用於新視圖進入動畫的配置
- `update` 可選，用於視圖更新時的動畫配置
- `delete` 可選，用於視圖移除時的動畫配置

傳遞給`create`、`update`或`delete`的配置包含以下鍵值：

- `type` 使用的[動畫類型](layoutanimation.md#types)
- `property` 要動畫化的[佈局屬性](layoutanimation.md#properties)（可選，但建議在`create`和`delete`中使用）
- `springDamping` （數值，可選，僅在`type: Type.spring`時使用）
- `initialVelocity` （數值，可選）
- `delay` （數值，可選）
- `duration` （數值，可選）

---

### `create()`

```tsx
static create(duration, type, creationProp)
```

輔助函數，創建包含`create`、`update`和`delete`字段的物件，用於傳入[`configureNext`](layoutanimation.md#configurenext)。`type`參數為[動畫類型](layoutanimation.md#types)，`creationProp`參數為[佈局屬性](layoutanimation.md#properties)。

**範例：**

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, {useState} from 'react';
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button,
} from 'react-native';

if (
  Platform.OS === 'android' &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

const App = () => {
  const [boxPosition, setBoxPosition] = useState('left');

  const toggleBox = () => {
    LayoutAnimation.configureNext({
      duration: 500,
      create: {type: 'linear', property: 'opacity'},
      update: {type: 'spring', springDamping: 0.4},
      delete: {type: 'linear', property: 'opacity'},
    });
    setBoxPosition(boxPosition === 'left' ? 'right' : 'left');
  };

  return (
    <View style={styles.container}>
      <View style={styles.buttonContainer}>
        <Button title="Toggle Layout" onPress={toggleBox} />
      </View>
      <View
        style={[styles.box, boxPosition === 'left' ? null : styles.moveRight]}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'flex-start',
    justifyContent: 'center',
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: 'blue',
  },
  moveRight: {
    alignSelf: 'flex-end',
    height: 200,
    width: 200,
  },
  buttonContainer: {
    alignSelf: 'center',
  },
});

export default App;
```

## 屬性

### 動畫類型

枚舉值，用於[`create`](layoutanimation.md#create)方法或[`configureNext`](layoutanimation.md#configurenext)的`create`/`update`/`delete`配置中指定動畫類型（用法示例：`LayoutAnimation.Types.easeIn`）。

| Types         |
| ------------- |
| spring        |
| linear        |
| easeInEaseOut |
| easeIn        |
| easeOut       |
| keyboard      |

---

### 佈局屬性

枚舉值，用於[`create`](layoutanimation.md#create)方法或[`configureNext`](layoutanimation.md#configurenext)的`create`/`update`/`delete`配置中指定要動畫化的佈局屬性（用法示例：`LayoutAnimation.Properties.opacity`）。

| Properties |
| ---------- |
| opacity    |
| scaleX     |
| scaleY     |
| scaleXY    |

---

### 預設配置

一組預定義的動畫配置，可直接傳入[`configureNext`](layoutanimation.md#configurenext)。

| Presets       | Value                                                                                                                                                          |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| easeInEaseOut | `create(300, 'easeInEaseOut', 'opacity')`                                                                                                                      |
| linear        | `create(500, 'linear', 'opacity')`                                                                                                                             |
| spring        | `{duration: 700, create: {type: 'linear', property: 'opacity'}, update: {type: 'spring', springDamping: 0.4}, delete: {type: 'linear', property: 'opacity'} }` |

---

### `easeInEaseOut`

使用`Presets.easeInEaseOut`配置調用`configureNext()`。

---

### `linear`

使用`Presets.linear`配置調用`configureNext()`。

---

### `spring`

使用`Presets.spring`配置調用`configureNext()`。

**範例：**

```SnackPlayer name=LayoutAnimation&supportedPlatforms=android,ios
import React, {useState} from 'react';
import {
  View,
  Platform,
  UIManager,
  LayoutAnimation,
  StyleSheet,
  Button,
} from 'react-native';

if (
  Platform.OS === 'android' &&
  UIManager.setLayoutAnimationEnabledExperimental
) {
  UIManager.setLayoutAnimationEnabledExperimental(true);
}

const App = () => {
  const [firstBoxPosition, setFirstBoxPosition] = useState('left');
  const [secondBoxPosition, setSecondBoxPosition] = useState('left');
  const [thirdBoxPosition, setThirdBoxPosition] = useState('left');

  const toggleFirstBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.easeInEaseOut);
    setFirstBoxPosition(firstBoxPosition === 'left' ? 'right' : 'left');
  };

  const toggleSecondBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.linear);
    setSecondBoxPosition(secondBoxPosition === 'left' ? 'right' : 'left');
  };

  const toggleThirdBox = () => {
    LayoutAnimation.configureNext(LayoutAnimation.Presets.spring);
    setThirdBoxPosition(thirdBoxPosition === 'left' ? 'right' : 'left');
  };

  return (
    <View style={styles.container}>
      <View style={styles.buttonContainer}>
        <Button title="EaseInEaseOut" onPress={toggleFirstBox} />
      </View>
      <View
        style={[
          styles.box,
          firstBoxPosition === 'left' ? null : styles.moveRight,
        ]}
      />
      <View style={styles.buttonContainer}>
        <Button title="Linear" onPress={toggleSecondBox} />
      </View>
      <View
        style={[
          styles.box,
          secondBoxPosition === 'left' ? null : styles.moveRight,
        ]}
      />
      <View style={styles.buttonContainer}>
        <Button title="Spring" onPress={toggleThirdBox} />
      </View>
      <View
        style={[
          styles.box,
          thirdBoxPosition === 'left' ? null : styles.moveRight,
        ]}
      />
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'flex-start',
    justifyContent: 'center',
  },
  box: {
    height: 100,
    width: 100,
    borderRadius: 5,
    margin: 8,
    backgroundColor: 'blue',
  },
  moveRight: {
    alignSelf: 'flex-end',
  },
  buttonContainer: {
    alignSelf: 'center',
  },
});

export default App;
```