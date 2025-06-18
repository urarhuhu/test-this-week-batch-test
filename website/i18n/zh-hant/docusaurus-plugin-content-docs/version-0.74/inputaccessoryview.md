---
id: inputaccessoryview
title: InputAccessoryView
---

一個允許在 iOS 上自定義鍵盤輸入附屬視圖的組件。當 `TextInput` 獲得焦點時，輸入附屬視圖會顯示在鍵盤上方。此組件可用於創建自定義工具欄。

使用此組件時，請將您的自定義工具欄用 `InputAccessoryView` 組件包裹，並設置一個 `nativeID`。然後，將該 `nativeID` 作為 `inputAccessoryViewID` 傳遞給您所需的任何 `TextInput`。基本範例：

```SnackPlayer name=InputAccessoryView&supportedPlatforms=ios
import React, {useState} from 'react';
import {Button, InputAccessoryView, ScrollView, TextInput} from 'react-native';

const App = () => {
  const inputAccessoryViewID = 'uniqueID';
  const initialText = '';
  const [text, setText] = useState(initialText);

  return (
    <>
      <ScrollView keyboardDismissMode="interactive">
        <TextInput
          style={{
            padding: 16,
            marginTop: 50,
          }}
          inputAccessoryViewID={inputAccessoryViewID}
          onChangeText={setText}
          value={text}
          placeholder={'Please type here…'}
        />
      </ScrollView>
      <InputAccessoryView nativeID={inputAccessoryViewID}>
        <Button onPress={() => setText(initialText)} title="Clear text" />
      </InputAccessoryView>
    </>
  );
};

export default App;
```

此組件還可用於創建黏性文本輸入（固定在鍵盤頂部的文本輸入）。為此，請用 `InputAccessoryView` 組件包裹一個 `TextInput`，並且不要設置 `nativeID`。有關範例，請參閱 [InputAccessoryViewExample.js](https://github.com/facebook/react-native/blob/main/packages/rn-tester/js/examples/InputAccessoryView/InputAccessoryViewExample.js)。

---

# 參考文獻

## 屬性

### `backgroundColor`

| Type               |
| ------------------ |
| [color](colors.md) |

---

### `nativeID`

用於將此 `InputAccessoryView` 與指定 `TextInput` 關聯的 ID。

| Type   |
| ------ |
| string |

---

### `style`

| Type                              |
| --------------------------------- |
| [View Style](view-style-props.md) |

# 已知問題

- [react-native#18997](https://github.com/facebook/react-native/issues/18997): 不支援多行 `TextInput`
- [react-native#20157](https://github.com/facebook/react-native/issues/20157): 無法與底部標籤欄一起使用