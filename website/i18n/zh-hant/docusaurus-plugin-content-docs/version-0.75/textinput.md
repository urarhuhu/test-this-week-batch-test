---
id: textinput
title: TextInput
---

一個基礎組件，用於通過鍵盤向應用程序輸入文本。屬性提供了多種功能的可配置性，例如自動校正、自動大寫、佔位文本以及不同的鍵盤類型（如數字鍵盤）。

最基本的用例是放置一個 `TextInput` 並訂閱 `onChangeText` 事件來讀取用戶輸入。還有其他事件，如 `onSubmitEditing` 和 `onFocus`，可以訂閱。一個最小示例：

```SnackPlayer name=TextInput
import React from 'react';
import {SafeAreaView, StyleSheet, TextInput} from 'react-native';

const TextInputExample = () => {
  const [text, onChangeText] = React.useState('Useless Text');
  const [number, onChangeNumber] = React.useState('');

  return (
    <SafeAreaView>
      <TextInput
        style={styles.input}
        onChangeText={onChangeText}
        value={text}
      />
      <TextInput
        style={styles.input}
        onChangeText={onChangeNumber}
        value={number}
        placeholder="useless placeholder"
        keyboardType="numeric"
      />
    </SafeAreaView>
  );
};

const styles = StyleSheet.create({
  input: {
    height: 40,
    margin: 12,
    borderWidth: 1,
    padding: 10,
  },
});

export default TextInputExample;
```

通過原生元素暴露的兩個方法是 .focus() 和 .blur()，可以以編程方式聚焦或模糊 TextInput。

請注意，某些屬性僅在 `multiline={true/false}` 時可用。此外，如果 `multiline=true`，則僅應用於元素一側的邊框樣式（例如 `borderBottomColor`、`borderLeftWidth` 等）將不會應用。要實現相同的效果，您可以將 `TextInput` 包裹在 `View` 中：

```SnackPlayer name=TextInput
import React from 'react';
import {View, TextInput} from 'react-native';

const MultilineTextInputExample = () => {
  const [value, onChangeText] = React.useState('Useless Multiline Placeholder');

  // If you type something in the text box that is a color, the background will change to that
  // color.
  return (
    <View
      style={{
        backgroundColor: value,
        borderBottomColor: '#000000',
        borderBottomWidth: 1,
      }}>
      <TextInput
        editable
        multiline
        numberOfLines={4}
        maxLength={40}
        onChangeText={text => onChangeText(text)}
        value={value}
        style={{padding: 10}}
      />
    </View>
  );
};

export default MultilineTextInputExample;
```

`TextInput` 默認在其視圖底部有一個邊框。此邊框的填充由系統提供的背景圖像設置，無法更改。避免此問題的解決方案是不要明確設置高度（在這種情況下，系統會負責在正確位置顯示邊框），或者通過將 `underlineColorAndroid` 設置為透明來不顯示邊框。

