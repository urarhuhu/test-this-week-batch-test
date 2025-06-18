# 附錄

## 一、術語表

- **Spec（規格）** - 用於描述 Turbo Native 模組或 Fabric Native 元件的 API 之 TypeScript 或 Flow 程式碼。由 **Codegen** 用於生成樣板程式碼。

- **Native Modules（原生模組）** - 不包含使用者介面(UI)的原生函式庫。例如持久化儲存、通知、網路事件等。這些可透過函式與物件形式供 JavaScript 應用程式碼存取。
- **Native Component（原生元件）** - 可透過 React 元件形式供應用程式 JavaScript 碼使用的原生平台視圖。

- **Legacy Native Components（舊版原生元件）** - 運行於舊版 React Native 架構的元件。
- **Legacy Native Modules（舊版原生模組）** - 運行於舊版 React Native 架構的模組。

## 二、Codegen 類型對照

下表列出支援的類型及其在各平台的映射關係，可供參考：

| Flow                                                                       | TypeScript                                          | Flow Nullable Support                                   | TypeScript Nullable Support                          | Android (Java)                       | iOS (ObjC)                                                     |
| -------------------------------------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------- |
| `string`                                                                   | `string`                                            | `?string`                                               | <code>string &#124; null</code>                      | `string`                             | `NSString`                                                     |
| `boolean`                                                                  | `boolean`                                           | `?boolean`                                              | <code>boolean &#124; null</code>                     | `Boolean`                            | `NSNumber`                                                     |
| Object Literal<br /><code>&#123;&#124; foo: string, ...&#124;&#125;</code> | <code>&#123; foo: string, ...&#125; as const</code> | <code>?&#123;&#124; foo: string, ...&#124;&#125;</code> | <code>?&#123; foo: string, ...&#125; as const</code> | \-                                   | \-                                                             |
| Object [[1](#notes)]                                                       | Object [[1](#notes)]                                | `?Object`                                               | <code>Object &#124; null</code>                      | `ReadableMap`                        | `@` (untyped dictionary)                                       |
| <code>Array&lt;T&gt;</code>                                                | <code>Array&lt;T&gt;</code>                         | <code>?Array&lt;T&gt;</code>                            | <code>Array&lt;T&gt; &#124; null</code>              | `ReadableArray`                      | `NSArray` (or `RCTConvertVecToArray` when used inside objects) |
| `Function`                                                                 | `Function`                                          | `?Function`                                             | <code>Function &#124; null</code>                    | \-                                   | \-                                                             |
| <code>Promise&lt;T&gt;</code>                                              | <code>Promise&lt;T&gt;</code>                       | <code>?Promise&lt;T&gt;</code>                          | <code>Promise&lt;T&gt; &#124; null</code>            | `com.facebook.react.bridge.Promise`  | `RCTPromiseResolve` and `RCTPromiseRejectBlock`                |
| Type Unions<br /><code>'SUCCESS'&#124;'FAIL'</code>                        | Type Unions<br /><code>'SUCCESS'&#124;'FAIL'</code> | Only as callbacks                                       |                                                      | \-                                   | \-                                                             |
| Callbacks<br />`() =>`                                                     | Callbacks<br />`() =>`                              | Yes                                                     |                                                      | `com.facebook.react.bridge.Callback` | `RCTResponseSenderBlock`                                       |
| `number`                                                                   | `number`                                            | No                                                      |                                                      | `double`                             | `NSNumber`                                                     |

### 注意事項：

<b>[1]</b> 強烈建議使用物件字面量(Object literals)而非物件(Objects)。

:::info
您亦可參考 React Native 核心模組的 JavaScript 規格文件，這些檔案位於 React Native 代碼庫的 `Libraries/` 目錄下。
:::