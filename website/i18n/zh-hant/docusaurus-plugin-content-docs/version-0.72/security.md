---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往容易被忽視。雖然確實無法打造完全無懈可擊的軟體（畢竟連銀行金庫都可能被攻破），但遭受惡意攻擊或暴露安全漏洞的機率，與你投入應用程式防護的努力成反比。就像普通掛鎖雖能被撬開，但比起櫃鉤仍難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及強化應用程式安全的工具。這不是一份檢查清單，而是各種選項的目錄，每個選項都能為你的應用程式和用戶提供更多保護。

## 儲存敏感資訊

切勿在應用程式代碼中儲存敏感的API金鑰。任何包含在代碼中的內容，都可能被檢查應用套件的人以明文形式存取。雖然像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這類工具很適合用於添加環境變數（如API端點），但它們與伺服器端的環境變數不同，後者常包含機密資訊和API金鑰。

若必須在應用程式中使用API金鑰或密鑰來存取資源，最安全的做法是在應用程式與資源之間建立協調層。這可以是無伺服器函數（如使用AWS Lambda或Google Cloud Functions），它能轉發請求並附加所需的API金鑰或密鑰。伺服器端代碼中的機密資訊無法像應用程式代碼中的機密那樣被API使用者存取。

**針對需持久化的用戶數據，請根據其敏感程度選擇合適的儲存方式。** 隨著應用程式的使用，你常需要將數據儲存在裝置上，無論是為了支援離線使用、減少網路請求，或是保存用戶的存取權杖以避免每次使用時重新驗證。

> **持久化 vs 非持久化** — 持久化數據會寫入裝置的磁碟，讓應用程式能在多次啟動間讀取數據，無需再次網路請求或用戶重新輸入。但這也使數據更容易被攻擊者存取。非持久化數據則永不寫入磁碟，因此無數據可被存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是React Native的社群維護模組，提供非同步、未加密的鍵值儲存。Async Storage不跨應用程式共享：每個應用程式都有自己的沙盒環境，無法存取其他應用程式的數據。

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

