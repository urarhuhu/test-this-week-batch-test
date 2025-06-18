---
title: Introducing Hot Reloading
author: Martín Bigio
authorTitle: Software Engineer at Instagram
authorURL: 'https://twitter.com/martinbigio'
authorImageURL: 'https://avatars3.githubusercontent.com/u/535661?v=3&s=128'
authorTwitter: martinbigio
tags: [engineering]
---

React Native's goal is to give you the best possible developer experience. A big part of it is the time it takes between you save a file and be able to see the changes. Our goal is to get this feedback loop to be under 1 second, even as your app grows.

We got close to this ideal via three main features:

- Use JavaScript as the language doesn't have a long compilation cycle time.
- Implement a tool called Packager that transforms es6/flow/jsx files into normal JavaScript that the VM can understand. It was designed as a server that keeps intermediate state in memory to enable fast incremental changes and uses multiple cores.
- Build a feature called Live Reload that reloads the app on save.

At this point, the bottleneck for developers is no longer the time it takes to reload the app but losing the state of your app. A common scenario is to work on a feature that is multiple screens away from the launch screen. Every time you reload, you've got to click on the same path again and again to get back to your feature, making the cycle multiple-seconds long.

## Hot Reloading

The idea behind hot reloading is to keep the app running and to inject new versions of the files that you edited at runtime. This way, you don't lose any of your state which is especially useful if you are tweaking the UI.

A video is worth a thousand words. Check out the difference between Live Reload (current) and Hot Reload (new).

<iframe
  width="100%"
  height="315"
  src="https://www.youtube.com/embed/2uQzVi-KFuc"
  frameborder="0"
  allowfullscreen></iframe>

If you look closely, you can notice that it is possible to recover from a red box and you can also start importing modules that were not previously there without having to do a full reload.

**Word of warning:** because JavaScript is a very stateful language, hot reloading cannot be perfectly implemented. In practice, we found out that the current setup is working well for a large amount of usual use cases and a full reload is always available in case something gets messed up.

Hot reloading is available as of 0.22, you can enable it:

- Open the developer menu
- Tap on "Enable Hot Reloading"

## Implementation in a nutshell

Now that we've seen why we want it and how to use it, the fun part begins: how it actually works.

