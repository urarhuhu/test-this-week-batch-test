---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統管理著應用中手勢的生命週期。當應用判斷用戶意圖時，一次觸控可能會經歷多個階段。例如，應用需要判斷觸控是滾動、滑動小部件還是點擊。甚至在觸控持續期間，這種判斷也可能改變。同時還可能存在多個並發觸控。

觸控響應系統的必要性在於：它允許組件在無需了解父組件或子組件的情況下，協調這些觸控交互。

### 最佳實踐

為了讓應用體驗更佳，每個操作應具備以下特性：

- 反饋/高亮效果 - 向用戶展示當前正在處理觸控的元素，以及釋放手勢後將發生的動作
- 可取消性 - 用戶在觸控過程中可通過移開手指中止正在執行的動作

這些特性能提升用戶使用應用時的舒適度，因為它允許人們在無需擔心操作失誤的情況下進行探索和交互。

### TouchableHighlight 與 Touchable* 組件

響應系統可能較為複雜。因此我們為需要"可點擊"的元素提供了抽象的 `Touchable` 實現。該實現利用響應系統，並允許您以聲明式配置點擊交互。在任何需要使用網頁按鈕或鏈接的地方，都可以使用 `TouchableHighlight`。

## 響應者生命週期

視圖可通過實現正確的協商方法成為觸控響應者。有兩種方法用於詢問視圖是否希望成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 該視圖是否希望在觸控開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖不是響應者時，每次觸控移動都會調用：該視圖是否想"聲明"觸控響應權？

如果視圖返回 true 並嘗試成為響應者，將會發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 視圖現在開始響應觸控事件。此時應高亮顯示並向用戶展示正在發生的操作
- `View.props.onResponderReject: evt => {}` - 當前已有其他元素作為響應者且不願釋放權限

如果視圖正在響應，可能會調用以下處理程序：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸控結束時觸發，即"touchUp"
- `View.props.onResponderTerminationRequest: evt => true` - 其他元素希望成為響應者。當前視圖是否應釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已被從視圖剝離。可能在調用 `onResponderTerminationRequest` 後被其他視圖獲取，也可能被操作系統未經詢問直接獲取（iOS 上的控制中心/通知中心會觸發此情況）

`evt` 是一個合成觸控事件，其結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件以來所有發生變化的觸控事件數組
  - `identifier` - 觸控標識符
  - `locationX` - 觸控點相對於元素的 X 座標
  - `locationY` - 觸控點相對於元素的 Y 座標
  - `pageX` - 觸控點相對於根元素的 X 座標
  - `pageY` - 觸控點相對於根元素的 Y 座標
  - `target` - 接收觸控事件的元素節點 ID
  - `timestamp` - 觸控時間標識符，用於計算速度
  - `touches` - 當前屏幕上所有觸控點的數組

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式呼叫，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的組件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控件和按鈕都可使用。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖想要阻止子組件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層次的手勢解譯，請參閱 [PanResponder](panresponder.md)。