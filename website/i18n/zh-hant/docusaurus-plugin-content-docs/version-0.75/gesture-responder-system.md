---
id: gesture-responder-system
title: Gesture Responder System
---

手勢響應系統負責管理應用程式中的手勢生命週期。當應用程式判斷使用者意圖時，一次觸碰可能會經歷多個階段。例如，應用需要判斷該觸碰是捲動、滑動元件還是點擊。這些判斷甚至可能在觸碰持續期間發生變化，且可能同時存在多點觸控。

觸碰響應系統的設計目的，是讓元件能在無需了解父元件或子元件的情況下，協調這些觸碰互動行為。

### 最佳實踐

要讓應用體驗更佳，每個操作都應具備以下特性：

- 反饋/高亮效果 - 向使用者顯示當前處理觸碰的元件，以及手勢結束時將觸發的動作
- 可取消性 - 執行操作時，使用者應能透過拖移手指中途中斷觸碰

這些特性能提升使用者操作應用的舒適度，讓人們能無懼失誤地進行實驗性互動。

### TouchableHighlight 與 Touchable* 元件

響應系統可能較複雜，因此我們為"可點擊"元素提供了抽象的 `Touchable` 實現。該實現基於響應系統，並允許以宣告式配置點擊互動。任何需要按鈕或連結的情境（類似網頁）都可使用 `TouchableHighlight`。

## 響應者生命週期

視圖可透過實作正確的協商方法成為觸碰響應者。有兩種方法詢問視圖是否願意成為響應者：

- `View.props.onStartShouldSetResponder: evt => true,` - 該視圖是否希望在觸碰開始時成為響應者？
- `View.props.onMoveShouldSetResponder: evt => true,` - 當視圖非響應者時，每次觸碰移動都會呼叫：該視圖是否想"聲明"觸碰響應權？

若視圖返回 true 並嘗試成為響應者，將發生以下情況之一：

- `View.props.onResponderGrant: evt => {}` - 視圖現已取得觸碰事件響應權。此時應顯示高亮效果告知使用者當前狀態
- `View.props.onResponderReject: evt => {}` - 其他元件已佔用響應權且不釋放

當視圖處於響應狀態時，可能觸發以下處理程序：

- `View.props.onResponderMove: evt => {}` - 使用者正在移動手指
- `View.props.onResponderRelease: evt => {}` - 觸碰結束時觸發（即 "touchUp"）
- `View.props.onResponderTerminationRequest: evt => true` - 其他元件請求響應權。當前視圖是否應釋放？返回 true 表示允許釋放
- `View.props.onResponderTerminate: evt => {}` - 響應權已被系統或其他元件剝奪（iOS 上可能因控制中心/通知中心直接取得權限而無需詢問）

`evt` 是合成觸碰事件，結構如下：

- `nativeEvent`
  - `changedTouches` - 自上次事件後所有變更的觸碰事件陣列
  - `identifier` - 觸碰識別碼
  - `locationX` - 觸碰相對於元素的 X 座標
  - `locationY` - 觸碰相對於元素的 Y 座標
  - `pageX` - 觸碰相對於根元素的 X 座標
  - `pageY` - 觸碰相對於根元素的 Y 座標
  - `target` - 接收觸碰事件的元素節點 ID
  - `timestamp` - 觸碰時間標記（用於計算速度）
  - `touches` - 當前螢幕上所有觸碰點的陣列

### 捕獲 ShouldSet 處理程序

`onStartShouldSetResponder` 和 `onMoveShouldSetResponder` 會以冒泡模式被呼叫，最深的節點會優先被觸發。這意味著當多個視圖對 `*ShouldSetResponder` 處理程序返回 true 時，最深的組件將成為響應者。在大多數情況下這是理想的，因為它能確保所有控件和按鈕都可使用。

然而，有時父組件會希望確保自己成為響應者。這可以通過使用捕獲階段來處理。在響應系統從最深組件冒泡之前，它會先執行捕獲階段，觸發 `on*ShouldSetResponderCapture`。因此，如果父視圖希望阻止子組件在觸摸開始時成為響應者，它應該設置一個返回 true 的 `onStartShouldSetResponderCapture` 處理程序。

- `View.props.onStartShouldSetResponderCapture: evt => true,`
- `View.props.onMoveShouldSetResponderCapture: evt => true,`

### PanResponder

如需更高層級的手勢解譯，請參閱 [PanResponder](panresponder.md)。