---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無懈可擊的軟體——畢竟我們至今仍未發明出完全無法破解的鎖（銀行金庫仍然會被攻破）。但遭受惡意攻擊或暴露安全漏洞的機率，與你願意投入多少心力來保護應用程式成反比。即使普通掛鎖能被撬開，它仍比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助強化應用程式安全的工具。這不是一份預檢清單，而是一系列選項的目錄，每個選項都能進一步保護你的應用程式與使用者。

## 儲存敏感資訊

切勿將敏感API金鑰儲存在應用程式碼中。任何包含在程式碼中的內容，都可能被檢查應用套件的人以明文形式存取。雖然像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這類工具很適合用於添加環境特定變數（如API端點），但它們不應與常包含密鑰和API金鑰的伺服器端環境變數混淆。

若必須使用API金鑰或密鑰從應用程式存取資源，最安全的處理方式是建立應用程式與資源之間的協調層。這可以是無伺服器函數（例如使用AWS Lambda或Google Cloud Functions），能轉發帶有所需API金鑰或密鑰的請求。伺服器端程式碼中的密鑰無法像應用程式碼中的密鑰那樣被API消費者存取。

**對於需持久化的使用者資料，請根據其敏感性選擇適當的儲存類型。** 隨著應用程式的使用，你會經常需要將資料儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是儲存使用者的存取權杖以便在會話間無需重新驗證。

> **持久化 vs 非持久化** — 持久化資料會被寫入裝置的磁碟，讓資料能在應用程式重新啟動時被讀取，無需再次發送網路請求或要求使用者重新輸入。但這也使得資料更容易被攻擊者存取。非持久化資料則永遠不會寫入磁碟——因此沒有資料可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native社群維護的模組，提供非同步、未加密的鍵值儲存。Async Storage不會在應用程式間共享：每個應用程式都有自己的沙箱環境，無法存取其他應用程式的資料。

| **Do** use async storage when...              | **Don't** use async storage for... |
| --------------------------------------------- | ---------------------------------- |
| Persisting non-sensitive data across app runs | Token storage                      |
| Persisting Redux state                        | Secrets                            |
| Persisting GraphQL state                      |                                    |
| Storing global app-wide variables             |                                    |

#### 開發者注意事項

<Tabs groupId="guide" queryString defaultValue="web" values={constants.getDevNotesTabs(["web"])}>

<TabItem value="web">

> Async Storage is the React Native equivalent of Local Storage from the web

</TabItem>
</Tabs>

### 安全儲存

