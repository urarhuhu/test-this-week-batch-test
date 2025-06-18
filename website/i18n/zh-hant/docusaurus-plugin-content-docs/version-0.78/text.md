---
id: text
title: Text
---

用於顯示文字的 React 元件。

`Text` 支援嵌套、樣式設定和觸控處理。

在以下範例中，嵌套的標題和正文文字將繼承 `styles.baseText` 的 `fontFamily`，但標題提供了自己的額外樣式。由於文字中的換行符，標題和正文將垂直堆疊：

```SnackPlayer name=Text%20Function%20Component%20Example
import React, {useState} from 'react';
import {Text, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const TextInANest = () => {
  const [titleText, setTitleText] = useState("Bird's Nest");
  const bodyText = 'This is not really a bird nest.';

  const onPressTitle = () => {
    setTitleText("Bird's Nest [pressed]");
  };

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text style={styles.baseText}>
          <Text style={styles.titleText} onPress={onPressTitle}>
            {titleText}
            {'\n'}
            {'\n'}
          </Text>
          <Text numberOfLines={5}>{bodyText}</Text>
        </Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  baseText: {
    fontFamily: 'Cochin',
  },
  titleText: {
    fontSize: 20,
    fontWeight: 'bold',
  },
});

export default TextInANest;
```

## 嵌套文字

Android 和 iOS 都允許您通過為字串的特定範圍添加格式（如粗體或彩色文字）來顯示格式化文字（iOS 上使用 `NSAttributedString`，Android 上使用 `SpannableString`）。實際上，這非常繁瑣。對於 React Native，我們決定使用網頁的範例來實現相同的效果，即通過嵌套文字來達成。

```SnackPlayer name=Nested%20Text%20Example
import React from 'react';
import {Text, StyleSheet} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const BoldAndBeautiful = () => (
  <SafeAreaProvider>
    <SafeAreaView style={styles.container}>
      <Text style={styles.baseText}>
        I am bold
        <Text style={styles.innerText}> and red</Text>
      </Text>
    </SafeAreaView>
  </SafeAreaProvider>
);

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  baseText: {
    fontWeight: 'bold',
  },
  innerText: {
    color: 'red',
  },
});

export default BoldAndBeautiful;
```

在底層，React Native 會將其轉換為一個扁平的 `NSAttributedString` 或 `SpannableString`，其中包含以下資訊：

```
"I am bold and red"
0-9: bold
9-17: bold, red
```

## 容器

`<Text>` 元素在佈局上是獨特的：其內部不再使用 Flexbox 佈局，而是使用文字佈局。這意味著 `<Text>` 內部的元素不再是矩形，而是在行尾時自動換行。

```tsx
<Text>
  <Text>First part and </Text>
  <Text>second part</Text>
</Text>
// Text container: the text will be inline, if the space allows it
// |First part and second part|

// otherwise, the text will flow as if it was one
// |First part |
// |and second |
// |part       |

<View>
  <Text>First part and </Text>
  <Text>second part</Text>
</View>
// View container: each text is its own block
// |First part and|
// |second part   |

// otherwise, the text will flow in its own block
// |First part |
// |and        |
// |second part|
```

## 有限的樣式繼承

在網頁上，通常為整個文件設定字體家族和大小的方式是利用繼承的 CSS 屬性，如下所示：

```css
html {
  font-family:
    'lucida grande', tahoma, verdana, arial, sans-serif;
  font-size: 11px;
  color: #141823;
}
```

除非元素或其父元素指定了新規則，否則文件中的所有元素都將繼承此字體。

在 React Native 中，我們對此更加嚴格：**您必須將所有文字節點包裹在 `<Text>` 元件中**。您不能直接在 `<View>` 下放置文字節點。

```tsx
// BAD: will raise exception, can't have a text node as child of a <View>
<View>
  Some text
</View>

// GOOD
<View>
  <Text>
    Some text
  </Text>
</View>
```

您也無法為整個子樹設定預設字體。同時，`fontFamily` 僅接受單一字體名稱，這與 CSS 中的 `font-family` 不同。建議在應用程式中使用一致的字體和大小的方式是創建一個包含這些樣式的元件 `MyAppText`，並在整個應用程式中使用此元件。您還可以使用此元件來創建更具體的元件，如 `MyAppHeaderText`，用於其他類型的文字。

```tsx
<View>
  <MyAppText>
    Text styled with the default font for the entire application
  </MyAppText>
  <MyAppHeaderText>Text styled as a header</MyAppHeaderText>
</View>
```

假設 `MyAppText` 是一個僅將其子元素渲染為帶有樣式的 `Text` 元件的元件，則 `MyAppHeaderText` 可以如下定義：

