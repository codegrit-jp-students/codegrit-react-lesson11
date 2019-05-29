# レッスン11. Firebaseを利用したアプリケーションの公開

## アプリケーションを公開しよう

前回のレッスンでは、クイズアプリの作成を行いました。このレッスンでは、作成したアプリを実際にインターネット上に公開していきます。このアプリの公開にあたっては、いくつか知っておくべき概念がありますので、それについて解説し実際に公開していきます。

## 環境とは

まず最初に知っておくべきは環境という概念です。最初に用語を上げるとこれまでのように自分のパソコン上で開発していて公開されていない環境のことを**開発環境**、公開しているアプリの環境を**本番環境**、そしてテストを行っている環境を**テスト環境**と呼びます。またアプリケーションによっては、開発環境と本番環境の間に、より本番環境に近い環境でアプリをテストをするために**ステージング環境**という環境を用意することもあります。

なぜ、このように環境が分かれるかというと、開発中と本番では様々な条件が異なってくるためです。例えば、大きなアプリではサーバーは一つだけでなく複数のサーバーを用意して負荷を分散させることが普通です。またMacやWindowsのサーバーではなく、Linuxというサーバーを使うためOS自体も違うことが普通です。

こうした、与えられた条件の違いに対応するために環境ごとに異なるアプリの設定が必要になってきます。

## JavaScriptアプリケーションのプロダクション(本番)ビルド

開発中のアプリケーションを、ES6からES5のコードにトランスパイルしたり、下記にあげる最適化などを行いブラウザにアップできる状態にすることを**ビルド**と呼びます。ビルドは本番環境向けのみではなく、他の環境用に行うこともありますので、環境名に併せて本番ビルド、開発ビルドなどと呼んだりします。

JavaScriptアプリケーションの本番ビルドにあたっては、本番環境と開発環境では特に以下の点が重要になります。

1. ファイルをまとめる
2. ファイルを最小化する

アプリケーションのファイルが分散したままサーバー上に公開すると、ブラウザ側はその全てのファイルの取得のために何度もHTTPリクエストをサーバーに対して送る必要があります。開発環境では、複数のファイルに分かれていたほうがエラーなどの場所の特定が簡単です。しかし本番環境では、複数のHTTPリクエストを送ることで発生するタイムラグがユーザーの体験を低下させてしまいます。

そこで、裏側では分かれているファイルをES6からES6にトランスパイルし、更にファイルにまとめるということをしていきます。ファイルは一個にまとめるという場合もあれば、複数ファイルにまとめることもあります。

次にまとめたファイルを最小化するということをします。開発中のファイルは、開発者が見やすいように適切なスペースを置いています。しかし、ブラウザはES5やHTML、CSSのルールにさえしたがっていればスペースが全くなくても問題なく読み取れます。

そこで、Minifyといって無駄なスペースを取り払ってファイルサイズを出来るだけ最小化するということをします。これもファイルをまとめるのと同様ユーザー体験を向上させるための手段です。

ここまで書いていくと難しそうに感じるかと思いますが、実は`create-react-app`ではこうした作業をある程度行うための設定が最初からされています。そのため、その設定をそのまま使うだけで多くの局面では問題ありません。(より厳密なコントロールをしたい場合はwebpackなどの設定を変更する必要があります。)

ただし、概念を知っておくことは重要ですので上記のようなことを行っているということは理解しておきましょう。

## 環境変数(.env)

### 環境変数とは

次に環境変数という概念について説明していきます。先程解説したとおり、環境ごとにサーバーの条件は異なります。たとえば今回のようにFirebaseを利用している場合は、Cloud Datastoreに保存されているデータも大きく違うことが普通です。例えば、ユーザー登録のあるアプリなら、登録されているユーザーも開発環境では少数のテスト用ユーザー、本番環境では実際に使っているユーザーがいます。そのため、Firebaseのプロジェクトも例えば、`MyQuiz Dev`、`MyQuiz Prod`のように2つのプロジェクトを作ったりすることが想定されます。

そのように開発環境ごとに異なるプロジェクトを用意すると、`firebaseConfig`の値も全て違ってきます。こうした異なる環境ごとの値を環境変数と呼びます。この環境変数を保存するための場所として`create-react-app`では`.env`で始まるファイルを利用します。ファイルは環境ごとに用意し、create-react-appでは環境ごとに、環境とあったファイルのデータを読み込みます。

例えば、開発環境であれば`.env.development`、本番環境では`.env.production`という名前をつけます。

また、`.env.development.local`、`.env.production.local`という名前もつけることが出来、create-react-appではこの2つのファイルは初期状態で`.gitignore`に入っています。

### 環境変数にデータを登録する

環境変数にデータを登録する場合は以下のように、データ名の部分は大文字でアンダーバー_を使ってスペースをつなぎます。また値の部分を書く前に`=`を使ってつなぎスペースは入れません。値の部分に文字列を使う場合`""`で囲っても囲まなくてもどちらでも書けます。

