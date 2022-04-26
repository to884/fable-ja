# 公式サンプルをチェック

## Fable のサンプルをダウンロードする

Fable は Node.js や ブラウザなど、さまざまな JavaScript ランタイムをターゲットにできます。Fable の使い方を簡単に調べるために、私たちは fable3-samples [Github repo](https://github.com/fable-compiler/fable3-samples) を作成しました。

このリポジトリには、すぐに使えるサンプルがいくつか用意されてあり、それぞれが個別のフォルダに置かれています。

!!! info
    もし、バックエンド（.NET を使用）とフロントエンド（Fable を使用）の両方をカバーするフルスタックの F# ソリューションに興味があるなら、[SAFE-Stack](https://safe-stack.github.io/)をチェックしてみてください。

- **ブラウザのサンプル**

  - シンプルな HTML5 canvas とブラウザ DOM_[/browser](https://github.com/fable-compiler/fable3-samples/tree/master/browser)_

  - フェッチ＆プロミス＆json パースでより複雑なブラウザアプリ。_[/promises](https://github.com/fable-compiler/fable3-samples/tree/master/promises)_

- **リアクトのサンプル**

  - [Elm](https://elm-lang.org/) ライクな [シングルページアプリケーション(SPA)](https://en.wikipedia.org/wiki/Single-page_application)を React で実現。_[/minimal](https://github.com/fable-compiler/fable3-samples/tree/master/minimal)_。

  - Bulma と React を使ったフロントエンドアプリ SPA を始める(`git clone https://github.com/MangelMaxime/fulma-demo`)

- **Node.js のサンプル**

  - フェッチ＆プロミスによる Node.js アプリ。_[/nodejs](https://github.com/fable-compiler/fable3-samples/tree/master/nodejs)_。

  - バンドルされた Node.js アプリ(fetch & promises 付き)。_[/nodejsbundle](https://github.com/fable-compiler/fable3-samples/tree/master/nodejsbundle)_ 

- **高度なサンプル**

  - .NET の依存関係を解決するために Paket を使用する。_[/withpaket](https://github.com/fable-compiler/fable3-samples/tree/master/withpaket)_

  - 相互運用性: Fable から JS コードを呼び出す。_[/interop](https://github.com/fable-compiler/fable3-samples/tree/master/interop)_

  - 相互運用性: JS から Fable のコードを呼び出す: _[/interopFableFromJs](https://github.com/fable-compiler/fable3-samples/tree/master/interopFableFromJs)_

## 依存関係のインストール

**JS の依存関係** は `package.json` ファイルに記述されています。npm を使う場合は、 `npm install` を実行すると、パッケージが `node_modules` フォルダにダウンロードされ、lock ファイルが作成されます。

Nuget と Paket のどちらを使用しているかによって、**.NET の依存関係** が `.fsproj` または `paket.references` ファイルにリストアップされます。`dotnet restore` を実行すれば、それらをインストールできますが、これはすでに Fable によって自動的に実行されます。

!!! info
    lock ファイル(npm を使用している場合は `package-lock.json` など)は、誰かがリポジトリをクローンするときに、再現性のあるビルドを確実にするためにコミットされるべきです。

## アプリのビルドと実行

依存関係が解決されたので、ウォッチモードでアプリを起動しましょう。テンプレートの種類によって手順は異なります。

**Web サンプル** の場合

Web サンプルの場合、特に指示がない限り、常に `npm start` となります。

これで、[http://localhost:8080/](http://localhost:8080/)から、お気に入りのブラウザでプロジェクトにアクセスすることができるようになります。

次に、プロジェクトをコードエディターで開き、`src` フォルダにある `App.fs` ファイルを変更します。保存して、コンパイルが成功したら、ブラウザで直接変更内容を見ることができるはずです。

**Node.js のサンプル**

Node.js のサンプルの場合、特にアドバイスがない限り、常に `npm build` となります。

すると、`build`フォルダの中に生成された JS ファイルが表示されるはずです。

!!! info
    常に、テンプレートに同梱されている `README.md` ファイルをチェックして、最新の指示を得てください。