```tsx
const MyAppHeaderText = ({children}) => {
  return (
    <MyAppText>
      <Text style={{fontSize: 20}}>{children}</Text>
    </MyAppText>
  );
};
```

以這種方式組合 `MyAppText` 確保我們從頂層元件獲取樣式，同時保留了在特定用例中添加/覆蓋樣式的能力。

React Native 仍然有樣式繼承的概念，但僅限於文字子樹。在這種情況下，第二部分將同時是粗體和紅色。

```tsx
<Text style={{fontWeight: 'bold'}}>
  I am bold
  <Text style={{color: 'red'}}>and red</Text>
</Text>
```

我們相信這種更受限的文字樣式設定方式將產生更好的應用程式：

- （開發者）React 元件設計時考慮了強隔離性：您應該能夠將元件放置在應用程式的任何位置，並相信只要屬性相同，它的外觀和行為就會相同。可以從屬性外部繼承的文字屬性將破壞這種隔離性。

- （實現者）React Native 的實現也得以簡化。我們不需要在每個元素上都有 `fontFamily` 字段，也不需要每次顯示文字節點時都遍歷樹直到根節點。樣式繼承僅編碼在原生 Text 元件內部，不會洩漏到其他元件或系統本身。

---

# 參考

## 屬性

### `accessibilityHint`

輔助功能提示幫助使用者理解當他們對輔助功能元素執行操作時會發生什麼，當該結果無法從輔助功能標籤中明確得知時。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

指示螢幕閱讀器在使用者與元素互動時應使用的語言值。該值應遵循 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage) 以獲取更多資訊。

| Type   |
| ------ |
| string |

---

### `accessibilityLabel`

覆蓋螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤是通過遍歷所有子元素並累積所有以空格分隔的 `Text` 節點來構建的。

| Type   |
| ------ |
| string |

---

### `accessibilityRole`

告知螢幕閱讀器將當前聚焦的元素視為具有特定角色。

