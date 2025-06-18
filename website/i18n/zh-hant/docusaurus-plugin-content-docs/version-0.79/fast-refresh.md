---
id: fast-refresh
title: Fast Refresh
---

Fast Refresh 是 React Native 的一項功能，可讓您即時獲得 React 元件變更的反饋。Fast Refresh 預設啟用，您可以在 [React Native 開發者選單](/docs/debugging#accessing-the-in-app-developer-menu) 中切換「啟用 Fast Refresh」。啟用後，大多數編輯內容應在一兩秒內可見。

## 運作原理

- 若編輯的模組**僅導出 React 元件**，Fast Refresh 僅更新該模組的程式碼並重新渲染元件。您可以修改該檔案中的任何內容，包括樣式、渲染邏輯、事件處理程序或副作用。
- 若編輯的模組導出內容包含_非_ React 元件，Fast Refresh 會重新執行該模組及其導入模組。例如 `Button.js` 和 `Modal.js` 都導入 `Theme.js`，編輯 `Theme.js` 將同時更新兩個元件。
- 最後，若編輯的檔案被**React 樹之外的模組導入**，Fast Refresh **將退回完整重新載入**。您可能有個檔案既渲染 React 元件又導出某個值，而該值被**非 React 元件**導入。例如元件導出常數，且非 React 工具模組導入該常數。此時可考慮將常數移至獨立檔案並讓雙方導入，這樣 Fast Refresh 就能重新運作。其他情況通常也能用類似方式解決。

## 錯誤恢復能力

若在 Fast Refresh 階段發生**語法錯誤**，修正後再次儲存檔案即可。紅色錯誤框會消失。系統會阻止執行含語法錯誤的模組，因此無需重新載入應用程式。

若在**模組初始化時發生執行階段錯誤**（例如輸入 `Style.create` 而非 `StyleSheet.create`），修正錯誤後 Fast Refresh 會繼續運作。紅色錯誤框消失，模組會更新。

若錯誤導致**元件內部執行階段錯誤**，修正後 Fast Refresh 階段_同樣_會繼續。此時 React 會使用更新後的程式碼重新掛載應用程式。

若應用程式中設有[錯誤邊界](https://reactjs.org/docs/error-boundaries.html)（對於生產環境的優雅降級是個好做法），下次編輯時會重試渲染。因此錯誤邊界能避免您總是被踢回根應用畫面。但請注意錯誤邊界不應_過於_細碎。它們在生產環境由 React 使用，應始終謹慎設計。

## 限制

Fast Refresh 會嘗試保留編輯中元件的本地 React 狀態，但僅在安全時執行。以下是可能導致每次編輯檔案時本地狀態被重置的原因：

- 類別元件的本地狀態不會保留（僅函式元件和 Hooks 能保留狀態）。
- 編輯的模組可能_除了_ React 元件外還導出其他內容。
- 有時模組會導出高階元件調用結果，如 `createNavigationContainer(MyScreen)`。若返回元件是類別，狀態將被重置。

長期來看，隨著程式碼庫逐漸改用函式元件和 Hooks，預期更多情況下狀態能獲得保留。

## 技巧

- Fast Refresh 預設保留函式元件（及 Hooks）的 React 本地狀態。
- 有時您可能需要_強制_重置狀態並重新掛載元件。例如調整僅在掛載時執行的動畫時就很實用。為此可在編輯檔案任意位置添加 `// @refresh reset`。此指令僅作用於當前檔案，會指示 Fast Refresh 在每次編輯時重新掛載該檔案定義的元件。

## Fast Refresh 與 Hooks

在可能的情況下，Fast Refresh 會嘗試保留元件在編輯之間的狀態。特別是 `useState` 和 `useRef`，只要你不改變它們的參數或 Hook 呼叫的順序，它們就會保留先前的值。

具有依賴項的 Hook（例如 `useEffect`、`useMemo` 和 `useCallback`）在 Fast Refresh 期間_總是_會更新。在 Fast Refresh 進行時，它們的依賴項列表會被忽略。

例如，當你將 `useMemo(() => x * 2, [x])` 編輯為 `useMemo(() => x * 10, [x])` 時，即使 `x`（依賴項）沒有改變，它也會重新執行。如果 React 不這樣做，你的編輯就不會反映在畫面上！

有時，這可能會導致意想不到的結果。例如，即使是一個具有空依賴項陣列的 `useEffect` 也會在 Fast Refresh 期間重新執行一次。然而，即使沒有 Fast Refresh，撰寫能夠應對 `useEffect` 偶爾重新執行的程式碼也是一個良好的實踐。這使得你之後更容易為它引入新的依賴項。