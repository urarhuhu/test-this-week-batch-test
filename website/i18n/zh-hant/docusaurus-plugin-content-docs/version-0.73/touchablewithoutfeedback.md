---
id: touchablewithoutfeedback
title: TouchableWithoutFeedback
---

> 如果您正在尋找更全面且具未來性的觸控輸入處理方式，請參閱 [Pressable](pressable.md) API。

除非有充分理由，否則請勿使用。所有響應按壓的元素在被觸碰時都應有視覺回饋。

`TouchableWithoutFeedback` 僅支援單個子元素。若需包含多個子元件，請將其包裹在 View 中。需特別注意，`TouchableWithoutFeedback` 的工作原理是克隆其子元素並套用回應屬性，因此任何中介元件都必須將這些屬性傳遞至底層的 React Native 元件。

## 使用模式

```tsx
function MyComponent(props: MyComponentProps) {
  return (
    <View {...props} style={{flex: 1, backgroundColor: '#fff'}}>
      <Text>My Component</Text>
    </View>
  );
}

<TouchableWithoutFeedback onPress={() => alert('Pressed!')}>
  <MyComponent />
</TouchableWithoutFeedback>;
```

## 範例

```SnackPlayer name=TouchableWithoutFeedback
import React, {useState} from 'react';
import {StyleSheet, TouchableWithoutFeedback, Text, View} from 'react-native';

const TouchableWithoutFeedbackExample = () => {
  const [count, setCount] = useState(0);

  const onPress = () => {
    setCount(count + 1);
  };

  return (
    <View style={styles.container}>
      <View style={styles.countContainer}>
        <Text style={styles.countText}>Count: {count}</Text>
      </View>
      <TouchableWithoutFeedback onPress={onPress}>
        <View style={styles.button}>
          <Text>Touch Here</Text>
        </View>
      </TouchableWithoutFeedback>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    paddingHorizontal: 10,
  },
  button: {
    alignItems: 'center',
    backgroundColor: '#DDDDDD',
    padding: 10,
  },
  countContainer: {
    alignItems: 'center',
    padding: 10,
  },
  countText: {
    color: '#FF00FF',
  },
});

export default TouchableWithoutFeedbackExample;
```

---

# 參考文獻

## 屬性

### `accessibilityIgnoresInvertColors`

| Type    |
| ------- |
| Boolean |

---

### `accessible`

當值為 `true` 時，表示該視圖為無障礙元素。預設情況下，所有可觸碰元素均具無障礙功能。

| Type |
| ---- |
| bool |

---

### `accessibilityLabel`

覆寫螢幕閱讀器在使用者與元素互動時朗讀的文字。預設情況下，標籤由遍歷所有子元素並以空格分隔累積的 `Text` 節點構成。

| Type   |
| ------ |
| string |

---

### `accessibilityLanguage` <div class="label ios">iOS</div>