在 iOS 上，這些角色會映射到對應的輔助功能特徵。圖片按鈕的功能與同時設定 'image' 和 'button' 特徵相同。詳見 [輔助功能指南](accessibility.md#accessibilitytraits-ios) 以獲取更多資訊。

在 Android 上，這些角色在 TalkBack 中的功能類似於 iOS 中 Voiceover 添加輔助功能特徵的方式。

| Type                                                 |
| ---------------------------------------------------- |
| [AccessibilityRole](accessibility#accessibilityrole) |

---

### `accessibilityState`

告知螢幕閱讀器將當前聚焦的元素視為處於特定狀態。

您可以提供一個狀態、無狀態或多個狀態。狀態必須通過物件傳遞，例如 `{selected: true, disabled: true}`。

| Type                                                   |
| ------------------------------------------------------ |
| [AccessibilityState](accessibility#accessibilitystate) |

---

### `accessibilityActions`

輔助功能操作允許輔助技術以程式化方式呼叫元件的操作。`accessibilityActions` 屬性應包含一個操作物件列表。每個操作物件應包含名稱和標籤欄位。

詳見 [輔助功能指南](accessibility.md#accessibility-actions) 以獲取更多資訊。

| Type  | Required |
| ----- | -------- |
| array | No       |

---

### `onAccessibilityAction`

當使用者執行輔助功能操作時呼叫。此函數的唯一參數是一個包含要執行操作名稱的事件。

詳見 [輔助功能指南](accessibility.md#accessibility-actions) 以獲取更多資訊。

| Type     | Required |
| -------- | -------- |
| function | No       |

---

### `accessible`

當設定為 `true` 時，表示該視圖是一個輔助功能元素。

詳見 [輔助功能指南](accessibility#accessible-ios-android) 以獲取更多資訊。

| Type    | Default |
| ------- | ------- |
| boolean | `true`  |

---

### `adjustsFontSizeToFit`

指定字體是否應自動縮小以適應給定的樣式約束。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `allowFontScaling`

指定字體是否應根據文字大小輔助功能設定進行縮放。

| Type    | Default |
| ------- | ------- |
| boolean | `true`  |

---

### `android_hyphenationFrequency` <div class="label android">Android</div>

在 Android API 等級 23+ 上設定自動斷字頻率，用於確定單詞斷開方式。

| Type                                | Default  |
| ----------------------------------- | -------- |
| enum(`'none'`, `'normal'`,`'full'`) | `'none'` |

---

### `aria-busy`

表示元素正在被修改，輔助技術可能需要等待變更完成後再通知使用者更新。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

表示可勾選元素的狀態。此屬性可接受布林值或「mixed」字串來表示混合勾選框。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

表示元素可感知但已被禁用，因此無法編輯或操作。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-expanded`

表示可展開元素當前是展開還是折疊狀態。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-label`

定義一個字串值，用於標記互動式元素。

| Type   |
| ------ |
| string |

---

### `aria-selected`

表示可選元素當前是否被選中。

| Type    |
| ------- |
| boolean |

### `dataDetectorType` <div class="label android">Android</div>

決定文字元素中哪些類型的資料會被轉換為可點擊的連結。預設不偵測任何資料類型。

只能提供一種類型。

| Type                                                          | Default  |
| ------------------------------------------------------------- | -------- |
| enum(`'phoneNumber'`, `'link'`, `'email'`, `'none'`, `'all'`) | `'none'` |

---

### `disabled` <div class="label android">Android</div>

指定文字視圖的禁用狀態，用於測試目的。

| Type | Default |
| ---- | ------- |
| bool | `false` |

---

### `dynamicTypeRamp` <div class="label ios">iOS</div>

在iOS上應用於此元素的[動態類型](https://developer.apple.com/documentation/uikit/uifont/scaling_fonts_automatically)級別。

| Type                                                                                                                                                     | Default  |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| enum(`'caption2'`, `'caption1'`, `'footnote'`, `'subheadline'`, `'callout'`, `'body'`, `'headline'`, `'title3'`, `'title2'`, `'title1'`, `'largeTitle'`) | `'body'` |

---

### `ellipsizeMode`

當設定`numberOfLines`時，此屬性定義文字如何被截斷。必須與`numberOfLines`一起使用。

可為以下值之一：

- `head` - 行尾顯示在容器內，行首缺失的文字以省略號表示。例如：「...wxyz」
- `middle` - 行首和行尾顯示在容器內，中間缺失的文字以省略號表示。例如：「ab...yz」
- `tail` - 行首顯示在容器內，行尾缺失的文字以省略號表示。例如：「abcd...」
- `clip` - 不繪製超出文字容器邊界的行。

> 在Android上，當`numberOfLines`設定為大於`1`的值時，只有`tail`值能正常運作。

| Type                                           | Default |
| ---------------------------------------------- | ------- |
| enum(`'head'`, `'middle'`, `'tail'`, `'clip'`) | `tail`  |

---

### `id`

用於從原生程式碼定位此視圖。優先級高於`nativeID`屬性。

| Type   |
| ------ |
| string |

---

### `maxFontSizeMultiplier`

指定當 `allowFontScaling` 啟用時，字體可達到的最大縮放比例。可能的值：

- `null/undefined`：繼承父節點或全域預設值 (0)
- `0`：無最大值限制，忽略父級/全域預設值
- `>= 1`：將此節點的 `maxFontSizeMultiplier` 設為此值

| Type   | Default     |
| ------ | ----------- |
| number | `undefined` |

---

### `minimumFontScale` <div class="label ios">iOS</div>

指定當 `adjustsFontSizeToFit` 啟用時，字體可縮小的最小比例 (值範圍 0.01-1.0)。

| Type   |
| ------ |
| number |

---

### `nativeID`

用於從原生程式碼定位此視圖。

| Type   |
| ------ |
| string |

---

### `numberOfLines`

用於在計算文字佈局（包括自動換行）後，以省略號截斷文字，使總行數不超過此數值。將此屬性設為 `0` 會取消此限制，意味著不應用任何行數限制。

此屬性通常與 `ellipsizeMode` 搭配使用。

| Type   | Default |
| ------ | ------- |
| number | `0`     |

---

### `onLayout`

在掛載時或佈局變更時觸發。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onLongPress`

此函式在長按時呼叫。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onMoveShouldSetResponder`

此視圖是否要「聲明」觸控回應權？當視圖不是響應者時，每次觸控移動都會呼叫此函式。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onPress`

使用者按壓時呼叫的函式，在 `onPressOut` 之後觸發。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onPressIn`

觸控開始時立即呼叫，在 `onPressOut` 和 `onPress` 之前。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onPressOut`

觸控結束時呼叫。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderGrant`

視圖現在開始響應觸控事件。此時應高亮顯示並向使用者展示當前狀態。

在 Android 上，從此回調返回 true 可阻止其他原生元件在此響應者終止前成為響應者。

| Type                                                              |
| ----------------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void ｜ boolean` |

---

### `onResponderMove`

使用者正在移動手指。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderRelease`

觸控結束時觸發。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminate`

響應者權限已從視圖移除。可能因呼叫 `onResponderTerminationRequest` 後被其他視圖取得，也可能被系統未經詢問直接取走（例如 iOS 的控制中心/通知中心）

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onResponderTerminationRequest`

當其他 `View` 想要成為觸控回應者並要求此 `View` 釋放其回應權時，返回 `true` 表示允許釋放。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onStartShouldSetResponderCapture`

若父層 `View` 欲阻止子層 `View` 在觸控開始時成為回應者，應設置此處理函式並返回 `true`。

| Type                                                      |
| --------------------------------------------------------- |
| `md ({nativeEvent: [PressEvent](pressevent)}) => boolean` |

---

### `onTextLayout`

在文字佈局變更時觸發。

| Type                                                 |
| ---------------------------------------------------- |
| ([`TextLayoutEvent`](text#textlayoutevent)) => mixed |

---

### `pressRetentionOffset`

當滾動視圖被禁用時，此屬性定義觸控點可偏離按鈕多遠才會停用按鈕。停用後若移回範圍內，按鈕會重新啟用。建議傳入常數值以減少記憶體分配。

| Type                 |
| -------------------- |
| [Rect](rect), number |

---

### `role`

`role` 向輔助技術使用者傳達元件的用途，其優先級高於 [`accessibilityRole`](text#accessibilityrole) 屬性。

| Type                       |
| -------------------------- |
| [Role](accessibility#role) |

---

### `selectable`

允許使用者選取文字，以使用原生複製貼上功能。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `selectionColor` <div class="label android">Android</div>

文字選取時的高亮顏色。

| Type            |
| --------------- |
| [color](colors) |

---

### `style`

| Type                                                                 |
| -------------------------------------------------------------------- |
| [Text Style](text-style-props), [View Style Props](view-style-props) |

---

### `suppressHighlighting` <div class="label ios">iOS</div>

設為 `true` 時，文字被按下不會有視覺變化。預設情況下，按下時會顯示灰色橢圓形高亮效果。

| Type    | Default |
| ------- | ------- |
| boolean | `false` |

---

### `testID`

用於在端對端測試中定位此視圖。

| Type   |
| ------ |
| string |

---

### `textBreakStrategy` <div class="label android">Android</div>

在 Android API 等級 23+ 設定文字斷行策略，可選值為 `simple`、`highQuality`、`balanced`。

| Type                                            | Default       |
| ----------------------------------------------- | ------------- |
| enum(`'simple'`, `'highQuality'`, `'balanced'`) | `highQuality` |

---

### `lineBreakStrategyIOS` <div class="label ios">iOS</div>

在 iOS 14+ 設定換行策略，可選值為 `none`、`standard`、`hangul-word` 及 `push-out`。

| Type                                                        | Default  |
| ----------------------------------------------------------- | -------- |
| enum(`'none'`, `'standard'`, `'hangul-word'`, `'push-out'`) | `'none'` |

## 型別定義

### TextLayout

`TextLayout` 物件是 [`TextLayoutEvent`](text#textlayoutevent) 回調的一部分，包含 `Text` 行文的測量數據。

#### 範例

```js
{
    capHeight: 10.496,
    ascender: 14.624,
    descender: 4,
    width: 28.224,
    height: 18.624,
    xHeight: 6.048,
    x: 0,
    y: 0
}
```

#### 屬性

| Name      | Type   | Optional | Description                                                         |
| --------- | ------ | -------- | ------------------------------------------------------------------- |
| ascender  | number | No       | The line ascender height after the text layout changes.             |
| capHeight | number | No       | Height of capital letter above the baseline.                        |
| descender | number | No       | The line descender height after the text layout changes.            |
| height    | number | No       | Height of the line after the text layout changes.                   |
| width     | number | No       | Width of the line after the text layout changes.                    |
| x         | number | No       | Line X coordinate inside the Text component.                        |
| xHeight   | number | No       | Distance between the baseline and median of the line (corpus size). |
| y         | number | No       | Line Y coordinate inside the Text component.                        |

### TextLayoutEvent

`TextLayoutEvent` 物件會在元件佈局變更時於回調函式中返回。它包含一個名為 `lines` 的鍵，其值為一個陣列，內含對應每個渲染文字行的 [`TextLayout`](text#textlayout) 物件。

#### 範例

```js
{
  lines: [
    TextLayout,
    TextLayout,
    // ...
  ];
  target: 1127;
}
```

#### 屬性

| Name   | Type                                    | Optional | Description                                           |
| ------ | --------------------------------------- | -------- | ----------------------------------------------------- |
| lines  | array of [TextLayout](text#textlayout)s | No       | Provides the TextLayout data for every rendered line. |
| target | number                                  | No       | The node id of the element.                           |