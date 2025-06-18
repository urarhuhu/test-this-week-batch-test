---
id: appstate
title: AppState
---

`AppState` 可以告訴您應用程式目前處於前景或背景狀態，並在狀態變更時發出通知。

AppState 常被用來判斷處理推播通知時的意圖和適當行為。

### 應用程式狀態

- `active` - 應用程式正在前景執行
- `background` - 應用程式正在背景執行。使用者可能處於以下情況：
  - 正在使用其他應用程式
  - 在主畫面
  - [Android] 在其他 `Activity` 中（即使是由您的應用程式啟動）
- [iOS] `inactive` - 此狀態發生在前景與背景轉換期間，以及進入多工檢視、開啟通知中心或來電等非活動期間。

更多資訊請參閱 [Apple 官方文件](https://developer.apple.com/documentation/uikit/app_and_scenes/managing_your_app_s_life_cycle)

## 基本用法

要查看當前狀態，您可以檢查 `AppState.currentState`，該值會保持最新狀態。但在啟動時，由於 `AppState` 需要透過橋接器獲取狀態，`currentState` 會暫時為 null。

```SnackPlayer name=AppState%20Example
import React, {useRef, useState, useEffect} from 'react';
import {AppState, StyleSheet, Text} from 'react-native';
import {SafeAreaView, SafeAreaProvider} from 'react-native-safe-area-context';

const AppStateExample = () => {
  const appState = useRef(AppState.currentState);
  const [appStateVisible, setAppStateVisible] = useState(appState.current);

  useEffect(() => {
    const subscription = AppState.addEventListener('change', nextAppState => {
      if (
        appState.current.match(/inactive|background/) &&
        nextAppState === 'active'
      ) {
        console.log('App has come to the foreground!');
      }

      appState.current = nextAppState;
      setAppStateVisible(appState.current);
      console.log('AppState', appState.current);
    });

    return () => {
      subscription.remove();
    };
  }, []);

  return (
    <SafeAreaProvider>
      <SafeAreaView style={styles.container}>
        <Text>Current state is: {appStateVisible}</Text>
      </SafeAreaView>
    </SafeAreaProvider>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default AppStateExample;
```

此範例永遠只會顯示「當前狀態為：active」，因為應用程式只有在 `active` 狀態時對使用者可見，而 null 狀態只會短暫出現。若想實驗此程式碼，建議使用自己的設備而非嵌入式預覽。

---

# 參考資料

## 事件

### `change`

當應用程式狀態變更時會觸發此事件。監聽器會收到[當前應用程式狀態值](appstate#app-states)之一作為參數。

### `memoryWarning`

此事件用於需要發出記憶體警告或釋放記憶體的情況。

### `focus` <div class="label android">Android</div>

當應用程式獲得焦點時觸發（使用者正在與應用程式互動）。

### `blur` <div class="label android">Android</div>

當使用者未主動與應用程式互動時觸發。適用於使用者下拉[通知抽屜](https://developer.android.com/guide/topics/ui/notifiers/notifications#bar-and-drawer)的情況。此時 `AppState` 不會改變，但會觸發 `blur` 事件。

## 方法

### `addEventListener()`

```tsx
static addEventListener(
  type: AppStateEvent,
  listener: (state: AppStateStatus) => void,
): NativeEventSubscription;
```

設定當 AppState 上發生指定事件類型時要呼叫的函式。有效的 `eventType` 值請參閱[上方列表](#events)。會返回 `EventSubscription`。

## 屬性

### `currentState`

```tsx
static currentState: AppStateStatus;
```