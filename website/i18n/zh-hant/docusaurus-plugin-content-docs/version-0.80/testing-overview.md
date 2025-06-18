---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。錯誤會導致糟糕的使用者體驗，最終造成業務損失。防止脆弱程式設計的方法之一，是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化方法，從靜態分析到端到端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為何需要測試

人類會犯錯，測試的重要性在於幫助發現這些錯誤並驗證程式碼是否正常運作。更重要的是，測試能確保未來在新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值超乎想像。修復程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修復錯誤後重新執行測試，若通過則表示錯誤已修復且不會再被引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後，更多的自動化測試意味著更少的手動<abbr title="Quality Assurance">QA</abbr>時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在撰寫程式碼時檢查錯誤，但無需實際執行該程式碼。

- **Linters** 分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的情況（例如使用製表符而非空格，或反之，取決於您的配置）。
- **類型檢查** 確保傳遞給函式的結構符合其設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於類型檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造過程為例——在模型首次起飛展示所有複雜系統協同運作前，會單獨測試零件以確保其安全性和功能性。例如：機翼需承受極端負載彎曲測試、引擎零件需通過耐久性測試、擋風玻璃需模擬鳥擊測試。

軟體開發同理。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試完整組裝的系統更徹底。因此，撰寫可測試程式碼與撰寫乾淨、模組化的程式碼息息相關。

要提升應用程式的可測試性，首先將 React 元件（視圖部分）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。如此一來，業務邏輯測試（不應依賴 React 元件）可獨立於主要負責渲染 UI 的元件！

理論上，您甚至可以將所有邏輯和資料獲取移出元件，使元件專注於渲染。應用狀態將完全獨立於元件，業務邏輯可在完全不使用 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

