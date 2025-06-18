---
id: react-devtools
title: React DevTools
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

[React DevTools](https://github.com/facebook/react/tree/main/packages/react-devtools) 可用於偵錯應用程式中的 React 元件層級結構。

獨立版本的 React DevTools 允許連接到 React Native 應用程式。要使用它，請安裝或執行 `react-devtools` 套件。它應該會在幾秒內連接到您的模擬器。

```sh
npx react-devtools
```

![React DevTools 介面](/docs/assets/debugging-react-devtools-detail.jpg)

<details>
<summary>💡 Installing React DevTools globally</summary>

We recommend running `react-devtools` via `npx`, but you can also install a given version globally.

<Tabs groupId="package-manager" defaultValue={constants.defaultPackageManager} values={constants.packageManagers}>
<TabItem value="npm">

```sh
npm install -g react-devtools
```

</TabItem>
<TabItem value="yarn">

```shell
yarn global add react-devtools
```

</TabItem>
</Tabs>

Then, run the global `react-devtools` command:

```sh
react-devtools
```

</details>

<details>
<summary>💡 Adding React DevTools as a project dependency</summary>

If you prefer to avoid global installations, you can add `react-devtools` as a project dependency. Add the `react-devtools` package to your project using `npm install --save-dev react-devtools`, then add `"react-devtools": "react-devtools"` to the `scripts` section in your `package.json`, and then run `npm run react-devtools` from your project folder to open the DevTools.

</details>

:::tip
在 [react.dev 上的 React 開發者工具指南](https://react.dev/learn/react-developer-tools) 中了解更多關於使用 DevTools 的資訊。
:::

## 與元素檢查器的整合

React Native 提供了一個元素檢查器，可在開發選單中選擇「顯示元素檢查器」來使用。檢查器讓您可以點擊任何 UI 元素並查看其相關資訊。

![元素檢查器介面的影片](/docs/assets/debugging-element-inspector.gif)

當 React DevTools 連接時，元素檢查器將進入**折疊模式**，並改以 DevTools 作為主要 UI。在此模式下，點擊模擬器中的內容將導航至 DevTools 中的相關元件。

您可以在同一選單中選擇「隱藏元素檢查器」以退出此模式。

![React DevTools 元素檢查器整合](/docs/assets/debugging-element-inspector-react-devtools.gif)

## 偵錯應用程式狀態

[Reactotron](https://github.com/infinitered/reactotron) 是一個開源桌面應用程式，允許您檢查 Redux 或 MobX-State-Tree 的應用程式狀態，以及查看自訂日誌、執行自訂命令（如重設狀態）、儲存和還原狀態快照，以及其他對 React Native 應用程式有幫助的偵錯功能。

您可以在 [README](https://github.com/infinitered/reactotron) 中查看安裝說明。如果您使用 Expo，這裡有一篇文章詳細說明 [如何在 Expo 上安裝](https://shift.infinite.red/start-using-reactotron-in-your-expo-project-today-in-3-easy-steps-a03d11032a7a)。

## 疑難排解

:::tip
一旦您啟動了 React DevTools，請按照指示操作。如果您在開啟 React DevTools 之前已經運行了應用程式，您可能需要 [開啟開發選單](./debugging#accessing-the-dev-menu) 來連接它。

![React DevTools 連接流程](/docs/assets/debugging-react-devtools-connection.gif)
:::

:::info
如果連接到 Android 模擬器時遇到問題，請嘗試在新的終端機中執行 `adb reverse tcp:8097 tcp:8097`。
:::