---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發連鎖反應，導致更嚴重的故障。程式錯誤會造成糟糕的使用者體驗，最終導致業務損失。預防脆弱程式設計的方法之一，就是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端對端測試，確保您的應用程式如預期般運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為什麼要測試

人非聖賢，孰能無過。測試之所以重要，是因為它能幫助您發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保在您新增功能、重構現有程式碼或升級專案主要依賴套件時，程式碼仍能持續運作。

測試的價值遠超您的想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修復錯誤後重新執行測試，若測試通過，代表錯誤已被修復且不會再次出現在程式碼庫中。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多自動化測試意味著減少手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在您撰寫程式碼時檢查錯誤，但不會實際執行這些程式碼。

- **Linters** 會分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的參數符合函式設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[Flow](https://flow.org/en/docs/) 用於型別檢查。您也可以使用 [TypeScript](typescript)，這是一種會編譯成純 JavaScript 的型別化語言。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試各個零件以確保其安全性和功能性。例如：機翼需承受極端負載彎曲測試、引擎零件需通過耐久性測試、擋風玻璃需模擬鳥擊測試。

軟體開發亦是如此。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試組裝後的整體進行更徹底的測試。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼息息相關。

要讓應用程式更具可測試性，首先需將視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，您就能讓業務邏輯測試（不應依賴 React 元件）獨立於元件本身，而元件的主要職責是渲染應用程式的 UI！

理論上，您甚至可以將所有邏輯和資料獲取移出元件，讓元件專注於渲染。應用狀態將完全獨立於元件，業務邏輯能在完全沒有 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

After writing testable code, it’s time to write some actual tests! The default template of React Native ships with [Jest](https://jestjs.io) testing framework. It includes a preset that's tailored to this environment so you can get productive without tweaking the configuration and mocks straight away—[more on mocks](#mocking) shortly. You can use Jest to write all types of tests featured in this guide.

> If you do test-driven development, you actually write tests first! That way, testability of your code is given.

### Structuring Tests

Your tests should be short and ideally test only one thing. Let's start with an example unit test written with Jest:

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

The test is described by the string passed to the [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) function. Take good care writing the description so that it’s clear what is being tested. Do your best to cover the following:

1. **Given** - some precondition
2. **When** - some action executed by the function that you’re testing
3. **Then** - the expected outcome

This is also known as AAA (Arrange, Act, Assert).

Jest offers [`describe`](https://jestjs.io/docs/en/api#describename-fn) function to help structure your tests. Use `describe` to group together all tests that belong to one functionality. Describes can be nested, if you need that. Other functions you'll commonly use are [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) or [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout) that you can use for setting up the objects you're testing. Read more in the [Jest api reference](https://jestjs.io/docs/en/api).

If your test has many steps or many expectations, you probably want to split it into multiple smaller ones. Also, ensure that your tests are completely independent of one another. Each test in your suite must be executable on its own without first running some other test. Conversely, if you run all your tests together, the first test must not influence the output of the second one.

Lastly, as developers we like when our code works great and doesn't crash. With tests, this is often the opposite. Think of a failed test as of a _good thing!_ When a test fails, it often means something is not right. This gives you an opportunity to fix the problem before it impacts the users.

## Unit tests

Unit tests cover the smallest parts of code, like individual functions or classes.

When the object being tested has any dependencies, you’ll often need to mock them out, as described in the next paragraph.

The great thing about unit tests is that they are quick to write and run. Therefore, as you work, you get fast feedback about whether your tests are passing. Jest even has an option to continuously run tests that are related to code you’re editing: [Watch mode](https://jestjs.io/docs/en/cli#watch).

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### Mocking

Sometimes, when your tested objects have external dependencies, you’ll want to “mock them out.” “Mocking” is when you replace some dependency of your code with your own implementation.

> Generally, using real objects in your tests is better than using mocks but there are situations where this is not possible. For example: when your JS unit test relies on a native module written in Java or Objective-C.

Imagine you’re writing an app that shows the current weather in your city and you’re using some external service or other dependency that provides you with the weather information. If the service tells you that it’s raining, you want to show an image with a rainy cloud. You don’t want to call that service in your tests, because:

- 這可能導致測試變慢且不穩定（因為涉及網路請求）
- 每次執行測試時，該服務可能返回不同的數據
- 第三方服務可能在您急需執行測試時離線！

因此，您可以提供該服務的模擬實現，實際上替換了數千行程式碼和某些連網的溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函數級別到模組級別的模擬都能處理。

## 整合測試

在編寫較大的軟體系統時，其各個部分需要相互交互。在單元測試中，如果您的單元依賴於另一個單元，有時最終會模擬該依賴項，用假的替換它。

在整合測試中，真實的單個單元被組合（與您的應用程式相同）並一起測試，以確保它們的合作按預期工作。這並不是說這裡不會發生模擬：您仍然需要模擬（例如，模擬與天氣服務的通信），但比單元測試中需要的少得多。

> 請注意，關於整合測試含義的術語並不總是一致的。此外，單元測試和整合測試之間的界限可能並不總是清晰。對於本指南，如果您的測試符合以下條件，則屬於「整合測試」：
>
> - 如上所述，組合了應用程式的多個模組
> - 使用外部系統
> - 向其他應用程式（例如天氣服務API）發送網路請求
> - 進行任何類型的檔案或數據庫<abbr title="輸入/輸出">I/O</abbr>操作

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React 元件負責渲染您的應用程式，用戶將直接與其輸出交互。即使您的應用程式的業務邏輯測試覆蓋率高且正確，如果沒有元件測試，您仍可能向用戶交付損壞的UI。元件測試可能屬於單元和整合測試，但因為它們是 React Native 的核心部分，我們將單獨介紹。

對於測試 React 元件，您可能需要測試兩件事：

- 交互：確保元件在用戶交互時行為正確（例如，當用戶按下按鈕時）
- 渲染：確保 React 使用的元件渲染輸出正確（例如，按鈕的外觀和在UI中的位置）

例如，如果您有一個帶有 `onPress` 監聽器的按鈕，您希望測試按鈕是否正確顯示以及點擊按鈕是否被元件正確處理。

有幾個庫可以幫助您測試這些：

- React 的[測試渲染器](https://reactjs.org/docs/test-renderer.html)，與其核心一起開發，提供了一個 React 渲染器，可用於將 React 元件渲染為純 JavaScript 對象，而不依賴於 DOM 或原生移動環境。
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 建立在 React 的測試渲染器之上，並添加了下一段中描述的 `fireEvent` 和 `query` API。

> 元件測試僅是在 Node.js 環境中運行的 JavaScript 測試。它們不考慮任何 iOS、Android 或其他支持 React Native 元件的平台代碼。因此，它們無法給您100%的信心，確保一切對用戶都正常運作。如果 iOS 或 Android 代碼中存在錯誤，它們將無法發現。

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試用戶交互

除了渲染某些UI外，您的元件還處理事件，如 `TextInput` 的 `onChangeText` 或 `Button` 的 `onPress`。它們可能還包含其他函數和事件回調。考慮以下示例：

```jsx
function GroceryShoppingList() {
  const [groceryItem, setGroceryItem] = useState('');
  const [items, setItems] = useState([]);

  const addNewItemToShoppingList = useCallback(() => {
    setItems([groceryItem, ...items]);
    setGroceryItem('');
  }, [groceryItem, items]);

  return (
    <>
      <TextInput
        value={groceryItem}
        placeholder="Enter grocery item"
        onChangeText={text => setGroceryItem(text)}
      />
      <Button
        title="Add the item to list"
        onPress={addNewItemToShoppingList}
      />
      {items.map(item => (
        <Text key={item}>{item}</Text>
      ))}
    </>
  );
}
```

在測試用戶交互時，從用戶的角度測試元件——頁面上有什麼？交互時會發生什麼變化？

作為經驗法則，優先使用用戶可以看到或聽到的內容：

- 使用渲染的文字或[無障礙輔助功能工具](https://reactnative.dev/docs/accessibility#accessibility-properties)進行斷言

相反地，你應該避免：

- 對元件的 props 或 state 進行斷言
- 使用 testID 查詢

避免測試實作細節如 props 或 state——雖然這類測試可行，但它們並非以使用者與元件互動的方式為導向，且容易因重構而失效（例如當你想重新命名某些東西或用 hooks 改寫類別元件時）。

> React 類別元件尤其容易測試其實作細節，如內部狀態、props 或事件處理器。為避免測試實作細節，建議優先使用帶有 Hooks 的函式元件，這會讓依賴元件內部細節變得更加困難。

像 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這樣的元件測試函式庫，透過精心選擇提供的 API，有助於撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法 `changeText` 和 `press` 來模擬使用者與元件的互動，以及查詢函式 `getAllByText` 來找出渲染輸出中匹配的 `Text` 節點。

```jsx
test('given empty GroceryShoppingList, user can add an item to it', () => {
  const {getByPlaceholder, getByText, getAllByText} = render(
    <GroceryShoppingList />,
  );

  fireEvent.changeText(
    getByPlaceholder('Enter grocery item'),
    'banana',
  );
  fireEvent.press(getByText('Add the item to list'));

  const bananaElements = getAllByText('banana');
  expect(bananaElements).toHaveLength(1); // expect 'banana' to be on the list
});
```

這個範例並非測試當你呼叫某個函式時狀態如何變化。它測試的是當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試方式。它是一個非常強大且底層的工具，因此使用時需格外謹慎。

「元件快照」是由 Jest 內建的自訂 React 序列化器所產生的 JSX 風格字串。這個序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換句話說：元件快照是你的元件在測試執行期間所產生之渲染輸出的文字表示。它可能看起來像這樣：

```jsx
<Text
  style={
    Object {
      "fontSize": 20,
      "textAlign": "center",
    }
  }>
  Welcome to React Native!
</Text>
```

進行快照測試時，通常你會先實作你的元件，然後執行快照測試。快照測試接著會建立一個快照，並將其作為參考快照儲存到你的代碼庫中的一個檔案裡。**該檔案會被提交並在代碼審查時檢查**。未來對元件渲染輸出的任何變更都會改變其快照，導致測試失敗。此時你需要更新儲存的參考快照才能使測試通過。該變更同樣需要被提交和審查。

快照有幾個弱點：

- 對開發者或審查者而言，可能難以判斷快照中的變更是有意為之還是錯誤的證據。特別是大型快照可能很快變得難以理解，其附加價值也會降低。
- 當快照被建立時，它在那個時間點被視為正確的——即使渲染輸出實際上可能是錯誤的。
- 當快照失敗時，開發者容易不經仔細檢查變更是否預期，就直接使用 `--updateSnapshot` jest 選項更新快照。因此需要一定的開發紀律。

快照本身並不能確保你的元件渲染邏輯正確，它們主要擅長防範意外變更，以及檢查測試中的 React 樹下的元件是否接收到預期的 props（樣式等）。

我們建議你只使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。如果你想測試兩個 React 元件狀態之間的變更，可以使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。如有疑問，優先選擇如前一節所述的明確期望值。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，你從使用者的角度驗證你的應用程式在裝置（或模擬器/仿真器）上是否如預期運作。

這需要將你的應用程式建置為發布版本，並針對該版本運行測試。在端對端測試中，你不再需要考慮 React 元件、React Native API、Redux 儲存或任何業務邏輯。這些並非端對端測試的目的，且在端對端測試過程中甚至無法存取這些內容。

相反地，端對端測試函式庫讓你能夠在應用程式畫面中尋找並控制元素：例如，你可以像真實用戶一樣實際點擊按鈕或在 `TextInput` 中輸入文字。接著，你可以斷言某個元素是否存在於應用程式畫面中、是否可見、包含什麼文字等等。

端對端測試能為你提供應用程式某部分正常運作的最大信心。其代價包括：

- 撰寫這些測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

嘗試用端對端測試覆蓋應用程式的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。你添加的測試越多，信心就越高，但維護和執行它們的時間也會越多。請權衡利弊，決定什麼最適合你。

有幾種可用的端對端測試工具：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個流行的框架，因為它是專為 React Native 應用程式設計的。另一個在 iOS 和 Android 應用程式領域流行的函式庫是 [Appium](http://appium.io/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

我們希望你在閱讀本指南時有所收穫。測試應用程式的方法有很多種，一開始可能很難決定使用什麼。然而，我們相信一旦你開始為出色的 React Native 應用程式添加測試，一切都會變得清晰。那麼還在等什麼？提高你的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文件](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](http://appium.io/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並完整貢獻。_