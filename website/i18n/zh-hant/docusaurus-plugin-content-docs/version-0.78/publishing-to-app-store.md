---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生iOS應用程式相同，但需注意一些額外考量事項。

:::info
若您使用Expo，請參閱Expo的[部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/)，以建置並提交應用程式至Apple App Store。本指南適用於任何React Native應用程式，可自動化部署流程。
:::

### 1. 設定發布方案

為App Store分發建置應用程式需在Xcode中使用`Release`方案。以`Release`建置的應用程式會自動停用應用內開發者選單，避免使用者在生產環境中誤觸。同時會將JavaScript程式碼本地打包，使您能在未連接電腦時於裝置上測試應用程式。

要設定應用程式使用`Release`方案建置，請前往**Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇**Run**標籤，然後將Build Configuration下拉選單設為`Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用程式套件體積增長時，您可能會在啟動畫面與根應用程式視圖之間看到空白閃屏。若發生此情況，可將以下程式碼加入`AppDelegate.m`以維持啟動畫面顯示過渡期間。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

即使處於Debug模式，每次針對實體裝置建置時都會生成靜態套件。若要節省時間，可在Xcode建置階段「Bundle React Native code and images」的shell腳本中加入以下內容來關閉Debug模式的套件生成：

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 2. 建置發布版應用程式

現在您可以透過按下<kbd>Cmd ⌘</kbd> + <kbd>B</kbd>或從選單欄選擇**Product** → **Build**來建置發布版應用程式。完成發布建置後，即可將應用程式分發給測試人員並提交至App Store。

:::info
您亦可使用`React Native CLI`執行此操作，透過`--mode`選項設定值為`Release`（例如在專案根目錄執行：`npm run ios -- --mode="Release"`或`yarn ios --mode Release`）。
:::

完成測試準備發布至App Store時，請遵循本指南後續步驟。

- 開啟終端機，導航至應用程式的iOS資料夾並輸入`open .`
- 雙擊YOUR_APP_NAME.xcworkspace，這將啟動Xcode
- 點擊`Product` → `Archive`，請確保裝置設定為「Any iOS Device (arm64)」

:::note
請檢查您的Bundle Identifier，確認與Apple開發者後台「Identifiers」中建立的完全一致。
:::

- 封存完成後，在封存視窗中點擊`Distribute App`
- 選擇`App Store Connect`（若需發布至App Store）
- 點擊`Upload` → 確保所有核取方塊已勾選 → 點擊`Next`
- 根據需求選擇`Automatically manage signing`或`Manually manage signing`
- 點擊`Upload`
- 現在您可在App Store Connect的TestFlight中找到該建置版本

填寫必要資訊後，在「Build」區段選擇應用程式建置版本，點擊`Save` → `Submit For Review`提交審核。

### 4. 螢幕截圖

Apple商店要求提供最新裝置的螢幕截圖。相關裝置規格參考可查閱[此處](https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications/)。請注意若已提供某些顯示尺寸的截圖，其他尺寸可能非必填項目。