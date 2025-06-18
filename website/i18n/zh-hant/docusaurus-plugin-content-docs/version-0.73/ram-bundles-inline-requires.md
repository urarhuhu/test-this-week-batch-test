---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您的應用程式規模龐大，可考慮採用隨機存取模組(RAM)套件格式並搭配行內(inline) require。此方式特別適合具有大量畫面、但典型使用情境中未必會開啟的應用程式。通常對於啟動後短時間內不需執行大量程式碼的應用程式很有幫助。例如：應用程式包含複雜的個人檔案頁面或較少使用的功能，但多數使用情境僅涉及主畫面更新。我們可透過RAM格式與行內require(在實際使用時才載入)來優化套件載入效率。

## JavaScript載入機制

在react-native執行JS程式碼前，該程式碼必須先載入記憶體並完成解析。傳統套件若載入50MB的套件，必須完整載入並解析全部50MB後才能開始執行。RAM套件的優化原理在於：您只需在啟動時載入實際需要的部分，並在需要其他功能時逐步載入套件的對應區塊。

## 行內require

行內require會延遲模組或檔案的載入時機，直到實際需要該檔案時才執行。基礎範例如下：

```tsx title="VeryExpensive.tsx"
import React, {Component} from 'react';
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

即使不使用RAM格式，行內require仍能改善啟動時間，因為VeryExpensive.js內的程式碼只會在首次被require時才會執行。

## 啟用RAM格式

在iOS平台，RAM格式會建立單一索引檔案，react native將逐個模組載入。Android平台預設會為每個模組建立獨立檔案。您可強制Android像iOS一樣建立單一檔案，但使用多個獨立檔案通常效能更佳且記憶體消耗更少。

在Xcode中編輯「Bundle React Native code and images」建置階段以啟用RAM格式。於`../node_modules/react-native/scripts/react-native-xcode.sh`前添加`export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

在Android平台編輯`android/app/build.gradle`檔案啟用RAM格式。於`apply from: "../../node_modules/react-native/react.gradle"`前添加或修改`project.ext.react`區塊：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
]
```

若要在Android平台使用單一索引檔案，請添加以下設定：

```
project.ext.react = [
  bundleCommand: "ram-bundle",
  extraPackagerArgs: ["--indexed-ram-bundle"]
]
```

:::info
若您使用[Hermes JS引擎](https://github.com/facebook/hermes)，則**不應**啟用RAM套件功能。Hermes載入bytecode時，`mmap`會確保不會載入整個檔案。Hermes與RAM套件併用可能導致問題，因兩者的運作機制互不相容。
:::

## 設定預載與行內require

使用RAM套件後，呼叫`require`會產生額外開銷。當遇到尚未載入的模組時，`require`需透過bridge傳送訊息。這對啟動階段影響最大，因為此時應用程式載入初始模組時可能需呼叫大量require。所幸我們可設定部分模組為預載模式，為此您需實作某種形式的行內require。

## 分析已載入模組

您可在根檔案(index.(ios|android).js)的初始import後添加以下程式碼：

```js
const modules = require.getModules();
const moduleIds = Object.keys(modules);
const loadedModuleNames = moduleIds
  .filter(moduleId => modules[moduleId].isInitialized)
  .map(moduleId => modules[moduleId].verboseName);
const waitingModuleNames = moduleIds
  .filter(moduleId => !modules[moduleId].isInitialized)
  .map(moduleId => modules[moduleId].verboseName);

// make sure that the modules you expect to be waiting are actually waiting
console.log(
  'loaded:',
  loadedModuleNames.length,
  'waiting:',
  waitingModuleNames.length,
);

// grab this text blob, and put it in a file named packager/modulePaths.js
console.log(
  `module.exports = ${JSON.stringify(
    loadedModuleNames.sort(),
    null,
    2,
  )};`,
);
```

執行應用程式時，可從控制台觀察已載入與待載入模組的數量。建議檢視moduleNames確認是否有非預期載入的模組。請注意行內require會在首次參照import時觸發。您可能需要分析並重構程式碼，確保啟動時僅載入必要模組。您可透過修改require上的Systrace物件來協助除錯有問題的require呼叫。

```js
require.Systrace.beginEvent = message => {
  if (message.includes(problematicModule)) {
    throw new Error();
  }
};
```

每個應用程式都不同，但合理的做法是只載入首屏所需的模組。當您確認無誤後，可將 loadedModuleNames 的輸出結果存入名為 `packager/modulePaths.js` 的檔案中。

## 更新 metro.config.js

現在我們需要更新專案根目錄下的 `metro.config.js` 檔案，以使用新生成的 `modulePaths.js` 檔案：

<!-- prettier-ignore -->

```js
const {getDefaultConfig, mergeConfig} = require('@react-native/metro-config');
const fs = require('fs');
const path = require('path');
const modulePaths = require('./packager/modulePaths');

const config = {
  transformer: {
    getTransformOptions: () => {
      const moduleMap = {};
      modulePaths.forEach(modulePath => {
        if (fs.existsSync(modulePath)) {
          moduleMap[path.resolve(modulePath)] = true;
        }
      });
      return {
        preloadedModules: moduleMap,
        transform: {inlineRequires: {blockList: moduleMap}},
      };
    },
  },
};

module.exports = mergeConfig(getDefaultConfig(__dirname), config);
```

另請參閱 [**設定 Metro**](/docs/metro#configuring-metro)。

設定檔中的 `preloadedModules` 條目標記了哪些模組應在建構 RAM bundle 時預先載入。當 bundle 載入時，這些模組會立即載入，甚至在執行任何 require 之前。而 `blockList` 條目則標記了哪些模組不應使用 inline require。由於這些模組已被預載，使用 inline require 不會帶來效能優勢。實際上，生成的 JavaScript 程式碼每次參照這些導入時，反而會額外耗費時間解析 inline require。

## 測試與效能改善評估

現在您應該已準備好使用 RAM 格式和 inline requires 來建置應用程式。請務必測量並比較啟用前後的啟動時間。