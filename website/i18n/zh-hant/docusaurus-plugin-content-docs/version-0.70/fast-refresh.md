---
id: fast-refresh
title: Fast Refresh
---

Fast Refresh 是 React Native 的一項功能，讓您能近乎即時地獲得對 React 元件修改的反饋。Fast Refresh 預設為啟用狀態，您可以在 [React Native 開發者選單](/docs/debugging#accessing-the-in-app-developer-menu) 中切換「啟用 Fast Refresh」。啟用後，大多數編輯應能在一兩秒內看到變化。

## 運作原理

- 若您編輯的模組**僅導出 React 元件**，Fast Refresh 僅會更新該模組的程式碼，並重新渲染您的元件。您可以修改該檔案中的任何內容，包括樣式、渲染邏輯、事件處理器或副作用。
- 若您編輯的模組導出內容包含_非_ React 元件，Fast Refresh 會重新執行該模組及其導入該模組的其他模組。例如，若 `Button.js` 和 `Modal.js` 都導入 `Theme.js`，編輯 `Theme.js` 將同時更新這兩個元件。
- 最後，若您編輯的檔案被**React 樹之外的模組導入**，Fast Refresh **將退回執行完整重新載入**。您可能有一個檔案既渲染 React 元件，又導出被**非 React 元件**導入的值。例如，您的元件可能同時導出一個常數，而一個非 React 工具模組導入該常數。此時，建議將該常數移至獨立檔案並讓兩個檔案分別導入，這樣可重新啟用 Fast Refresh。其他情況通常也能以類似方式解決。

## 錯誤恢復能力

若您在 Fast Refresh 過程中出現**語法錯誤**，修正後再次儲存檔案即可。錯誤提示框會消失。含有語法錯誤的模組會被阻止執行，因此您無需重新載入應用程式。

若您在**模組初始化時發生執行階段錯誤**（例如輸入 `Style.create` 而非 `StyleSheet.create`），修正錯誤後 Fast Refresh 會繼續執行。錯誤提示框會消失，模組也會更新。

若您的錯誤導致**元件內部發生執行階段錯誤**，修正後 Fast Refresh 仍會繼續執行。此時，React 會使用更新後的程式碼重新掛載您的應用程式。

若您的應用中設有[錯誤邊界](https://reactjs.org/docs/error-boundaries.html)（這在生產環境中能優雅處理錯誤，是個好做法），它們會在下次編輯時嘗試重新渲染。因此，錯誤邊界能避免您總是被踢回應用程式的根畫面。但請注意，錯誤邊界不應_過於_細粒度。它們在生產環境中由 React 使用，應始終有意識地設計。

## 限制

Fast Refresh 會嘗試保留您正在編輯的元件中的本地 React 狀態，但僅在安全的情況下。以下幾種情況可能會導致每次編輯檔案時本地狀態被重置：

- 類別元件的本地狀態不會保留（僅函式元件和 Hooks 能保留狀態）。
- 您編輯的模組可能_除了_ React 元件外還導出其他內容。
- 有時，模組可能導出高階元件的調用結果，如 `createNavigationContainer(MyScreen)`。若返回的元件是類別，狀態將被重置。

長期來看，隨著更多程式碼轉為函式元件和 Hooks，預期狀態保留的情況會增加。

## 提示

- Fast Refresh 預設會保留函式元件（及 Hooks）中的 React 本地狀態。
- 有時您可能想_強制_重置狀態並重新掛載元件。例如，這在調整僅在掛載時執行的動畫時很有用。為此，您可以在編輯的檔案中任意位置加入 `// @refresh reset`。此指令僅作用於該檔案，會指示 Fast Refresh 在每次編輯時重新掛載該檔案中定義的元件。

## Fast Refresh 與 Hooks

在可能的情況下，Fast Refresh 會嘗試保留元件在編輯之間的狀態。特別是 `useState` 和 `useRef`，只要你不改變它們的參數或 Hook 調用的順序，它們就會保留之前的值。

具有依賴項的 Hook——例如 `useEffect`、`useMemo` 和 `useCallback`——在 Fast Refresh 期間總是會更新。在 Fast Refresh 進行時，它們的依賴項列表會被忽略。

例如，當你將 `useMemo(() => x * 2, [x])` 編輯為 `useMemo(() => x * 10, [x])` 時，即使 `x`（依賴項）沒有改變，它也會重新運行。如果 React 不這樣做，你的編輯就不會反映在螢幕上！

有時，這可能會導致意想不到的結果。例如，即使是一個具有空依賴項數組的 `useEffect` 在 Fast Refresh 期間仍然會重新運行一次。然而，即使沒有 Fast Refresh，編寫能夠偶爾重新運行 `useEffect` 的彈性代碼也是一個好習慣。這使得你以後更容易為它引入新的依賴項。