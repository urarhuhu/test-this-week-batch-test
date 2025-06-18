---
id: props
title: Props
---

多數元件在創建時可透過不同參數進行客製化，這些創建參數稱為「props」（屬性的簡稱）。

舉例來說，React Native 的基本元件之一為 `Image`。當您創建圖片時，可使用名為 `source` 的 prop 來控制其顯示的圖片。

```SnackPlayer name=Props
import React from 'react';
import { Image } from 'react-native';

const Bananas = () => {
    let pic = {
      uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg'
    };
    return (
      <Image source={pic} style={{width: 193, height: 110, marginTop:50}}/>
    );
}

export default Bananas;
```

請注意包圍 `{pic}` 的大括號——這會將變數 `pic` 嵌入 JSX 中。您可以在 JSX 的大括號內放入任何 JavaScript 表達式。

您自訂的元件同樣能使用 `props`。這讓您能創建單一元件並在應用程式的多處使用，透過在 `render` 函式中引用 `props` 來實現各處微調屬性。以下為範例：

```SnackPlayer name=Props
import React from 'react';
import { Text, View } from 'react-native';

const Greeting = (props) => {
    return (
      <View style={{alignItems: 'center'}}>
        <Text>Hello {props.name}!</Text>
      </View>
    );
}

export default LotsOfGreetings = () => {
    return (
      <View style={{alignItems: 'center', top: 50}}>
        <Greeting name='Rexxar' />
        <Greeting name='Jaina' />
        <Greeting name='Valeera' />
      </View>
    );
}
```

透過將 `name` 作為 prop 使用，我們能客製化 `Greeting` 元件，從而重複利用該元件來實現各問候語。此範例亦在 JSX 中使用 `Greeting` 元件，類似於[核心元件](intro-react-native-components)。這正是 React 的強大之處——若您希望使用不同的 UI 原始元件，完全可以自行創建新的元件。

此處另一新概念是 [`View`](view.md) 元件。[`View`](view.md) 作為其他元件的容器非常實用，有助於控制樣式與佈局。

結合 `props` 與基礎的 [`Text`](text.md)、[`Image`](image.md) 及 [`View`](view.md) 元件，您便能建構各式靜態畫面。若要讓應用程式隨時間產生變化，請[學習 State 的用法](state.md)。