---
title: Using AWS with React Native
author: Richard Threlkeld
authorTitle: Senior Technical Product Manager at AWS Mobile
authorURL: 'https://twitter.com/undef_obj'
authorImageURL: 'https://pbs.twimg.com/profile_images/811239086581227520/APX1eZwF_400x400.jpg'
authorTwitter: undef_obj
tags: [engineering]
---

AWS 在科技產業中以雲端服務供應商聞名，其服務範疇涵蓋運算、儲存、資料庫技術，以及全託管的無伺服器解決方案。AWS Mobile 團隊持續與客戶及 JavaScript 生態系成員密切合作，致力於讓雲端連線的行動與網頁應用程式更安全、可擴展且易於開發部署。我們最初提供了一個[完整入門套件](https://github.com/awslabs/aws-mobile-react-native-starter)，而後續還有更多新進展。

本篇部落格將探討 React 與 React Native 開發者會感興趣的內容：

- [**AWS Amplify**](https://github.com/aws/aws-amplify)：用於雲端服務的 JavaScript 應用程式宣告式函式庫
- [**AWS AppSync**](https://aws.amazon.com/appsync/)：具備離線與即時功能的全託管 GraphQL 服務

## AWS Amplify

使用 Create React Native App 或 Expo 等工具快速建立 React Native 應用程式相當容易，但當您嘗試將應用場景與基礎架構服務匹配時，連接至雲端的過程可能充滿挑戰。例如，您的 React Native 應用程式可能需要上傳照片。這些照片是否需要按使用者進行保護？這可能意味著您需要某種註冊或登入流程。您希望使用自己的使用者目錄，還是透過社群媒體供應商進行驗證？或許您的應用程式還需要在使用者登入後呼叫具有自訂業務邏輯的 API。

為協助 JavaScript 開發者解決這些問題，我們發布了名為 AWS Amplify 的函式庫。其設計將任務劃分為不同「類別」，而非針對 AWS 特定實作。舉例來說，若您希望使用者註冊、登入後上傳私人照片，只需在應用程式中引入 `Auth` 與 `Storage` 類別：

```
import { Auth } from 'aws-amplify';

Auth.signIn(username, password)
    .then(user => console.log(user))
    .catch(err => console.log(err));

Auth.confirmSignIn(user, code)
    .then(data => console.log(data))
    .catch(err => console.log(err));
```

上述程式碼展示了 Amplify 協助處理的常見任務範例，例如透過電子郵件或簡訊使用多重要素驗證（MFA）代碼。目前支援的類別包括：

- **Auth**：提供憑證自動化功能。開箱即用的實作使用 AWS 憑證進行簽署，以及來自 [Amazon Cognito](https://aws.amazon.com/cognito/) 的 OIDC JWT 權杖。常見功能如 MFA 皆受支援。
- **Analytics**：僅需一行程式碼，即可在 [Amazon Pinpoint](https://aws.amazon.com/pinpoint/) 中追蹤已驗證或未驗證使用者。您可依需求擴展自訂指標或屬性。
- **API**：以安全方式與 RESTful API 互動，利用 [AWS Signature Version 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html)。此模組特別適合搭配 [Amazon API Gateway](https://aws.amazon.com/api-gateway/) 的無伺服器架構。
- **Storage**：簡化上傳、下載及列出 [Amazon S3](https://aws.amazon.com/s3/) 內容的操作指令。您還能輕鬆按使用者將資料分組為公開或私有內容。
- **Caching**：跨網頁應用程式與 React Native 的 LRU 快取介面，採用實作特定的持久化機制。
- **i18n and Logging**：提供國際化、本地化功能，以及除錯與日誌記錄能力。

Amplify 的一大優點在於其針對您的程式設計環境，將「最佳實踐」編碼至設計中。例如，我們在與客戶及 React Native 開發者合作時發現，開發期間為快速完成功能而採取的捷徑，可能會直接進入生產環境堆疊。這些做法可能影響擴展性或安全性，並迫使基礎架構重新設計與程式碼重構。

One example of how we help developers avoid this is the [Serverless Reference Architectures with AWS Lambda](https://www.allthingsdistributed.com/2016/06/aws-lambda-serverless-reference-architectures.html). These show you best practices around using Amazon API Gateway and AWS Lambda together when building your backend. This pattern is encoded into the `API` category of Amplify. You can use this pattern to interact with several different REST endpoints, and pass headers all the way through to your Lambda function for custom business logic. We’ve also released an [AWS Mobile CLI](https://docs.aws.amazon.com/aws-mobile/latest/developerguide/react-native-getting-started.html) for bootstrapping new or existing React Native projects with these features. To get started, just install via **npm**, and follow the configuration prompts:

```
npm install --global awsmobile-cli
awsmobile configure
```

Another example of encoded best practices that is specific to the mobile ecosystem is password security. The default `Auth` category implementation leverages Amazon Cognito user pools for user registration and sign-in. This service implements [Secure Remote Password protocol](https://srp.stanford.edu) as a way of protecting users during authentication attempts. If you're inclined to read through the [mathematics of the protocol](https://srp.stanford.edu/ndss.html#SECTION00032200000000000000), you'll notice that you must use a large prime number when calculating the password verifier over a primitive root to generate a Group. In React Native environments, [JIT is disabled](/docs/javascript-environment). This makes BigInteger calculations for security operations such as this less performant. To account for this, we've released native bridges in Android and iOS that you can link inside your project:

```
npm install --save aws-amplify-react-native
react-native link amazon-cognito-identity-js
```

We're also excited to see that the Expo team has included this [in their latest SDK](https://blog.expo.io/expo-sdk-v25-0-0-is-now-available-714d10a8c3f7) so that you can use Amplify without ejecting.

Finally, specific to React Native (and React) development, Amplify contains [higher order components (HOCs)](https://reactjs.org/docs/higher-order-components.html) for easily wrapping functionality, such as for sign-up and sign-in to your app:

```
import Amplify, { withAuthenticator } from 'aws-amplify-react-native';
import aws_exports from './aws-exports';

Amplify.configure(aws_exports);

class App extends React.Component {
...
}

export default withAuthenticator(App);
```

The underlying component is also provided as `<Authenticator />`, which gives you full control to customize the UI. It also gives you some properties around managing the state of the user, such as if they've signed in or are waiting for MFA confirmation, and callbacks that you can fire when state changes.

Similarly, you'll find general React components that you can use for different use cases. You can customize these to your needs, for example, to show all private images from Amazon S3 in the `Storage` module:

```
<S3Album
  level="private"
  path={path}
  filter={(item) => /jpg/i.test(item.path)}/>
```

You can control many of the component features via props, as shown earlier, with public or private storage options. There are even capabilities to automatically gather analytics when users interact with certain UI components:

```
return <S3Album track/>
```

AWS Amplify favors a convention over configuration style of development, with a global initialization routine or initialization at the category level. The quickest way to get started is with an [aws-exports file](https://aws.amazon.com/blogs/mobile/enhanced-javascript-development-with-aws-mobile-hub/). However, developers can also use the library independently with existing resources.

For a deep dive into the philosophy and to see a full demo, check out the video from [AWS re\:Invent](https://www.youtube.com/watch?v=vAjf3lyjf8c).

## AWS AppSync

Shortly after the launch of AWS Amplify, we also released [AWS AppSync](https://aws.amazon.com/appsync/). This is a fully managed GraphQL service that has both offline and real-time capabilities. Although you can use GraphQL in different client programming languages (including native Android and iOS), it's quite popular among React Native developers. This is because the data model fits nicely into a unidirectional data flow and component hierarchy.

AWS AppSync 讓您能夠連接到自己 AWS 帳戶中的資源，意味著您擁有並控制自己的數據。這是通過使用數據源實現的，該服務支援 [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)、[Amazon Elasticsearch](https://aws.amazon.com/elasticsearch-service/) 和 [AWS Lambda](https://aws.amazon.com/lambda/)。這使您能夠在單一 GraphQL API 中結合功能（例如 NoSQL 和全文搜索）作為一個架構。這讓您可以混合搭配不同的數據源。AppSync 服務還可以 [根據架構進行配置](https://docs.aws.amazon.com/appsync/latest/devguide/provision-from-schema.html)，因此如果您不熟悉 AWS 服務，只需編寫 GraphQL SDL，點擊一個按鈕，即可自動啟動並運行。

AWS AppSync 中的實時功能是通過 [基於 GraphQL 訂閱的知名事件模式](https://graphql.org/blog/subscriptions-in-graphql-and-relay/) 來控制的。由於 AWS AppSync 中的訂閱是 [在架構上通過 GraphQL 指令控制的](https://docs.aws.amazon.com/appsync/latest/devguide/real-time-data.html)，而架構可以使用任何數據源，這意味著您可以通過 Amazon DynamoDB 和 Amazon Elasticsearch Service 的數據庫操作觸發通知，或者通過 AWS Lambda 從基礎設施的其他部分觸發通知。

與 AWS Amplify 類似，您可以在 GraphQL API 上使用 [企業級安全功能](https://docs.aws.amazon.com/appsync/latest/devguide/security.html) 與 AWS AppSync。該服務讓您可以快速開始使用 API 密鑰。然而，當您進入生產環境時，可以過渡到使用 AWS Identity and Access Management (IAM) 或來自 Amazon Cognito 用戶池的 OIDC 令牌。您可以在解析器級別通過類型策略控制訪問。您甚至可以使用邏輯檢查來進行 [細粒度訪問控制](https://docs.aws.amazon.com/appsync/latest/devguide/security.html#fine-grained-access-control) 的運行時檢查，例如檢測用戶是否是特定數據庫資源的所有者。還有功能可以檢查群組成員資格以執行解析器或個別數據庫記錄的訪問。

為了幫助 React Native 開發者了解更多關於這些技術的資訊，AWS AppSync 控制台主頁上有一個 [內置的 GraphQL 示例架構](https://docs.aws.amazon.com/appsync/latest/devguide/quickstart.html)，您可以啟動。此示例會部署一個 GraphQL 架構，配置數據庫表，並自動為您連接查詢、變更和訂閱。還有一個可運行的 [AWS AppSync 的 React Native 示例](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react-native)，它利用了這個內置架構（以及一個 [React 示例](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react)），讓您可以在幾分鐘內啟動客戶端和雲端組件。

使用 `AWSAppSyncClient` 開始非常簡單，它可以插入到 [Apollo Client](https://github.com/apollographql/apollo-client) 中。`AWSAppSyncClient` 會處理 GraphQL API 的安全性和簽名、離線功能以及訂閱握手和協商過程：

```
import AWSAppSyncClient from "aws-appsync";
import { Rehydrated } from 'aws-appsync-react';
import { AUTH_TYPE } from "aws-appsync/lib/link/auth-link";

const client = new AWSAppSyncClient({
  url: awsconfig.graphqlEndpoint,
  region: awsconfig.region,
  auth: {type: AUTH_TYPE.API_KEY, apiKey: awsconfig.apiKey}
});
```

AppSync 控制台提供了一個可供下載的配置文件，其中包含您的 GraphQL 端點、AWS 區域和 API 密鑰。然後，您可以將客戶端與 [React Apollo](https://github.com/apollographql/react-apollo) 一起使用：

```
const WithProvider = () => (
  <ApolloProvider client={client}>
      <Rehydrated>
          <App />
      </Rehydrated>
  </ApolloProvider>
);
```

此時，您可以使用標準的 GraphQL 查詢：

```
query ListEvents {
    listEvents{
      items{
        __typename
        id
        name
        where
        when
        description
        comments{
          __typename
          items{
            __typename
            eventId
            commentId
            content
            createdAt
          }
          nextToken
        }
      }
    }
}
```

上面的示例展示了使用 AppSync 配置的示例應用架構進行查詢。它不僅展示了與 DynamoDB 的交互，還包括數據的分頁（包括加密令牌）以及 `Events` 和 `Comments` 之間的類型關係。由於應用配置了 `AWSAppSyncClient`，數據會自動離線持久化，並在設備重新連接時同步。

您可以通過 [這段影片](https://www.youtube.com/watch?v=FtkVlIal_m0) 深入了解背後的客戶端技術和 React Native 演示。

<iframe
  width="560"
  height="315"
  src="https://www.youtube-nocookie.com/embed/FtkVlIal_m0?rel=0"
  frameborder="0"
  allow="autoplay; encrypted-media"
  allowfullscreen></iframe>

## 反饋

The team behind the libraries is eager to hear how these libraries and services work for you. They also want to hear what else we can do to make React and React Native development with cloud services easier for you. Reach out to the AWS Mobile team on GitHub for [AWS Amplify](https://github.com/aws/aws-amplify) or [AWS AppSync](https://github.com/aws-samples/aws-mobile-appsync-events-starter-react-native).