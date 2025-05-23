# 認証認可

## 認証（Authentication）

ユーザが「誰」であるかを確認すること

HTTP ステータスコードだと 401 Unauthorized

## 認可（Authorization）

権利を確認すること

HTTP ステータスコードだと 403 Forbidden

## OAuth 2.0

外部サービスやサードパーティが絡んだときに発生する**認可**の仕組み

### 背景

1. セキュリティリスク
   - ID / PW が漏洩した際、攻撃者が全ての権限を得ることになってしまうので、**認証** と切り離した**認可**の仕組みが必要だった。
   - 上記により、安全に外部にアクセス権を渡したかった。
2. 細かいアクセス権限の管理
   - サードパーティに対して一部のデータの閲覧を許可など細かくアクセス制御をしたい。
   - 不要になった権限は後から削除したい。

### 4 つのロール

| 役割                 | 説明                                                                     | 例（Google ログインを使う場合） |
| -------------------- | ------------------------------------------------------------------------ | ------------------------------- |
| **リソースオーナー** | 資源（データ）の持ち主（＝ユーザー）                                     | Google アカウントの持ち主       |
| **クライアント**     | OAuth を使って外部サービスのデータにアクセスするアプリ                   | 自作の Web アプリ               |
| **リソースサーバ**   | ユーザーのデータを持つサーバ。<br />アクセスにはアクセストークンが必要。 | Google の API（カレンダーなど） |
| **認可サーバ**       | 認可（アクセストークンの発行）を担当                                     |                                 |

### 4 つの値

| 項目                         | 説明                                                                                                                                                                                                                       | 例（Google OAuth）                                                                                            |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **クライアント ID**          | クライアント（アプリ）が認可コードを取得するために、認可サーバと通信する際に使う鍵。<br />公開しても構わない。                                                                                                             |                                                                                                               |
| **クライアントシークレット** | クライアント（アプリ）が認可サーバと通信する際に使う秘密の鍵。<br/>サーバ側とアプリ側でのみ共有される。                                                                                                                    | Web アプリのバックエンドが持つ、OAuth 認証プロセスを通じて発行されたシークレット                              |
| **認可コード**               | ユーザーが認可サーバに認証し、認可を与えた後に発行される一時的なコード。<br/>これを使ってアクセストークンを取得する。                                                                                                      | ユーザーが Google にログイン後、認可サーバから受け取る一時的なコード（`authorization_code`）                  |
| **アクセストークン**         | クライアントがリソースサーバ（外部 API）にデータをリクエストする際に使う、アクセス許可を示すトークン。<br/>**アクセストークン** 自体にスコープを意味する値を含めるので、ユーザーが許可した範囲内でデータにアクセスできる。 | Google の API（カレンダーなど）をリクエストする際に使用するアクセストークン（`access_token`）                 |
| **リフレッシュトークン**     | アクセストークンが期限切れになったときに、新しいアクセストークンを取得するために使用する。<br/>ユーザーの再認証なしで、アクセストークンを再取得できる。                                                                    | アクセストークンが期限切れになった後、再度アクセストークンを取得するために使用するトークン（`refresh_token`） |

1. クライアント ID とクライアントシークレット\*\*

   - **クライアント ID**：
     → 「このリクエストはどのアプリから来た？」を識別するための**ラベルのようなもの**。

   - **クライアントシークレット**：
     → 「このアプリが**本当に正規のクライアントか？**」を確認するための**パスワードのようなもの**。

     | 項目                   | クライアント ID                                    | クライアントシークレット                                   |
     | ---------------------- | -------------------------------------------------- | ---------------------------------------------------------- |
     | **役割**               | クライアント（アプリ）を識別するための**公開情報** | クライアントの**認証（なりすまし防止）**に使う秘密鍵       |
     | **使われるタイミング** | 認可コードの取得時（リダイレクト時など）           | アクセストークンの取得時（サーバ間通信）                   |
     | **誰が知っている？**   | クライアントと認可サーバ、ユーザーに見えることも   | クライアントと認可サーバのみ（ユーザーに見せてはいけない） |
     | **セキュリティレベル** | 公開して OK（URL に含まれても問題なし）            | 公開 NG（クライアントの秘密情報）                          |

