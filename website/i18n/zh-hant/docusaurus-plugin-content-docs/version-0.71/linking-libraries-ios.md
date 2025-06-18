---
id: linking-libraries-ios
title: Linking Libraries
---

並非每個應用程式都會使用所有原生功能，而包含支援這些功能的程式碼會影響二進位檔案的大小...但我們仍希望在您需要時能隨時添加這些功能。

基於這個考量，我們將許多這類功能以獨立的靜態函式庫形式提供。

大多數函式庫只需拖曳兩個檔案即可完成連結，有時可能需要第三步驟，但不會超過這個範圍。

:::note
所有與 React Native 一同發布的函式庫都存放在儲存庫根目錄的 `Libraries` 資料夾中。其中一些是純 JavaScript，您只需 `require` 即可使用。
其他函式庫還依賴於某些原生程式碼，在這種情況下，您必須將這些檔案添加到您的應用程式中，否則當您嘗試使用該函式庫時，應用程式會立即拋出錯誤。
:::

## 以下是連結包含原生程式碼的函式庫的幾個步驟

### 自動連結

安裝具有原生依賴的函式庫：

```shell
npm install <library-with-native-dependencies> --save
```

:::info
`--save` 或 `--save-dev` 標記在此步驟中非常重要。React Native 會根據您 `package.json` 檔案中的 `dependencies` 和 `devDependencies` 來連結您的函式庫。
:::

就是這樣！下次建置您的應用程式時，原生程式碼將透過[自動連結](https://github.com/react-native-community/cli/blob/main/docs/autolinking.md)機制完成連結。

### 手動連結

#### 步驟 1

如果函式庫包含原生程式碼，其資料夾內必須有一個 `.xcodeproj` 檔案。將此檔案拖曳到 Xcode 中的專案（通常在 Xcode 的 `Libraries` 群組下）；

![](/docs/assets/AddToLibraries.png)

#### 步驟 2

點擊您的主要專案檔案（代表 `.xcodeproj` 的那個），選擇 `Build Phases`，然後從您正在導入的函式庫的 `Products` 資料夾中拖曳靜態函式庫到 `Link Binary With Libraries`。

![](/docs/assets/AddToBuildPhases.png)

#### 步驟 3

並非每個函式庫都需要此步驟，您需要考慮的是：

_我是否需要在編譯時知道函式庫的內容？_

這意味著，您是在原生端使用此函式庫，還是僅在 JavaScript 中使用？如果僅在 JavaScript 中使用，那麼您已經完成了！

如果您確實需要從原生端呼叫它，那麼我們需要知道函式庫的標頭檔。為此，您需要前往專案的檔案，選擇 `Build Settings` 並搜尋 `Header Search Paths`。在那裡，您應該包含函式庫的路徑。（此文件過去建議使用 `recursive`，但這不再推薦，因為它可能導致細微的建置失敗，尤其是在使用 CocoaPods 時。）

![](/docs/assets/AddToSearchPaths.png)