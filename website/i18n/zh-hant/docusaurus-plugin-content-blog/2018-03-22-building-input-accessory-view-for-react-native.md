---
title: 'Building <InputAccessoryView> For React Native'
author: Peter Argany
authorTitle: Software Engineer at Facebook
authorURL: 'https://github.com/PeteTheHeat'
authorImageURL: 'https://avatars3.githubusercontent.com/u/6011080?s=400&u=028e28081107d0ab16a5cb22baca43c080f5fa50&v=4'
authorTwitter: peterargany
tags: [engineering]
---

## 開發動機

三年前，GitHub 上就有議題提出希望 React Native 能支援輸入輔助視圖（input accessory view）。

<img src="/blog/assets/input-accessory-1.png" />

在接下來的幾年裡，這個議題累積了無數的「+1」、各種變通方案，但 React Native 本身卻始終沒有實質改變——直到今天。我們從 iOS 開始，[公開了一個 API](/docs/inputaccessoryview) 來存取原生的輸入輔助視圖，並很興奮能分享我們是如何實現它的。

## 背景知識

究竟什麼是輸入輔助視圖？閱讀 [Apple 的開發者文件](https://developer.apple.com/documentation/uikit/uiresponder/1621119-inputaccessoryview?language=objc)後，我們了解到這是一個自訂視圖，可以在接收者成為第一響應者時，錨定在系統鍵盤的頂部。任何繼承自 `UIResponder` 的類別都可以將 `.inputAccessoryView` 屬性重新宣告為可讀寫，並在此管理自訂視圖。響應者基礎設施會掛載這個視圖，並保持它與系統鍵盤同步。像是拖曳或點擊等會關閉鍵盤的手勢，會在框架層級應用到輸入輔助視圖上。這讓我們能打造出具有互動式鍵盤關閉功能的內容，這是像 iMessage 和 WhatsApp 這類頂尖通訊應用不可或缺的功能。

將視圖錨定在鍵盤頂部有兩種常見的使用情境。第一種是建立鍵盤工具列，像是 Facebook 發文框的背景選擇器。

<img src="/blog/assets/input-accessory-2.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

在這個情境中，鍵盤聚焦在文字輸入欄位，而輸入輔助視圖則用來提供額外的鍵盤功能。這些功能會根據輸入欄位的類型而有所不同。在地圖應用中可能是地址建議，而在文字編輯器中則可能是富文本格式工具。

<hr style={{clear: 'both', marginBottom: 20}} />

在這個情境中，擁有 `<InputAccessoryView>` 的 Objective-C UIResponder 應該很明確。`<TextInput>` 成為了第一響應者，而在底層這會變成 `UITextView` 或 `UITextField` 的實例。

第二種常見情境是黏性文字輸入：

<img src="/blog/assets/input-accessory-3.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

在這裡，文字輸入實際上是輸入輔助視圖本身的一部分。這常見於通訊應用中，讓使用者在滾動瀏覽先前訊息的同時也能撰寫新訊息。

<hr style={{clear: 'both', marginBottom: 20}} />

在這個例子中，誰擁有 `<InputAccessoryView>`？能再次是 `UITextView` 或 `UITextField` 嗎？文字輸入位於輸入輔助視圖「內部」，這聽起來像是循環依賴。光是解決這個問題就足以寫成[另一篇部落格文章](https://derpturkey.com/uitextfield-docked-like-ios-messenger/)。劇透：擁有者是一個通用的 `UIView` 子類別，我們手動讓它[成為第一響應者](https://developer.apple.com/documentation/uikit/uiresponder/1621113-becomefirstresponder?language=objc)。

## API 設計

現在我們知道了什麼是 `<InputAccessoryView>`，以及我們想如何使用它。下一步是設計一個能同時適用兩種使用情境，並能與現有 React Native 元件（如 `<TextInput>`）良好配合的 API。

對於鍵盤工具列，有幾點我們需要考慮：

1. 我們希望能將任何通用的 React Native 視圖層級提升到 `<InputAccessoryView>` 中。
2. 我們希望這個通用且獨立的視圖層級能接收觸控，並能操作應用程式狀態。
3. 我們希望能將 `<InputAccessoryView>` 連結到特定的 `<TextInput>`。
4. 我們希望能跨多個文字輸入共享同一個 `<InputAccessoryView>`，而無需重複任何程式碼。

我們可以使用類似 [React portals](https://reactjs.org/docs/portals.html) 的概念來實現第 1 點。在這個設計中，我們將 React Native 視圖傳送到由響應者基礎設施管理的 `UIView` 層級。由於 React Native 視圖會渲染為 UIViews，這其實相當直觀——我們只需要覆寫：

`- (void)insertReactSubview:(UIView *)subview atIndex:(NSInteger)atIndex`

and pipe all the subviews to a new UIView hierarchy. For #2, we set up a new [RCTTouchHandler](https://github.com/facebook/react-native/blob/master/React/Base/RCTTouchHandler.h) for the `<InputAccessoryView>`. State updates are achieved by using regular event callbacks. For #3 and #4, we use the [nativeID](https://github.com/facebook/react-native/blob/master/React/Views/UIView%2BReact.h#L28) field to locate the accessory view UIView hierarchy in native code during the creation of a `<TextInput>` component. This function uses the `.inputAccessoryView` property of the underlying native text input. Doing this effectively links `<InputAccessoryView>` to `<TextInput>` in their ObjC implementations.

Supporting sticky text inputs (scenario 2) adds a few more constraints. For this design, the input accessory view has a text input as a child, so linking via nativeID is not an option. Instead, we set the `.inputAccessoryView` of a generic off-screen `UIView` to our native `<InputAccessoryView>` hierarchy. By manually telling this generic `UIView` to become first responder, the hierarchy is mounted by responder infrastructure. This concept is explained thoroughly in the aforementioned blog post.

## Pitfalls

Of course not everything was smooth sailing while building this API. Here are a few pitfalls we encountered, along with how we fixed them.

An initial idea for building this API involved listening to `NSNotificationCenter` for UIKeyboardWill(Show/Hide/ChangeFrame) events. This pattern is used in some open-sourced libraries, and internally in some parts of the Facebook app. Unfortunately, `UIKeyboardDidChangeFrame` events were not being called in time to update the `<InputAccessoryView>` frame on swipes. Also, changes in keyboard height are not captured by these events. This creates a class of bugs that manifest like this:

<img src="/blog/assets/input-accessory-4.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

On iPhone X, text and emoji keyboard are different heights. Most applications using keyboard events to manipulate text input frames had to fix the above bug. Our solution was to commit to using the `.inputAccessoryView` property, which meant that the responder infrastructure handles frame updates like this.

<hr style={{clear: 'both', marginBottom: 20}} />

Another tricky bug we encountered was avoiding the home pill on iPhone X. You may be thinking, “Apple developed [safeAreaLayoutGuide](https://developer.apple.com/documentation/uikit/uiview/2891102-safearealayoutguide?language=objc) for this very reason, this is trivial!”. We were just as naive. The first issue is that the native `<InputAccessoryView>` implementation has no window to anchor to until the moment it is about to appear. That's alright, we can override `-(BOOL)becomeFirstResponder` and enforce layout constraints there. Adhering to these constraints bumps the accessory view up, but another bug arises: <img src="/blog/assets/input-accessory-5.gif" style={{float: 'left', paddingRight: 70, paddingTop: 20}} />

The input accessory view successfully avoids the home pill, but now content behind the unsafe area is visible. The solution lies in this [radar](https://www.openradar.me/34411433). I wrapped the native `<InputAccessoryView>` hierarchy in a container which doesn't conform to the `safeAreaLayoutGuide` constraints. The native container covers the content in the unsafe area, while the `<InputAccessoryView>` stays within the safe area boundaries.

<hr style={{clear: 'both', marginBottom: 20}} />

## Example Usage

Here's an example which builds a keyboard toolbar button to reset `<TextInput>` state.

```jsx
class TextInputAccessoryViewExample extends React.Component<
  {},
  *,
> {
  constructor(props) {
    super(props);
    this.state = {text: 'Placeholder Text'};
  }

  render() {
    const inputAccessoryViewID = 'inputAccessoryView1';
    return (
      <View>
        <TextInput
          style={styles.default}
          inputAccessoryViewID={inputAccessoryViewID}
          onChangeText={text => this.setState({text})}
          value={this.state.text}
        />
        <InputAccessoryView nativeID={inputAccessoryViewID}>
          <View style={{backgroundColor: 'white'}}>
            <Button
              onPress={() =>
                this.setState({text: 'Placeholder Text'})
              }
              title="Reset Text"
            />
          </View>
        </InputAccessoryView>
      </View>
    );
  }
}
```

Another example for [Sticky Text Inputs can be found in the repository](https://github.com/facebook/react-native/blob/84ef7bc372ad870127b3e1fb8c13399fe09ecd4d/RNTester/js/InputAccessoryViewExample.js).

## When will I be able to use this?

The full commit for this feature implementation is [here](https://github.com/facebook/react-native/commit/38197c8230657d567170cdaf8ff4bbb4aee732b8). [`<InputAccessoryView>`](/docs/next/inputaccessoryview) will be available in the upcoming v0.55.0 release.

Happy keyboarding :)