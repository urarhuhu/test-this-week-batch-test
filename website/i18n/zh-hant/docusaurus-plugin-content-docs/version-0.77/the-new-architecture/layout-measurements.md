# 測量佈局

有時，您需要測量當前的佈局，以便對整體佈局進行一些更改，或是做出決策並調用特定的邏輯。

React Native 提供了一些原生方法來獲取視圖的測量值。

調用這些方法的最佳方式是在 `useLayoutEffect` 鉤子中：這將為您提供這些測量的最新值，並讓您能夠在計算測量的同一幀中應用更改。

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
這裡描述的方法在 React Native 提供的大多數默認組件上都可用。然而，它們在那些沒有直接由原生視圖支持的複合組件上是不可用的。這通常包括您在應用中定義的大多數組件。
:::

## measure(callback)

確定給定視圖在屏幕上的位置（`x` 和 `y`）、`width` 和 `height`。通過異步回調返回這些值。如果成功，回調將被調用並帶有以下參數：

- `x`：測量視圖在視口中的原點（左上角）的 `x` 坐標。
- `y`：測量視圖在視口中的原點（左上角）的 `y` 坐標。
- `width`：視圖的 `width`。
- `height`：視圖的 `height`。
- `pageX`：視圖在視口中的 `x` 坐標（通常是整個屏幕）。
- `pageY`：視圖在視口中的 `y` 坐標（通常是整個屏幕）。

此外，`measure()` 返回的 `width` 和 `height` 是組件在視口中的 `width` 和 `height`。

## measureInWindow(callback)

確定給定視圖在窗口中的位置（`x` 和 `y`），並通過異步回調返回這些值。如果 React 根視圖嵌入到另一個原生視圖中，這將為您提供絕對坐標。如果成功，回調將被調用並帶有以下參數：

- `x`：視圖在當前窗口中的 `x` 坐標。
- `y`：視圖在當前窗口中的 `y` 坐標。
- `width`：視圖的 `width`。
- `height`：視圖的 `height`。