# Remix Indie スタック

![The Remix Indie Stack](https://repository-images.githubusercontent.com/465928257/a241fa49-bd4d-485a-a2a5-5cb8e4ee0abf)

[Remix Stacks](https://remix.run/stacks)についてもっと学ぶ。

```sh
npx create-remix@latest --template remix-run/indie-stack
```

## スタック内容

- [Fly app deployment](https://fly.io) と [Docker](https://www.docker.com/) を使用した本番環境へのデプロイ
- 本番環境に適した [SQLiteデータベース](https://sqlite.org)
- [Flyバックアップリージョンフォールバック用のヘルスチェックエンドポイント](https://fly.io/docs/reference/configuration/#services-http_checks)
- 本番環境とステージング環境へのマージ時のデプロイのための [GitHub Actions](https://github.com/features/actions)
- [cookieベースのセッション](https://remix.run/utils/sessions#md-createcookiesessionstorage)を使ったメール/パスワード認証
- [Prisma](https://prisma.io)を使ったデータベースORM
- [Tailwind](https://tailwindcss.com/)を使ったスタイリング
- [Cypress](https://cypress.io)を使ったエンドツーエンドテスト
- [MSW](https://mswjs.io)を使ったローカルサードパーティリクエストモッキング
- [Vitest](https://vitest.dev)と[Testing Library](https://testing-library.com)を使ったユニットテスト
- [Prettier](https://prettier.io)を使ったコードフォーマット
- [ESLint](https://eslint.org)を使ったリンティング
- [TypeScript](https://typescriptlang.org)を使った静的タイプ

スタックの一部が気に入らない場合は、フォークして変更し、`npx create-remix --template your/repo`を使って自分のものにしてください。

## クイックスタート

このボタンをクリックして、プロジェクトが設定され、Flyがプリインストールされた[Gitpod](https://gitpod.io)のワークスペースを作成します。

[![Gitpod Ready-to-Code](https://img.shields.io/badge/Gitpod-Ready--to--Code-blue?logo=gitpod)](https://gitpod.io/#https://github.com/remix-run/indie-stack/tree/main)

## 開発

- 初期設定：

  ```sh
  npm run setup
  ```

- 開発サーバーの開始：

  ```sh
  npm run dev
  ```

これにより、ファイルの変更時にアセットをリビルドする開発モードでアプリが開始されます。

データベースシードスクリプトは、始めるために使用できるいくつかのデータを持つ新しいユーザーを作成します：

- メール： `rachel@remix.run`
- パスワード： `racheliscool`

### 関連するコード：

これはかなりシンプルなメモアプリですが、PrismaとRemixを使用してフルスタックアプリを構築する方法の良い例です。主な機能はユーザーの作成、ログインとログアウト、そしてメモの作成と削除です。

- ユーザーの作成、ログインとログアウト [./app/models/user.server.ts](./app/models/user.server.ts)
- ユーザーセッション、およびその検証 [./app/session.server.ts](./app/session.server.ts)
- メモの作成と削除 [./app/models/note.server.ts](./app/models/note.server.ts)

## デプロイメント

このRemix Stackには、アプリを本番環境とステージング環境に自動的にデプロイするための2つのGitHub Actionsが含まれています。

初回デプロイメントの前に、いくつかの作業が必要です：

- [Flyをインストール](https://fly.io/docs/getting-started/installing-flyctl/)

- Flyにサインアップしてログインします

  ```sh
  fly auth signup
  ```

  > **注記：** 複数のFlyアカウントを持っている場合は、Fly CLIでサインインしているアカウントがブラウザでサインインしているアカウントと同じであることを確認してください。ターミナルで`fly auth whoami`を実行し、メールがブラウザでサインインしているFlyアカウントと一致していることを確認します。

- ステージング用と本番用の2つのFlyアプリを作成します：

  ```sh
  fly apps create blog-tutorial-75f2
  fly apps create blog-tutorial-75f2-staging
  ```

  > **注記：** この名前が`fly.toml`ファイルに設定されている`app`と一致していることを確認してください。そうでない場合、デプロイできません。

  - Gitを初期化します。

  ```sh
  git init
  ```

- 新しい[GitHubリポジトリ](https://repo.new)を作成し、それをプロジェクトのリモートとして追加します。**まだアプリをプッシュしないでください！**

  ```sh
  git remote add origin <ORIGIN_URL>
  ```

- GitHubリポジトリに`FLY_API_TOKEN`を追加します。これを行うには、Flyのユーザー設定にアクセスして新しい[token](https://web.fly.io/user/personal_access_tokens/new)を作成し、[リポジトリのシークレット](https://docs.github.com/en/actions/security-guides/encrypted-secrets)に`FLY_API_TOKEN`の名前で追加します。

- Flyアプリのシークレットに`SESSION_SECRET`を追加します。これを行うには、以下のコマンドを実行します：

  ```sh
  fly secrets set SESSION_SECRET=$(openssl rand -hex 32) --app blog-tutorial-75f2
  fly secrets set SESSION_SECRET=$(openssl rand -hex 32) --app blog-tutorial-75f2-staging
  ```

  opensslがインストールされていない場合は、[1Password](https://1password.com/password-generator)などでランダムなシークレットを生成し、`$(openssl rand -hex 32)`を生成されたシークレットに置き換えても構いません。

- ステージングおよび本番環境のSQLiteデータベース用に永続ボリュームを作成します。以下を実行します：

  ```sh
  fly volumes create data --size 1 --app blog-tutorial-75f2
  fly volumes create data --size 1 --app blog-tutorial-75f2-staging
  ```

すべての設定が完了したら、変更をコミットしてリポジトリにプッシュします。`main`ブランチへの各コミットは本番環境へのデプロイを、`dev`ブランチへの各コミットはステージング環境へのデプロイをトリガーします。

### データベースへの接続

デプロイされたアプリケーション内にあるsqliteデータベースは`/data/sqlite.db`に存在します。`fly ssh console -C database-cli`を実行することで、ライブデータベースに接続できます。

### デプロイメントのサポートを受ける

Flyへのデプロイメントで問題が発生した場合は、上記のすべてのステップを確実に実行していることを確認し、それでも問題が解決しない場合は、デプロイメントに関する詳細（アプリ名を含む）を[Flyサポートコミュニティ](https://community.fly.io)に投稿してください。通常、そちらでは迅速な対応が期待でき、デプロイメントの問題や質問に対する解決策を提供してくれることがあります。

## GitHub Actions

このプロジェクトでは、継続的な統合とデプロイメントのためにGitHub Actionsを使用しています。`main`ブランチに入ったものは、テスト/ビルドなどを経て本番環境にデプロイされます。`dev`ブランチ内のものはステージング環境にデプロイされます。

## テスト

### Cypress

このプロジェクトではエンドツーエンドテストにCypressを使用しています。これらは`cypress`ディレクトリ内にあります。変更を加える際には、既存のファイルに追加するか、`cypress/e2e`ディレクトリ内に新しいファイルを作成して変更をテストしてください。

ページ上の要素を意味的に選択するために[`@testing-library/cypress`](https://testing-library.com/cypress)を使用しています。

これらのテストを開発環境で実行するには、`npm run test:e2e:dev`を実行してアプリの開発サーバーとCypressクライアントを起動します。上述のようにデータベースがdockerで実行されていることを確認してください。

ログインフローを経ずに認証された機能をテストするためのユーティリティがあります：

```ts
cy.login();
// 新しいユーザーとしてログインしています
```

また、テストの終わりにユーザーを自動的に削除するユーティリティもあります。各テストファイルに以下を追加してください：

```ts
afterEach(() => {
  cy.cleanupUser();
});
```

これにより、ローカルDBをクリーンに保ち、テストを互いに分離することができます。

### Vitest

ユーティリティや個々のコンポーネントの低レベルのテストには`vitest`を使用しています。DOM固有のアサーションヘルパーは[`@testing-library/jest-dom`](https://testing-library.com/jest-dom)を通じて提供されています。

### タイプチェック

このプロジェクトはTypeScriptを使用しています。タイプチェックとオートコンプリートを活用するために、エディターにTypeScriptを設定することをお勧めします。プロジェクト全体でタイプチェックを実行するには、`npm run typecheck`を実行してください。

### リンティング

このプロジェクトはリンティングのた

めにESLintを使用しています。それは`.eslintrc.js`で設定されています。

### フォーマット

このプロジェクトでは自動フォーマットのために[Prettier](https://prettier.io/)を使用しています。保存時に自動フォーマットを行うために、エディタープラグイン（例えば[VSCode Prettierプラグイン](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)）のインストールをお勧めします。プロジェクト内のすべてのファイルをフォーマットするために実行できる`npm run format`スクリプトもあります。
