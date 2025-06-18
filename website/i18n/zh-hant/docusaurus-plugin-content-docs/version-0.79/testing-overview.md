---
id: testing-overview
title: Testing
author: Vojtech Novak
authorURL: 'https://twitter.com/vonovak'
description: This guide introduces React Native developers to the key concepts behind testing, how to write good tests, and what kinds of tests you can incorporate into your workflow.
---

隨著程式碼庫的擴展，未預期的小錯誤和邊緣案例可能引發更大的故障。錯誤會導致糟糕的使用者體驗，最終造成業務損失。預防脆弱程式設計的方法之一，是在將程式碼發布前進行測試。

本指南將涵蓋多種自動化方法，從靜態分析到端到端測試，確保您的應用程式如預期運作。

<img src="/docs/assets/diagram_testing.svg" alt="Testing is a cycle of fixing, testing, and either passing to release or failing back into testing." />

## 為什麼要測試

我們是人類，而人類會犯錯。測試之所以重要，是因為它能幫助您發現這些錯誤並驗證程式碼是否正常運作。或許更重要的是，測試能確保在您新增功能、重構現有程式碼或升級專案主要依賴項時，程式碼仍能持續運作。

測試的價值可能超出您的認知。修正程式錯誤的最佳方法之一，就是撰寫一個能暴露該錯誤的失敗測試。當您修正錯誤並重新執行測試時，若測試通過，表示錯誤已修復且不會再次引入程式碼庫。

測試也能作為新團隊成員的技術文件。對於初次接觸程式碼庫的人來說，閱讀測試能幫助他們理解現有程式碼的運作方式。

最後但同樣重要的是，更多的自動化測試意味著更少的手動<abbr title="Quality Assurance">QA</abbr>時間，釋放寶貴資源。

## 靜態分析

提升程式碼品質的第一步是使用靜態分析工具。靜態分析會在您撰寫程式碼時檢查錯誤，但無需實際執行該程式碼。

- **Linters** 會分析程式碼以捕捉常見錯誤（如未使用的程式碼），幫助避免陷阱，並標記違反風格指南的行為（例如使用製表符而非空格，或反之，取決於您的配置）。
- **型別檢查** 確保傳遞給函式的結構符合函式設計預期，例如防止將字串傳遞給預期接收數字的計數函式。

