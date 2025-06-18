---
id: stylesheet
title: StyleSheet
---

StyleSheet 是一個類似於 CSS 樣式表的抽象層

```SnackPlayer name=StyleSheet
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.title}>React Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#eaeaea',
  },
  title: {
    marginTop: 16,
    paddingVertical: 8,
    borderWidth: 4,
    borderColor: '#20232a',
    borderRadius: 6,
    backgroundColor: '#61dafb',
    color: '#20232a',
    textAlign: 'center',
    fontSize: 30,
    fontWeight: 'bold',
  },
});

export default App;
```

程式碼品質建議：

- 將樣式移出渲染函式能使程式碼更易理解。
- 為樣式命名有助於為渲染函式中的底層元件賦予意義。

---

# 參考文獻

## 方法

### `compose()`

```tsx
static compose(style1: Object, style2: Object): Object | Object[];
```

合併兩種樣式，使 `style2` 會覆寫 `style1` 中的任何樣式。若任一樣式為假值，則直接返回另一個樣式而不分配陣列，節省記憶體分配並維持 PureComponent 檢查的參考相等性。

```SnackPlayer name=Compose
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={container}>
    <Text style={text}>React Native</Text>
  </View>
);

const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    backgroundColor: '#fff',
  },
  text: {
    fontSize: 30,
    color: '#000',
  },
});

const lists = StyleSheet.create({
  listContainer: {
    flex: 1,
    backgroundColor: '#61dafb',
  },
  listItem: {
    fontStyle: 'italic',
    fontWeight: 'bold',
  },
});

const container = StyleSheet.compose(page.container, lists.listContainer);
const text = StyleSheet.compose(page.text, lists.listItem);

export default App;
```

---

### `create()`

```tsx
static create(styles: Object): Object;
```

從給定物件創建 StyleSheet 樣式參照。

---

### `flatten()`

```tsx
static flatten(style: Object[]): Object;
```

將樣式物件陣列扁平化為單一聚合樣式物件。此方法亦可用於查閱由 `StyleSheet.register` 返回的 ID。

> **注意：** 請謹慎使用，過度濫用可能影響優化效果。ID 能透過橋接機制和記憶體帶來優化。直接引用樣式物件將使您無法享受這些優化。

```SnackPlayer name=Flatten
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={page.container}>
    <Text style={flattenStyle}>React Native</Text>
    <Text>Flatten Style</Text>
    <Text style={page.code}>{JSON.stringify(flattenStyle, null, 2)}</Text>
  </View>
);

const page = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
    alignItems: 'center',
  },
  text: {
    color: '#000',
    fontSize: 14,
    fontWeight: 'bold',
  },
  code: {
    marginTop: 12,
    padding: 12,
    borderRadius: 8,
    color: '#666',
    backgroundColor: '#eaeaea',
  },
});

const typography = StyleSheet.create({
  header: {
    color: '#61dafb',
    fontSize: 30,
    marginBottom: 36,
  },
});

const flattenStyle = StyleSheet.flatten([page.text, typography.header]);

export default App;
```

此方法內部使用 `StyleSheetRegistry.getStyleByID(style)` 來解析以 ID 表示的樣式物件。因此，樣式物件陣列（`StyleSheet.create()` 的實例）會被逐一解析為對應物件，合併後返回。這也解釋了該方法的替代用途。

---

### `setStyleAttributePreprocessor()`

> **警告：實驗性功能。** 可能頻繁發生破壞性變更且不保證會事先公告。整個功能隨時可能被移除，誰知道呢？使用風險自負。

```tsx
static setStyleAttributePreprocessor(
  property: string,
  process: (propValue: any) => any,
);
```

設定用於預處理樣式屬性值的函式。此方法內部用於處理顏色和變形值。除非您確知自己在做什麼且已窮盡其他選項，否則不應使用此方法。

## 屬性

---

### `absoluteFill`

絕對定位全屏覆蓋是常見模式（`position: 'absolute', left: 0, right: 0, top: 0, bottom: 0`），為方便起見並減少重複樣式，可使用 `absoluteFill`。您亦可透過 absoluteFill 在 StyleSheet 中創建自定義項目，例如：

```SnackPlayer name=absoluteFill
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <View style={styles.box1}>
      <Text style={styles.text}>1</Text>
    </View>
    <View style={[styles.box2, StyleSheet.absoluteFill]}>
      <Text style={styles.text}>2</Text>
    </View>
    <View style={styles.box3}>
      <Text style={styles.text}>3</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  box1: {
    position: 'absolute',
    top: 40,
    left: 40,
    width: 100,
    height: 100,
    backgroundColor: 'red',
  },
  box2: {
    width: 100,
    height: 100,
    backgroundColor: 'blue',
  },
  box3: {
    position: 'absolute',
    top: 120,
    left: 120,
    width: 100,
    height: 100,
    backgroundColor: 'green',
  },
  text: {
    color: '#FFF',
    fontSize: 80,
  },
});

export default App;
```

---

### `absoluteFillObject`

有時您可能需要微調版的 `absoluteFill`——可使用 `absoluteFillObject` 在 `StyleSheet` 中創建自定義項目，例如：

```SnackPlayer name=absoluteFillObject
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <View style={styles.box1}>
      <Text style={styles.text}>1</Text>
    </View>
    <View style={styles.box2}>
      <Text style={styles.text}>2</Text>
    </View>
    <View style={styles.box3}>
      <Text style={styles.text}>3</Text>
    </View>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  box1: {
    position: 'absolute',
    top: 40,
    left: 40,
    width: 100,
    height: 100,
    backgroundColor: 'red',
  },
  box2: {
    ...StyleSheet.absoluteFillObject,
    top: 120,
    left: 50,
    width: 100,
    height: 100,
    backgroundColor: 'blue',
  },
  box3: {
    ...StyleSheet.absoluteFillObject,
    top: 120,
    left: 120,
    width: 100,
    height: 100,
    backgroundColor: 'green',
  },
  text: {
    color: '#FFF',
    fontSize: 80,
  },
});

export default App;
```

---

### `hairlineWidth`

此常數定義為平台上細線的寬度。可用作邊框粗細或兩元素間分隔線。範例：

```SnackPlayer name=hairlineWidth
import React from 'react';
import {StyleSheet, Text, View} from 'react-native';

const App = () => (
  <View style={styles.container}>
    <Text style={styles.row}>React</Text>
    <Text style={styles.row}>Native</Text>
  </View>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 24,
  },
  row: {
    padding: 4,
    borderBottomColor: 'red',
    borderBottomWidth: StyleSheet.hairlineWidth,
  },
});

export default App;
```

此常數永遠是整數像素值（確保線條清晰），並會嘗試匹配底層平台的標準細線寬度。但請勿依賴其恆定不變，因不同平台和螢幕密度可能採用不同計算方式。

若模擬器縮放比例過低，hairline 寬度的線條可能無法顯示。