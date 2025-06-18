---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無法攻破的軟體——畢竟我們至今仍未發明完全無法破解的鎖（銀行的保險庫終究還是會被闖入）。但遭受惡意攻擊或因安全漏洞而暴露的機率，與你願意投入多少努力來保護應用程式免受此類事件影響成反比。普通的掛鎖雖然能被撬開，但比起櫥櫃掛鉤仍然難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助你強化應用程式安全的工具。這不是一份預檢清單——而是選項目錄，每個選項都能進一步保護你的應用程式與使用者。

## 儲存敏感資訊

切勿在應用程式代碼中儲存敏感的 API 金鑰。任何包含在代碼中的內容都可能被檢查應用套件的人以純文字形式存取。像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這樣的工具非常適合添加環境特定的變數（如 API 端點），但它們不應與伺服器端的環境變數混淆，後者通常包含密鑰和 API 金鑰。

如果必須在應用程式中使用 API 金鑰或密鑰來存取某些資源，最安全的處理方式是構建一個位於應用程式和資源之間的協調層。這可以是無伺服器函數（例如使用 AWS Lambda 或 Google Cloud Functions），它可以轉發帶有所需 API 金鑰或密鑰的請求。伺服器端代碼中的密鑰無法像應用程式代碼中的密鑰那樣被 API 消費者存取。

**對於持久化的使用者數據，根據其敏感性選擇合適的儲存類型。** 隨著應用程式的使用，你會經常發現需要在設備上保存數據，無論是為了支持離線使用、減少網路請求，還是在會話之間保存使用者的存取令牌，這樣他們就不必每次使用應用程式時重新驗證。

> **持久化 vs 非持久化** — 持久化數據會被寫入設備的磁碟，這使得數據可以在應用程式多次啟動之間被讀取，而無需再次發送網路請求或要求使用者重新輸入。但這也可能使數據更容易被攻擊者存取。非持久化數據永遠不會寫入磁碟——因此沒有數據可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是 React Native 的一個社區維護模組，提供了一個非同步、未加密的鍵值儲存。Async Storage 不會在應用程式之間共享：每個應用程式都有自己的沙盒環境，無法存取其他應用程式的數據。

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

React Native 並未內建任何儲存敏感數據的方式。然而，Android 和 iOS 平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 允許你安全地儲存使用者的少量敏感資訊。這是儲存證書、令牌、密碼以及任何其他不應放在 Async Storage 中的敏感資訊的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 上相當於持久化鍵值數據儲存的機制。**共享偏好設定中的數據預設未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 為 Android 封裝了共享偏好設定類，並自動加密鍵和值。

#### Android - 金鑰庫

