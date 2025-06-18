---
id: ram-bundles-inline-requires
title: RAM Bundles and Inline Requires
---

若您開發的是大型應用程式，可考慮採用隨機存取模組(RAM)套件格式並搭配行內(inline)引入(requires)。這對於包含大量畫面、但典型使用情境中多數畫面不會被開啟的應用程式特別有用。通常適用於啟動後短時間內不需載入大量程式碼的場景，例如應用程式內含複雜的個人檔案頁面或較少使用的功能，但多數使用者會話僅涉及主畫面更新。透過RAM格式與行內引入機制，我們能優化套件載入效率——僅在實際使用時才載入這些功能與畫面。

## JavaScript載入機制

在react-native執行JS程式碼前，這些程式碼必須先載入記憶體並完成解析。傳統套件若載入50MB的套件，必須完整載入並解析全部50MB後才能開始執行。RAM套件的優化原理在於：啟動時僅載入您實際需要的部分內容，後續再按需漸進式載入套件的其他區段。

## 行內引入(Inline Requires)

行內引入會延遲模組或檔案的引入時機，直到該檔案被實際使用時才載入。基礎範例如下：

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

即使未採用RAM格式，行內引入仍能改善啟動時間，因為VeryExpensive.js內的程式碼僅會在首次被引入時執行。

## 啟用RAM格式

在iOS平台，RAM格式會建立單一索引檔案，react native將逐個模組載入。Android平台預設會為每個模組建立獨立檔案，您可強制Android像iOS一樣建立單一檔案，但使用多個獨立檔案通常更具效能優勢且記憶體消耗更低。

在Xcode中編輯「Bundle React Native code and images」建置階段以啟用RAM格式。於`../node_modules/react-native/scripts/react-native-xcode.sh`前添加`export BUNDLE_COMMAND="ram-bundle"`：

```
export BUNDLE_COMMAND="ram-bundle"
export NODE_BINARY=node
../node_modules/react-native/scripts/react-native-xcode.sh
```

在Android平台，編輯`android/app/build.gradle`檔案啟用RAM格式。於`apply from: "../../node_modules/react-native/react.gradle"`前添加或修改`project.ext.react`區塊：

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
若您使用[Hermes JS引擎](https://github.com/facebook/hermes)，則**不應**啟用RAM套件功能。Hermes載入位元組碼時，`mmap`機制會確保不會載入整個檔案。Hermes與RAM套件併用可能導致問題，因兩者的運作機制互不相容。
:::

## 預載與行內引入配置

採用RAM套件後，呼叫`require`會產生額外開銷。當遇到尚未載入的模組時，`require`需透過橋接層(bridge)發送訊息。這對啟動階段影響最大，因初始模組載入期間會集中大量require呼叫。所幸我們可配置部分模組進行預載，為此您需實作某種形式的行內引入機制。

## 分析已載入模組

在您的根檔案(index.(ios|android).js)中，可於初始引入(imports)後添加以下程式碼：

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

執行應用程式時，可透過控制台觀察已載入與待載入模組數量。建議檢視moduleNames確認是否有意外載入的模組。請注意行內引入會在首次參照到導入內容時觸發，您可能需要重構程式碼以確保啟動時僅載入必要模組。您可透過修改require上的Systrace物件來協助除錯有問題的引入。

```js
require.Systrace.beginEvent = message => {
  if (message.includes(problematicModule)) {
    throw new Error();
  }
};
```

每個應用程式的情況不同，但合理的做法是僅載入首屏所需的模組。當您確認模組清單後，請將 loadedModuleNames 的輸出內容儲存為 `packager/modulePaths.js` 檔案。

## 更新 metro.config.js 設定

現在我們需要更新專案根目錄下的 `metro.config.js` 檔案，以使用新生成的 `modulePaths.js` 檔案：

```js
const modulePaths = require('./packager/modulePaths');
const resolve = require('path').resolve;
const fs = require('fs');

// Update the following line if the root folder of your app is somewhere else.
const ROOT_FOLDER = resolve(__dirname, '..');

const config = {
  transformer: {
    getTransformOptions: () => {
      const moduleMap = {};
      modulePaths.forEach(path => {
        if (fs.existsSync(path)) {
          moduleMap[resolve(path)] = true;
        }
      });
      return {
        preloadedModules: moduleMap,
        transform: {inlineRequires: {blockList: moduleMap}},
      };
    },
  },
  projectRoot: ROOT_FOLDER,
};

module.exports = config;
```

設定中的 `preloadedModules` 參數用於指定建構 RAM bundle 時應標記為預載的模組。當 bundle 載入時，這些模組會在所有 require 執行前立即載入。而 `blockList` 參數則表示這些模組不應使用 inline require。由於它們已被預載，使用 inline require 不會帶來效能優勢。實際上，生成的 JavaScript 程式碼每次解析這些引入時，反而會耗費額外時間處理 inline require。

## 測試與效能改善評估

現在您應該已準備好使用 RAM 格式與 inline requires 來建置應用程式。請務必測量並比較啟用前後的啟動時間差異。