React Native 預設配置了兩種工具：[ESLint](https://eslint.org/) 用於程式碼檢查，[TypeScript](typescript) 用於型別檢查。

## 撰寫可測試的程式碼

要開始測試，首先需要撰寫可測試的程式碼。以飛機製造流程為例——在模型首次起飛展示所有複雜系統協同運作前，會先測試個別零件以確保其安全性和功能。例如，機翼會在極端負載下進行彎曲測試；引擎零件會測試耐用性；擋風玻璃會模擬鳥擊測試。

軟體開發也是如此。與其將整個程式寫在一個龐大檔案中，不如將程式碼拆分為多個小型模組，這樣能比測試完整組裝的程式進行更徹底的測試。因此，撰寫可測試的程式碼與撰寫乾淨、模組化的程式碼息息相關。

要讓應用程式更具可測試性，首先應將應用的視圖部分（React 元件）與業務邏輯和應用狀態分離（無論您使用 Redux、MobX 或其他解決方案）。這樣一來，您可以將業務邏輯測試（不應依賴 React 元件）與元件本身的測試分開，後者的主要職責是渲染應用程式的 UI！

理論上，您可以將所有邏輯和資料獲取完全移出元件。這樣一來，元件將僅專注於渲染。您的狀態將完全獨立於元件。應用的邏輯甚至可以在沒有任何 React 元件的情況下運作！

> 我們鼓勵您透過其他學習資源進一步探索可測試程式碼的主題。

## 撰寫測試

撰寫完可測試的程式碼後，是時候撰寫實際測試了！React Native 的預設模板附帶 [Jest](https://jestjs.io) 測試框架。它包含一個針對此環境量身打造的預設配置，讓您無需立即調整配置和模擬（mock）就能開始工作——稍後將詳細介紹[模擬](#mocking)。您可以使用 Jest 撰寫本指南中提到的所有測試類型。

> 如果你採用測試驅動開發（TDD），實際上你會先寫測試！這樣一來，程式碼的可測試性自然就具備了。

### 測試結構設計

你的測試應該簡潔，且理想情況下只測試單一功能。讓我們從一個用 Jest 編寫的單元測試範例開始：

```js
it('given a date in the past, colorForDueDate() returns red', () => {
  expect(colorForDueDate('2000-10-20')).toBe('red');
});
```

測試的描述由傳遞給 [`it`](https://jestjs.io/docs/en/api#testname-fn-timeout) 函式的字串定義。請仔細撰寫描述，確保清楚說明測試內容。盡量涵蓋以下要素：

1. **Given** - 某些前置條件  
2. **When** - 被測函式執行的某個動作  
3. **Then** - 預期的結果

這也被稱為 AAA 模式（Arrange, Act, Assert）。

Jest 提供 [`describe`](https://jestjs.io/docs/en/api#describename-fn) 函式來組織測試。用 `describe` 將屬於同一功能的所有測試分組。如果需要，`describe` 還可以嵌套使用。其他常用函式包括 [`beforeEach`](https://jestjs.io/docs/en/api#beforeeachfn-timeout) 或 [`beforeAll`](https://jestjs.io/docs/en/api#beforeallfn-timeout)，可用於設置測試對象。詳見 [Jest API 文檔](https://jestjs.io/docs/en/api)。

如果測試步驟繁多或有多個斷言，建議拆分成多個小測試。同時確保測試之間完全獨立——每個測試都必須能單獨執行，且不受其他測試影響。換言之，當你執行全部測試時，第一個測試的結果不應影響第二個測試的輸出。

最後要記住：開發者通常希望程式碼完美運行不出錯，但測試恰恰相反。請將測試失敗視為「好事」！當測試失敗時，通常意味著某些地方需要修正，這讓你有機會在問題影響用戶前解決它。

## 單元測試

單元測試涵蓋程式碼的最小單位，例如獨立函式或類別。

當被測對象有依賴項時，通常需要將其模擬（mock）掉，如下一節所述。

單元測試的優勢在於編寫和執行速度快，能即時反饋測試結果。Jest 還提供 [Watch 模式](https://jestjs.io/docs/en/cli#watch)，可持續運行與你編輯程式碼相關的測試。

<img src="/docs/assets/p_tests-unit.svg" alt=" " />

### 模擬（Mocking）

當被測對象有外部依賴時，你可能需要「模擬」這些依賴。「模擬」是指用自訂實現替換程式碼的某些依賴項。

> 通常，在測試中使用真實對象比模擬更好，但在某些情況下無法實現。例如：當你的 JS 單元測試依賴於用 Java 或 Objective-C 編寫的原生模組時。

假設你正在開發一個顯示當前城市天氣的應用，並使用某個提供天氣資訊的外部服務。如果服務回報正在下雨，你需要顯示雨天雲圖。但你不希望在測試中實際調用該服務，因為：

- 這會使測試變慢且不穩定（涉及網路請求）  
- 服務每次可能返回不同的數據  
- 第三方服務可能在測試時剛好離線！

因此，你可以提供該服務的模擬實現，從而替換成千上萬行程式碼和聯網溫度計！

> Jest 內建[模擬支援](https://jestjs.io/docs/en/mock-functions#mocking-modules)，從函式級到模組級模擬都能實現。

## 整合測試

在開發大型軟體系統時，各個組件需要相互協作。在單元測試中，若某個單元依賴於另一個單元，有時會需要透過模擬(mock)來替換該依賴項，使用虛擬版本進行測試。

整合測試則是將實際的獨立單元（如同在應用程式中）組合起來共同測試，以確保它們的協作符合預期。這並非意味著完全不需要模擬——例如仍需模擬與天氣服務的通信——但使用頻率會遠低於單元測試。

> 請注意，關於「整合測試」的術語定義並不完全一致。單元測試與整合測試的界線有時也較模糊。本指南中，符合以下條件即歸類為整合測試：
>
> - 組合應用程式中多個模組進行測試
> - 使用外部系統
> - 對其他應用程式發起網路請求（如天氣服務API）
> - 涉及任何檔案或資料庫的<abbr title="輸入/輸出">I/O</abbr>操作

<img src="/docs/assets/p_tests-integration.svg" alt=" " />

## 元件測試

React元件負責渲染應用程式界面，使用者將直接與其輸出互動。即使業務邏輯測試覆蓋率高且正確，若缺少元件測試，仍可能向使用者交付有缺陷的UI。元件測試可歸類為單元測試或整合測試，但由於其是React Native的核心部分，我們將單獨討論。

測試React元件時，主要有兩個面向需要驗證：

- 互動行為：確保元件在使用者操作時反應正確（例如點擊按鈕）
- 渲染輸出：確保React使用的元件渲染結果正確（例如按鈕外觀與UI位置）

舉例來說，若某按鈕設有`onPress`監聽器，需同時測試其外觀顯示是否正確，以及點擊事件是否被元件正確處理。

以下工具庫可協助測試：

- React官方提供的[Test Renderer](https://reactjs.org/docs/test-renderer.html)，能在不依賴DOM或原生環境的情況下，將元件渲染為純JavaScript物件
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)基於React測試渲染器，擴增了後文將說明的`fireEvent`與`query`API

> 元件測試僅是在Node.js環境中運行的JavaScript測試，不涉及iOS/Android等支撐React Native元件的平台程式碼。因此無法100%保證使用者體驗無誤——若底層原生代碼存在缺陷，此類測試無法發現。

<img src="/docs/assets/p_tests-component.svg" alt=" " />

### 測試使用者互動

除了渲染UI，元件還需處理如`TextInput`的`onChangeText`或`Button`的`onPress`等事件，可能包含其他函數與回調。參考以下範例：

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

測試使用者互動時，應從使用者角度出發——頁面顯示什麼內容？互動後會產生什麼變化？

基本原則建議優先測試使用者可感知的元素：

- 透過渲染文字或[無障礙輔助功能](https://reactnative.dev/docs/accessibility#accessibility-properties)進行驗證

相對地，應避免：

- 對元件props或state進行斷言
- 使用testID查詢元件

避免測試實作細節（如props或state），這類測試雖可行，但未聚焦使用者互動模式，且容易因重構（例如重命名或改用Hooks改寫class元件）而失效。

> React 類別元件特別容易測試其實作細節，例如內部狀態、props 或事件處理器。為避免測試實作細節，建議優先使用帶有 Hooks 的函式元件，這會讓依賴元件內部邏輯變得更加困難。

像 [React Native Testing Library](https://callstack.github.io/react-native-testing-library/) 這類元件測試函式庫，透過精心設計的 API 來協助編寫以使用者為中心的測試。以下範例使用 `fireEvent` 方法中的 `changeText` 和 `press` 來模擬使用者與元件的互動，並使用查詢函式 `getAllByText` 在渲染輸出中尋找匹配的 `Text` 節點。

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

這個範例並非測試呼叫函式時狀態如何變化，而是測試當使用者在 `TextInput` 中變更文字並按下 `Button` 時會發生什麼！

### 測試渲染輸出

[快照測試](https://jestjs.io/docs/en/snapshot-testing) 是 Jest 提供的一種進階測試方法。它是一個非常強大且底層的工具，因此使用時需格外謹慎。

「元件快照」是由 Jest 內建的自訂 React 序列化器產生的類 JSX 字串。此序列化器讓 Jest 能將 React 元件樹轉換為人類可讀的字串。換句話說：元件快照是測試運行期間生成的元件渲染輸出的文字表徵。它可能看起來像這樣：

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

進行快照測試時，通常會先實作元件，然後執行快照測試。測試會建立快照並將其作為參考快照儲存到程式碼庫的檔案中。**該檔案會被提交並在程式碼審查時檢查**。未來任何對元件渲染輸出的變更都會導致快照變化，從而使測試失敗。此時需更新儲存的參考快照才能使測試通過，而此變更同樣需要提交並審查。

快照測試有幾個弱點：

- 對開發者或審查者而言，難以判斷快照變更是否為預期行為或代表錯誤。特別是大型快照可能迅速變得難以理解，其附加價值隨之降低。
- 快照建立時即被視為正確——即使實際渲染輸出存在錯誤。
- 當快照失敗時，開發者容易直接使用 Jest 的 `--updateSnapshot` 選項更新快照，而未能仔細檢查變更是否合理。因此需要一定的開發紀律。

快照本身無法確保元件渲染邏輯正確，其主要作用是防範意外變更，並檢查測試中 React 樹下的元件是否收到預期的 props（樣式等）。

建議僅使用小型快照（參見 [`no-large-snapshots` 規則](https://github.com/jest-community/eslint-plugin-jest/blob/master/docs/rules/no-large-snapshots.md)）。若需測試兩個 React 元件狀態間的差異，可使用 [`snapshot-diff`](https://github.com/jest-community/snapshot-diff)。若有疑慮，建議優先採用前文所述的明確斷言方法。

<img src="/docs/assets/p_tests-snapshot.svg" alt=" " />

## 端對端測試

在端對端（E2E）測試中，您會從使用者角度驗證應用程式在真實裝置（或模擬器）上的運作是否符合預期。

這需要以發布配置建置應用程式，並對其執行測試。E2E 測試不再考慮 React 元件、React Native API、Redux store 或任何業務邏輯——這些並非 E2E 測試的目的，且在測試期間也無法存取。

相反地，E2E 測試函式庫讓您能尋找並控制應用畫面中的元素：例如，您可以像真實使用者一樣實際點擊按鈕或在 `TextInput` 中輸入文字。接著可對畫面元素進行斷言，例如特定元素是否存在、是否可見、包含什麼文字等。

E2E 測試能提供最高程度的信心，確保應用程式的某部分正常運作。其代價包括：

- 撰寫這類測試比其他類型的測試更耗時
- 執行速度較慢
- 更容易出現不穩定情況（「不穩定測試」是指隨機通過或失敗，而程式碼並未變更的測試）

請盡量用端對端測試覆蓋應用的關鍵部分：身份驗證流程、核心功能、支付等。對於非關鍵部分，可使用更快速的 JavaScript 測試。測試越多，信心越高，但維護和執行測試的時間也會增加。請權衡利弊，選擇最適合的方案。

現有多種端對端測試工具可供選擇：在 React Native 社群中，[Detox](https://github.com/wix/detox/) 是熱門框架，因其專為 React Native 應用量身打造。在 iOS 和 Android 應用領域，另一個常用庫是 [Appium](https://appium.io/) 或 [Maestro](https://maestro.mobile.dev/)。

<img src="/docs/assets/p_tests-e2e.svg" alt=" " />

## 總結

希望您喜歡閱讀本指南並有所收穫。測試應用的方法多種多樣，初學者可能難以抉擇。但我們相信，一旦開始為您出色的 React Native 應用添加測試，一切自會豁然開朗。還在等什麼？快來提高測試覆蓋率吧！

### 相關連結

- [React 測試概述](https://reactjs.org/docs/testing.html)
- [React Native Testing Library](https://callstack.github.io/react-native-testing-library/)
- [Jest 文檔](https://jestjs.io/docs/en/tutorial-react-native)
- [Detox](https://github.com/wix/detox/)
- [Appium](https://appium.io/)
- [Maestro](https://maestro.mobile.dev/)

---

_本指南由 [Vojtech Novak](https://twitter.com/vonovak) 原創撰寫並貢獻全部內容。_