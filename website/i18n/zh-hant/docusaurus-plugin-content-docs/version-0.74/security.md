---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無法攻破的軟體（畢竟銀行金庫仍會被闖入），但遭受惡意攻擊或暴露於安全漏洞的機率，與你願意投入多少心力來保護應用程式成反比。即使普通掛鎖能被撬開，它仍比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及協助強化應用程式安全的工具。這並非起飛前檢查清單，而是一系列選項的目錄，每項都能為你的應用程式和用戶提供更進一步的保護。

## 儲存敏感資訊

切勿將敏感 API 金鑰儲存在應用程式代碼中。任何包含在代碼中的內容，都可能被檢查應用程式套件的人以純文字形式存取。雖然像 [react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv) 和 [react-native-config](https://github.com/luggit/react-native-config/) 這類工具很適合用來添加環境變數（如 API 端點），但它們不應與伺服器端的環境變數混淆，後者常包含密鑰和 API 金鑰。

若必須在應用程式中使用 API 金鑰或密鑰來存取資源，最安全的做法是在應用程式與資源間建立協調層。這可以是無伺服器函數（例如使用 AWS Lambda 或 Google Cloud Functions），它能轉發請求並附上所需的 API 金鑰或密鑰。伺服器端代碼中的密鑰無法像應用程式代碼中的密鑰那樣被 API 使用者存取。

**對於需持久化的用戶數據，請根據其敏感性選擇適當的儲存類型。** 隨著應用程式的使用，你會經常需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是保存用戶的存取權杖以避免每次使用時重新驗證。

> **持久化 vs 非持久化** — 持久化數據會被寫入裝置的磁碟，讓應用程式能在不同啟動間讀取數據，無需再次發送網路請求或要求用戶重新輸入。但這也使得數據更容易被攻擊者存取。非持久化數據則從不會寫入磁碟——因此根本沒有數據可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage) 是 React Native 的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage 不會在應用程式間共享：每個應用程式都有自己的沙箱環境，無法存取其他應用程式的數據。

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

