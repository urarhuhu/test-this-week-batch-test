---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。這些錯誤會導致糟糕的使用者體驗，最終造成業務損失。防止程式碼脆弱的方法之一，是在發布前進行測試。

本指南將介紹多種自動化測試方法，從靜態分析到端到端測試，確保應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人非聖賢，孰能無過。測試的重要性在於幫助發現錯誤並驗證程式碼是否正常運作。更重要的是，它能確保未來新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值超乎想像。修復程式錯誤的最佳方法之一，就是撰寫能暴露該錯誤的失敗測試。當你修正錯誤後重新執行測試，若通過則代表錯誤已被修復且不會再次引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助理解現有程式碼的運作方式。

最後，自動化測試意味著減少手動<abbr title="Quality Assurance">QA</abbr>的時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但無需實際執行。

- **Linters** 分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記風格指南禁止的事項（例如使用製表符而非空格，或反之，取決於配置）。
- **類型檢查** 確保傳遞給函式的結構符合預期，防止將字串傳遞給預期數字的計數函式等情況。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於代碼檢查，[TypeScript](typescript) 用於類型檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造為例——在模型首次起飛展示所有複雜系統協同運作前，會單獨測試每個部件以確保安全性和功能性。例如：機翼需通過極端負載下的彎曲測試；引擎零件需通過耐久性測試；擋風玻璃需通過模擬鳥擊測試。

軟體開發亦同。與其將整個程式寫在一個龐大檔案中，不如拆分成多個小模組進行更徹底的測試。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼息息相關。

要提升應用程式的可測試性，首先應將視圖部分（React 元件）與業務邏輯和應用狀態分離（無論使用 Redux、MobX 或其他方案）。如此一來，業務邏輯測試（不應依賴 React 元件）就能獨立於主要負責渲染 UI 的元件！

理論上，甚至可以將所有邏輯和數據獲取移出元件，使元件專注於渲染。應用狀態完全獨立於元件，業務邏輯甚至能在沒有 React 元件的情況下運作！

> 建議透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

完成可測試程式碼後，接下來就是實際撰寫測試！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架，包含針對此環境量身打造的預設配置，讓你能立即投入測試而無需調整設定和模擬——稍後將詳細說明[模擬技巧](#mocking)。你可以使用 Jest 撰寫本指南提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔且理想情況下只驗證單一事項。以下是一個用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試描述透過傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述以明確說明測試內容，建議涵蓋以下要素：

1. **Given** - 前置條件
2. **When** - 被測函式執行的動作
3. **Then** - 預期結果

此模式亦稱為 AAA（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式協助組織測試。用 `describe` 將相同功能的測試分組，必要時可嵌套使用。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout) 可用於設置測試對象，詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟或驗證點過多，建議拆分成多個小測試。同時確保測試間完全獨立：每個測試都應能單獨執行，且測試套件中前序測試不影響後續測試結果。

最後要記得：對開發者而言，程式正常運作是樂見的，但測試情境恰好相反。將測試失敗視為「好事」！這代表發現了潛在問題，讓你能在影響用戶前及時修復。

## 單元測試

單元測試針對程式最小可測單元（如獨立函式或類別）進行驗證。

當被測對象存在依賴項時，通常需要進行模擬（mock），詳見下節說明。

單元測試的優勢在於快速撰寫與執行，能即時反饋程式碼狀態。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬測試

當被測程式存在外部依賴時，常需進行「模擬」（mocking），即用自訂實現替代真實依賴項。

> 原則上，測試中使用真實對象優於模擬對象，但某些情境無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你開發的天氣應用需呼叫外部服務獲取數據。測試時不應實際呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定
- 服務每次可能返回不同數據
- 第三方服務可能在測試時離線！

此時可提供服務的模擬實現，用幾行程式碼替代真實的聯網溫度計群！

