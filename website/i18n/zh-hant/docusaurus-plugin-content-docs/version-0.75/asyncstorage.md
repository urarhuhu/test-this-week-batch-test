---
id: asyncstorage
title: '🚧 AsyncStorage'
---

> **已移除。** 請改用 [社群套件](https://reactnative.directory/?search=storage) 替代方案。

`AsyncStorage` 是一個未加密、非同步、持久化且應用程式全域的鍵值儲存系統，應取代 LocalStorage 使用。

建議在輕量使用之外的情境中，應基於 `AsyncStorage` 封裝抽象層，而非直接操作全域性的 `AsyncStorage`。

在 iOS 上，`AsyncStorage` 由原生程式碼實現，小數值會儲存在序列化字典中，較大數值則存於獨立檔案。Android 平台會根據可用性選擇使用 [RocksDB](https://rocksdb.org/) 或 SQLite 作為底層儲存引擎。

`AsyncStorage` 的 JavaScript 層提供清晰 API 介面、真實的 `Error` 物件與非批次操作方法，所有 API 均返回 `Promise` 物件。

導入 `AsyncStorage` 函式庫：

```jsx
import {AsyncStorage} from 'react-native';
```

持久化資料：

```jsx
_storeData = async () => {
  try {
    await AsyncStorage.setItem(
      '@MySuperStore:key',
      'I like to save it.',
    );
  } catch (error) {
    // Error saving data
  }
};
```

讀取資料：

```jsx
_retrieveData = async () => {
  try {
    const value = await AsyncStorage.getItem('TASKS');
    if (value !== null) {
      // We have data!!
      console.log(value);
    }
  } catch (error) {
    // Error retrieving data
  }
};
```

---

# 參考文件

## 方法

### `getItem()`

```jsx
static getItem(key: string, [callback]: ?(error: ?Error, result: ?string) => void)
```

根據 `key` 取得對應項目並在完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                                        | Required | Description                                                       |
| -------- | ------------------------------------------- | -------- | ----------------------------------------------------------------- |
| key      | string                                      | Yes      | Key of the item to fetch.                                         |
| callback | `?(error: ?Error, result: ?string) => void` | No       | Function that will be called with a result if found or any error. |

---

### `setItem()`

```jsx
static setItem(key: string, value: string, [callback]: ?(error: ?Error) => void)
```

為指定 `key` 設置數值並在完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to set.                      |
| value    | string                     | Yes      | Value to set for the `key`.                  |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `removeItem()`

```jsx
static removeItem(key: string, [callback]: ?(error: ?Error) => void)
```

移除指定 `key` 的項目並在完成時觸發回調。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to remove.                   |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `mergeItem()`

```jsx
static mergeItem(key: string, value: string, [callback]: ?(error: ?Error) => void)
```

合併現有 `key` 值與輸入值（假設兩者均為 JSON 字串）。返回 `Promise` 物件。

**注意：** 並非所有原生實作都支援此功能。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| key      | string                     | Yes      | Key of the item to modify.                   |
| value    | string                     | Yes      | New value to merge for the `key`.            |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

範例：

```jsx
let UID123_object = {
  name: 'Chris',
  age: 30,
  traits: {hair: 'brown', eyes: 'brown'},
};
// You only need to define what will be added or updated
let UID123_delta = {
  age: 31,
  traits: {eyes: 'blue', shoe_size: 10},
};

AsyncStorage.setItem(
  'UID123',
  JSON.stringify(UID123_object),
  () => {
    AsyncStorage.mergeItem(
      'UID123',
      JSON.stringify(UID123_delta),
      () => {
        AsyncStorage.getItem('UID123', (err, result) => {
          console.log(result);
        });
      },
    );
  },
);

// Console log result:
// => {'name':'Chris','age':31,'traits':
//    {'shoe_size':10,'hair':'brown','eyes':'blue'}}
```

---

### `clear()`

```jsx
static clear([callback]: ?(error: ?Error) => void)
```

清除 _所有_ `AsyncStorage` 資料（含其他客戶端與函式庫的資料）。通常不建議直接呼叫，應改用 `removeItem` 或 `multiRemove` 僅清除自身應用程式的鍵值。返回 `Promise` 物件。

**參數：**

| Name     | Type                       | Required | Description                                  |
| -------- | -------------------------- | -------- | -------------------------------------------- |
| callback | `?(error: ?Error) => void` | No       | Function that will be called with any error. |

---

### `getAllKeys()`

```jsx
static getAllKeys([callback]: ?(error: ?Error, keys: ?Array<string>) => void)
```

取得應用程式 _所有_ 已知鍵值（含其他呼叫方與函式庫的鍵值）。返回 `Promise` 物件。

**參數：**

| Name     | Type                                             | Required | Description                                                     |
| -------- | ------------------------------------------------ | -------- | --------------------------------------------------------------- |
| callback | `?(error: ?Error, keys: ?Array<string>) => void` | No       | Function that will be called with all keys found and any error. |

---

### `flushGetRequests()`

```jsx
static flushGetRequests(): [object Object]
```

使用單一批次呼叫來刷新所有待處理的請求以獲取資料。

---

### `multiGet()`

```jsx
static multiGet(keys: Array<string>, [callback]: ?(errors: ?Array<Error>, result: ?Array<Array<string>>) => void)
```

此方法允許您批次獲取給定`key`陣列對應的項目。您的回調函式將被調用並傳入一組找到的鍵值對：

```
multiGet(['k1', 'k2'], cb) -> cb([['k1', 'val1'], ['k2', 'val2']])
```

該方法返回一個`Promise`物件。

**參數：**

| Name     | Type                                                              | Required | Description                                                                                                         |
| -------- | ----------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------- |
| keys     | `Array<string>`                                                   | Yes      | Array of key for the items to get.                                                                                  |
| callback | `?(errors: ?Array<Error>, result: ?Array<Array<string>>) => void` | No       | Function that will be called with a key-value array of the results, plus an array of any key-specific errors found. |

範例：

```jsx
AsyncStorage.getAllKeys((err, keys) => {
  AsyncStorage.multiGet(keys, (err, stores) => {
    stores.map((result, i, store) => {
      // get at each store's key/value so you can work with it
      let key = store[i][0];
      let value = store[i][1];
    });
  });
});
```

---

### `multiSet()`

```jsx
static multiSet(keyValuePairs: Array<Array<string>>, [callback]: ?(errors: ?Array<Error>) => void)
```

使用此方法批次儲存多組鍵值對。操作完成時，您將收到一個包含任何錯誤的單一回調：

```
multiSet([['k1', 'val1'], ['k2', 'val2']], cb);
```

該方法返回一個`Promise`物件。

**參數：**

| Name          | Type                               | Required | Description                                                                  |
| ------------- | ---------------------------------- | -------- | ---------------------------------------------------------------------------- |
| keyValuePairs | `Array<Array<string>>`             | Yes      | Array of key-value array for the items to set.                               |
| callback      | `?(errors: ?Array<Error>) => void` | No       | Function that will be called with an array of any key-specific errors found. |

---

### `multiRemove()`

```jsx
static multiRemove(keys: Array<string>, [callback]: ?(errors: ?Array<Error>) => void)
```

呼叫此方法以批次刪除`keys`陣列中的所有鍵。返回一個`Promise`物件。

**參數：**

| Name     | Type                               | Required | Description                                                             |
| -------- | ---------------------------------- | -------- | ----------------------------------------------------------------------- |
| keys     | `Array<string>`                    | Yes      | Array of key for the items to delete.                                   |
| callback | `?(errors: ?Array<Error>) => void` | No       | Function that will be called an array of any key-specific errors found. |

範例：

```jsx
let keys = ['k1', 'k2'];
AsyncStorage.multiRemove(keys, err => {
  // keys k1 & k2 removed, if they existed
  // do most stuff after removal (if you want)
});
```

---

### `multiMerge()`

```jsx
static multiMerge(keyValuePairs: Array<Array<string>>, [callback]: ?(errors: ?Array<Error>) => void)
```

批次操作以合併指定鍵集的現有值與新值。此操作假設值為字串化的JSON。返回一個`Promise`物件。

**注意**：並非所有原生實作都支援此功能。

**參數：**

| Name          | Type                               | Required | Description                                                                  |
| ------------- | ---------------------------------- | -------- | ---------------------------------------------------------------------------- |
| keyValuePairs | `Array<Array<string>>`             | Yes      | Array of key-value array for the items to merge.                             |
| callback      | `?(errors: ?Array<Error>) => void` | No       | Function that will be called with an array of any key-specific errors found. |

範例：

```jsx
// first user, initial values
let UID234_object = {
  name: 'Chris',
  age: 30,
  traits: {hair: 'brown', eyes: 'brown'},
};

// first user, delta values
let UID234_delta = {
  age: 31,
  traits: {eyes: 'blue', shoe_size: 10},
};

// second user, initial values
let UID345_object = {
  name: 'Marge',
  age: 25,
  traits: {hair: 'blonde', eyes: 'blue'},
};

// second user, delta values
let UID345_delta = {
  age: 26,
  traits: {eyes: 'green', shoe_size: 6},
};

let multi_set_pairs = [
  ['UID234', JSON.stringify(UID234_object)],
  ['UID345', JSON.stringify(UID345_object)],
];
let multi_merge_pairs = [
  ['UID234', JSON.stringify(UID234_delta)],
  ['UID345', JSON.stringify(UID345_delta)],
];

AsyncStorage.multiSet(multi_set_pairs, err => {
  AsyncStorage.multiMerge(multi_merge_pairs, err => {
    AsyncStorage.multiGet(['UID234', 'UID345'], (err, stores) => {
      stores.map((result, i, store) => {
        let key = store[i][0];
        let val = store[i][1];
        console.log(key, val);
      });
    });
  });
});

// Console log results:
// => UID234 {"name":"Chris","age":31,"traits":{"shoe_size":10,"hair":"brown","eyes":"blue"}}
// => UID345 {"name":"Marge","age":26,"traits":{"shoe_size":6,"hair":"blonde","eyes":"green"}}
```