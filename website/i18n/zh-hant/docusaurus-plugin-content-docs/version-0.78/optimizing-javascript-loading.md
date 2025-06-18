---
id: optimizing-javascript-loading
title: Optimizing JavaScript loading
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

解析和執行 JavaScript 程式碼需要記憶體和時間。因此，隨著應用程式的增長，延遲載入程式碼直到首次需要時才執行通常很有幫助。React Native 預設啟用了一些標準優化，您也可以在自己的程式碼中採用一些技巧來幫助 React 更有效率地載入應用程式。對於非常大型的應用程式，還有一些進階的自動優化方法（各有其取捨）。

## 推薦：使用 Hermes

Hermes 是新版 React Native 應用程式的預設引擎，並針對高效程式碼載入進行了高度優化。在正式版建置中，JavaScript 程式碼會預先完全編譯為位元組碼。位元組碼是按需載入到記憶體中的，不需要像純 JavaScript 那樣進行解析。

:::info
更多關於在 React Native 中使用 Hermes 的資訊，請參閱[這裡](./hermes)。
:::

## 推薦：延遲載入大型元件

如果一個包含大量程式碼/依賴的元件在應用程式初始渲染時不太可能被使用，您可以使用 React 的 [`lazy`](https://react.dev/reference/react/lazy) API 來延遲載入其程式碼，直到首次渲染時才執行。通常，您應該考慮延遲載入應用程式中的畫面級元件，這樣新增畫面不會增加應用程式的啟動時間。

:::info
更多關於[使用 Suspense 延遲載入元件](https://react.dev/reference/react/lazy#suspense-for-code-splitting)的資訊，包括程式碼範例，請參閱 React 的文件。
:::

### 提示：避免模組副作用

如果您的元件模組（或其依賴項）具有_副作用_，例如修改全域變數或在元件外部訂閱事件，延遲載入元件可能會改變應用程式的行為。React 應用程式中的大多數模組不應該有任何副作用。

```tsx title="SideEffects.tsx"
import Logger from './utils/Logger';

//  🚩 🚩 🚩 Side effect! This must be executed before React can even begin to
// render the SplashScreen component, and can unexpectedly break code elsewhere
// in your app if you later decide to lazy-load SplashScreen.
global.logger = new Logger();

export function SplashScreen() {
  // ...
}
```

## 進階：內聯呼叫 `require`

有時您可能希望延遲載入某些程式碼，直到首次使用時才執行，而不使用 `lazy` 或非同步的 `import()`。您可以在檔案頂部使用 [`require()`](https://metrobundler.dev/docs/module-api/#require) 函數來實現這一點，而不是使用靜態的 `import`。

```tsx title="VeryExpensive.tsx"
import {Component} from 'react';
import {Text} from 'react-native';
// ... import some very expensive modules

export default function VeryExpensive() {
  // ... lots and lots of rendering logic
  return <Text>Very Expensive Component</Text>;
}
```

```tsx title="Optimized.tsx"
import {useCallback, useState} from 'react';
import {TouchableOpacity, View, Text} from 'react-native';
// Usually we would write a static import:
// import VeryExpensive from './VeryExpensive';

let VeryExpensive = null;

export default function Optimize() {
  const [needsExpensive, setNeedsExpensive] = useState(false);
  const didPress = useCallback(() => {
    if (VeryExpensive == null) {
      VeryExpensive = require('./VeryExpensive').default;
    }

    setNeedsExpensive(true);
  }, []);

  return (
    <View style={{marginTop: 20}}>
      <TouchableOpacity onPress={didPress}>
        <Text>Load</Text>
      </TouchableOpacity>
      {needsExpensive ? <VeryExpensive /> : null}
    </View>
  );
}
```

## 進階：自動內聯 `require` 呼叫

如果您使用 React Native CLI 來建置應用程式，`require` 呼叫（但不包括 `import`）會自動為您內聯，無論是在您的程式碼中還是在您使用的任何第三方套件（`node_modules`）中。

```tsx
import {useCallback, useState} from 'react';
import {TouchableOpacity, View, Text} from 'react-native';

// This top-level require call will be evaluated lazily as part of the component below.
const VeryExpensive = require('./VeryExpensive').default;

export default function Optimize() {
  const [needsExpensive, setNeedsExpensive] = useState(false);
  const didPress = useCallback(() => {
    setNeedsExpensive(true);
  }, []);

  return (
    <View style={{marginTop: 20}}>
      <TouchableOpacity onPress={didPress}>
        <Text>Load</Text>
      </TouchableOpacity>
      {needsExpensive ? <VeryExpensive /> : null}
    </View>
  );
}
```

:::info
一些 React Native 框架會禁用此行為。特別是，在 Expo 專案中，`require` 呼叫預設不會內聯。您可以通過編輯專案的 Metro 配置並在 [`getTransformOptions`](https://metrobundler.dev/docs/configuration#gettransformoptions) 中設置 `inlineRequires: true` 來啟用此優化。
:::

### 內聯 `require` 的陷阱

內聯 `require` 呼叫會改變模組的評估順序，甚至可能導致某些模組_永遠_不被評估。這通常是安全的，因為 JavaScript 模組通常是無副作用的。

如果您的某個模組確實有副作用——例如，如果它初始化了某些日誌機制，或修補了其他程式碼使用的全域 API——那麼您可能會看到意外的行為甚至崩潰。在這些情況下，您可能希望將某些模組排除在此優化之外，或完全禁用它。

要**禁用所有自動內聯 `require` 呼叫：**

更新您的 `metro.config.js`，將 `inlineRequires` 轉換器選項設置為 `false`：

```tsx title="metro.config.js"
module.exports = {
  transformer: {
    async getTransformOptions() {
      return {
        transform: {
          inlineRequires: false,
        },
      };
    },
  },
};
```

要僅**排除某些模組不進行 `require` 內聯：**

There are two relevant transformer options: `inlineRequires.blockList` and `nonInlinedRequires`. See the code snippet for examples of how to use each one.

```tsx title="metro.config.js"
module.exports = {
  transformer: {
    async getTransformOptions() {
      return {
        transform: {
          inlineRequires: {
            blockList: {
              // require() calls in `DoNotInlineHere.js` will not be inlined.
              [require.resolve('./src/DoNotInlineHere.js')]: true,

              // require() calls anywhere else will be inlined, unless they
              // match any entry nonInlinedRequires (see below).
            },
          },
          nonInlinedRequires: [
            // require('react') calls will not be inlined anywhere
            'react',
          ],
        },
      };
    },
  },
};
```

See the documentation for [`getTransformOptions` in Metro](https://metrobundler.dev/docs/configuration#gettransformoptions) for more details on setting up and fine-tuning your inline `require`s.

## Advanced: Use random access module bundles (non-Hermes)

:::info
**Not supported when [using Hermes](#use-hermes).** Hermes bytecode is not compatible with the RAM bundle format, and provides the same (or better) performance in all use cases.
:::

Random access module bundles (also known as RAM bundles) work in conjunction with the techniques mentioned above to limit the amount of JavaScript code that needs to be parsed and loaded into memory. Each module is stored as a separate string (or file) which is only parsed when the module needs to be executed.

RAM bundles may be physically split into separate files, or they may use the _indexed_ format, consisting of a lookup table of multiple modules in a single file.

<Tabs groupId="platform" queryString defaultValue={constants.defaultPlatform} values={constants.platforms}>
<TabItem value="android">

On Android enable the RAM format by editing your `android/app/build.gradle` file. Before the line `apply from: "../../node_modules/react-native/react.gradle"` add or amend the `project.ext.react` block:

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

Use the following lines on Android if you want to use a single indexed file:

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

</TabItem>
<TabItem value="ios">

On iOS, RAM bundles are always indexed ( = single file).

Enable the RAM format in Xcode by editing the build phase "Bundle React Native code and images". Before `../node_modules/react-native/scripts/react-native-xcode.sh` add `export BUNDLE_COMMAND="ram-bundle"`:

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

</TabItem>
</Tabs>

See the documentation for [`getTransformOptions` in Metro](https://metrobundler.dev/docs/configuration#gettransformoptions) for more details on setting up and fine-tuning your RAM bundle build.