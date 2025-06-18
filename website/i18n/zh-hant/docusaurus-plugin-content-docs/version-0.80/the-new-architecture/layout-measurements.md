# 測量佈局

有時，您需要測量當前的佈局，以便對整體佈局進行調整或根據測量結果做出決策並執行特定邏輯。

React Native 提供了一些原生方法來獲取視圖的測量數據。

調用這些方法的最佳時機是在 `useLayoutEffect` 鉤子中：這樣可以獲取最新的測量值，並讓您在同一幀中應用基於這些測量的變更。

典型的代碼如下所示：

```tsx
function AComponent(children) {
  const targetRef = React.useRef(null)

  useLayoutEffect(() => {
    targetRef.current?.measure((x, y, width, height, pageX, pageY) => {
      //do something with the measurements
    });
  }, [ /* add dependencies here */]);

  return (
    <View ref={targetRef}>
     {children}
    <View />
  );
}
```

:::note
此處描述的方法在 React Native 提供的大多數默認組件中均可使用。然而，對於那些沒有直接對應原生視圖的複合組件（通常包括您在應用中自定義的大多數組件），這些方法將不可用。
:::

## measure(callback)

確定指定視圖在視口中的位置（`x` 和 `y` 坐標）、`寬度` 和 `高度`。通過異步回調返回這些值。如果成功，回調函數將接收以下參數：

- `x`：被測量視圖在視口中的原點（左上角）的 `x` 坐標。
- `y`：被測量視圖在視口中的原點（左上角）的 `y` 坐標。
- `width`：視圖的 `寬度`。
- `height`：視圖的 `高度`。
- `pageX`：視圖在視口（通常是整個屏幕）中的 `x` 坐標。
- `pageY`：視圖在視口（通常是整個屏幕）中的 `y` 坐標。

此外，`measure()` 返回的 `width` 和 `height` 是組件在視口中的實際尺寸。

## measureInWindow(callback)

確定指定視圖在當前窗口中的位置（`x` 和 `y` 坐標），並通過異步回調返回這些值。如果 React 根視圖嵌入在另一個原生視圖中，此方法將返回絕對坐標。如果成功，回調函數將接收以下參數：

- `x`：視圖在當前窗口中的 `x` 坐標。
- `y`：視圖在當前窗口中的 `y` 坐標。
- `width`：視圖的 `寬度`。
- `height`：視圖的 `高度`。