### 3 つのエンドポイント

| 項目                           | 所属                                   | 呼び出し元                                         | 役割                                                                         |
| ------------------------------ | -------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------- |
| **認可エンドポイント**         | **認可サーバ**のエンドポイント         | **リソースオーナー（ユーザー）**がブラウザから呼ぶ | ブラウザからユーザーがアクセスして認可画面を表示し、**認可コード**を発行する |
| **トークンエンドポイント**     | **認可サーバ**のエンドポイント         | **クライアント（バックエンド）**が呼ぶ             | 認可コードを送信して、**アクセストークン / リフレッシュトークン**を取得する  |
| **リダイレクトエンドポイント** | **クライアントアプリ**のエンドポイント | 認可サーバ → ユーザーのブラウザ                    | 認可サーバが認可コードを付けてユーザーを**クライアントにリダイレクトする**   |

### 認可のフロー

| 手順 | 主体                                  | 処理内容                                                           |
| ---- | ------------------------------------- | ------------------------------------------------------------------ |
| 1    | ユーザー                              | クライアントのボタンを押す                                         |
| 2    | クライアント → 認可サーバ             | 認可エンドポイントにリダイレクト（認可コード要求）                 |
| 3    | ユーザー → 認可サーバ                 | ログイン＋許可ボタンを押す                                         |
| 4    | 認可サーバ → クライアント             | 認可コードを付けてリダイレクト                                     |
| 5    | クライアント（サーバー）→ 認可サーバ  | トークンエンドポイントに認可コード＋クライアントシークレットを送信 |
| 6    | 認可サーバ → クライアント（サーバー） | アクセストークンとリフレッシュトークンを返す                       |

## PKCE（Proof Key of Code Exchange）

### 概要

ピクシーと呼ぶ。
リダイレクト先でのなりすまし（攻撃者のサイトにリダイレクトしてしまう）を防ぐ仕組み。

モバイルアプリには必須のセキュリティ。
Web アプリはドメインによる一意性があり、リダイレクト先のなりすましが難しい。
一方で、モバイルアプリはカスタムスキーム（例：myapp://callback）を使うため、スキームの競合やなりすましのリスクが高く、PKCE などの対策が必須である。

OAuth 2.0 の拡張仕様

## 手法

1. クライアントからブラウザを起動する際に、ランダムな文字列を渡す。
2. その値を認可リクエスト（認可コードを要求するリクエスト）にそのまま含ませる
3. 認可レスポンス（認可コードを受け取るレスポンス）のリダイレクトで起動したクライアントは、トークンリクエスト（アクセストークンを取得するリクエスト）にランダム文字列を添えてアクセストークンを取得する。
4. （厳密には違うが、）要はランダム文字列はクライアントシークレットみたいなもの

## OpenID Connect 1.0

OAuth 2.0 を認証として活用するために拡張した、ID 連携のための仕組み

### 背景

1. OAuth の使い道が広すぎた
   1. アクセストークンはあくまでスコープ付きのリソースの使用権限だが、それを持っている人＝本人として扱う OAuth による認証をし始めてしまった。
2. OAuth だけでは、誰がログインしているかわからない
   1. OAuth では認証時の情報（日時や手段）やユーザの情報（ID や名前、メアドなど）を取得する仕様が定められていない
3. 現場がバラバラな実装をし始めた
   1. 仕様がバラバラ
   2. セキュリティもまちまち
   3. 実装コストも高い

以上の理由から OpenID Connect という形で標準化が図られた。

### 概略

ざっくり、アクセストークンに加えて、ID トークンも得ることができる

#### ID トークン

name, profile, picture, email などのユーザ情報を持つものが JWT 形式になったもの

### OAuth 2.0 との違い

1. 認可リクエストに scope=openid をつける。これをつけると OIDC として認識される。
2. トークンエンドポイントを叩いた時にアクセストークンと一緒に ID トークンが返ってくる。
3. userinfo エンドポイントで詳細なユーザ情報を取得できる

## 参考

https://zenn.dev/suzuki_hoge/books/2021-05-authentication-and-authorization-0259d3f
