# 新しいプロジェクトを開始する

さあ、準備ができたら、Fable を使って新しいプロジェクトを始めてみましょう!

## Fable テンプレートを使ってみよう

Fable を使い始める最も簡単な方法は、テンプレートを使ってみることです (詳しくは [dotnet テンプレート](https://docs.microsoft.com/en-us/dotnet/core/tools/custom-templates#installing-a-template) を見てください)。最小限の Fable プロジェクトでは、新しいディレクトリを作成して移動し、次のコマンドを実行します(最初のコマンドは、システムにまだテンプレートをインストールしていない場合にのみ必要です)。

1. `dotnet new --install Fable.Template` を実行します。
2. `dotnet new fable` を実行します。

このドキュメントの残りの部分は、Fable.Template に当てはまります。また、具体的な例や、より多くのツールやライブラリがインストールされたよりまとまったテンプレートが必要な場合は、次のいずれかをチェックしてみてください。

- [Fable 3 samples](https://github.com/fable-compiler/fable3-samples) リポジトリをクローンして、いろいろな種類のアプリ（ブラウザ、nodejs、テストなど）で Fable を使用する方法を学ぶ。
- [Feliz テンプレート](https://zaid-ajaj.github.io/Feliz/#/Feliz/ProjectTemplate) を使って、F#で [React](https://reactjs.org/) アプリを構築する。
- [Sutil](https://davedawkins.github.io/Sutil/#documentation-installation) を使って JS に依存しないフロントエンドアプリを F#で完全に書く。
- フロントエンドとバックエンドの両方をカバーする [SAFE テンプレート](https://safe-stack.github.io/docs/quickstart/) を使ってスピードアップする。

## 依存関係のインストール

**JS の依存関係** は、 `package.json` ファイルに記述されます。npm install`を実行すると、必要なパッケージが`node_modules` フォルダにダウンロードされ、lock ファイルが作成されます。

**.NET の依存関係** は `src/App.fsproj` ファイルに記述されます。`dotnet restore src`を実行することでインストールできますが、これは Fable がすでに自動的に行っていることです。

!!! info
    lock ファイル（npm を使用している場合は `package-lock.json` など）は、誰かがあなたのリポジトリをクローンするときに、確実に再現性のあるビルドを行うためにコミットする必要があります。.NET の依存ファイルについては、[Paket](https://fsprojects.github.io/Paket/) を使用して lock ファイルを作成することができます。

## アプリのビルドと実行

いよいよです。JS の依存関係を既にインストールしている場合は、`npm start` を実行するだけです。数秒後、あなたの Web サーバーアプリは、あなたが消すまでバックグラウンドで実行されます。

- お気に入りのブラウザで、[http://localhost:8080/](http://localhost:8080/) にあるあなたのプロジェクトにアクセスできます。
- サーバーは "watch "モードになっています。F#のソースファイル `.fs` を保存するたびに、Fable は自動的にプロジェクトをリビルドします。ビルドが成功すれば、ページを更新することなく、ブラウザに変更内容が表示されます。成功しなければ、ブラウザには何も表示されず、サーバーのコンソール出力に表示されるビルドエラーを確認してみてください。

!!! info
    `npm start` コマンドは `dotnet fable watch src --run webpack-dev-server` のエイリアスに過ぎないので、`package.json` ファイルの "scripts" セクションを確認してください。また、テンプレートに同梱されている `README.md` ファイルをチェックして、最新の手順を確認してください。
