---
id: fast-refresh
title: Fast Refresh
---

Fast Refresh 是 React Native 的一項功能，可讓您在修改 React 元件時獲得近乎即時的反饋。Fast Refresh 預設為啟用狀態，您可以在 [React Native 開發者選單](/docs/debugging#accessing-the-in-app-developer-menu) 中切換「啟用 Fast Refresh」。啟用後，大多數編輯內容應能在一兩秒內顯示變更。

## 運作原理

- 若您編輯的模組**僅導出 React 元件**，Fast Refresh 僅會更新該模組的程式碼並重新渲染元件。您可以修改該檔案中的任何內容，包括樣式、渲染邏輯、事件處理器或副作用。
- 若編輯的模組包含**非 React 元件**的導出，Fast Refresh 會重新執行該模組及其導入模組。例如 `Button.js` 和 `Modal.js` 都導入 `Theme.js`，則編輯 `Theme.js` 會同時更新兩個元件。
- 最後，若您編輯的檔案被**React 樹之外的模組導入**，Fast Refresh **會退回完整重新載入**。例如某個檔案同時導出 React 元件和常數值，而該常數被非 React 工具模組導入。此時可考慮將常數移至獨立檔案，再分別導入以恢復 Fast Refresh 功能。其他情況通常也能用類似方式解決。

## 錯誤恢復能力

在 Fast Refresh 過程中若發生**語法錯誤**，修正後重新存檔即可。紅屏錯誤會消失，含有語法錯誤的模組不會執行，因此無需重新載入應用程式。

若在**模組初始化階段**發生運行時錯誤（例如誤將 `StyleSheet.create` 打成 `Style.create`），修正錯誤後 Fast Refresh 會繼續運作。紅屏消失後模組即會更新。

若錯誤導致**元件內部發生運行時錯誤**，修正後 Fast Refresh 同樣會繼續運作。此時 React 會使用更新後的程式碼重新掛載應用程式。

若應用程式中設有[錯誤邊界](https://reactjs.org/docs/error-boundaries.html)（這在生產環境中能實現優雅降級），紅屏錯誤後的下次編輯會觸發重新渲染。因此錯誤邊界能避免您被強制退回根畫面。但請注意錯誤邊界不宜過度細分，它們在生產環境中會被 React 使用，應始終謹慎設計。

## 限制

Fast Refresh 會嘗試保留編輯中元件的本地 React 狀態，但僅在安全情況下進行。以下是可能導致每次編輯後本地狀態被重置的原因：

- 類別元件的本地狀態無法保留（僅函式元件和 Hook 能保留狀態）
- 編輯的模組可能**同時導出其他內容**（不僅僅是 React 元件）
- 有時模組會導出高階元件的調用結果（如 `createNavigationContainer(MyScreen)`）。若返回的是類別元件，狀態會被重置

長期來看，隨著程式碼庫逐漸改用函式元件和 Hook，狀態保留的適用場景將會增加。

## 技巧

- Fast Refresh 預設會保留函式元件（及 Hook）的 React 本地狀態
- 有時您可能需要**強制重置狀態**並重新掛載元件（例如調整僅在掛載時執行的動畫效果）。此時可在編輯檔案中添加 `// @refresh reset` 指令。該指令僅作用於當前檔案，會使 Fast Refresh 在每次編輯時重新掛載該檔案定義的元件

## Fast Refresh 與 Hook

When possible, Fast Refresh attempts to preserve the state of your component between edits. In particular, `useState` and `useRef` preserve their previous values as long as you don't change their arguments or the order of the Hook calls.

Hooks with dependencies—such as `useEffect`, `useMemo`, and `useCallback`—will _always_ update during Fast Refresh. Their list of dependencies will be ignored while Fast Refresh is happening.

For example, when you edit `useMemo(() => x * 2, [x])` to `useMemo(() => x * 10, [x])`, it will re-run even though `x` (the dependency) has not changed. If React didn't do that, your edit wouldn't reflect on the screen!

Sometimes, this can lead to unexpected results. For example, even a `useEffect` with an empty array of dependencies would still re-run once during Fast Refresh. However, writing code resilient to an occasional re-running of `useEffect` is a good practice even without Fast Refresh. This makes it easier for you to later introduce new dependencies to it.