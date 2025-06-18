---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生iOS應用程式相同，但需注意一些額外事項。

:::info
若您使用Expo，請參閱Expo的[部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/)，以構建並提交應用至Apple App Store。本指南適用於任何React Native應用程式，可自動化部署流程。
:::

### 1. 配置發布方案

為App Store分發構建應用需在Xcode中使用`Release`方案。`Release`構建會自動禁用應用內開發者選單，防止用戶在生產環境中誤觸。同時會將JavaScript代碼本地打包，使應用可在未連接電腦的設備上測試。

配置應用使用`Release`方案：前往**Product** → **Scheme** → **Edit Scheme**。在側邊欄選擇**Run**標籤，將Build Configuration下拉選單設為`Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用包體積增大時，可能會在啟動畫面與主應用視圖間出現短暫白屏。若發生此情況，可將以下代碼加入`AppDelegate.m`以延長啟動畫面顯示時間。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

即使處於Debug模式，每次針對實體設備構建時都會生成靜態包。若需節省時間，可在Xcode構建階段`Bundle React Native code and images`的shell腳本中添加以下內容來禁用Debug模式的包生成：

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 2. 構建發布版本

現在可通過按<kbd>Cmd ⌘</kbd> + <kbd>B</kbd>或選擇選單欄**Product** → **Build**來構建發布版本。完成後即可分發應用給測試人員或提交至App Store。

:::info
亦可使用`React Native CLI`執行此操作，透過`--mode`選項設定值為`Release`（例如在項目根目錄執行：`npm run ios -- --mode="Release"`或`yarn ios --mode Release`）。
:::

完成測試準備發布至App Store時，請依循本指南操作。

- 啟動終端機，導航至應用的iOS文件夾並輸入`open .`
- 雙擊YOUR_APP_NAME.xcworkspace，將啟動Xcode
- 點擊`Product` → `Archive`，確保設備設為"Any iOS Device (arm64)"

:::note
請檢查Bundle Identifier是否與Apple開發者後台創建的標識符完全一致。
:::

- 歸檔完成後，在歸檔視窗點擊`Distribute App`
- 選擇`App Store Connect`（若需發布至App Store）
- 點擊`Upload` → 確保所有複選框已勾選 → 點擊`Next`
- 根據需求選擇`自動管理簽名`或`手動管理簽名`
- 點擊`Upload`
- 完成後可在App Store Connect的TestFlight中找到

填寫必要資訊後，在Build區段選擇應用構建版本，點擊`Save` → `Submit For Review`

### 4. 螢幕截圖

Apple商店要求提供最新設備的截圖。具體規格可參考[此處](https://developer.apple.com/help/app-store-connect/reference/screenshot-specifications/)。請注意若已提供某些尺寸的截圖，其他尺寸可能非必填。