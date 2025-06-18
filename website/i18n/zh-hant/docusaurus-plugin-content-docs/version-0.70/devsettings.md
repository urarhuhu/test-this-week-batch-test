---
id: devsettings
title: DevSettings
---

`DevSettings` 模組提供了在開發階段供開發者自訂設定的方法。

---

# 參考文獻

## 方法

### `addMenuItem()`

```jsx
static addMenuItem(title, handler)
```

在開發者選單中新增自訂選項。

**參數：**

| Name                                                         | Type     |
| ------------------------------------------------------------ | -------- |
| title <div className="label basic required">Required</div>   | string   |
| handler <div className="label basic required">Required</div> | function |

**範例：**

```jsx
DevSettings.addMenuItem('Show Secret Dev Screen', () => {
  Alert.alert('Showing secret dev screen!');
});
```

---

### `reload()`

```jsx
static reload()
```

重新載入應用程式。可直接呼叫或透過使用者互動觸發。

**範例：**

```jsx
<Button title="Reload" onPress={() => DevSettings.reload()} />
```