# ビルドと実行

プロジェクトの準備ができましたか？それなら、いよいよビルドして実行しましょう。

しかし、このプロセスは、まだ開発を進めている段階なのか、それともすでにプロジェクトを本番環境にデプロイする準備をしている段階なのかによって変わってきます。開発段階では、最適化にはあまりこだわらず、変更した結果を画面上ですぐに確認できるよう、高速な ビルドが欲しいところです。一方、本番環境では、より最適化された JS コードを得るために、より遅いビルドを我慢することになります。

Webpack の設定は少し難しいかもしれません。ほとんどの [サンプル](https://github.com/fable-compiler/fable3-samples) には `webpack.config.js` ファイルがあり、参考にできます。また、[webpack config template for Fable](https://github.com/fable-compiler/webpack-config-template/blob/master/webpack.config.js) もあり、ほとんどのプロジェクトで動作するよう用意されています。

## 開発

すでに自前のサーバーを持っている場合でも、フロントエンドを開発するときには、コードの変更を検知してアプリを再起動せずに読み込むことができるサーバーが欲しくなります! これは  [webpack-dev-server](https://github.com/webpack/webpack-dev-server) が行います。通常、ウォッチモードで Fable と一緒に実行します。

> 多くの Fable プロジェクトでは、Webpack に引数を渡すために `webpack.config.js` ファイルを使用するので、`webpack serve` コマンドしか目に入らないでしょう。

[npm-scripts](https://docs.npmjs.com/misc/scripts) を共通のアクションのショートカットとして使うのであれば、開発モードで実行するために `npm start` を使うのが一般的な方法です。次は `package.json` の例です。

```json
  "scripts": {
    "start": "dotnet fable watch src --run webpack serve".
  },
```

> 自前のサーバーも持っているのであれば、APIコールをそちらにリダイレクトさせたいこともあるでしょう。そのためには [devServer.proxy](https://webpack.js.org/configuration/dev-server#devserverproxy) という Webpack の設定オプションを使用します。

webpack-dev-server は、あなたがプロセスを終了させるまで実行し続け、ファイルを保存した後にあなたのコードの変更を検知します。[Hot Module Replacement](https://elmish.github.io/hmr/) を使うと、アプリを再起動せずに変更を取り込もうとします。そうでないときは、Web ページを更新するだけです。

webpack-dev-server は生成されたファイルがメモリ上にあり、実際にディスクに書き込まれないことに気をつけてください。これは次のステップで行います。つまり、本番環境用にビルドするのです!

## 本番環境

デプロイ用のファイルを準備するとき、特別なサーバーは必要ありません。Webpack CLIを本番モードで直接呼び出して、[JS minification](https://webpack.js.org/configuration/optimization#optimizationminimize): `dotnet fable src --run webpack --mode production` などの最適化を有効にすることができます。

> 前述のように、ほとんどの設定では `--mode` 引数を明示的に設定する必要はありません。また、この操作のために "build" (場合によっては "deploy") という npm-script を用意するのが一般的です。

```json
  "scripts": {
    "start": "dotnet fable watch src --run webpack serve",
    "build": "dotnet fable src --run webpack"
  },
```

Webpack は一度だけ実行され、生成されたファイルは [output.path](https://webpack.js.org/configuration/output#outputpath) に書き込まれます。このディレクトリはホストにデプロイする必要があります!

## Webpack は使いたくない

ときおり、選んだサンプルによって、ビルドオプションが違う場合があります。詳細は、使いたいサンプルの README ファイルを参照してください。