請注意，在 Android 上，在輸入中執行文本選擇可能會將應用的活動 `windowSoftInputMode` 參數更改為 `adjustResize`。這可能會導致在鍵盤處於活動狀態時具有 position: 'absolute' 的組件出現問題。要避免這種行為，可以在 AndroidManifest.xml ( https://developer.android.com/guide/topics/manifest/activity-element.html ) 中指定 `windowSoftInputMode`，或者通過原生代碼以編程方式控制此參數。

---

# 參考

## 屬性

### [視圖屬性](view.md#props)

繼承 [視圖屬性](view.md#props)。

---

### `allowFontScaling`

指定字體是否應縮放以尊重文本大小的無障礙設置。默認為 `true`。

| Type |
| ---- |
| bool |

---

### `autoCapitalize`

告訴 `TextInput` 自動大寫某些字符。此屬性不受某些鍵盤類型（如 `name-phone-pad`）支持。

- `characters`：所有字符。
- `words`：每個單詞的首字母。
- `sentences`：每個句子的首字母（_默認_）。
- `none`：不自動大寫任何內容。

| Type                                             |
| ------------------------------------------------ |
| enum('none', 'sentences', 'words', 'characters') |

---

### `autoComplete`

為系統指定自動完成提示，以便提供自動填充。在 Android 上，系統將始終嘗試通過使用啟發式方法來識別內容類型來提供自動填充。要禁用自動完成，請將 `autoComplete` 設置為 `off`。

以下值在所有平台上都有效：

- `additional-name` (額外名稱)
- `address-line1` (地址行1)
- `address-line2` (地址行2)
- `birthdate-day` (出生日期-日，iOS 17+)
- `birthdate-full` (出生日期-完整，iOS 17+)
- `birthdate-month` (出生日期-月，iOS 17+)
- `birthdate-year` (出生日期-年，iOS 17+)
- `cc-csc` (信用卡安全碼，iOS 17+)
- `cc-exp` (信用卡到期日，iOS 17+)
- `cc-exp-day` (信用卡到期日-日，iOS 17+)
- `cc-exp-month` (信用卡到期日-月，iOS 17+)
- `cc-exp-year` (信用卡到期日-年，iOS 17+)
- `cc-number` (信用卡號碼)
- `country` (國家)
- `current-password` (當前密碼)
- `email` (電子郵件)
- `family-name` (姓氏)
- `given-name` (名字)
- `honorific-prefix` (敬稱前綴)
- `honorific-suffix` (敬稱後綴)
- `name` (姓名)
- `new-password` (新密碼)
- `off` (關閉)
- `one-time-code` (一次性驗證碼)
- `postal-code` (郵遞區號)
- `street-address` (街道地址)
- `tel` (電話號碼)
- `username` (使用者名稱)

<div class="label basic ios">iOS</div>

以下值僅在 iOS 上有效：

- `cc-family-name` (信用卡姓氏，iOS 17+)
- `cc-given-name` (信用卡名字，iOS 17+)
- `cc-middle-name` (信用卡中間名，iOS 17+)
- `cc-name` (信用卡姓名，iOS 17+)
- `cc-type` (信用卡類型，iOS 17+)
- `nickname` (暱稱)
- `organization` (組織)
- `organization-title` (組織職稱)
- `url` (網址)

<div class="label basic android">Android</div>

以下值僅在 Android 上有效：

- `gender` (性別)
- `name-family` (姓氏)
- `name-given` (名字)
- `name-middle` (中間名)
- `name-middle-initial` (中間名縮寫)
- `name-prefix` (名前綴)
- `name-suffix` (名後綴)
- `password` (密碼)
- `password-new` (新密碼)
- `postal-address` (郵寄地址)
- `postal-address-country` (郵寄地址國家)
- `postal-address-extended` (郵寄地址擴展)
- `postal-address-extended-postal-code` (郵寄地址擴展郵遞區號)
- `postal-address-locality` (郵寄地址地區)
- `postal-address-region` (郵寄地址區域)
- `sms-otp` (簡訊一次性密碼)
- `tel-country-code` (電話國碼)
- `tel-device` (裝置電話)
- `tel-national` (國內電話)
- `username-new` (新使用者名稱)

| Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('additional-name', 'address-line1', 'address-line2', 'birthdate-day', 'birthdate-full', 'birthdate-month', 'birthdate-year', 'cc-csc', 'cc-exp', 'cc-exp-day', 'cc-exp-month', 'cc-exp-year', 'cc-number', 'country', 'current-password', 'email', 'family-name', 'given-name', 'honorific-prefix', 'honorific-suffix', 'name', 'new-password', 'off', 'one-time-code', 'postal-code', 'street-address', 'tel', 'username', 'cc-family-name', 'cc-given-name', 'cc-middle-name', 'cc-name', 'cc-type', 'nickname', 'organization', 'organization-title', 'url', 'gender', 'name-family', 'name-given', 'name-middle', 'name-middle-initial', 'name-prefix', 'name-suffix', 'password', 'password-new', 'postal-address', 'postal-address-country', 'postal-address-extended', 'postal-address-extended-postal-code', 'postal-address-locality', 'postal-address-region', 'sms-otp', 'tel-country-code', 'tel-device', 'tel-national', 'username-new') |

---

### `autoCorrect`

若為 `false`，則停用自動校正。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `autoFocus`

若為 `true`，則在 `componentDidMount` 或 `useEffect` 時聚焦輸入框。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `blurOnSubmit`

若為 `true`，則提交時文字欄位會失去焦點。單行欄位的預設值為 true，多行欄位則為 false。請注意，對於多行欄位，將 `blurOnSubmit` 設為 `true` 表示按下返回鍵會使欄位失去焦點並觸發 `onSubmitEditing` 事件，而非在欄位中插入換行。

| Type |
| ---- |
| bool |

---

### `caretHidden`

若為 `true`，則隱藏游標。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `clearButtonMode` <div class="label ios">iOS</div>

清除按鈕應出現在文字視圖右側的時機。此屬性僅支援單行 TextInput 元件。預設值為 `never`。

| Type                                                       |
| ---------------------------------------------------------- |
| enum('never', 'while-editing', 'unless-editing', 'always') |

---

### `clearTextOnFocus` <div class="label ios">iOS</div>

若為 `true`，則在開始編輯時自動清除文字欄位。

| Type |
| ---- |
| bool |

---

### `contextMenuHidden`

若為 `true`，則隱藏上下文選單。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `dataDetectorTypes` <div class="label ios">iOS</div>

決定文字輸入框中哪些類型的資料會被轉換為可點擊的連結。僅在 `multiline={true}` 且 `editable={false}` 時有效。預設不偵測任何資料類型。

可指定單一類型或多種類型組成的陣列。

`dataDetectorTypes` 的可能值包括：

- `'phoneNumber'`
- `'link'`
- `'address'`
- `'calendarEvent'`
- `'none'`
- `'all'`

| Type                                                                                                                                                     |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('phoneNumber', 'link', 'address', 'calendarEvent', 'none', 'all'), ,array of enum('phoneNumber', 'link', 'address', 'calendarEvent', 'none', 'all') |

---

### `defaultValue`

提供初始值，該值會在使用者開始輸入時改變。適用於不想處理事件監聽與手動更新受控狀態值的場景。

| Type   |
| ------ |
| string |

---

### `cursorColor` <div class="label android">Android</div>

設定元件中游標（或稱「插入符」）的顏色。與 `selectionColor` 的行為不同，此屬性會獨立於文字選取框的顏色設定游標顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `disableFullscreenUI` <div class="label android">Android</div>

當值為 `false` 時，若文字輸入框周圍可用空間較少（例如手機橫向模式），作業系統可能讓使用者在全螢幕文字輸入模式中編輯文字。設為 `true` 會停用此功能，使用者將始終直接在輸入框中編輯文字。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `editable`

若為 `false`，文字內容不可編輯。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `enablesReturnKeyAutomatically` <div class="label ios">iOS</div>

若為 `true`，當輸入框無文字時會停用返回鍵，並在有文字時自動啟用。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `enterKeyHint`

決定返回鍵顯示的文字內容。此屬性優先於 `returnKeyType`。

以下為跨平台支援的值：

- `enter`
- `done`
- `next`
- `search`
- `send`

_僅限 Android_

以下為僅 Android 支援的值：

- `previous`

| Type                                                        |
| ----------------------------------------------------------- |
| enum('enter', 'done', 'next', 'previous', 'search', 'send') |

---

### `importantForAutofill` <div class="label android">Android</div>

告知作業系統是否應將應用程式中的個別欄位納入 Android API 26+ 的自動填充視圖結構。可能值為 `auto`、`no`、`noExcludeDescendants`、`yes` 和 `yesExcludeDescendants`，預設值為 `auto`。

- `auto`：由 Android 系統自行判斷視圖對自動填充的重要性。
- `no`：此視圖對自動填充不重要。
- `noExcludeDescendants`：此視圖及其子視圖對自動填充不重要。
- `yes`：此視圖對自動填充重要。
- `yesExcludeDescendants`：此視圖對自動填充重要，但其子視圖不重要。

| Type                                                                       |
| -------------------------------------------------------------------------- |
| enum('auto', 'no', 'noExcludeDescendants', 'yes', 'yesExcludeDescendants') |

---

### `inlineImageLeft` <div class="label android">Android</div>

若定義此屬性，提供的圖片資源將顯示在輸入框左側。圖片資源必須位於 `/android/app/src/main/res/drawable` 目錄下，並以如下方式引用：

```
<TextInput
 inlineImageLeft='search_icon'
/>
```

| Type   |
| ------ |
| string |

---

### `inlineImagePadding` <div class="label android">Android</div>

設定內嵌圖片（如有）與文字輸入框本身的間距。

| Type   |
| ------ |
| number |

---

### `inputAccessoryViewID` <div class="label ios">iOS</div>

可選標識符，用於將自訂的 [InputAccessoryView](inputaccessoryview.md) 與此文字輸入框連結。當此輸入框獲得焦點時，InputAccessoryView 會顯示在鍵盤上方。

| Type   |
| ------ |
| string |

---

### `inputMode`

功能類似 HTML 的 `inputmode` 屬性，決定開啟哪種鍵盤（例如 `numeric`），其優先級高於 `keyboardType`。

支援以下數值：

- `none`
- `text`
- `decimal`
- `numeric`
- `tel`
- `search`
- `email`
- `url`

| Type                                                                        |
| --------------------------------------------------------------------------- |
| enum('decimal', 'email', 'none', 'numeric', 'search', 'tel', 'text', 'url') |

---

### `keyboardAppearance` <div class="label ios">iOS</div>

決定鍵盤的顏色。

| Type                             |
| -------------------------------- |
| enum('default', 'light', 'dark') |

---

### `keyboardType`

決定開啟哪種鍵盤，例如 `numeric`。

所有鍵盤類型的截圖可參閱[此處](https://lefkowitz.me/2018/04/30/visual-guide-to-react-native-textinput-keyboardtype-options/)。

以下數值跨平台通用：

- `default`
- `number-pad`
- `decimal-pad`
- `numeric`
- `email-address`
- `phone-pad`
- `url`

_iOS 專用_

以下數值僅在 iOS 平台有效：

- `ascii-capable`
- `numbers-and-punctuation`
- `name-phone-pad`
- `twitter`
- `web-search`

_Android 專用_

以下數值僅在 Android 平台有效：

- `visible-password`

| Type                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| enum('default', 'email-address', 'numeric', 'phone-pad', 'ascii-capable', 'numbers-and-punctuation', 'url', 'number-pad', 'name-phone-pad', 'decimal-pad', 'twitter', 'web-search', 'visible-password') |

---

### `maxFontSizeMultiplier`

當啟用 `allowFontScaling` 時，指定字體可達到的最大縮放比例。可能數值：

- `null/undefined`（預設）：繼承父節點或全域預設值（0）
- `0`：無上限，忽略父級/全域預設值
- `>= 1`：將此節點的 `maxFontSizeMultiplier` 設為該值

| Type   |
| ------ |
| number |

---

### `maxLength`

限制可輸入的最大字元數。使用此屬性替代在 JavaScript 中實作邏輯，以避免畫面閃爍。

| Type   |
| ------ |
| number |

---

### `multiline`

若為 `true`，文字輸入框可支援多行輸入。預設值為 `false`。

:::note
請注意，這會使文字在 iOS 上對齊頂部，而在 Android 上居中。若要讓兩個平台的行為一致，請將 `textAlignVertical` 設為 `top`。
:::

| Type |
| ---- |
| bool |

---

### `numberOfLines` <div class="label android">Android</div>

設定 `TextInput` 的行數。需搭配 `multiline` 設為 `true` 才能填滿這些行。

| Type   |
| ------ |
| number |

---

### `onBlur`

當文字輸入框失去焦點時觸發的回調函數。

> 注意：若您嘗試從 `nativeEvent` 存取 `text` 值，請注意結果可能為 `undefined`，這可能導致非預期的錯誤。若要取得 TextInput 的最終值，可使用 [`onEndEditing`](textinput#onendediting) 事件，該事件會在編輯完成時觸發。

| Type     |
| -------- |
| function |

---

### `onChange`

當文字輸入框的文字變更時觸發的回調函數。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {eventCount, target, text}}`) => void |

---

### `onChangeText`

當文字輸入框的文字變更時觸發的回調函數。變更後的文字會以單一字串參數的形式傳遞給回調處理函數。

| Type     |
| -------- |
| function |

---

### `onContentSizeChange`

當文字輸入框的內容大小變更時觸發的回調函數。

僅適用於多行文字輸入框。

| Type                                                       |
| ---------------------------------------------------------- |
| (`{nativeEvent: {contentSize: {width, height} }}`) => void |

---

### `onEndEditing`

當文字輸入結束時觸發的回調函數。

| Type     |
| -------- |
| function |

---

### `onPressIn`

當觸碰開始時觸發的回調函數。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onPressOut`

當觸碰結束時觸發的回調函數。

| Type                                                   |
| ------------------------------------------------------ |
| `md ({nativeEvent: [PressEvent](pressevent)}) => void` |

---

### `onFocus`

當文字輸入框獲得焦點時觸發的回調函數。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onKeyPress`

當按鍵被按下時觸發的回調函數。會傳遞一個物件，其中 `keyValue` 為 `'Enter'` 或 `'Backspace'`（對應按鍵），其他情況下則為輸入的字元（包括空格 `' '`）。會在 `onChange` 回調之前觸發。注意：在 Android 上僅處理軟鍵盤的輸入，不處理硬體鍵盤的輸入。

| Type                                        |
| ------------------------------------------- |
| (`{nativeEvent: {key: keyValue} }`) => void |

---

### `onLayout`

在掛載時或佈局變更時觸發。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onScroll`

當內容滾動時觸發。可能包含 `ScrollEvent` 的其他屬性，但在 Android 上為了效能考量不會提供 `contentSize`。

| Type                                                |
| --------------------------------------------------- |
| (`{nativeEvent: {contentOffset: {x, y} }}`) => void |

---

### `onSelectionChange`

當文字輸入框的選取範圍變更時觸發的回調函數。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {selection: {start, end} }}`) => void |

---

### `onSubmitEditing`

當文字輸入的提交按鈕被按下時觸發的回調函數。

| Type                                                  |
| ----------------------------------------------------- |
| (`{nativeEvent: {text, eventCount, target}}`) => void |

請注意，在 iOS 上使用 `keyboardType="phone-pad"` 時此方法不會被呼叫。

---

### `placeholder`

在文字輸入尚未輸入時顯示的字串。

| Type   |
| ------ |
| string |

---

### `placeholderTextColor`

佔位符字串的文字顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `readOnly`

若為 `true`，則文字不可編輯。預設值為 `false`。

| Type |
| ---- |
| bool |

---

### `returnKeyLabel` <div class="label android">Android</div>

設定返回鍵的標籤。可替代 `returnKeyType` 使用。

| Type   |
| ------ |
| string |

---

### `returnKeyType`

決定返回鍵的外觀。在 Android 上也可使用 `returnKeyLabel`。

_跨平台_

以下數值適用於所有平台：

- `done`
- `go`
- `next`
- `search`
- `send`

_僅限 Android_

以下數值僅適用於 Android：

- `none`
- `previous`

_僅限 iOS_

以下數值僅適用於 iOS：

- `default`
- `emergency-call`
- `google`
- `join`
- `route`
- `yahoo`

| Type                                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------- |
| enum('done', 'go', 'next', 'search', 'send', 'none', 'previous', 'default', 'emergency-call', 'google', 'join', 'route', 'yahoo') |

### `rejectResponderTermination` <div class="label ios">iOS</div>

若為 `true`，允許 TextInput 將觸控事件傳遞給父元件。這使得如 SwipeableListView 等元件可以從 TextInput 上滑動，如同 Android 上的預設行為。若為 `false`，TextInput 總是要求處理輸入（除非被禁用）。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `rows` <div class="label android">Android</div>

設定 `TextInput` 的行數。需與 `multiline` 設為 `true` 一起使用以填滿行數。

| Type   |
| ------ |
| number |

---

### `scrollEnabled` <div class="label ios">iOS</div>

若為 `false`，將禁用文字視窗的滾動。預設值為 `true`。僅在 `multiline={true}` 時有效。

| Type |
| ---- |
| bool |

---

### `secureTextEntry`

若為 `true`，文字輸入會隱藏輸入的文字以保護如密碼等敏感資訊。預設值為 `false`。不適用於 `multiline={true}`。

| Type |
| ---- |
| bool |

---

### `selection`

文字輸入選取的起始與結束位置。將起始與結束設為相同值以定位游標。

| Type                                  |
| ------------------------------------- |
| object: `{start: number,end: number}` |

---

### `selectionColor`

文字輸入框的高亮、選取把手和游標顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `selectionHandleColor` <div class="label android">Android</div>

設定選取把手的顏色。與 `selectionColor` 不同，它允許獨立於選取顏色自訂選取把手的顏色。

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `selectTextOnFocus`

若為 `true`，聚焦時會自動選取所有文字。

| Type |
| ---- |
| bool |

---

### `showSoftInputOnFocus`

當設為 `false` 時，將阻止欄位聚焦時顯示軟鍵盤。預設值為 `true`。

| Type |
| ---- |
| bool |

---

### `spellCheck` <div class="label ios">iOS</div>

若為 `false`，則停用拼字檢查樣式（即紅色底線）。預設值繼承自 `autoCorrect`。

| Type |
| ---- |
| bool |

---

### `textAlign`

將輸入文字對齊輸入欄位的左側、中央或右側。

`textAlign` 的可能值為：

- `left`
- `center`
- `right`

| Type                            |
| ------------------------------- |
| enum('left', 'center', 'right') |

---

### `textContentType` <div class="label ios">iOS</div>

向鍵盤和系統提供關於使用者輸入內容預期語義意義的資訊。

:::note
[`autoComplete`](#autocomplete) 提供相同功能且支援所有平台。您可以使用 [`Platform.select`](/docs/next/platform#select) 來處理不同平台的行為差異。

避免同時使用 `textContentType` 和 `autoComplete`。為了向後兼容，當兩個屬性都設定時，`textContentType` 會優先生效。
:::

您可以將 `textContentType` 設為 `username` 或 `password`，以啟用從裝置鑰匙圈自動填入登入資訊的功能。

`newPassword` 可用於表示使用者可能想儲存在鑰匙圈的新密碼輸入欄位，而 `oneTimeCode` 則表示該欄位可透過簡訊收到的驗證碼自動填入。

若要停用自動填入功能，請將 `textContentType` 設為 `none`。

`textContentType` 的可能值為：

- `none`
- `addressCity`
- `addressCityAndState`
- `addressState`
- `birthdate` (iOS 17+)
- `birthdateDay` (iOS 17+)
- `birthdateMonth` (iOS 17+)
- `birthdateYear` (iOS 17+)
- `countryName`
- `creditCardExpiration` (iOS 17+)
- `creditCardExpirationMonth` (iOS 17+)
- `creditCardExpirationYear` (iOS 17+)
- `creditCardFamilyName` (iOS 17+)
- `creditCardGivenName` (iOS 17+)
- `creditCardMiddleName` (iOS 17+)
- `creditCardName` (iOS 17+)
- `creditCardNumber`
- `creditCardSecurityCode` (iOS 17+)
- `creditCardType` (iOS 17+)
- `emailAddress`
- `familyName`
- `fullStreetAddress`
- `givenName`
- `jobTitle`
- `location`
- `middleName`
- `name`
- `namePrefix`
- `nameSuffix`
- `newPassword`
- `nickname`
- `oneTimeCode`
- `organizationName`
- `password`
- `postalCode`
- `streetAddressLine1`
- `streetAddressLine2`
- `sublocality`
- `telephoneNumber`
- `URL`
- `username`

| Type                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| enum('none', 'addressCity', 'addressCityAndState', 'addressState', 'birthdate', 'birthdateDay', 'birthdateMonth', 'birthdateYear', 'countryName', 'creditCardExpiration', 'creditCardExpirationMonth', 'creditCardExpirationYear', 'creditCardFamilyName', 'creditCardGivenName', 'creditCardMiddleName', 'creditCardName', 'creditCardNumber', 'creditCardSecurityCode', 'creditCardType', 'emailAddress', 'familyName', 'fullStreetAddress', 'givenName', 'jobTitle', 'location', 'middleName', 'name', 'namePrefix', 'nameSuffix', 'newPassword', 'nickname', 'oneTimeCode', 'organizationName', 'password', 'postalCode', 'streetAddressLine1', 'streetAddressLine2', 'sublocality', 'telephoneNumber', 'URL', 'username') |

---

### `passwordRules` <div class="label ios">iOS</div>

When using `textContentType` as `newPassword` on iOS we can let the OS know the minimum requirements of the password so that it can generate one that will satisfy them. In order to create a valid string for `PasswordRules` take a look to the [Apple Docs](https://developer.apple.com/password-rules/).

> If passwords generation dialog doesn't appear please make sure that:
>
> - AutoFill is enabled: **Settings** → **Passwords & Accounts** → toggle "On" the **AutoFill Passwords**,
> - iCloud Keychain is used: **Settings** → **Apple ID** → **iCloud** → **Keychain** → toggle "On" the **iCloud Keychain**.

| Type   |
| ------ |
| string |

---

### `style`

Note that not all Text styles are supported, an incomplete list of what is not supported includes:

- `borderLeftWidth`
- `borderTopWidth`
- `borderRightWidth`
- `borderBottomWidth`
- `borderTopLeftRadius`
- `borderTopRightRadius`
- `borderBottomRightRadius`
- `borderBottomLeftRadius`

[Styles](style.md)

| Type                  |
| --------------------- |
| [Text](text.md#style) |

---

### `textBreakStrategy` <div class="label android">Android</div>

Set text break strategy on Android API Level 23+, possible values are `simple`, `highQuality`, `balanced` The default value is `highQuality`.

| Type                                      |
| ----------------------------------------- |
| enum('simple', 'highQuality', 'balanced') |

---

### `underlineColorAndroid` <div class="label android">Android</div>

The color of the `TextInput` underline.

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `value`

The value to show for the text input. `TextInput` is a controlled component, which means the native value will be forced to match this value prop if provided. For most uses, this works great, but in some cases this may cause flickering - one common cause is preventing edits by keeping value the same. In addition to setting the same value, either set `editable={false}`, or set/update `maxLength` to prevent unwanted edits without flicker.

| Type   |
| ------ |
| string |

---

### `lineBreakStrategyIOS` <div class="label ios">iOS</div>

Set line break strategy on iOS 14+. Possible values are `none`, `standard`, `hangul-word` and `push-out`.

| Type                                                        | Default  |
| ----------------------------------------------------------- | -------- |
| enum(`'none'`, `'standard'`, `'hangul-word'`, `'push-out'`) | `'none'` |

## Methods

### `.focus()`

```tsx
focus();
```

Makes the native input request focus.

### `.blur()`

```tsx
blur();
```

Makes the native input lose focus.

### `clear()`

```tsx
clear();
```

Removes all text from the `TextInput`.

---

### `isFocused()`

```tsx
isFocused(): boolean;
```

Returns `true` if the input is currently focused; `false` otherwise.

# Known issues

- [react-native#19096](https://github.com/facebook/react-native/issues/19096): Doesn't support Android's `onKeyPreIme`.
- [react-native#19366](https://github.com/facebook/react-native/issues/19366): Calling .focus() after closing Android's keyboard via back button doesn't bring keyboard up again.
- [react-native#26799](https://github.com/facebook/react-native/issues/26799): Doesn't support Android's `secureTextEntry` when `keyboardType="email-address"` or `keyboardType="phone-pad"`.