Hot Reloading is built on top of a feature [Hot Module Replacement](https://webpack.js.org/guides/hot-module-replacement/), or HMR. It was first introduced by webpack and we implemented it inside of React Native Packager. HMR makes the Packager watch for file changes and send HMR updates to a thin HMR runtime included on the app.

In a nutshell, the HMR update contains the new code of the JS modules that changed. When the runtime receives them, it replaces the old modules' code with the new one:

![](/blog/assets/hmr-architecture.png)

The HMR update contains a bit more than just the module's code we want to change because replacing it, it's not enough for the runtime to pick up the changes. The problem is that the module system may have already cached the _exports_ of the module we want to update. For instance, say you have an app composed of these two modules:

```
// log.js
function log(message) {
  const time = require('./time');
  console.log(`[${time()}] ${message}`);
}

module.exports = log;
```

```
// time.js
function time() {
  return new Date().getTime();
}

module.exports = time;
```

The module `log`, prints out the provided message including the current date provided by the module `time`.

When the app is bundled, React Native registers each module on the module system using the `__d` function. For this app, among many `__d` definitions, there will one for `log`:

```
__d('log', function() {
  ... // module's code
});
```

This invocation wraps each module's code into an anonymous function which we generally refer to as the factory function. The module system runtime keeps track of each module's factory function, whether it has already been executed, and the result of such execution (exports). When a module is required, the module system either provides the already cached exports or executes the module's factory function for the first time and saves the result.

So say you start your app and require `log`. At this point, neither `log` nor `time`'s factory functions have been executed so no exports have been cached. Then, the user modifies `time` to return the date in `MM/DD`:

```js
// time.js
function bar() {
  const date = new Date();
  return `${date.getMonth() + 1}/${date.getDate()}`;
}

module.exports = bar;
```

The Packager will send time's new code to the runtime (step 1), and when `log` gets eventually required the exported function gets executed it will do so with `time`'s changes (step 2):

![](/blog/assets/hmr-step.png)

Now say the code of `log` requires `time` as a top level require:

```
const time = require('./time'); // top level require

// log.js
function log(message) {
  console.log(`[${time()}] ${message}`);
}

module.exports = log;
```

When `log` is required, the runtime will cache its exports and `time`'s one. (step 1). Then, when `time` is modified, the HMR process cannot simply finish after replacing `time`'s code. If it did, when `log` gets executed, it would do so with a cached copy of `time` (old code).

For `log` to pick up `time` changes, we'll need to clear its cached exports because one of the modules it depends on was hot swapped (step 3). Finally, when `log` gets required again, its factory function will get executed requiring `time` and getting its new code.

![](/blog/assets/hmr-log.png)

## HMR API

HMR in React Native extends the module system by introducing the `hot` object. This API is based on [webpack](https://webpack.github.io/hot-module-replacement.md)'s one. The `hot` object exposes a function called `accept` which allows you to define a callback that will be executed when the module needs to be hot swapped. For instance, if we would change `time`'s code as follows, every time we save time, we'll see “time changed” in the console:

```
// time.js
function time() {
  ... // new code
}

module.hot.accept(() => {
  console.log('time changed');
});

module.exports = time;
```

Note that only in rare cases you would need to use this API manually. Hot Reloading should work out of the box for the most common use cases.

## HMR Runtime

As we've seen before, sometimes it's not enough only accepting the HMR update because a module that uses the one being hot swapped may have been already executed and its imports cached. For instance, suppose the dependency tree for the movies app example had a top-level `MovieRouter` that depended on the `MovieSearch` and `MovieScreen` views, which depended on the `log` and `time` modules from the previous examples:

![](/blog/assets/hmr-diamond.png)

If the user accesses the movies' search view but not the other one, all the modules except for `MovieScreen` would have cached exports. If a change is made to module `time`, the runtime will have to clear the exports of `log` for it to pick up `time`'s changes. The process wouldn't finish there: the runtime will repeat this process recursively up until all the parents have been accepted. So, it'll grab the modules that depend on `log` and try to accept them. For `MovieScreen` it can bail, as it hasn't been required yet. For `MovieSearch`, it will have to clear its exports and process its parents recursively. Finally, it will do the same thing for `MovieRouter` and finish there as no modules depends on it.

In order to walk the dependency tree, the runtime receives the inverse dependency tree from the Packager on the HMR update. For this example the runtime will receive a JSON object like this one:

```
{
  modules: [
    {
      name: 'time',
      code: /* time's new code */
    }
  ],
  inverseDependencies: {
    MovieRouter: [],
    MovieScreen: ['MovieRouter'],
    MovieSearch: ['MovieRouter'],
    log: ['MovieScreen', 'MovieSearch'],
    time: ['log'],
  }
}
```

## React Components

React 元件要實現熱重載（Hot Reloading）會稍微困難一些。問題在於我們無法單純替換舊程式碼，否則會遺失元件的狀態。針對 React 網頁應用，[Dan Abramov](https://twitter.com/dan_abramov) 實作了一個 Babel [轉換器](https://gaearon.github.io/react-hot-loader/)，透過 webpack 的 HMR API 來解決此問題。簡而言之，他的解決方案是在_轉換階段_為每個 React 元件建立代理（proxy）。這些代理會保存元件狀態，並將生命週期方法委派給實際的元件（即我們進行熱重載的目標）：

![](/blog/assets/hmr-proxy.png)

除了建立代理元件外，該轉換器還會定義 `accept` 函數，其中包含強制 React 重新渲染元件的程式碼。如此一來，我們就能在不丟失應用程式狀態的情況下，熱重載渲染相關的程式碼。

React Native 內建的[轉換器](https://github.com/facebook/react-native/blob/master/packager/transformer.js#L92-L95)使用 `babel-preset-react-native`，其[配置](https://github.com/facebook/react-native/blob/master/babel-preset/configs/hmr.js#L24-L31)會啟用 `react-transform`，方式與在基於 webpack 的 React 網頁專案中相同。

## Redux 狀態庫

要讓 [Redux](https://redux.js.org/) 狀態庫支援熱重載，只需像在基於 webpack 的網頁專案中那樣使用 HMR API：

```
// configureStore.js
import { createStore, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
import reducer from '../reducers';

export default function configureStore(initialState) {
  const store = createStore(
    reducer,
    initialState,
    applyMiddleware(thunk),
  );

  if (module.hot) {
    module.hot.accept(() => {
      const nextRootReducer = require('../reducers/index').default;
      store.replaceReducer(nextRootReducer);
    });
  }

  return store;
};
```

當你修改 reducer 時，接受該 reducer 的程式碼會被發送至客戶端。客戶端會發現該 reducer 無法自行處理更新，因此會查找所有引用它的模組並嘗試接受更新。最終，流程會抵達單一狀態庫（即 `configureStore` 模組），由它來接受 HMR 更新。

## 結論

若你有興趣協助改進熱重載功能，建議閱讀 [Dan Abramov 關於熱重載未來的文章](https://medium.com/@dan_abramov/hot-reloading-in-react-1140438583bf#.jmivpvmz4)並參與貢獻。例如，Johny Days 正著手[實現多客戶端連線支援](https://github.com/facebook/react-native/pull/6179)。我們仰賴大家的協助來維護與強化此功能。

React Native 讓我們能重新思考應用程式的建構方式，以打造更優異的開發者體驗。熱重載只是其中一環，我們還能運用哪些巧妙技巧來進一步提升？