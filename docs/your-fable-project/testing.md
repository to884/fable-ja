# テスト

JS エコシステムのすべてのツールを使えます。

一部の js ライブラリはすでに Fable バインディングを備えています。

- mocha: [https://github.com/Zaid-Ajaj/Fable.Mocha](https://github.com/Zaid-Ajaj/Fable.Mocha)
- jest: [https://github.com/Shmew/Fable.Jester](https://github.com/Shmew/Fable.Jester)

## jest を使った例

### セットアップ

js のテストランナーをインストールしましょう。

```sh
  npm install jest --save-dev
```

そして、Fable バインディングをインストールします。

```sh
  # nuget
  dotnet add package Fable.Jester
  # paket
  paket add Fable.Jester --project ./project/path
```

### テストを書く

さて、最初のテストを書いてみましょう。

```fsharp
open Fable.Jester

Jest.describe("基本的なテストを実行できます", fun () ->
    Jest.test("running a test", fun () ->)
        Jest.expect(1+1).toEqual(2)
    )
)
```

より詳細な情報は Jester ドキュメントを参照してください。[https://shmew.github.io/Fable.Jester/](https://shmew.github.io/Fable.Jester/)

### 実行

テストを実行する前に、プロジェクトを JS に変換する必要がありますが、テストランナーは一般的に一つの大きなファイルよりも小さなファイルを持つことを好むので、Webpack でバンドルする必要はありません。ですので、Fable コンパイラを実行し、生成されたコードを出力ディレクトリに置くだけです。

```sh
  dotnet fable src -o output
```

Jest の設定は、コンフィグファイル `jest.config.js` で行います。

```js
module.exports = {
  moduleFileExtensions: ['js'],
  roots: ['./output'],
  testMatch: ['<rootDir>/**/*.Test.js'],
  coveragePathIgnorePatterns: ['/\.fable/', '/[fF]able.*/', '/node_modules/'],
  testEnvironment: 'node',
  transform: {}
};
```

`root` はコンパイラの `outDir` と同じである必要があります。
`testMatch` はテストするファイルのパターン名を表します。
`coveragePathIgnorePatterns`, `testEnvironment`, `transform` はランナーのパフォーマンスを改善します。
詳しくはJestのドキュメントを参照してください : [https://jestjs.io/docs/jp/configuration](https://jestjs.io/docs/en/configuration)

これで、テストを実行できるようになりました。

```sh
  npx jest --config=jest.config.js
```

これでテストが完了です。テスト結果を見てみましょう。

このコマンドは、npm の `package.json` で指定します。

```json
{
  "scripts": {
    "test": "dotnet fable src -o output --run jest --config=jest.config.js",
  },
}
```

そして、今度はコマンド1つで実行します。

```sh
  npm test
```

### ウォッチモード

テストを毎回実行していると時間がかかります。
ウォッチ機能を使えば、コンパイラとランナーのキャッシュを活用し、ファイルが変更されるたびにテストを実行することができます。

現在、Fable には様々なランナーに対する公式プラグインがありません。
そのため、以下の 2 つのコマンドを並列に実行する必要があります。

```sh
dotnet fable watch src -o 出力
npx jest --config=jest.config.js --watchAll
```

`package.json` に npm スクリプトを追加します。

```json
{
  "scripts": {
    "test": "dotnet fable src -o output && jest --config=jest.config.js",
    "watch-test:build": "dotnet fable watch src -o output",
    "watch-test:run": "jest --config=jest.config.js --watchAll",
    "watch-test": "npm-run-all --parallel watch-test:*"
  },
}
```

私は、複数のコマンドを並列に実行するために `npm-run-all` を使っています。これをインストールしておくとよいでしょう。

```sh
npm install --save-dev npm-run-all
```

次に、以下のコマンドを実行します。

```sh
  npm run-script watch-test
```

お楽しみください！