[Android Keystore](https://developer.android.com/training/articles/keystore) 系統可讓您將加密金鑰儲存在容器中，使其更難以從裝置中提取。

若要使用 iOS Keychain 服務或 Android 安全共享偏好設定，您可以自行編寫橋接程式碼，或使用封裝這些功能並提供統一 API 的函式庫（但需自行承擔風險）。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - 在 iOS 上使用 Keychain，在 Android 上使用 EncryptedSharedPreferences。
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - 在 iOS 上是安全的，但在 Android 上使用 Shared Preferences（預設不安全）。不過有一個[分支](https://github.com/mCodex/react-native-sensitive-info/tree/keystore)使用 Android Keystore。
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - 為 Redux 封裝 react-native-sensitive-info。

> **注意無意中儲存或暴露敏感資訊的情況。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個狀態樹持久化到 Async Storage。或將用戶令牌和個人資訊發送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

行動應用程式有一個在網頁上不存在的獨特漏洞：**深度連結**。深度連結是一種從外部來源直接向原生應用程式發送資料的方式。深度連結的形式類似 `app://`，其中 `app` 是您的應用程式方案，而 `//` 後面的內容可用於內部處理請求。

例如，如果您正在構建一個電子商務應用程式，可以使用 `app://products/1` 來深度連結到您的應用程式，並打開 id 為 1 的產品詳細頁面。您可以將其視為類似於網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您不應在其中發送任何敏感資訊。

深度連結不安全的原因是沒有集中註冊 URL 方案的方法。作為應用程式開發者，您可以通過在 iOS 的 [Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) 或在 Android 的 [添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL 方案。

沒有任何機制可以阻止惡意應用程式通過註冊相同的方案來劫持您的深度連結，從而獲取連結中包含的資料。發送像 `app://products/1` 這樣的內容無害，但發送令牌則是一個安全問題。

當操作系統在打開連結時有兩個或更多應用程式可供選擇時，Android 會向用戶顯示一個 [歧義對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，並詢問他們選擇使用哪個應用程式來打開連結。然而在 iOS 上，操作系統會為您做出選擇，因此用戶將毫不知情。Apple 在後續的 iOS 版本（iOS 11）中採取了措施來解決這個問題，實施了先到先得的原則，儘管這種漏洞仍可能以其他方式被利用，您可以在此[閱讀更多](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可以在 iOS 中安全地連結到應用程式內的內容。

### OAuth2 與重定向

OAuth2 認證協議在當今極為流行，被譽為最完善且安全的協議。OpenID Connect 協議也是基於此。在 OAuth2 中，用戶需透過第三方進行身份驗證。成功完成後，該第三方會重定向回請求應用程式，並附帶一個可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網絡上安全傳輸資訊的開放標準。

在網絡上，此重定向步驟是安全的，因為網址具有唯一性保證。但對應用程式而言並非如此，如前所述，並無集中式方法來註冊網址方案！為解決此安全隱患，必須額外加入 PKCE 形式的檢查機制。

[PKCE](https://oauth.net/2/pkce/)（讀作「Pixy」）全稱為 Proof of Key Code Exchange，是 OAuth 2 規範的擴展。它透過增加一層安全驗證，確保認證與權杖交換請求來自同一客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 能為任意大小的文字或檔案產生獨特「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向不可逆（無法逆向推導原始輸入）

此時您將持有兩個數值：

- **code_verifier** - 由客戶端生成的大型隨機字串
- **code_challenge** - code_verifier 的 SHA 256 雜湊值

在初始的 `/authorize` 請求中，客戶端會同時發送記憶體中的 `code_verifier` 所對應的 `code_challenge`。當授權請求正確返回後，客戶端再發送用於生成 `code_challenge` 的原始 `code_verifier`。身份提供者（IDP）將重新計算 `code_challenge`，核對是否與初始 `/authorize` 請求設定的值相符，僅在匹配時才會發送存取權杖。

此機制確保只有觸發初始授權流程的應用程式能成功兌換驗證碼為 JWT。即使惡意應用取得驗證碼，單獨持有也毫無用處。實際運作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

若需實作原生 OAuth，可考慮使用 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth) 函式庫。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，能與 OAuth2 供應商溝通並支援 PKCE。

> 僅當您的身份供應商支援時，react-native-app-auth 方能實作 PKCE。

![OAuth2 搭配 PKCE](/docs/assets/diagram_pkce.svg)

## 網路安全

您的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在伺服器傳送至客戶端過程中被以明文竊取。安全端點可透過 `https://` 開頭（而非 `http://`）識別。

### SSL 憑證綁定

即使使用 https 端點，資料仍可能遭攔截。在 https 架構下，客戶端僅信任由預裝可信憑證授權機構簽署的有效憑證。攻擊者可藉由在用戶裝置安裝惡意根憑證，使客戶端誤信攻擊者簽署的所有憑證。因此，單純依賴憑證仍可能使您暴露於[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險中。

**SSL 憑證綁定**是一種可在客戶端使用的技術，用以防範此類攻擊。其原理是在開發階段將一組受信任的憑證清單嵌入（或「綁定」）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證綁定時，需注意憑證的有效期限。憑證通常每 1-2 年會過期，屆時不僅伺服器端需更新憑證，應用程式內嵌的憑證也必須同步更新。一旦伺服器憑證更新後，任何仍內嵌舊憑證的應用程式將立即無法運作。

## 總結

沒有任何方法能絕對保證安全，但透過有意識的努力與嚴謹措施，可大幅降低應用程式遭受安全漏洞攻擊的可能性。請根據應用程式儲存資料的敏感性、用戶數量及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：根本未請求的資訊，駭客要取得自然困難得多。