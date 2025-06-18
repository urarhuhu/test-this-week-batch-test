---
title: Using TypeScript with React Native
author: Ash Furrow
authorTitle: Software Engineer at Artsy
authorURL: 'https://github.com/ashfurrow'
authorImageURL: 'https://avatars2.githubusercontent.com/u/498212?s=460&v=4'
authorTwitter: ashfurrow
tags: [engineering]
---

JavaScript! We all love it. But some of us also love [types](https://en.wikipedia.org/wiki/Data_type). Luckily, options exist to add stronger types to JavaScript. My favourite is [TypeScript](https://www.typescriptlang.org), but React Native supports [Flow](https://flow.org) out of the box. Which you prefer is a matter of preference, they each have their own approach on how to add the magic of types to JavaScript. Today, we're going to look at how to use TypeScript in React Native apps.

This post uses Microsoft's [TypeScript-React-Native-Starter](https://github.com/Microsoft/TypeScript-React-Native-Starter) repo as a guide.

**Update**: Since this blog post was written, things have gotten even easier. You can replace all the set up described in this blog post by running just one command:

```sh
npx react-native init MyAwesomeProject --template react-native-template-typescript
```

However, there _are_ some limitations to Babel's TypeScript support, which the blog post above goes into in detail. The steps outlined in _this_ post still work, and Artsy is still using [react-native-typescript-transformer](https://github.com/ds300/react-native-typescript-transformer) in production, but the fastest way to get up and running with React Native and TypeScript is using the above command. You can always switch later if you have to.

In any case, have fun! The original blog post continues below.

## Prerequisites

Because you might be developing on one of several different platforms, targeting several different types of devices, basic setup can be involved. You should first ensure that you can run a plain React Native app without TypeScript. Follow [the instructions on the React Native website to get started](/docs/getting-started). When you've managed to deploy to a device or emulator, you'll be ready to start a TypeScript React Native app.

You will also need [Node.js](https://nodejs.org/en/), [npm](https://www.npmjs.com), and [Yarn](https://yarnpkg.com/lang/en).

## Initializing

Once you've tried scaffolding out an ordinary React Native project, you'll be ready to start adding TypeScript. Let's go ahead and do that.

```sh
react-native init MyAwesomeProject
cd MyAwesomeProject
```

## Adding TypeScript

The next step is to add TypeScript to your project. The following commands will:

- add TypeScript to your project
- add [React Native TypeScript Transformer](https://github.com/ds300/react-native-typescript-transformer) to your project
- initialize an empty TypeScript config file, which we'll configure next
- add an empty React Native TypeScript Transformer config file, which we'll configure next
- adds [typings](https://github.com/DefinitelyTyped/DefinitelyTyped) for React and React Native

Okay, let's go ahead and run these.

```sh
yarn add --dev typescript
yarn add --dev react-native-typescript-transformer
yarn tsc --init --pretty --jsx react
touch rn-cli.config.js
yarn add --dev @types/react @types/react-native
```

The `tsconfig.json` file contains all the settings for the TypeScript compiler. The defaults created by the command above are mostly fine, but open the file and uncomment the following line:

```js
{
  /* Search the config file for the following line and uncomment it. */
  // "allowSyntheticDefaultImports": true,  /* Allow default imports from modules with no default export. This does not affect code emit, just typechecking. */
}
```

The `rn-cli.config.js` contains the settings for the React Native TypeScript Transformer. Open it and add the following:

```js
module.exports = {
  getTransformModulePath() {
    return require.resolve('react-native-typescript-transformer');
  },
  getSourceExts() {
    return ['ts', 'tsx'];
  },
};
```

## Migrating to TypeScript

Rename the generated `App.js` and `__tests_/App.js` files to `App.tsx`. `index.js` needs to use the `.js` extension. All new files should use the `.tsx` extension (or `.ts` if the file doesn't contain any JSX).

If you tried to run the app now, you'd get an error like `object prototype may only be an object or null`. This is caused by a failure to import the default export from React as well as a named export on the same line. Open `App.tsx` and modify the import at the top of the file:

```diff
-import React, { Component } from 'react';
+import React from 'react'
+import { Component } from 'react';
```

這部分與 Babel 和 TypeScript 在 CommonJS 模組上的交互方式差異有關。未來兩者將穩定採用相同的行為模式。

至此，你應該能夠運行 React Native 應用程式了。

## 添加 TypeScript 測試基礎設施

React Native 內建 [Jest](https://github.com/facebook/jest)，因此要在 TypeScript 環境中測試 React Native 應用，我們需將 [ts-jest](https://www.npmjs.com/package/ts-jest) 加入 `devDependencies`。

```sh
yarn add --dev ts-jest
```

接著開啟 `package.json`，將 `jest` 欄位替換為以下內容：

```js
{
  "jest": {
    "preset": "react-native",
    "moduleFileExtensions": [
      "ts",
      "tsx",
      "js"
    ],
    "transform": {
      "^.+\\.(js)$": "<rootDir>/node_modules/babel-jest",
      "\\.(ts|tsx)$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
    },
    "testRegex": "(/__tests__/.*|\\.(test|spec))\\.(ts|tsx|js)$",
    "testPathIgnorePatterns": [
      "\\.snap$",
      "<rootDir>/node_modules/"
    ],
    "cacheDirectory": ".jest/cache"
  }
}
```

這會配置 Jest 使用 `ts-jest` 來處理 `.ts` 和 `.tsx` 檔案。

## 安裝依賴項的型別宣告

為了獲得最佳的 TypeScript 開發體驗，我們需要讓型別檢查器理解依賴項的結構與 API。部分函式庫會在發佈套件時包含 `.d.ts` 檔案（型別宣告/型別定義檔案），這些檔案能描述底層 JavaScript 的結構。對於其他函式庫，我們需明確安裝 `@types/` npm 範圍內的對應套件。

例如，此處我們需要 Jest、React、React Native 和 React Test Renderer 的型別定義。

```ts
yarn add --dev @types/jest @types/react @types/react-native @types/react-test-renderer
```

我們將這些宣告檔案套件保存為 _開發_ 依賴項，因為這是個 React Native _應用程式_，僅在開發時使用這些依賴項而非運行時。如果我們要發佈函式庫到 NPM，可能需要將部分型別依賴項設為常規依賴項。

可參閱[此處了解更多關於取得 `.d.ts` 檔案](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html)的資訊。

## 忽略更多檔案

為了版本控制，你需要開始忽略 `.jest` 資料夾。若使用 git，只需在 `.gitignore` 檔案中新增條目。

```config
# Jest
#
.jest/
```

作為檢查點，建議將檔案提交至版本控制系統。

```sh
git init
git add .gitignore # import to do this first, to ignore our files
git add .
git commit -am "Initial commit."
```

## 添加元件

讓我們為應用添加一個元件。現在建立一個 `Hello.tsx` 元件，這是個教學用元件（非實際開發中會寫的內容），但能展示如何在 React Native 中使用 TypeScript 實現非簡單功能。

建立 `components` 目錄並加入以下範例。

```ts
// components/Hello.tsx
import React from 'react';
import {Button, StyleSheet, Text, View} from 'react-native';

export interface Props {
  name: string;
  enthusiasmLevel?: number;
}

interface State {
  enthusiasmLevel: number;
}

export class Hello extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);

    if ((props.enthusiasmLevel || 0) <= 0) {
      throw new Error(
        'You could be a little more enthusiastic. :D',
      );
    }

    this.state = {
      enthusiasmLevel: props.enthusiasmLevel || 1,
    };
  }

  onIncrement = () =>
    this.setState({
      enthusiasmLevel: this.state.enthusiasmLevel + 1,
    });
  onDecrement = () =>
    this.setState({
      enthusiasmLevel: this.state.enthusiasmLevel - 1,
    });
  getExclamationMarks = (numChars: number) =>
    Array(numChars + 1).join('!');

  render() {
    return (
      <View style={styles.root}>
        <Text style={styles.greeting}>
          Hello{' '}
          {this.props.name +
            this.getExclamationMarks(this.state.enthusiasmLevel)}
        </Text>

        <View style={styles.buttons}>
          <View style={styles.button}>
            <Button
              title="-"
              onPress={this.onDecrement}
              accessibilityLabel="decrement"
              color="red"
            />
          </View>

          <View style={styles.button}>
            <Button
              title="+"
              onPress={this.onIncrement}
              accessibilityLabel="increment"
              color="blue"
            />
          </View>
        </View>
      </View>
    );
  }
}

// styles
const styles = StyleSheet.create({
  root: {
    alignItems: 'center',
    alignSelf: 'center',
  },
  buttons: {
    flexDirection: 'row',
    minHeight: 70,
    alignItems: 'stretch',
    alignSelf: 'center',
    borderWidth: 5,
  },
  button: {
    flex: 1,
    paddingVertical: 0,
  },
  greeting: {
    color: '#999',
    fontWeight: 'bold',
  },
});
```

哇！內容很多，讓我們分解說明：

- 我們渲染的是 `View` 和 `Button` 等跨平台原生元件，而非 `div`、`span`、`h1` 等 HTML 元素
- 樣式透過 React Native 提供的 `StyleSheet.create` 函數定義。React 的樣式表允許使用 Flexbox 控制佈局，並採用類似 CSS 的結構進行樣式設定

## 添加元件測試

現在我們有了元件，來試著測試它。

我們已安裝 Jest 作為測試執行器。接下來要為元件編寫快照測試，先添加快照測試所需的附加套件：

```sh
yarn add --dev react-addons-test-utils
```

現在於 `components` 目錄建立 `__tests__` 資料夾，並為 `Hello.tsx` 添加測試：

```ts
// components/__tests__/Hello.tsx
import React from 'react';
import renderer from 'react-test-renderer';

import {Hello} from '../Hello';

it('renders correctly with defaults', () => {
  const button = renderer
    .create(<Hello name="World" enthusiasmLevel={1} />)
    .toJSON();
  expect(button).toMatchSnapshot();
});
```

The first time the test is run, it will create a snapshot of the rendered component and store it in the `components/__tests__/__snapshots__/Hello.tsx.snap` file. When you modify your component, you'll need to update the snapshots and review the update for inadvertent changes. You can read more about testing React Native components [here](https://facebook.github.io/jest/docs/en/tutorial-react-native.html).

## Next Steps

Check out the official [React tutorial](https://reactjs.org/tutorial/tutorial.html) and state-management library [Redux](https://redux.js.org). These resources can be helpful when writing React Native apps. Additionally, you may want to look at [ReactXP](https://microsoft.github.io/reactxp/), a component library written entirely in TypeScript that supports both React on the web as well as React Native.

Have fun in a more type-safe React Native development environment!