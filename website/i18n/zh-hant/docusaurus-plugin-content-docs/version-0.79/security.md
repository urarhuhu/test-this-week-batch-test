---
id: security
title: Security
---

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

在構建應用程式時，安全性往往被忽視。雖然確實無法打造完全無法穿透的軟體——畢竟我們至今仍未發明出完全無法破解的鎖（銀行金庫仍然會被攻破）。然而，遭受惡意攻擊或暴露於安全漏洞的機率，與您願意投入保護應用程式的努力成反比。即使普通的掛鎖可以被撬開，它仍然比櫥櫃掛鉤難突破得多！

<img src="/docs/assets/d_security_chart.svg" width={283} alt=" " style={{float: 'right'}} />

本指南將介紹儲存敏感資訊、身份驗證、網路安全的最佳實踐，以及幫助您強化應用程式安全的工具。這不是一份起飛前檢查清單——而是一個選項目錄，每個選項都能進一步保護您的應用程式和用戶。

## 儲存敏感資訊

切勿在應用程式代碼中儲存敏感的API金鑰。任何包含在代碼中的內容，都可能被檢查應用程式套件的人以純文字形式存取。像[react-native-dotenv](https://github.com/goatandsheep/react-native-dotenv)和[react-native-config](https://github.com/luggit/react-native-config/)這樣的工具非常適合添加環境特定的變數，如API端點，但它們不應與伺服器端的環境變數混淆，後者通常包含秘密和API金鑰。

如果必須在應用程式中使用API金鑰或秘密來存取某些資源，最安全的處理方式是構建一個介於應用程式和資源之間的協調層。這可以是一個無伺服器函數（例如使用AWS Lambda或Google Cloud Functions），它可以轉發帶有所需API金鑰或秘密的請求。伺服器端代碼中的秘密無法像應用程式代碼中的秘密那樣被API消費者存取。

**對於持久化的用戶數據，根據其敏感性選擇合適的儲存類型。** 隨著應用程式的使用，您經常會發現需要在設備上保存數據，無論是為了支持應用程式在離線狀態下使用、減少網路請求，還是在會話之間保存用戶的存取令牌，以免他們每次使用應用程式時都需要重新驗證。

> **持久化與非持久化** —— 持久化數據被寫入設備的磁盤，這使得數據可以在應用程式啟動之間被讀取，而無需再次進行網路請求或要求用戶重新輸入。但這也使得數據更容易被攻擊者存取。非持久化數據從未被寫入磁盤——因此沒有數據可供存取！

### Async Storage

[Async Storage](https://github.com/react-native-async-storage/async-storage)是一個由社區維護的React Native模組，提供了一個非同步、未加密的鍵值儲存。Async Storage不在應用程式之間共享：每個應用程式都有自己的沙盒環境，無法存取其他應用程式的數據。

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

React Native並未內建任何儲存敏感數據的方式。然而，Android和iOS平台已有現成的解決方案。

#### iOS - 鑰匙圈服務

[鑰匙圈服務](https://developer.apple.com/documentation/security/keychain_services)允許您安全地儲存用戶的小塊敏感資訊。這是儲存證書、令牌、密碼以及任何不應放在Async Storage中的敏感資訊的理想場所。

#### Android - 安全共享偏好設定

[共享偏好設定](https://developer.android.com/reference/android/content/SharedPreferences)是Android上相當於持久化鍵值數據儲存的機制。**共享偏好設定中的數據預設未加密**，但[加密共享偏好設定](https://developer.android.com/topic/security/data)為Android包裝了共享偏好設定類，並自動加密鍵和值。

#### Android - 金鑰庫

[Android Keystore](https://developer.android.com/training/articles/keystore) 系統可讓您將加密金鑰儲存在容器中，使其更難從裝置中提取。

若要使用 iOS Keychain 服務或 Android Secure Shared Preferences，您可以自行編寫橋接程式碼，或選擇使用封裝這些功能的函式庫（但需自行承擔風險），這些函式庫會提供統一的 API。以下是一些可考慮的函式庫：

- [expo-secure-store](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-keychain](https://github.com/oblador/react-native-keychain)

> **請注意無意間儲存或暴露敏感資訊的情況。** 這可能意外發生，例如將敏感表單資料儲存在 redux state 中，並將整個 state 樹持久化到 Async Storage。或是將使用者 token 和個人資訊傳送到應用程式監控服務（如 Sentry 或 Crashlytics）。

## 身份驗證與深度連結

<img src="/docs/assets/d_security_deep-linking.svg" width={225} alt=" " style={{float: 'right', margin: '0 0 1em 1em'}} />

行動應用程式有一個在網頁上不存在的獨特漏洞：**深度連結**。深度連結是一種從外部來源直接將資料傳送到原生應用程式的方式。深度連結的格式類似 `app://`，其中 `app` 是您的應用程式 scheme，而 `//` 後面的內容可用於內部處理請求。

舉例來說，如果您正在開發一個電子商務應用程式，您可以使用 `app://products/1` 來深度連結到您的應用程式，並開啟 id 為 1 的產品詳細頁面。您可以將這些連結想像成網頁上的 URL，但有一個關鍵區別：

深度連結並不安全，您絕對不應該在其中傳送任何敏感資訊。

深度連結不安全的原因是因為沒有集中式的 URL scheme 註冊方法。作為應用程式開發者，您可以透過在 iOS 的 [Xcode 中配置](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app) 或 Android 的 [新增 intent](https://developer.android.com/training/app-links/deep-linking) 來使用幾乎任何 URL scheme。

沒有任何機制能阻止惡意應用程式透過註冊相同的 scheme 來劫持您的深度連結，並取得連結中包含的資料。傳送像 `app://products/1` 這樣的內容並無害處，但傳送 token 則會引發安全疑慮。

當作業系統在開啟連結時有兩個或更多應用程式可選擇時，Android 會向使用者顯示 [選擇器對話框](https://developer.android.com/training/basics/intents/sending#disambiguation-dialog)，詢問他們要使用哪個應用程式來開啟連結。然而在 iOS 上，作業系統會替使用者做出選擇，因此使用者可能完全不知情。Apple 已在後續的 iOS 版本（iOS 11）中採取措施解決此問題，實施了先到先得的原則，但此漏洞仍可能以其他方式被利用，您可以在[這裡](https://thehackernews.com/2019/07/ios-custom-url-scheme.html)閱讀更多相關資訊。使用 [universal links](https://developer.apple.com/ios/universal-links/) 可在 iOS 中安全地連結到您應用程式內的內容。

### OAuth2 與重新導向

OAuth2 身份驗證協議現今非常流行，被譽為最完整且安全的協議。OpenID Connect 協議也是基於此協議。在 OAuth2 中，使用者會被要求透過第三方進行身份驗證。成功完成後，該第三方會重新導向回請求的應用程式，並附帶一個驗證碼，該驗證碼可兌換為 JWT — [JSON Web Token](https://jwt.io/introduction/)。JWT 是一種在網路上安全傳輸資訊的開放標準。

在網頁上，這個重新導向步驟是安全的，因為網頁上的 URL 保證是唯一的。但這對應用程式來說並不成立，因為如前所述，沒有集中式的 URL scheme 註冊方法！為了解決這個安全問題，必須以 PKCE 的形式加入額外的檢查。

[PKCE](https://oauth.net/2/pkce/)，發音為「Pixy」，全名為「Proof of Key Code Exchange」，是 OAuth 2 規範的擴充功能。它透過新增一層安全驗證機制，確保驗證請求與令牌交換請求來自同一個客戶端。PKCE 採用 [SHA 256](https://www.movable-type.co.uk/scripts/sha256.html) 加密雜湊演算法。SHA 256 能為任何大小的文字或檔案產生獨特的「簽章」，其特性如下：

- 無論輸入檔案大小，輸出長度固定
- 相同輸入必定產生相同結果
- 單向不可逆（無法從簽章反推原始輸入）

此時你會得到兩個數值：

- **code_verifier** - 由客戶端產生的大型隨機字串
- **code_challenge** - code_verifier 的 SHA 256 雜湊值

在初始的 `/authorize` 請求中，客戶端會同時發送記憶體中保存的 code_verifier 所對應的 code_challenge。當授權請求成功返回後，客戶端再發送當初用於產生 code_challenge 的 code_verifier。身份提供者（IDP）會重新計算 code_challenge，確認其與初始 `/authorize` 請求中的數值相符後，才會發放存取令牌。

此機制確保只有發起初始授權流程的應用程式能成功用驗證碼兌換 JWT。即使惡意應用取得驗證碼，單獨持有也毫無用處。實際運作範例可參考[此示範](https://aaronparecki.com/oauth-2-simplified/#mobile-apps)。

適用於原生 OAuth 的推薦函式庫是 [react-native-app-auth](https://github.com/FormidableLabs/react-native-app-auth)。該 SDK 封裝了原生 [AppAuth-iOS](https://github.com/openid/AppAuth-iOS) 與 [AppAuth-Android](https://github.com/openid/AppAuth-Android) 函式庫，能與 OAuth2 供應商溝通並支援 PKCE。

> 注意：react-native-app-auth 的 PKCE 支援取決於你的身份供應商是否實作此功能。

![OAuth2 搭配 PKCE 流程圖](/docs/assets/diagram_pkce.svg)

## 網路安全

API 必須全程使用 [SSL 加密](https://www.ssl.com/faqs/faq-what-is-ssl/)。SSL 能防止資料在伺服器與客戶端傳輸過程中被以明文竊取。安全端點的識別特徵是其網址開頭為 `https://` 而非 `http://`。

### SSL 憑證綁定

僅使用 https 端點仍可能使資料遭攔截。在 https 架構下，客戶端僅信任由預裝可信憑證機構（CA）簽署的伺服器憑證。攻擊者可能透過在用戶裝置安裝惡意根 CA 憑證，使客戶端誤認攻擊者簽發的憑證為可信憑證，導致傳統憑證驗證機制仍存在[中間人攻擊](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)風險。

**SSL 憑證綁定**技術可防範此類攻擊。其原理是在開發階段將受信任的憑證清單嵌入（綁定）至客戶端，使應用程式僅接受由預存憑證簽署的請求，拒絕任何自簽憑證。

> 實作 SSL 綁定時需注意憑證有效期。憑證通常每 1-2 年會過期，屆時需同步更新伺服器與應用程式內的憑證。若伺服器更新憑證後，用戶端應用仍使用舊版綁定憑證，將導致連線失敗。

## 總結

雖然沒有萬無一失的安全處理方式，但透過有意識的努力與謹慎，仍能大幅降低應用程式遭受安全漏洞的風險。請根據應用程式儲存資料的敏感性、用戶數量，以及駭客入侵可能造成的損害程度，投入相應的安全防護資源。切記：那些從未被請求過的資訊，往往最難被竊取。