---
title: 'idx: The Existential Function'
author: Timothy Yung
authorTitle: Engineering Manager at Facebook
authorURL: 'https://github.com/yungsters'
authorImageURL: 'https://pbs.twimg.com/profile_images/1592444107/image.jpg'
authorTwitter: yungsters
tags: [engineering]
---

在Facebook，我們經常需要存取透過GraphQL獲取的資料結構中的深層嵌套值。在存取這些深層嵌套值的過程中，經常會遇到一個或多個中間欄位為可空值的情況。這些中間欄位可能因多種原因而為空，從隱私檢查失敗到僅因null恰好是表示非致命錯誤最靈活的方式。

不幸的是，目前存取這些深層嵌套值的過程既繁瑣又冗長。

```jsx
props.user &&
  props.user.friends &&
  props.user.friends[0] &&
  props.user.friends[0].friends;
```

目前有一個[ECMAScript提案旨在引入存在運算子](https://github.com/claudepache/es-optional-chaining)，這將使此操作更加便利。但在該提案最終確定之前，我們需要一個解決方案來改善開發體驗、維持現有語言語義，並透過Flow鼓勵型別安全。

我們提出了一個稱為`idx`的存在性_函式_。

```jsx
idx(props, _ => _.user.friends[0].friends);
```

此程式碼片段中的呼叫行為與上述程式碼片段中的布林表達式類似，但重複性大幅降低。`idx`函式接受兩個參數：

- 任何值，通常是您想要存取嵌套值的物件或陣列。
- 一個接收第一個參數並在其上存取嵌套值的函式。

理論上，`idx`函式會嘗試捕捉因存取null或undefined屬性而產生的錯誤。如果捕捉到此類錯誤，它將返回null或undefined。（您可以在[idx.js](https://github.com/facebookincubator/idx/blob/master/packages/idx/src/idx.js)中查看其實現方式。）

實際上，對每個嵌套屬性存取進行try-catch操作效能較低，且區分不同類型的TypeErrors較為脆弱。為了解決這些缺點，我們創建了一個Babel插件，將上述`idx`呼叫轉換為以下表達式：

```jsx
props.user == null
  ? props.user
  : props.user.friends == null
    ? props.user.friends
    : props.user.friends[0] == null
      ? props.user.friends[0]
      : props.user.friends[0].friends;
```

最後，我們為`idx`添加了一個自訂的Flow型別宣告，允許在第二個參數中的遍歷過程進行適當的型別檢查，同時允許對可空屬性進行嵌套存取。

該函式、Babel插件和Flow宣告現已[在GitHub上提供](https://github.com/facebookincubator/idx)。使用方式是安裝**idx**和**babel-plugin-idx** npm套件，並將"idx"添加到您的`.babelrc`檔案中的插件列表。