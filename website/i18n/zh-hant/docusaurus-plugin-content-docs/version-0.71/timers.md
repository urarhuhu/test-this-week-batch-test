---
id: timers
title: Timers
---

計時器是應用程式的重要組成部分，React Native 實現了[瀏覽器計時器](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Asynchronous/Timeouts_and_intervals)的功能。

## 計時器

- setTimeout, clearTimeout
- setInterval, clearInterval
- setImmediate, clearImmediate
- requestAnimationFrame, cancelAnimationFrame

`requestAnimationFrame(fn)` 與 `setTimeout(fn, 0)` 不同——前者會在所有畫面刷新後觸發，而後者會盡可能快地觸發（在 iPhone 5S 上每秒超過 1000 次）。

`setImmediate` 會在當前 JavaScript 執行區塊結束時執行，就在將批次回應送回原生端之前。請注意，如果在 `setImmediate` 回調中再次呼叫 `setImmediate`，它會立即執行，不會在中途返回原生端。

`Promise` 的實現使用 `setImmediate` 作為其異步處理機制。

:::note
在 Android 上進行偵錯時，如果偵錯工具與設備的時間出現偏差，可能會導致動畫、事件行為等功能無法正常運作或結果不準確。
請在偵錯機器上執行 ``adb shell "date `date +%m%d%H%M%Y.%S%3N`"`` 來修正此問題。實際裝置上使用需要 root 權限。
:::

## InteractionManager

原生應用程式之所以感覺流暢，一個重要原因是它們在互動和動畫期間避免了耗時操作。在 React Native 中，目前有一個限制是只有單一的 JS 執行線程，但你可以使用 `InteractionManager` 來確保長時間運行的任務被安排在互動/動畫完成後才開始。

應用程式可以通過以下方式安排任務在互動後執行：

```tsx
InteractionManager.runAfterInteractions(() => {
  // ...long-running synchronous task...
});
```

與其他排程替代方案比較：

- requestAnimationFrame(): 用於隨時間推移動畫化視圖的代碼。
- setImmediate/setTimeout/setInterval(): 稍後執行程式碼，請注意這可能會延遲動畫。
- runAfterInteractions(): 稍後執行程式碼，且不會延遲正在進行的動畫。

觸控處理系統將一個或多個活躍觸控視為「互動」，並會延遲 `runAfterInteractions()` 回調，直到所有觸控結束或被取消。

InteractionManager 還允許應用程式通過在動畫開始時創建互動「句柄」，並在完成時清除它來註冊動畫：

```tsx
const handle = InteractionManager.createInteractionHandle();
// run animation... (`runAfterInteractions` tasks are queued)
// later, on animation completion:
InteractionManager.clearInteractionHandle(handle);
// queued tasks run if all handles were cleared
```