React Native 並未內建任何儲存敏感數據的方式。不過，Android 和 iOS 平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services) 可讓你安全地儲存用戶的小塊敏感資訊。這是儲存憑證、權杖、密碼等不應放入 Async Storage 之敏感數據的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences) 是 Android 上等效的持久化鍵值數據儲存。**共享偏好設定中的數據預設未加密**，但 [加密共享偏好設定](https://developer.android.com/topic/security/data) 封裝了 Android 的 SharedPreferences 類別，會自動加密鍵與值。

#### Android - 金鑰庫

[Android Keystore](https://developer.android.com/training/articles/keystore) 系統可讓您將加密金鑰儲存在容器中，使其更難從裝置中提取。

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或選擇使用封裝這些功能的函式庫（但需自行承擔風險）。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意避免無意間儲存或暴露敏感資訊。** 這種情況可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個狀態樹持久化到 Async Storage，或將使用者令牌和個人資訊傳送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 認證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

行動應用程式具有一個網頁環境中不存在的獨特漏洞：**深度連結**。深度連結是一種從外部來源直接將資料傳送到原生應用程式的方式。深度連結的形式為 `app://`，其中 `app` 是您的應用程式 scheme，而 `//` 後面的內容可用於內部處理請求。

舉例來說，如果您正在開發一個電子商務應用程式，可以使用 `app://products/1` 來深度連結到您的應用程式，並開啟 ID 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕對不應該在其中傳送任何敏感資訊。

深度連結不安全的理由是：沒有集中式的 URL scheme 註冊方法。作為應用程式開發者，您可以透過 [在 Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app)（iOS）或 [在 Android 上添加 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL scheme。

惡意應用程式可以透過註冊相同的 scheme 來劫持您的深度連結，從而獲取連結中包含的資料。傳送像 `app://products/1` 這樣的連結無害，但傳送令牌則會造成安全隱患。

當作業系統需要選擇開啟連結的應用程式時，Android 會顯示 [歧義解析對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) 讓使用者選擇，而 iOS 則會自行決定（使用者完全不知情）。蘋果在後續 iOS 版本（iOS 11）中採取了「先到先得」原則來解決此問題，但此漏洞仍可能透過其他方式被利用（詳見[此處](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)）。使用 [通用連結](https://developer.apple.com/ios/universal-links/) 可在 iOS 中安全地連結至應用程式內的內容。

### OAuth2 與重新導向

OAuth2 認證協議現今極為流行，被譽為最完整且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，使用者需透過第三方進行認證。成功完成後，第三方會重新導回請求的應用程式，並附上可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是用於在網路各方之間安全傳輸資訊的開放標準。

在網頁環境中，此重新導向步驟是安全的，因為網頁 URL 保證唯一。但這不適用於應用程式，如前所述，沒有集中式的 URL scheme 註冊方法！為解決此安全問題，必須透過 PKCE 添加額外的檢查層。

[PKCE](https://oauth.net/2/pkce/), pronounced “Pixy” stands for Proof of Key Code Exchange, and is an extension to the OAuth 2 spec. This involves adding an additional layer of security which verifies that the authentication and token exchange requests come from the same client. PKCE uses the [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) Cryptographic Hash Algorithm. SHA 256 creates a unique “signature” for a text or file of any size, but it is:

- Always the same length regardless of the input file
- Guaranteed to always produce the same result for the same input
- One way (that is, you can’t reverse engineer it to reveal the original input)

Now you have two values:

- **code_verifier** - a large random string generated by the client
- **code_challenge** - the SHA 256 of the code_verifier

During the initial `/authorize` request, the client also sends the `code_challenge` for the `code_verifier` it keeps in memory. After the authorize request has returned correctly, the client also sends the `code_verifier` that was used to generate the `code_challenge`. The IDP will then calculate the `code_challenge`, see if it matches what was set on the very first `/authorize` request, and only send the access token if the values match.

This guarantees that only the application that triggered the initial authorization flow would be able to successfully exchange the verification code for a JWT. So even if a malicious application gets access to the verification code, it will be useless on its own. To see this in action, check out [this example](https://aaronparecki.com/oauth-2-simplified/#mobile-apps).

A library to consider for native OAuth is [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth). React-native-app-auth is an SDK for communicating with OAuth2 providers. It wraps the native [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) and [AppAuth-Android](https://github.com/openid/AppAuth-Android) libraries and can support PKCE.

> React-native-app-auth can support PKCE only if your Identity Provider supports it.

![OAuth2 with PKCE](/docs/assets/diagram_pkce.svg)

## Network Security

Your APIs should always use [SSL encryption](https://www.ssl.com/faqs/faq-what-is-ssl/). SSL encryption protects against the requested data being read in plain text between when it leaves the server and before it reaches the client. You’ll know the endpoint is secure, because it starts with `https://` instead of `http://`.

### SSL Pinning

Using https endpoints could still leave your data vulnerable to interception. With https, the client will only trust the server if it can provide a valid certificate that is signed by a trusted Certificate Authority that is pre-installed on the client. An attacker could take advantage of this by installing a malicious root CA certificate to the user’s device, so the client would trust all certificates that are signed by the attacker. Thus, relying on certificates alone could still leave you vulnerable to a [man-in-the-middle attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack).

**SSL pinning** is a technique that can be used on the client side to avoid this attack. It works by embedding (or pinning) a list of trusted certificates to the client during development, so that only the requests signed with one of the trusted certificates will be accepted, and any self-signed certificates will not be.

> When using SSL pinning, you should be mindful of certificate expiry. Certificates expire every 1-2 years and when one does, it’ll need to be updated in the app as well as on the server. As soon as the certificate on the server has been updated, any apps with the old certificate embedded in them will cease to work.

## Summary

雖然沒有萬無一失的方式能確保安全性，但透過有意識的努力與謹慎，仍可大幅降低應用程式遭受安全漏洞的風險。請根據應用程式儲存資料的敏感度、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：那些從未被請求過的資訊，要存取它們的難度會高上許多。