React Native並未內建儲存敏感數據的方式，但Android和iOS平台已有現成解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)可安全儲存用戶的小型敏感資訊，是存放憑證、權杖、密碼等不適合放在Async Storage之數據的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android的持久化鍵值數據儲存方案。**預設情況下，共享偏好設定中的數據未加密**，但[加密共享偏好設定](https://developer.android.com/topic/security/data)封裝了該類別，會自動加密鍵與值。

#### Android - 金鑰庫

The [Android Keystore](https://developer.android.com/training/articles/keystore) system lets you store cryptographic keys in a container to make it more difficult to extract from the device.

In order to use iOS Keychain services or Android Secure Shared Preferences, you can either write a bridge yourself or use a library which wraps them for you and provides a unified API at your own risk. Some libraries to consider:

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-encrypted-storage](https://github.com/emeraldsanto/react-native-encrypted-storage) - uses Keychain on iOS and EncryptedSharedPreferences on Android.
- [react-native-keychain](https://github.com/oblador/react-native-keychain)
- [react-native-sensitive-info](https://github.com/mCodex/react-native-sensitive-info) - secure for iOS, but uses Android Shared Preferences for Android (which is not secure by default). There is however a [branch](https://github.com/mCodex/react-native-sensitive-info/tree/keystore) that uses Android Keystore.
  - [redux-persist-sensitive-storage](https://github.com/CodingZeal/redux-persist-sensitive-storage) - wraps react-native-sensitive-info for Redux.

> **Be mindful of unintentionally storing or exposing sensitive info.** This could happen accidentally, for example saving sensitive form data in redux state and persisting the whole state tree in Async Storage. Or sending user tokens and personal info to an application monitoring service such as Sentry or Crashlytics.

## Authentication and Deep Linking

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

Mobile apps have a unique vulnerability that is non-existent in the web: **deep linking**. Deep linking is a way of sending data directly to a native application from an outside source. A deep link looks like `app://` where `app` is your app scheme and anything following the // could be used internally to handle the request.

For example, if you were building an ecommerce app, you could use `app://products/1` to deep link to your app and open the product detail page for a product with id 1. You can think of these kind of like URLs on the web, but with one crucial distinction:

Deep links are not secure and you should never send any sensitive information in them.

The reason deep links are not secure is because there is no centralized method of registering URL schemes. As an application developer, you can use almost any url scheme you choose by [configuring it in Xcode](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) for iOS or [adding an intent on Android](https://developer.android.com/training/app-links/deep-linking).

There is nothing stopping a malicious application from hijacking your deep link by also registering to the same scheme and then obtaining access to the data your link contains. Sending something like `app://products/1` is not harmful, but sending tokens is a security concern.

When the operating system has two or more applications to choose from when opening a link, Android will show the user a [Disambiguation dialog](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog) and ask them to choose which application to use to open the link. On iOS however, the operating system will make the choice for you, so the user will be blissfully unaware. Apple has made steps to address this issue in later iOS versions (iOS 11) where they instituted a first-come-first-served principle, although this vulnerability could still be exploited in different ways which you can read more about [here](https://thehackernews.com/2019/07/ios-custom-url-scheme.html). Using [universal links](https://developer.apple.com/ios/universal-links/) will allow linking to content within your app securely in iOS.

### OAuth2 and Redirects

OAuth2 認證協議在當今極為流行，被譽為最完善且安全的協議。OpenID Connect 協議也基於此。在 OAuth2 中，使用者需透過第三方進行身份驗證。成功完成後，該第三方會重定向回請求應用程式，並附帶一個可兌換為 JWT（[JSON Web Token](https://jwt.io/introduction/)）的驗證碼。JWT 是一種在網路各方之間安全傳輸資訊的開放標準。

在網頁上，此重定向步驟是安全的，因為網頁的 URL 保證是唯一的。但對應用程式而言並非如此，如前所述，並無集中式方法來註冊 URL 方案！為解決此安全疑慮，必須以 PKCE 形式加入額外檢查。

[PKCE](https://oauth.net/2/pkce/)（發音為「Pixy」）代表 Proof of Key Code Exchange，是 OAuth 2 規範的擴充。它透過加入額外的安全層來驗證認證與權杖交換請求是否來自同一客戶端。PKCE 使用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 會為任何大小的文字或檔案建立獨特的「簽章」，但其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向性（無法逆向推導原始輸入）

現在您將擁有兩個數值：

- **code_verifier** - 由客戶端產生的大型隨機字串
- **code_challenge** - code_verifier 的 SHA 256 雜湊值

在初始的 `/authorize` 請求中，客戶端會同時發送記憶體中保存的 code_verifier 所對應的 code_challenge。當授權請求正確返回後，客戶端還會發送用於生成 code_challenge 的 code_verifier。身份提供者（IDP）將計算 code_challenge，確認是否與首次 `/authorize` 請求設定的值相符，僅在匹配時發送存取權杖。

這確保只有觸發初始授權流程的應用程式能成功將驗證碼兌換為 JWT。即使惡意應用取得驗證碼，單獨使用也毫無價值。實際範例可參考[此處](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

原生 OAuth 可考慮使用 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth) 函式庫。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，能支援 PKCE。

> react-native-app-auth 僅在您的身份提供者支援時才能啟用 PKCE 功能。

![OAuth2 搭配 PKCE](/docs/assets/diagram_pkce.svg)

## 網路安全

您的 API 應始終使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 加密能防止資料在離開伺服器至抵達客戶端期間以明文形式被讀取。安全端點的特徵是網址以 `https://` 開頭而非 `http://`。

### SSL 憑證綁定

即使使用 https 端點，資料仍可能遭攔截。在 https 中，客戶端僅在伺服器能提供由可信憑證機構（預先安裝於客戶端）簽署的有效憑證時才會信任它。攻擊者可藉由在使用者裝置安裝惡意根 CA 憑證來利用此機制，使客戶端信任攻擊者簽署的所有憑證。因此，僅依賴憑證仍可能使您暴露於[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險中。

**SSL 憑證固定**是一種可在客戶端使用的技術，用於避免此類攻擊。其原理是在開發階段將一組受信任的憑證嵌入（或固定）至客戶端，使得只有由這些受信任憑證簽署的請求才會被接受，任何自簽憑證都將被拒絕。

> 使用 SSL 憑證固定時，需注意憑證的有效期限。憑證通常每 1-2 年會到期，屆時不僅伺服器端需要更新憑證，應用程式內嵌的憑證也必須同步更新。一旦伺服器憑證更新後，仍使用舊版憑證的應用程式將立即無法運作。

## 總結

沒有任何方法能絕對保證安全性，但透過有意識的努力與嚴謹措施，可大幅降低應用程式遭受安全漏洞攻擊的風險。請根據應用程式儲存的資料敏感度、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：最安全的資料，永遠是那些從未被請求過的資料。