此值指示螢幕閱讀器在使用者與元素互動時應使用的語言，需符合 [BCP 47 規範](https://www.rfc-editor.org/info/bcp47)。

詳見 [iOS `accessibilityLanguage` 文件](https://developer.apple.com/documentation/objectivec/nsobject/1615192-accessibilitylanguage)以獲取更多資訊。

| Type   |
| ------ |
| string |

---

### `accessibilityHint`

無障礙提示協助使用者在無障礙標籤未能明確說明時，理解對無障礙元素執行動作後的結果。

| Type   |
| ------ |
| string |

---

### `accessibilityRole`

`accessibilityRole` 向輔助技術使用者傳達元件的用途。

`accessibilityRole` 可為以下其中一項：

- `'none'` - 當元素沒有角色時使用。
- `'button'` - 當元素應被視為按鈕時使用。
- `'link'` - 當元素應被視為連結時使用。
- `'search'` - 當文字輸入框元素也應被視為搜尋欄位時使用。
- `'image'` - 當元素應被視為圖片時使用。可與按鈕或連結等組合使用。
- `'keyboardkey'` - 當元素作為鍵盤按鍵時使用。
- `'text'` - 當元素應被視為無法變更的靜態文字時使用。
- `'adjustable'` - 當元素可被「調整」時使用（例如滑桿）。
- `'imagebutton'` - 當元素應被視為按鈕且同時是圖片時使用。
- `'header'` - 當元素作為內容區段的標題時使用（例如導覽列的標題）。
- `'summary'` - 當元素可用於提供應用程式啟動時當前條件的快速摘要時使用。
- `'alert'` - 當元素包含需向使用者展示的重要文字時使用。
- `'checkbox'` - 當元素代表可勾選、取消勾選或混合勾選狀態的核取方塊時使用。
- `'combobox'` - 當元素代表可讓使用者從多個選項中選擇的下拉式方塊時使用。
- `'menu'` - 當元件是選單時使用。
- `'menubar'` - 當元件是多個選單的容器時使用。
- `'menuitem'` - 用於代表選單中的項目。
- `'progressbar'` - 用於代表指示任務進度的元件。
- `'radio'` - 用於代表單選按鈕。
- `'radiogroup'` - 用於代表一組單選按鈕。
- `'scrollbar'` - 用於代表捲軸。
- `'spinbutton'` - 用於代表可開啟選項清單的按鈕。
- `'switch'` - 用於代表可開啟或關閉的開關。
- `'tab'` - 用於代表分頁標籤。
- `'tablist'` - 用於代表分頁標籤清單。
- `'timer'` - 用於代表計時器。
- `'toolbar'` - 用於代表工具列（動作按鈕或元件的容器）。

| Type   |
| ------ |
| string |

---

### `accessibilityState`

向輔助技術使用者描述元件當前的狀態。

詳見[無障礙指南](accessibility.md#accessibilitystate-ios-android)。

| Type                                                                                             |
| ------------------------------------------------------------------------------------------------ |
| object: `{disabled: bool, selected: bool, checked: bool or 'mixed', busy: bool, expanded: bool}` |

---

### `accessibilityActions`

無障礙操作允許輔助技術以程式化方式呼叫元件的操作。`accessibilityActions`屬性應包含操作物件的清單，每個操作物件應包含名稱和標籤欄位。

詳見[無障礙指南](accessibility.md#accessibility-actions)。

| Type  |
| ----- |
| array |

---

### `aria-busy`

表示元素正在修改中，輔助技術可能需要等待變更完成後再通知使用者。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-checked`

表示可勾選元素的狀態。此欄位可接受布林值或「mixed」字串來代表混合狀態的核取方塊。

| Type             | Default |
| ---------------- | ------- |
| boolean, 'mixed' | false   |

---

### `aria-disabled`

表示元素可感知但已停用，因此無法編輯或操作。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-expanded`

表示可展開元素當前是展開還是摺疊狀態。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-hidden`

指示此無障礙元素內包含的無障礙元素是否隱藏。

例如，在包含兄弟視圖 `A` 和 `B` 的視窗中，將視圖 `B` 的 `aria-hidden` 設為 `true` 會導致 VoiceOver 忽略視圖 `B` 中的元素。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-label`

定義一個字串值，用於標記互動元素。

| Type   |
| ------ |
| string |

---

### `aria-live` <div class="label android">Android</div>

指示元素將被更新，並描述用戶代理、無障礙技術和用戶可以從實時區域期望的更新類型。

- **off** 無障礙服務不應宣布對此視圖的更改。
- **polite** 無障礙服務應宣布對此視圖的更改。
- **assertive** 無障礙服務應中斷正在進行的語音，立即宣布對此視圖的更改。

| Type                                     | Default |
| ---------------------------------------- | ------- |
| enum(`'assertive'`, `'off'`, `'polite'`) | `'off'` |

---

### `aria-modal` <div class="label ios">iOS</div>

布爾值，指示 VoiceOver 是否應忽略接收者兄弟視圖中的元素。優先於 [`accessibilityViewIsModal`](#accessibilityviewismodal-ios) 屬性。

| Type    | Default |
| ------- | ------- |
| boolean | false   |

---

### `aria-selected`

指示可選元素當前是否被選中。

| Type    |
| ------- |
| boolean |

### `onAccessibilityAction`

當用戶執行無障礙操作時調用。此函數的唯一參數是一個包含要執行動作名稱的事件。

詳見[無障礙指南](accessibility.md#accessibility-actions)以獲取更多資訊。

| Type     |
| -------- |
| function |

---

### `accessibilityValue`

表示組件的當前值。它可以是組件值的文字描述，或者對於基於範圍的組件（如滑塊和進度條），它包含範圍資訊（最小值、當前值和最大值）。

詳見[無障礙指南](accessibility.md#accessibilityvalue-ios-android)以獲取更多資訊。

| Type                                                            |
| --------------------------------------------------------------- |
| object: `{min: number, max: number, now: number, text: string}` |

---

### `aria-valuemax`

表示基於範圍的組件（如滑塊和進度條）的最大值。優先於 `accessibilityValue` 屬性中的 `max` 值。

| Type   |
| ------ |
| number |

---

### `aria-valuemin`

表示基於範圍的組件（如滑塊和進度條）的最小值。優先於 `accessibilityValue` 屬性中的 `min` 值。

| Type   |
| ------ |
| number |

---

### `aria-valuenow`

表示基於範圍的組件（如滑塊和進度條）的當前值。優先於 `accessibilityValue` 屬性中的 `now` 值。

| Type   |
| ------ |
| number |

---

### `aria-valuetext`

表示組件的文字描述。優先於 `accessibilityValue` 屬性中的 `text` 值。

| Type   |
| ------ |
| string |

---

### `delayLongPress`

從 `onPressIn` 開始到呼叫 `onLongPress` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `delayPressIn`

從觸碰開始到呼叫 `onPressIn` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `delayPressOut`

從觸碰釋放到呼叫 `onPressOut` 的持續時間（以毫秒為單位）。

| Type   |
| ------ |
| number |

---

### `disabled`

若為 true，則停用此元件的所有互動功能。

| Type |
| ---- |
| bool |

---

### `hitSlop`

定義觸碰可以從按鈕多遠的距離開始。當移出按鈕時，此值會加到 `pressRetentionOffset` 上。

> 觸碰區域永遠不會超出父視圖的邊界，且當觸碰同時命中兩個重疊的視圖時，兄弟視圖的 Z-index 永遠優先。

| Type                   |
| ---------------------- |
| [Rect](rect) or number |

### `id`

用於在原生程式碼中定位此視圖。優先於 `nativeID` 屬性。

| Type   |
| ------ |
| string |

---

### `onBlur`

當項目失去焦點時呼叫。

| Type     |
| -------- |
| function |

---

### `onFocus`

當項目獲得焦點時呼叫。

| Type     |
| -------- |
| function |

---

### `onLayout`

在掛載時和佈局變更時呼叫。

| Type                                                     |
| -------------------------------------------------------- |
| `md ({nativeEvent: [LayoutEvent](layoutevent)}) => void` |

---

### `onLongPress`

若 `onPressIn` 後的時間超過 370 毫秒則呼叫。此時間段可透過 [`delayLongPress`](#delaylongpress) 自訂。

| Type     |
| -------- |
| function |

---

### `onPress`

當觸碰釋放時呼叫，但若被取消（例如因滾動搶佔了回應器鎖）則不呼叫。第一個函數參數是一個 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `onPressIn`

當可觸碰元件被按下時立即呼叫，甚至在 `onPress` 之前呼叫。這在進行網路請求時很有用。第一個函數參數是一個 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `onPressOut`

當觸碰釋放時立即呼叫，甚至在 `onPress` 之前呼叫。第一個函數參數是一個 [PressEvent](pressevent) 形式的事件。

| Type     |
| -------- |
| function |

---

### `pressRetentionOffset`

當滾動視圖被停用時，此屬性定義觸碰可以移出按鈕多遠的距離才會停用按鈕。一旦停用，嘗試將其移回，您會看到按鈕再次被啟用！在滾動視圖停用時來回移動多次。請確保傳入常數以減少記憶體分配。

| Type                   |
| ---------------------- |
| [Rect](rect) or number |

---

### `nativeID`

| Type   |
| ------ |
| string |

---

### `testID`

用於在端到端測試中定位此視圖。

| Type   |
| ------ |
| string |

---

### `touchSoundDisabled` <div class="label android">Android</div>

若為 true，觸摸時不會播放系統聲音。

| Type    |
| ------- |
| Boolean |