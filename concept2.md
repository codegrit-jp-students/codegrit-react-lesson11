# Firebaseにアプリを公開しよう

## Firebase CLIのインストール

さて、ここまででアプリを公開する準備が整いました。アプリの公開先は色々な場所が選べますが今回はFirebaseをここまで利用してきていますのでFirebaseを利用して公開しましょう。

まずはパソコンにFirebase CLIをインストールします。

```bash
$ yarn add global firebase-tools
```

次に、自分のFirebaseのアカウントを利用してFirebaseにログインします。

```bash
$ firebase login
```

## ビルドする

次に、開発してきたアプリケーションを本番ビルドします。`create-react-app`では本番ビルドのためのNPMスクリプトが最初から用意されています。

```bash
yarn run build
```

少し待っているとビルドが完了します。プロジェクトのフォルダーを見てみると`build`というフォルダーが自動で追加され、その中にビルドされたファイルが入っていることが分かるかと思います。このフォルダーは、リポジトリのサイズを大きくしてしまうので`.gitignore`に含まれています。

## デプロイする

後は、ビルドされたコードをFirebase上に公開します。この公開の作業のことを専門用語でデプロイと呼びます。

まずは、プロジェクトのリポジトリをFirebase公開用に初期化します。

```bash
$ firebase init
```

すると、いくつか質問が出てきますので以下のように回答していきます。

- "Which Firebase CLI features do you want to setup for this folder? Press Space to select features, then Enter to confirm your choices."

DatabaseとHostingの2つを選びます。

- What file should be used for Database Rules?

そのままEnterキーを押してスキップします。

- What do you want to use as your public directory?

buildと入力します

- Configure as a single-page app (rewrite all urls to /index.html)? (y/N) 

yを入力します。


この初期化が終わると、firebase.jsonなどいくつかの設定ファイルが作成されその中に回答をもとにした設定が登録されます。

では最後に次のコマンドを入力してください。

```bash
$ firebase deploy
```

このコマンドでFirebase上に自動的にサイトが公開されます。公開されたら、コンソール上に表示された公開先のURLにアクセスしましょう。いかがでしょうか、サイトは無事公開されているでしょうか？

## 更に学ぼう

- [Create React App公式 - Adding Custom Environment Variables](https://facebook.github.io/create-react-app/docs/adding-custom-environment-variables)
- [Create React App公式 - Deployment](https://facebook.github.io/create-react-app/docs/deployment)
- [React Helmet公式](https://github.com/nfl/react-helmet)
- [Firebase公式 - サイトをデプロイする](https://firebase.google.com/docs/hosting/deploying)