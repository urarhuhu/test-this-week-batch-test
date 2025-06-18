---
id: publishing-to-app-store
title: Publishing to Apple App Store
---

發布流程與其他原生 iOS 應用程式相同，但需注意一些額外事項。

:::info
若使用 Expo，請參閱 Expo 的 [部署至應用商店指南](https://docs.expo.dev/distribution/app-stores/) 來建置並提交應用至 Apple App Store。本指南適用於任何 React Native 應用程式，可自動化部署流程。
:::

### 1. 配置發布方案

為 App Store 分發建置應用需在 Xcode 中使用 `Release` 方案。`Release` 建置的應用會自動停用開發者選單，避免使用者在生產環境誤觸，同時會將 JavaScript 程式碼本地打包，使裝置離線時仍能測試。

配置 `Release` 方案：前往 **Product** → **Scheme** → **Edit Scheme**，在側邊欄選擇 **Run** 標籤，並將 Build Configuration 下拉選單設為 `Release`。

![](/docs/assets/ConfigureReleaseScheme.png)

#### 專業建議

當應用套件體積增大時，啟動時可能出現閃白屏（介於啟動畫面與根視圖之間）。若發生此情況，可在 `AppDelegate.m` 中加入以下程式碼，使啟動畫面持續顯示至過渡完成。

```objectivec
  // Place this code after "[self.window makeKeyAndVisible]" and before "return YES;"
  UIStoryboard *sb = [UIStoryboard storyboardWithName:@"LaunchScreen" bundle:nil];
  UIViewController *vc = [sb instantiateInitialViewController];
  rootView.loadingView = vc.view;
```

即使處於 Debug 模式，每次針對實體裝置建置時都會生成靜態套件。若需節省時間，可在 Xcode Build Phase 的 `Bundle React Native code and images` 腳本中加入以下內容來停用 Debug 模式的套件生成：

```shell
 if [ "${CONFIGURATION}" == "Debug" ]; then
  export SKIP_BUNDLING=true
 fi
```

### 2. 建置發布版應用

按下 <kbd>Cmd ⌘</kbd> + <kbd>B</kbd> 或從選單選擇 **Product** → **Build** 即可建置發布版應用。完成後可將應用分發給測試人員或提交至 App Store。

:::info
亦可使用 `React Native CLI` 執行此操作：在專案根目錄執行 `npm run ios -- --mode="Release"` 或 `yarn ios --mode Release`。
:::

測試完成後，請依以下步驟提交至 App Store：

- 開啟終端機，進入應用程式的 iOS 資料夾並輸入 `open .`
- 雙擊 YOUR_APP_NAME.xcworkspace 以在 Xcode 中開啟
- 點選 `Product` → `Archive`，確保裝置設為 "Any iOS Device (arm64)"

:::note
請確認 Bundle Identifier 與 Apple Developer Dashboard 中建立的 Identifiers 完全一致。
:::

- 封存完成後，在封存視窗點選 `Distribute App`
- 選擇 `App Store Connect`（若需發布至 App Store）
- 點選 `Upload` → 勾選所有選項 → `Next`
- 依需求選擇 `Automatically manage signing` 或 `Manually manage signing`
- 點選 `Upload`
- 完成後可在 App Store Connect 的 TestFlight 中找到該建置版本

填寫必要資訊後，在 Build 區段選擇應用建置版本，點選 `Save` → `Submit For Review` 提交審核。