> Jest 內建完整的[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級到模組級皆可模擬。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若被測試單元依賴其他組件，有時會需要透過模擬(mock)來替換這些依賴項。

整合測試則是將實際的獨立單元（如同在應用程式中）組合起來共同測試，以確保它們的協作符合預期。這並非意味著完全不需要模擬——例如仍需模擬與天氣服務的通信——但相較單元測試，所需模擬的場景會大幅減少。

> 請注意，關於整合測試的定義術語並不完全一致。單元測試與整合測試的界線有時也較為模糊。在本指南中，符合以下任一條件即歸類為「整合測試」：
>
> - 組合應用程式中多個模組進行測試
> - 使用外部系統
> - 進行網路呼叫（如呼叫天氣服務API）
> - 執行任何檔案或資料庫的<abbr title="輸入/輸出">I/O</abbr>操作

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件直接負責渲染應用界面並與用戶互動。即使業務邏輯已通過完整測試，若缺乏元件測試，仍可能交付有缺陷的UI。元件測試可歸類於單元測試或整合測試，但由於其在React Native中的核心地位，我們將獨立探討。

測試React元件時，主要關注兩個面向：

- 互動行為：確保元件對用戶操作（如點擊按鈕）的反應符合預期
- 渲染輸出：確保React使用的渲染結果正確（如按鈕外觀與UI位置）

例如，對於帶有`onPress`監聽器的按鈕，需同時驗證其視覺呈現正確性與點擊事件的處理邏輯。

以下工具庫可協助測試：

- React官方提供的[Test Renderer](https://reactjs.org/docs/test-renderer.html)，能在不依賴DOM或原生環境的情況下，將元件渲染為純JavaScript對象
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於React測試渲染器，擴充了後文將說明的`fireEvent`與`query`API

> 元件測試僅在Node.js環境中執行JavaScript測試，不涉及iOS/Android等平台的底層原生代碼。因此無法100%保證用戶端運作完全正常——若原生代碼存在缺陷，此類測試將無法發現。

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試用戶互動

除了渲染UI，元件還需處理如`TextInput`的`onChangeText`或`Button`的`onPress`等事件。測試時應從用戶視角出發：

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

驗證「頁面顯示內容」與「互動後的變化」，遵循以下原則：

建議優先基於用戶可感知的元素進行驗證：

- 使用渲染文字或[無障礙輔助屬性](https://reactnative.dev/docs/accessibility#accessibility-properties)建立斷言

反之應避免：

- 對元件props或state進行斷言
- 使用testID查詢元件

避免測試實作細節（如props/state），這類測試雖可運行，但與用戶實際互動方式脫節，且容易因重構（例如改用Hooks改寫class元件）而失效。

> React class components are especially prone to testing their implementation details such as internal state, props or event handlers. To avoid testing implementation details, prefer using function components with Hooks, which make relying on component internals _harder_.

Component testing libraries such as [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) facilitate writing user-centric tests by careful choice of provided APIs. The following example uses `fireEvent` methods `changeText` and `press` that simulate a user interacting with the component and a query function `getAllByText` that finds matching `Text` nodes in the rendered output.

```tsx
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

This example is not testing how some state changes when you call a function. It tests what happens when a user changes text in the `TextInput` and presses the `Button`!

### Testing Rendered Output

[Snapshot testing](https://jestjs.io/docs/en/snapshot-testing) is an advanced kind of testing enabled by Jest. It is a very powerful and low-level tool, so extra attention is advised when using it.

A "component snapshot" is a JSX-like string created by a custom React serializer built into Jest. This serializer lets Jest translate React component trees to string that's human-readable. Put another way: a component snapshot is a textual representation of your component’s render output _generated_ during a test run. It may look like this:

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

With snapshot testing, you typically first implement your component and then run the snapshot test. The snapshot test then creates a snapshot and saves it to a file in your repo as a reference snapshot. **The file is then committed and checked during code review**. Any future changes to the component render output will change its snapshot, which will cause the test to fail. You then need to update the stored reference snapshot for the test to pass. That change again needs to be committed and reviewed.

Snapshots have several weak points:

- For you as a developer or reviewer, it can be hard to tell whether a change in snapshot is intended or whether it's evidence of a bug. Especially large snapshots can quickly become hard to understand and their added value becomes low.
- When snapshot is created, at that point it is considered to be correct-even in the case when the rendered output is actually wrong.
- When a snapshot fails, it's tempting to update it using the `--updateSnapshot` jest option without taking proper care to investigate whether the change is expected. Certain developer discipline is thus needed.

Snapshots themselves do not ensure that your component render logic is correct, they are merely good at guarding against unexpected changes and for checking that the components in the React tree under test receive the expected props (styles and etc.).

We recommend that you only use small snapshots (see [`no-large-snapshots` rule](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)). If you want to test a _change_ between two React component states, use [`snapshot-diff`](https://github.com/jest-community/snapshot-diff). When in doubt, prefer explicit expectations as described in the previous paragraph.

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## End-to-End Tests

In end-to-end (E2E) tests, you verify your app is working as expected on a device (or a simulator / emulator) from the user perspective.

This is done by building your app in the release configuration and running the tests against it. In E2E tests, you no longer think about React components, React Native APIs, Redux stores or any business logic. That is not the purpose of E2E tests and those are not even accessible to you during E2E testing.

Instead, E2E testing libraries allow you to find and control elements in the screen of your app: for example, you can _actually_ tap buttons or insert text into `TextInputs` the same way a real user would. Then you can make assertions about whether or not a certain element exists in the app’s screen, whether or not it’s visible, what text it contains, and so on.

E2E tests give you the highest possible confidence that part of your app is working. The tradeoffs include:

- writing them is more time consuming compared to the other types of tests
- they are slower to run
- they are more prone to flakiness (a "flaky" test is a test which randomly passes and fails without any change to code)

Try to cover the vital parts of your app with E2E tests: authentication flow, core functionalities, payments, etc. Use faster JS tests for the non-vital parts of your app. The more tests you add, the higher your confidence, but also, the more time you'll spend maintaining and running them. Consider the tradeoffs and decide what's best for you.

There are several E2E testing tools available: in the React Native community, [Detox](https://github.com/wix/detox/) is a popular framework because it’s tailored for React Native apps. Another popular library in the space of iOS and Android apps is [Appium](http://appium.io/).

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## Summary

We hope you enjoyed reading and learned something from this guide. There are many ways you can test your apps. It may be hard to decide what to use at first. However, we believe it all will make sense once you start adding tests to your awesome React Native app. So what are you waiting for? Get your coverage up!

### Links

- [React testing overview](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest docs](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](http://appium.io/)

---

_This guide originally authored and contributed in full by [Vojtech Novak](https://twitter.com/vonovak)._