`NAME_OF_DATA=data`

また、この.envのファイルへ登録する環境変数名にも`create-react-app`ではルールがあります。簡単なルールで名前の始めに`REACT_APP_`という頭文字をつけます。

例えば、今回のようなFirebaseに関する環境変数であれば、以下のようにします。

```
REACT_APP_FIREBASE_API_KEY="FirebaseのAPIキー"
...省略
```

### 環境変数を利用する

環境変数を登録したら、アプリのファイル内で以下のようにして登録した変数にアクセス出来ます。

```js
process.env.REACT_APP_FIREBASE_API_KEY
```

### 環境変数を隠す

環境変数に登録するデータには一般に公開したくないデータが含まれることも多くあります。自分のFirebaseのプロジェクトにアクセスするためのデータは直接見える状態にしておくべきではありません。サーバーサイドのプロジェクトはこの環境変数で外部APIの秘密にしておくべき情報を公開してしまい気づいたら多額の請求をされていたというトラブルも発生しています。

そのため、.envのファイルは`.gitignore`に含めてアップロードされないようにします。尚、create-react-appではこの設定を最初からされています。

これも知識として非常に重要ですので覚えておいてください。

### firebaseConfigを書き換える

さて、説明が長くなりましたが今回のアプリで`firebaseDb.js`のfirebaseConfigの部分を環境変数を使って書き換えましょう。最終的に以下のようになります。

```js
const firebaseConfig = {
  apiKey: process.env.REACT_APP_REACT_APP_FIREBASE_API_KEY,
  databaseURL: process.env.REACT_APP_FIREBASE_DATABASE_URL,
  projectId: process.env.REACT_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.REACT_APP_FIREBASE_STORAGE_BUCKET,
  appId: process.env.REACT_APP_FIREBASE_APP_ID
};
```

今回のアプリでは、データは読み込みを行うのみなので`.env.development`と`.env.production`とでは同じものを使って大丈夫です。`.gitignore`への追記も忘れないでください。

## データベースのセキュリティを設定する

次にFirebaseデータストア上の設定を行います。Firebaseのコンソール内で、Databaseのページにアクセスするとメニューの中に`Rules`という部分があります。そのタブをクリックするとCloud Datastoreのアクセスのルールを設定することが出来るページに移動します。現状は以下の画像のように警告が出ているはずです。

![cloud-datastore-security-rules-1](https://firebasestorage.googleapis.com/v0/b/codegrit-images.appspot.com/o/codegrit-react%2FLesson11%2Fcloud-datastore-security-rules-1-2.png?alt=media&token=754d6995-ff36-415f-871a-916b522c0af4)

これは、現状データベースに対して誰でも書き込み、読み込みが出来る状態になっているためです。より複雑なアプリでは、アドミン(管理者)ユーザーのみが書き換えたり読み込めるデータ、ログイン中のユーザーのみがアクセス出来るデータなどユーザー権限によってデータのアクセス権をコントロールすることが普通です。

今回のアプリの場合は、データは読み込みしか行わないので全てのユーザーに対して書き込みを禁止する設定を行います。具体的には以下のようにします。

```
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read;
    }
  }
}
```

## メタデータを設定する

次に行うことがメタデータの設定です。通常メタデータはHTMLファイルに書きます。しかしJavaScriptアプリケーションの場合、HTMLファイルは一つだけでも、複数ページを持つということも多くあります。(複数ページのアプリの作り方はReduxコースで学びます。)

このような場合でもページごとにタイトルなどのメタ情報を変更しなければ、Googleなどの検索エンジンは全てのページを同じタイトルやdescriptionで検索結果に表示してしまいます。

そこで`React Helmet`というパッケージを利用してReactのコード上でメタデータを設定していきます。豆知識ですが、このパッケージはアメリカンフットボールのNFLがオープンソースで公開しており、アメフトのヘルメットとかけてこの名前がつけられています。

では実際にこのパッケージを使っていきましょう。まずはインストールを行います。

```bash
yarn add react-helmet
```

インストールが終わったらApp.jsを書き換えていきます。登録するメタデータは多くありますが今回はシンプルにタイトルと説明だけを登録します。

```js
import React from 'react';
import Helmet from 'react-helmet';
import Top from './Top';

function App() {
  return (
    <div className="App">
      <Helmet>
        <title>MyQuiz App</title>
        <meta name="description" content="CodeGritのReactコースレッスン10のために作られたクイズアプリです。" />
      </Helmet>
      <Top />
    </div>
  );
}

export default App;
```

React Helmetでは、このようにApp.js上で基本のメタデータを設定しても、他のコンポーネントで同じ名前のメタデータを登録すると上書きしてくれます。そのため基本のデータはアプリの一番上のApp.jsに登録しても大丈夫です。

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