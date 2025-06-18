---
id: props
title: Props
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

多數元件在創建時可透過不同參數進行客製化，這些創建參數稱為「props」（屬性的簡稱）。

舉例來說，React Native 的基本元件之一是 `Image`。當您創建圖片時，可以使用名為 `source` 的 prop 來控制顯示的圖片。

```SnackPlayer name=Props
import React from 'react';
import {Image} from 'react-native';

const Bananas = () => {
  let pic = {
    uri: 'https://upload.wikimedia.org/wikipedia/commons/d/de/Bananavarieties.jpg',
  };
  return (
    <Image source={pic} style={{width: 193, height: 110, marginTop: 50}} />
  );
};

export default Bananas;
```

請注意包圍 `{pic}` 的大括號——這會將變數 `pic` 嵌入 JSX 中。您可以在 JSX 的大括號內放入任何 JavaScript 表達式。

您自己的元件也可以使用 `props`。這讓您能創建單一元件並在應用程式的多個地方使用，只需在 `render` 函式中引用 `props` 即可微調每個實例的屬性。以下為範例：

<Tabs groupId="language" queryString defaultValue={constants.defaultSnackLanguage} values={constants.snackLanguages}>
<TabItem value="javascript">

```SnackPlayer name=Props&ext=js
import React from 'react';
import {Text, View} from 'react-native';

const Greeting = props => {
  return (
    <View style={{alignItems: 'center'}}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={{alignItems: 'center', top: 50}}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
<TabItem value="typescript">

```SnackPlayer name=Props&ext=tsx
import React from 'react';
import {Text, View} from 'react-native';

type GreetingProps = {
  name: string;
};

const Greeting = (props: GreetingProps) => {
  return (
    <View style={{alignItems: 'center'}}>
      <Text>Hello {props.name}!</Text>
    </View>
  );
};

const LotsOfGreetings = () => {
  return (
    <View style={{alignItems: 'center', top: 50}}>
      <Greeting name="Rexxar" />
      <Greeting name="Jaina" />
      <Greeting name="Valeera" />
    </View>
  );
};

export default LotsOfGreetings;
```

</TabItem>
</Tabs>

透過 `name` prop 可客製化 `Greeting` 元件，讓我們能重複使用該元件來顯示不同問候語。此範例也像[核心元件](intro-react-native-components)那樣在 JSX 中使用 `Greeting` 元件。這種能力正是 React 的強大之處——如果您希望有另一組 UI 基礎元件可供使用，完全可以自行創建新的元件。

這裡另一個新出現的是 [`View`](view.md) 元件。[`View`](view.md) 作為其他元件的容器非常有用，可協助控制樣式與佈局。

透過 `props` 與基本的 [`Text`](text.md)、[`Image`](image.md) 和 [`View`](view.md) 元件，您能建構各種靜態畫面。若要讓應用程式隨時間變化，您需要[學習 State](state.md)。