React Native並未內建任何儲存敏感資料的方式。不過，Android和iOS平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)允許你安全地儲存使用者的少量敏感資訊。這是儲存憑證、權杖、密碼等不應放在Async Storage中的敏感資料的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android上等效的持久化鍵值資料儲存。**共享偏好設定中的資料預設未加密**，但[加密共享偏好設定](https://developer.android.com/topic/security/data)封裝了Android的Shared Preferences類別，會自動加密鍵與值。

#### Android - 金鑰庫

[Android Keystore](https://developer.android.com/training/articles/keystore) 系統可讓您將加密金鑰儲存在容器中，使其更難從裝置中提取。

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式，或使用封裝這些功能的函式庫（需自行承擔風險），並提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意間儲存或暴露敏感資訊的情況。** 這可能意外發生，例如將敏感表單資料儲存在 redux 狀態中，並將整個狀態樹持久化到 Async Storage。或將使用者令牌和個人資訊傳送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

行動應用程式有一個網頁應用程式所沒有的獨特漏洞：**深度連結**。深度連結是一種從外部來源直接將資料傳送到原生應用程式的方式。深度連結的形式類似 `app://`，其中 `app` 是您的應用程式方案，而 `//` 之後的任何內容可用於內部處理請求。

例如，如果您正在開發一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並開啟 id 為 1 的產品詳細頁面。您可以將這些連結視為網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應在其中傳送任何敏感資訊。

深度連結不安全的原因在於沒有集中式的 URL 方案註冊方法。作為應用程式開發者，您可以透過在 iOS 的 [Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) 或在 Android 的 [新增意圖](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

沒有任何機制能阻止惡意應用程式劫持您的深度連結，只需註冊相同的方案即可取得連結中包含的資料。傳送像 `app://products/1` 這樣的內容並無害處，但傳送令牌則會引發安全問題。

當作業系統在開啟連結時有兩個或更多應用程式可選擇時，Android 會向使用者顯示 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，並詢問使用者選擇哪個應用程式來開啟連結。然而在 iOS 上，作業系統會替您做出選擇，因此使用者可能完全不知情。Apple 在後續的 iOS 版本（iOS 11）中已採取措施解決此問題，實施了先到先得的原則，但此漏洞仍可能以其他方式被利用，您可以在[這裡](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)閱讀更多相關資訊。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可在 iOS 中安全地連結到您的應用程式內容。

### OAuth2 與重新導向

OAuth2 認證協定在現今極為流行，被譽為最完整且安全的協定。OpenID Connect 協定也基於此。在 OAuth2 中，使用者需透過第三方進行認證。成功完成後，第三方會重新導向回請求的應用程式，並附上可交換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，此重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但對應用程式而言並非如此，如前所述，沒有集中式的 URL 方案註冊方法！為了解決此安全問題，必須以 PKCE 的形式加入額外的檢查。

[PKCE](https://oauth.net/2/pkce/)，發音為「Pixy」，全名為「Proof of Key Code Exchange」，是 OAuth 2 規範的擴充功能。它透過新增一層安全驗證機制，確保認證請求與令牌交換請求來自同一個客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法，該演算法能為任何大小的文字或檔案產生獨特的「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 具單向性（無法逆向推導原始輸入）

此時您會持有兩個數值：

- **code_verifier**：由客戶端生成的隨機長字串
- **code_challenge**：code_verifier 的 SHA 256 雜湊值

在初始的 `/authorize` 請求中，客戶端會同時發送記憶體中的 `code_verifier` 所對應的 `code_challenge`。當授權請求成功返回後，客戶端需再傳送用於生成 `code_challenge` 的原始 `code_verifier`。身份提供者（IDP）將重新計算 `code_challenge`，核對是否與初始請求設定的值相符，僅在匹配時才會發放存取令牌。

此機制確保只有發起初始授權流程的應用程式能成功兌換驗證碼為 JWT。即使惡意應用取得驗證碼，單獨持有也毫無用處。實作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

針對原生 OAuth 實作，可考慮使用 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth) 函式庫。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 庫，能支援 PKCE 機制。

> 注意：react-native-app-auth 的 PKCE 支援需視您的身份提供者（Identity Provider）是否實作該功能而定。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 應全程採用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 能防止資料在伺服器與客戶端傳輸過程中被以明文竊取，安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

即使使用 https 仍可能遭受攔截攻擊。在標準 https 流程中，客戶端僅信任由預裝可信憑證機構（CA）簽署的伺服器憑證。攻擊者可能透過在用戶裝置安裝惡意根憑證，使客戶端誤認攻擊者簽發的憑證可信，導致傳統憑證驗證仍存在[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險。

**SSL 憑證綁定**技術可防範此類攻擊。其原理是在開發階段將受信任的憑證清單嵌入（綁定）至客戶端，使應用程式僅接受綁定清單內憑證簽署的請求，自動拒絕自簽憑證。

> 實作 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年會過期，屆時需同步更新伺服器與應用內嵌的憑證。若伺服器更新憑證後，仍使用舊憑證的應用將立即失效。

## 總結

雖然沒有萬無一失的方式能確保安全性，但透過有意識的努力與謹慎處理，仍可大幅降低應用程式遭受安全漏洞的風險。請根據應用程式儲存資料的敏感度、用戶數量，以及駭客入侵帳戶可能造成的損害程度，投入相應比例的安全防護資源。切記：那些從未被請求過的資訊，要存取它們的難度自然高出許多。