完成可測試程式碼後，接下來就是撰寫實際測試！React Native 的預設模板內建 [Jest](https://jestjs.io) 測試框架，包含針對此環境量身打造的預設配置，讓您無需立即調整設定和模擬（mock）即可開始工作——稍後將詳細說明[模擬技巧](#mocking)。您可以使用 Jest 撰寫本指南提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

測試應該簡潔，且理想情況下只測試單一功能。以下是一個用 Jest 撰寫的單元測試範例：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試的描述文字會傳入 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式。請仔細撰寫描述，確保測試目的明確。盡量涵蓋以下要素：

1. **Given** - 前置條件
2. **When** - 被測試函式執行的動作
3. **Then** - 預期結果

這也被稱為 AAA 模式（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式來組織測試。用 `describe` 將同一功能的測試分組，必要時可嵌套使用。其他常用函式如 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 和 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文件](https://jestjs.io/docs/en/api)。

若測試步驟繁多或有多個斷言，建議拆分成多個小測試。同時確保測試之間完全獨立——每個測試都應能單獨執行，且測試順序不影響結果。

最後要記得：對開發者而言，程式正常運作是好事，但測試失敗反而是_有益的_！測試失敗通常意味著潛在問題，讓你能在影響用戶前及時修復。

## 單元測試

單元測試針對程式的最小單位，例如獨立函式或類別。

當測試對象有依賴項時，常需使用模擬（mock）取代，詳見下節說明。

單元測試的優勢在於快速撰寫和執行，能即時反饋程式碼狀態。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當測試對象有外部依賴時，你可能需要「模擬」這些依賴。「模擬」是指用自訂實作取代程式碼的某些依賴項。

> 原則上，測試中使用真實對象優於模擬，但某些情況無法避免。例如：當 JS 單元測試依賴 Java 或 Objective-C 寫的原生模組時。

假設你正在開發顯示當地天氣的應用程式，並使用某個提供天氣資訊的外部服務。若服務回報正在下雨，你需要顯示雨天雲圖標。但測試中不應直接呼叫該服務，因為：

- 網路請求會導致測試變慢且不穩定
- 服務每次可能返回不同數據
- 第三方服務可能在需要測試時離線！

此時可提供該服務的模擬實作，用幾行程式碼取代數千行網路連線的溫度計程式！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級到模組級皆可模擬。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若測試單元依賴其他組件，有時會需要透過模擬(mock)來替換這些依賴項。

整合測試會將實際的獨立單元組合（如同在應用程式中一樣）並共同測試，以確保它們的協作符合預期。這並非表示不需要模擬：您仍可能需要模擬（例如模擬與天氣服務的通訊），但使用頻率會遠低於單元測試。

> 請注意，關於整合測試的術語定義並非總是統一。此外，單元測試與整合測試的界線有時也較模糊。在本指南中，若您的測試符合以下條件即歸類為「整合測試」：
>
> - 如前述組合多個應用模組
> - 使用外部系統
> - 對其他應用程式發送網路請求（如天氣服務API）
> - 進行任何形式的檔案或資料庫<abbr title="輸入/輸出">I/O</abbr>操作

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用界面，使用者將直接與其輸出互動。即使應用的業務邏輯測試覆蓋率高且正確，若缺少元件測試，仍可能向用戶交付有缺陷的UI。元件測試可歸類為單元或整合測試，但由於其是React Native的核心部分，我們將單獨說明。

測試React元件時，您可能需要驗證兩個面向：

- 互動性：確保元件在使用者操作時行為正確（例如點擊按鈕）
- 渲染結果：確保React使用的元件渲染輸出正確（例如按鈕外觀與UI位置）

舉例來說，若按鈕設有`onPress`監聽器，您需同時驗證按鈕顯示正確性與點擊事件的處理邏輯。

以下套件可協助進行這類測試：

- React官方推出的[Test Renderer](https://reactjs.org/docs/test-renderer.html)，能在不依賴DOM或原生環境的情況下，將元件渲染為純JavaScript物件
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於React測試渲染器，擴增了後文將說明的`fireEvent`與`query`API

> 元件測試僅是在Node.js環境執行的JavaScript測試，不涉及React Native元件底層的iOS、Android或其他平台程式碼。因此無法100%保證使用者端的完整運作，若原生代碼存在錯誤，這類測試將無法偵測。

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試使用者互動

除了渲染UI外，您的元件還需處理如`TextInput`的`onChangeText`或`Button`的`onPress`等事件，可能包含其他函數與事件回調。參考以下範例：

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

測試使用者互動時，應從使用者角度出發——頁面顯示什麼？互動後產生哪些變化？

基本原則建議優先採用使用者可感知的元素：

- 透過渲染文字或[無障礙輔助屬性](https://reactnative.dev/docs/accessibility#accessibility-properties)進行斷言

相對地，應避免：

- 對元件props或state進行斷言
- 使用testID查詢

避免測試實作細節如props或state——這類測試雖可運作，但未聚焦使用者互動模式，且容易因重構（例如重命名或改用Hook改寫class元件）而失效。

> React 類別元件尤其容易測試其實作細節，例如內部狀態、props 或事件處理器。為避免測試實作細節，建議優先使用搭配 Hooks 的函式元件，這會讓依賴元件內部邏輯變得更加困難。

諸如 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 來協助撰寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法 `changeText` 和 `press` 來模擬使用者與元件的互動，並透過查詢函式 `getAllByText` 在渲染輸出中尋找匹配的 `Text` 節點。

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

[快照測試](https://jestjs.io/docs/en/snapshot-testing)是 Jest 提供的一種進階測試方式。由於這是極強大且底層的工具，使用時需格外謹慎。

「元件快照」是由 Jest 內建的自訂 React 序列化器產生的類 JSX 字串。此序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換言之：元件快照是測試執行期間_生成_的元件渲染輸出的文字表徵，其內容可能如下：

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

進行快照測試時，通常會先實作元件再執行快照測試。測試會建立快照並將其作為參考快照存入程式庫中的檔案。**該檔案需提交並在程式碼審查時檢查**。後續任何元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而此變更同樣需提交並審查。

快照測試存在若干弱點：

- 對開發者或審查者而言，難以判斷快照變更屬預期行為或錯誤證據。龐大的快照尤其容易變得難以理解，其附加價值隨之降低。
- 快照建立當下即被視為正確——即便實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易直接使用 `--updateSnapshot` 選項更新快照，而未仔細檢查變更是否合理。因此需要相當的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要價值在於防範意外變更，以及驗證測試中 React 樹下的元件是否接收預期的 props（樣式等）。

建議僅使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。若需測試兩個 React 元件狀態間的_差異_，可使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。若有疑慮，建議優先採用前文所述的明確斷言方式。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您需驗證應用程式在裝置（或模擬器）上是否如預期般運作，測試角度完全模擬使用者行為。

此測試需以發布配置建置應用程式，並對其執行測試。E2E 測試不再考量 React 元件、React Native API、Redux 儲存或任何業務邏輯——這些並非 E2E 測試目的，且在測試過程中亦無法存取。

相反地，E2E 測試函式庫讓您能尋找並控制應用畫面中的元素：例如，您可以_實際_點擊按鈕或在 `TextInput` 中輸入文字，完全模擬真實使用者操作。接著可對畫面元素進行斷言，例如特定元素是否存在、是否可見、包含何種文字等。

E2E 測試能為應用程式功能提供最高程度的信心保證，但其代價包括：

- 與其他類型的測試相比，撰寫這些測試更耗時
- 執行速度較慢
- 更容易出現不穩定性（「不穩定」測試是指隨機通過和失敗，而程式碼沒有任何變更的測試）

嘗試用端到端測試涵蓋應用程式的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，使用更快的 JavaScript 測試。添加的測試越多，信心就越高，但維護和執行它們的時間也會越多。請權衡利弊，決定什麼最適合您。

有幾種可用的端到端測試工具：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是一個流行的框架，因為它是專為 React Native 應用程式設計的。在 iOS 和 Android 應用程式領域，另一個流行的庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀並從本指南中學到一些東西。測試應用程式的方法有很多種。一開始可能很難決定使用什麼。然而，我們相信，一旦您開始為您出色的 React Native 應用程式添加測試，一切都會變得有意義。那麼還在等什麼呢？提高您的測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文檔](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南最初由 [Vojtech Novak](https://twitter.com/vonovak) 撰寫並貢獻完整內容。_