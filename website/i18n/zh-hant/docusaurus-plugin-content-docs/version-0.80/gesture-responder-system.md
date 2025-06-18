---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用程式中的手勢生命週期。一個觸控操作在應用判斷用戶意圖時會經歷多個階段。例如，應用需要判斷觸控是滾動、滑動元件還是點擊。這些判斷甚至可能在觸控持續期間發生變化，且可能同時存在多個觸控點。

觸控響應系統的設計目的，是讓元件能在無需了解父元件或子元件的情況下，協調這些觸控互動行為。

### 最佳實踐

要讓應用操作體驗流暢，每個動作都應具備以下特性：

- 反饋/高亮效果 - 向用戶顯示當前處理觸控的元件，以及手勢釋放後將觸發的動作
- 可取消性 - 執行動作時，用戶應能透過拖移手指中途中斷操作

這些特性能提升用戶使用應用的舒適度，因為它允許人們在無需擔心操作失誤的情況下進行探索與互動。

### TouchableHighlight 與 Touchable* 元件

響應系統可能較複雜難用。因此我們為需要「可點擊」的場景提供了抽象的 `Touchable` 實現。該實現基於響應系統，並允許您以宣告式配置點擊互動。任何在網頁上使用按鈕或連結的場景，都可改用 `TouchableHighlight`。

## 響應者生命週期

視圖可透過實作正確的協商方法成為觸控響應者。有兩種方法可詢問視圖是否願意成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 此視圖是否希望在觸控開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸控移動都會呼叫：此視圖是否想「聲明」觸控響應權？

若視圖返回 true 並嘗試成為響應者，將發生以下其中一種情況：

- `View.props.onResponderGrant: evt => {}` - 視圖現在開始響應觸控事件。此時應顯示高亮效果讓用戶知曉狀態變化
- `View.props.onResponderReject: evt => {}` - 當前已有其他元件佔用響應權且不釋放

若視圖正在響應，則可能觸發以下處理程序：

- `View.props.onResponderMove: evt => {}` - 用戶正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸控結束時觸發（即「touchUp」）
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件請求獲取響應權。此視圖是否應釋放響應權？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已被剝奪。可能因 `onResponderTerminationRequest` 呼叫後被其他視圖取得，也可能被系統無預警收回（如 iOS 控制中心/通知中心觸發時）

`evt` 為合成觸控事件，結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有發生變化的觸控事件陣列
  - `identifier` - 觸控點識別碼
  - `locationX` - 觸控點相對於元件的 X 座標
  - `locationY` - 觸控點相對於元件的 Y 座標
  - `pageX` - 觸控點相對於根元件的 X 座標
  - `pageY` - 觸控點相對於根元件的 Y 座標
  - `target` - 接收觸控事件的元件節點 ID
  - `timestamp` - 觸控時間標記（用於計算速度）
  - `touches` - 當前螢幕上所有觸控點的陣列

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式被調用，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的組件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控件和按鈕都可使用。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖想要阻止子組件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高級的手勢解譯，請參閱 [PanResponder](panresponder.md)。