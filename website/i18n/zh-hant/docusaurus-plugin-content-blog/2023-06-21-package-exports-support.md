---
title: 'Package Exports Support in React Native'
authors: [huntie]
tags: [announcement, metro]
date: 2023-06-21
---

# Package Exports Support in React Native

With the release of [React Native 0.72](/blog/2023/06/21/0.72-metro-package-exports-symlinks), Metro — our JavaScript build tool — now includes beta support for the `package.json` [`"exports"`](https://nodejs.org/docs/latest-v18.x/api/packages.html#exports) field. When [enabled](/blog/2023/06/21/package-exports-support#enabling-package-exports-beta), it adds the following functionality:

- [React Native projects will work with more npm packages out-of-the-box](/blog/2023/06/21/package-exports-support#for-app-developers)
- [New capabilities for packages to define their API and target React Native](/blog/2023/06/21/package-exports-support#for-package-maintainers-preview)
- [Some breaking changes to package resolution (in edge cases)](/blog/2023/06/21/package-exports-support#breaking-changes)

In this post we'll cover how Package Exports works, and what these changes mean for you as a React Native app developer or package maintainer.

<!-- truncate -->

## What is Package Exports?

Introduced in Node.js 12.7.0, Package Exports is the modern approach for npm packages to specify **entry points** — the mapping of package subpaths which can be externally imported and which file(s) they should resolve to.

Supporting `"exports"` improves how React Native projects will work with the wider JavaScript ecosystem ([used in ~16.6k packages today](https://github.com/search?q=path%3A%2A%2A%2Fpackage.json+%22%5C%22access%5C%22%3A+%5C%22public%5C%22%22+%22%5C%22exports%5C%22%22+NOT+path%3A%2A%2A%2Fnode_modules+NOT+is%3Afork+NOT+is%3Aarchived&type=code)), and gives package authors a standardised feature set for multiplatform packages to target React Native.

[`"exports"`](https://nodejs.org/docs/latest-v19.x/api/packages.html#exports) can be used alongside, or instead of, [`"main"`](https://nodejs.org/docs/latest-v19.x/api/packages.html#main) in a `package.json` file.

```json
{
  "name": "@storybook/addon-actions",
  "main": "./dist/index.js",
  ...
  "exports": {
    ".": {
      "node": "./dist/index.js",
      "import": "./dist/index.mjs",
      "default": "./dist/index.js"
    },
    "./preview": {
      "import": "./dist/preview.mjs",
      "default": "./dist/preview.js"
    },
    ...
    "./package.json": "./package.json"
  }
}
```

Here's some app code consuming the above package by importing different subpaths of `@storybook/addon-actions`.

```js
import {action} from '@storybook/addon-actions';
// -> '@storybook/addon-actions/dist/index.js'

import {action} from '@storybook/addon-actions/preview';
// -> '@storybook/addon-actions/dist/preview.js'

import helpers from '@storybook/addon-actions/src/preset/addArgsHelpers';
// Inaccessible - not listed in "exports"!
```

The headlining features of Package Exports are:

- **Package encapsulation**: Only subpaths defined in `"exports"` can be imported from outside the package — giving packages control over their public API.
- **Subpath aliases**: Packages can define custom subpaths which map to a different file location (including via [subpath patterns](https://nodejs.org/docs/latest-v19.x/api/packages.html#subpath-patterns)) — allowing relocation of files while preserving the public API.
- **Conditional exports**: A subpath may resolve to a different underlying file depending on environment. For example, to target `"node"`, `"browser"`, or `"react-native"` runtimes — replacing the [`"browser"` field spec](https://github.com/defunctzombie/package-browser-field-spec).

:::note
The full capabilities for `"exports"` are detailed in the [Node.js Package Entry Points spec](https://nodejs.org/docs/latest-v19.x/api/packages.html#package-entry-points).

Since these features overlap with existing React Native concepts (such as [platform-specific extensions](/docs/platform-specific-code)), and since `"exports"` had been live in the npm ecosystem for some time, we reached out to the React Native community to make sure our implementation would meet developers' needs ([PR](https://github.com/react-native-community/discussions-and-proposals/pull/534), [final RFC](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0534-metro-package-exports-support.md)).
:::

## 給應用程式開發者

Package Exports 功能現已開放測試版啟用。

- 現在應能正常導入依賴 Package Exports 功能的套件（如 [**Firebase**](https://www.npmjs.com/package/firebase) 和 [**Storybook**](https://www.npmjs.com/search?q=%40storybook)）
- 使用 Metro 的 React Native for Web 專案現在可運用 `"browser"` 條件式導出，無需再使用替代方案

啟用 Package Exports 可能導致少數[邊緣案例的破壞性變更](#breaking-changes)，您可[立即測試](#validating-changes-in-your-project)是否影響專案。

**未來 React Native 版本將預設啟用 Package Exports**。由於雞生蛋問題，部分套件先前因 React Native 應用程式未支援而延遲遷移至 `"exports"`，或採用我們的 `"react-native"` 根欄位應急方案。Metro 支援這些功能後，將推動整個生態系統向前發展。

### 啟用 Package Exports（測試版）

可透過 [`resolver.unstable_enablePackageExports`](https://metrobundler.dev/docs/configuration/#unstable_enablepackageexports-experimental) 選項，在應用程式的 [**metro.config.js**](https://github.com/facebook/react-native/blob/0.72-stable/packages/react-native/template/metro.config.js) 檔案中啟用 Package Exports。

```js
const config = {
  // ...
  resolver: {
    unstable_enablePackageExports: true,
  },
};
```

Metro 另提供兩個解析器選項來設定條件式導出行為：

- [`unstable_conditionNames`](https://metrobundler.dev/docs/configuration/#unstable_conditionnames-experimental) — 解析條件式導出時要驗證的[條件名稱](https://nodejs.org/docs/latest-v19.x/api/packages.html#community-conditions-definitions)集合。預設匹配 `['require', 'import', 'react-native']`
- [`unstable_conditionsByPlatform`](https://metrobundler.dev/docs/configuration/#unstable_conditionsbyplatform-experimental) — 針對特定平台目標解析時要額外驗證的條件名稱。當平台為 `'web'` 時，預設匹配 `'browser'`

:::tip
**請記得使用 React Native 的 [Jest 預設配置](https://github.com/facebook/react-native/blob/main/template/jest.config.js#L2)！** Jest 預設支援 Package Exports。測試中可透過 [`testEnvironmentOptions`](https://jestjs.io/docs/configuration#testenvironmentoptions-object) 選項覆寫要解析的 `customExportConditions`。

**若使用 TypeScript**，可在專案的 `tsconfig.json` 中設定 [`moduleResolution: 'bundler'`](https://www.typescriptlang.org/tsconfig#moduleResolution) 和 [`resolvePackageJsonImports: false`](https://www.typescriptlang.org/tsconfig#resolvePackageJsonExports) 來匹配解析行為。
:::

#### 驗證專案中的變更

對於現有專案，建議早期採用者依下列步驟確認啟用 `unstable_enablePackageExports` 後是否發生解析變更。此為一次性流程，很可能完全無變更，但我們希望開發者能明確選擇啟用。

<details>
<summary>💡 Validating changes in your project</summary>

:::note
If you are not using Yarn, substitute `yarn` for `npx` (or the relevant tool used in your project).
:::

1. Get all resolved dependencies (before changes):

   ```sh
   # Replace index.js with your entry file if needed, such as App.js
   yarn metro get-dependencies index.js --platform android --output before.txt
   ```

   - **Expo CLI**: Run `npx expo customize metro.config.js` if your project doesn't have a `metro.config.js` file yet.
   - For full coverage, substitute `--platform android` for the other platforms in use by your app (e.g. `ios`, `web`).

2. Enable `resolver.unstable_enablePackageExports` in `metro.config.js`.
3. Get all resolved dependencies (after changes):

   ```sh
   yarn metro get-dependencies index.js --platform android --output after.txt
   ```

4. Compare!

   ```sh
   diff before.txt after.txt
   ```

</details>

### 破壞性變更

我們決定在 Metro 中實作符合規範的 Package Exports（這需要一些破壞性變更），但同時保持向後相容性（協助現有導入的應用程式逐步遷移）。

關鍵破壞性變更是：當套件提供 `"exports"` 時，將優先參考該欄位（而非其他 `package.json` 欄位）——且會直接使用匹配的子路徑目標。

- Metro 不會針對導入指示詞展開 [`sourceExts`](https://metrobundler.dev/docs/configuration/#sourceexts)
- Metro 不會針對目標檔案解析[平台特定副檔名](/docs/platform-specific-code)

詳情請參閱 Metro 文件中的[所有破壞性變更](https://metrobundler.dev/docs/package-exports#summary-of-breaking-changes)。

### Package encapsulation is lenient

When Metro encounters a subpath that isn't listed in `"exports"`, **it will fall back to legacy resolution**. This is a compatibility feature intended to reduce user friction for previously allowable imports in existing React Native projects.

Instead of throwing an error, Metro will log a warning.

```sh
warn: You have imported the module "foo/private/fn.js" which is not listed in
the "exports" of "foo". Consider updating your call site or asking the package
maintainer(s) to expose this API.
```

:::note
We plan to implement a strict mode for package encapsulation in future, to align with Node's default behaviour. Therefore, **we recommend that all developers address these warnings** if raised by users.
:::

## For package maintainers (preview)

:::info
Per our [rollout plan](#the-future-stable-exports-enabled-by-default), Package Exports will be enabled for most projects in the next React Native release (0.73) later this year.

**We have no plans to remove support for the `"main"` field and other current package resolution features any time soon.**
:::

Package Exports provides the ability to restrict access to your package's internals, and more predictable capabilities for libraries to target React Native and React Native for Web.

### If you are using `"exports"` today

If your package uses `"exports"` alongside the current `"react-native"` root field, please bear in mind the [breaking changes](#breaking-changes) for users above. For users enabling this feature in Metro, `"exports"` will now be considered first during module resolution.

In practice, we anticipate the main change for users will be the enforcement (via warnings) of any inaccessible subpaths in their apps, from respecting `"exports"` package encapsulation.

### Migrating to `"exports"`

**Adding an `"exports"` field to your package is entirely optional**. Existing package resolution features will behave identically for packages which don't use `"exports"` — and we have no plans to remove this behaviour.

We believe that the new features of `"exports"` provide a compelling feature set for React Native package maintainers.

- **Tighten your package API**: This is a great time to review the module API of your package, which can now be formally defined via exported subpath aliases. This prevents users from accessing internal APIs, reducing surface area for bugs.
- **Conditional exports**: If your package targets React Native for Web (i.e. `"react-native"` and `"browser"`), we now give packages control of the resolution order of these conditions (see next heading).

If you decide to introduce `"exports"`, **we recommend making this as a breaking change**. We've prepared a [**migration guide**](https://metrobundler.dev/docs/package-exports#migration-guide-for-package-maintainers) in the Metro docs which includes how to replace features such as platform-specific extensions.

:::note
**Please do not rely on the lenient behaviours of Metro's implementation.** While Metro is backwards-compatible, packages should follow how `"exports"` is documented in the spec and strictly implemented by other tools.
:::

### The new `"react-native"` condition

We've introduced `"react-native"` as a community condition (for use with conditional exports). This represents React Native, the framework, sitting alongside other recognised runtimes such as `"node"` and `"deno"` ([RFC](https://github.com/nodejs/node/pull/45367)).

> [Community Conditions Definitions — **`"react-native"`**](https://nodejs.org/docs/latest-v19.x/api/packages.html#community-conditions-definitions)
>
> _Will be matched by the React Native framework (all platforms). To target React Native for Web, "browser" should be specified before this condition._

This replaces the previous `"react-native"` root field. The priority order for how this was previously resolved was determined by projects, [which created ambiguity when using React Native for Web](https://github.com/expo/router/issues/37#issuecomment-1275925758). Under `"exports"`, _packages concretely define the resolution order for conditional entry points_ — removing this ambiguity.

```json
  "exports": {
    "browser": "./dist/index-browser.js",
    "react-native": "./dist/index-react-native.js",
    "default": "./dist/index.js"
  }
```

:::note
We chose not to introduce `"android"` and `"ios"` conditions, due to the prevalence of other existing platform selection methods, and the complexity of how this behaviour might work across frameworks. Please use the [`Platform.select()`](/docs/platform#select) API instead.
:::

## The future: Stable `"exports"`, enabled by default

In the next React Native release, we are aiming to remove the `unstable_` prefix for this feature (having addressed planned performance work and any bugs) and will enable Package Exports resolution by default.

With `"exports"` enabled for everyone, we can begin taking the React Native community forward — for example, React Native's core packages could be updated to better separate public and internal modules.

![Rollout plan for Package Exports support](../static/blog/assets/package-exports-rollout.png)

## Thanks

Thanks to members of the React Native community that gave feedback on the RFC: [@SimenB](https://github.com/SimenB), [@tido64](https://github.com/tido64), [@byCedric](https://github.com/byCedric), [@thymikee](https://github.com/thymikee).

Huge thanks to [@motiz88](https://github.com/motiz88) and [@robhogan](https://github.com/robhogan) at Meta for supporting the development of this feature.