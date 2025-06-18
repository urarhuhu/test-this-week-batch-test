---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化測試方法，從靜態分析到端到端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人類難免犯錯。測試的重要性在於幫助發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值超乎想像。修復程式錯誤的最佳方法之一，就是撰寫能暴露該錯誤的失敗測試。當您修復錯誤後重新執行測試，若通過則表示錯誤已修復且不會再次引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助理解現有程式碼的運作方式。

最後，自動化測試能減少手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但不會實際執行這些程式碼。

- **Linter**會分析程式碼，捕捉常見錯誤（如未使用的程式碼），避免潛在陷阱，並標記違反風格指南的行為（例如使用Tab而非空格，或反之，取決於您的配置）。
- **型別檢查**確保傳遞給函式的結構符合預期，防止將字串傳遞給預期接收數字的計數函式等情況。

React Native預設配置了兩種工具：[ESLint](https://eslint.org/)用於程式碼檢查，[TypeScript](typescript)用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造為例——在模型首次起飛展示所有複雜系統協同運作前，會單獨測試每個零件以確保安全性和功能性。例如機翼需承受極端負載彎曲測試，引擎零件需通過耐久性測試，擋風玻璃需模擬鳥擊測試。

軟體開發同理。與其在單一龐大檔案中撰寫數千行程式碼，不如將程式碼拆分為多個小型模組，這樣能比測試完整組裝的系統更徹底。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼密切相關。

要提高應用程式的可測試性，首先應將React元件（視圖部分）與業務邏輯和應用狀態分離（無論您使用Redux、MobX或其他解決方案）。如此一來，業務邏輯測試（不應依賴React元件）就能獨立於主要負責渲染UI的元件！

理論上，您可以將所有邏輯和資料獲取完全移出元件，使元件專注於渲染。應用狀態將完全獨立於元件，業務邏輯甚至能在沒有任何React元件的情況下運作！

> 我們建議您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

完成可測試程式碼後，就是時候撰寫實際測試了！React Native的預設模板內建[Jest](https://jestjs.io)測試框架，包含針對此環境量身打造的預設配置，讓您無需立即調整設定和模擬（mock）就能開始工作——稍後將[詳細說明模擬](#mocking)。您可以使用Jest撰寫本指南提到的所有測試類型。

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

## Unit Tests

Unit tests cover the smallest parts of code, like individual functions or classes.

When the object being tested has any dependencies, you’ll often need to mock them out, as described in the next paragraph.

The great thing about unit tests is that they are quick to write and run. Therefore, as you work, you get fast feedback about whether your tests are passing. Jest even has an option to continuously run tests that are related to code you’re editing: [Watch mode](https://jestjs.io/docs/en/cli#watch).

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### Mocking

Sometimes, when your tested objects have external dependencies, you’ll want to “mock them out.” “Mocking” is when you replace some dependency of your code with your own implementation.

> Generally, using real objects in your tests is better than using mocks but there are situations where this is not possible. For example: when your JS unit test relies on a native module written in Java or Objective-C.

Imagine you’re writing an app that shows the current weather in your city and you’re using some external service or other dependency that provides you with the weather information. If the service tells you that it’s raining, you want to show an image with a rainy cloud. You don’t want to call that service in your tests, because:

- It could make the tests slow and unstable (because of the network requests involved)
- The service may return different data every time you run the test
- Third party services can go offline when you really need to run tests!

Therefore, you can provide a mock implementation of the service, effectively replacing thousands of lines of code and some internet-connected thermometers!

> Jest comes with [support for mocking](https://jestjs.io/docs/en/mock-functions#mocking-modules) from function level all the way to module level mocking.

## Integration Tests

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若被測試單元依賴其他組件，往往需要透過模擬(mock)來替換這些依賴項。

整合測試則是將實際的獨立單元（如同在應用程式中那樣）組合起來共同測試，以確保它們的協作符合預期。這並非意味著完全不需要模擬——例如仍需模擬與天氣服務的通信——但相較單元測試，模擬的使用會大幅減少。

> 請注意「整合測試」的定義在業界並不完全一致。單元測試與整合測試的界線有時也較模糊。本指南中，符合以下任一條件即歸類為整合測試：
>
> - 組合多個應用模組進行測試
> - 使用外部系統
> - 進行網路呼叫（如呼叫天氣服務API）
> - 涉及檔案或資料庫<abbr title="輸入/輸出">I/O</abbr>操作

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件直接負責渲染介面並與用戶互動。即使業務邏輯已通過完整測試，若缺乏元件測試，仍可能交付有缺陷的UI。元件測試可歸類為單元或整合測試，但由於其在React Native中的核心地位，我們單獨討論。

React元件測試主要關注兩個面向：

- 互動行為：確保元件對用戶操作（如點擊按鈕）的反符合預期
- 渲染輸出：確保React生成的渲染結果正確（如按鈕外觀與UI位置）

例如測試帶有`onPress`監聽的按鈕時，需同時驗證按鈕視覺呈現與點擊事件處理的正確性。

常用測試工具包括：

- React官方[Test Renderer](https://reactjs.org/docs/test-renderer.html)：可在不依賴DOM或原生環境的情況下，將元件渲染為純JavaScript對象
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)：基於Test Renderer擴充，提供後文將介紹的`fireEvent`與`query`API

> 元件測試僅在Node.js環境執行JavaScript測試，不涉及iOS/Android等平台的底層原生代碼。因此無法100%保證用戶端運作完全正常——若原生代碼存在缺陷，此類測試無法發現。

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試用戶互動

除了渲染UI，元件還需處理如`TextInput`的`onChangeText`或`Button`的`onPress`等事件。參考以下範例：

```tsx
function GroceryShoppingList() {
  const [groceryItem, setGroceryItem] = useState('');
  const [items, setItems] = useState<string[]>([]);

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

測試用戶互動時，應從用戶視角出發：頁面顯示什麼？互動後產生哪些變化？

基本原則：優先基於用戶可感知的元素進行驗證：

- 使用渲染文字或[無障礙輔助屬性](https://reactnative.dev/docs/accessibility#accessibility-properties)進行斷言

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢元件

避免測試實作細節（如props/state）。這類測試雖可行，但未貼近真實用戶交互情境，且容易因重構（例如改用Hooks改寫class元件）而失效。

> React 類別元件特別容易測試其實作細節，例如內部狀態、props 或事件處理器。為避免測試實作細節，建議優先使用搭配 Hooks 的函式元件，這會讓依賴元件內部邏輯變得更加困難。

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 來協助撰寫以使用者為核心的測試。以下範例使用 `fireEvent` 方法的 `changeText` 和 `press` 來模擬使用者與元件的互動，並透過查詢函式 `getAllByText` 在渲染輸出中尋找符合的 `Text` 節點。

```tsx
test('given empty GroceryShoppingList, user can add an item to it', () => {
  const {getByPlaceholderText, getByText, getAllByText} = render(
    <GroceryShoppingList />,
  );

  fireEvent.changeText(
    getByPlaceholderText('Enter grocery item'),
    'banana',
  );
  fireEvent.press(getByText('Add the item to list'));

  const bananaElements = getAllByText('banana');
  expect(bananaElements).toHaveLength(1); // expect 'banana' to be on the list
});
```

此範例並非測試呼叫函式時狀態如何變化，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試技術。由於這是極強大且底層的工具，使用時需格外謹慎。

「元件快照」是由 Jest 內建的自訂 React 序列化器產生的 JSX 風格字串。此序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換言之：元件快照是測試執行期間_生成_的元件渲染輸出的文字表示。其外觀可能如下：

```tsx
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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照儲存至程式庫中的檔案。**該檔案需提交並在程式碼審查時檢查**。後續任何元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而此變更同樣需提交並審查。

快照測試有以下弱點：

- 對開發者或審查者而言，難以判斷快照變更是否為預期行為或錯誤證據。大型快照尤其容易變得難以理解，其附加價值隨之降低。
- 快照建立時即被視為正確版本——即使實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易直接使用 `--updateSnapshot` 選項更新快照，而未仔細檢查變更是否合理。因此需要一定的開發紀律。

快照本身無法保證元件渲染邏輯的正確性，其主要價值在於防範意外變更，以及驗證測試中 React 樹下的元件是否接收預期的 props（樣式等）。

建議僅使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。若需測試兩個 React 元件狀態間的_差異_，請使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。若有疑慮，建議採用前文所述的明確斷言方式。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在裝置（或模擬器）上是否如預期般運作，測試角度完全模擬使用者行為。

此測試需以發布配置建置應用程式，並對其執行測試。E2E 測試不再考量 React 元件、React Native API、Redux 儲存庫或任何業務邏輯——這些並非 E2E 測試的目的，且在測試過程中甚至無法存取這些內容。

相反地，E2E 測試函式庫讓您能尋找並控制應用畫面中的元素：例如，您可以_實際_點擊按鈕或在 `TextInput` 中輸入文字，完全模擬真實使用者操作。接著可對畫面元素進行斷言，例如判斷特定元素是否存在、是否可見、包含何種文字等。

E2E 測試能為應用程式的特定功能提供最高級別的信心保證。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過或失敗，而程式碼沒有任何變更的測試）

嘗試用端對端測試涵蓋應用的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會增加。請權衡利弊，選擇最適合的方案。

目前有幾種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個熱門框架，因為它專為 React Native 應用設計。在 iOS 和 Android 應用領域，另一個流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用的方法有很多種，一開始可能很難決定使用哪種方式。但我們相信，一旦您開始為出色的 React Native 應用添加測試，一切就會變得清晰。還在等什麼？趕緊提升測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文檔](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_