# .NET と F#の互換性

Fable は、.NET BCL (Base Class Library)の一部のクラスと、FSharp.Core ライブラリのほとんどのサポートを実現しています。できるかぎり、Fable は.NET の型とメソッドをネイティブの JavaScript API に変換して、オーバーヘッドを最小限にとどめています。

## .NET ベースクラスライブラリ

次のクラスは JS に変換され、そのほとんどのメソッド（静的およびインスタンス）が Fable で使えるはずです。

| .NET                                  | JavaScript            |
| ------------------------------------- | --------------------- |
| Numeric Types                         | number                |
| Arrays                                | Array / Typed Arrays  |
| Events                                | fable-core/Event      |
| System.Boolean                        | boolean               |
| System.Char                           | string                |
| System.String                         | string                |
| System.Guid                           | string                |
| System.TimeSpan                       | number                |
| System.DateTime                       | Date                  |
| System.DateTimeOffset                 | Date                  |
| System.DateOnly                       | Date                  |
| System.TimeOnly                       | number                |
| System.Timers.Timer                   | fable-core/Timer      |
| System.Collections.Generic.List       | Array                 |
| System.Collections.Generic.HashSet    | Set                   |
| System.Collections.Generic.Dictionary | Map                   |
| System.Text.RegularExpressions.Regex  | RegExp                |
| System.Lazy                           | fable-core/Lazy       |
| System.Random                         | {}                    |
| System.Math                           | (native JS functions) |

次の静的メソッドも使えます。

- `System.Console.WriteLine` (フォーマットも可能)
- `System.Diagnostics.Debug.WriteLine` (フォーマットあり)
- `System.Diagnostics.Debug.Assert(condition: bool)` システム診断.デバッグ.アサート(condition: bool)
- `システム.診断.デバッグ.Break()` の実行
- システム.アクティベータ.CreateInstance<'T>()

数値型の変換や文字列のパースもサポートされていますので、[変換テスト](https://github.com/fable-compiler/Fable/blob/master/tests/Main/ConvertTests.fs) をチェックしてください。

### 注意点

- `int64`、`uint64`、`bigint`、`decimal` を除くすべての数値型は JSの `number` (64-bit 浮動小数点型) に対応します。.NET と JS の違いについては、[数値型](numbers.md) のセクションをチェックしてください。
- 数値型配列はできるだけ [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) にコンパイルします。
- 数値型(`byte 500` のような明示的な変換を行わない限り)や配列のインデックスに対する境界チェックは行いません。
- `Regex` は常に `RegexOptions.ECMAScript` フラグを渡されたように動作します (例えば、負の look-behind や named group は使用しません)。
- `DateOnly`/`TimeOnly` を使うには、.fsproj ファイルで `net6.0` をターゲットにする必要があります。

## FSharp.Core

FSharp.Core のほとんどの演算子と、`sprintf`、`printfn`、`failwithf` (`String.Format` も利用可能)によるフォーマットがサポートされています。
FSharp.Core lib の以下の型と対応するモジュールも同様に JS に変換されます。

| .NET             | JavaScript                                                        |
| ---------------- | ----------------------------------------------------------------- |
| Tuples           | Array                                                             |
| Option           | (erased)                                                          |
| Choice           | fable-core/Choice                                                 |
| Result           | fable-core/Result                                                 |
| String           | fable-core/String (module)                                        |
| Seq              | [Iterable](http://babeljs.io/docs/learn-es2015/#iterators-for-of) |
| List             | fable-core/List                                                   |
| Map              | fable-core/Map                                                    |
| Set              | fable-core/Set                                                    |
| Async            | fable-core/Async                                                  |
| Event            | fable-core/Event (module)                                         |
| Observable       | fable-core/Observable (module)                                    |
| Arrays           | Array / Typed Arrays                                              |
| Events           | fable-core/Event                                                  |
| MailboxProcessor | fable-core/MailboxProcessor (limited support)                     |

### 注意点 II

- オプションは JS では **消去されます** (`Some 5` は JS ではただの`5`になり、`None`は`null`に変換されます)。これはたとえば、TypeScript の [オプションのプロパティ](https://www.typescriptlang.org/docs/handbook/interfaces.html#optional-properties) を表現するために必要なものです。しかし、いくつかの場合(ネストされたオプションなど)では、ランタイムにオプションの実際の表現が存在します。
- `Async.RunSynchronously`はサポートされていません。
- `MailboxProcessor` は JS ではシングルスレッドで、現在は`Start`, `Receive`, `Post`, `PostAndAsyncReply` のみ実装されています (`cancellationToken` や `timeout` オプション引数はサポートされていません).

## オブジェクト指向プログラミング

F#のオブジェクト指向機能のほとんどは、Fable と互換性があります：インターフェースと抽象クラス、構造体、継承、オーバーローディングなど。ただし、[ES2015 クラス](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes) のいくつかの制限のため、生成されたコードは代わりに [プロトタイプチェーン](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain) を使う点に注意してください。また、インスタンスメンバーはプロトタイプにアタッチされないので、ネイティブ JS コードからアクセスできないことに注意してください。このルールの例外は、**インターフェースと抽象メンバー**の実装です。

### 注意点 III

- インターフェースやジェネリック型に対して型テストを行うことはできません。

## リフレクションとジェネリック

Fable ではリフレクションのサポートがあります。現在何ができるかは [リフレクションテスト](https://github.com/fable-compiler/Fable/blob/master/tests/Main/ReflectionTests.fs) でチェックすると良いでしょう。

生成された JS コードでは、ジェネリックが既定で消去されます。しかし、`inline` で関数をマークすることで、実行時にジェネリック情報(例えば `typeof<'T>`)にアクセスすることはできます。

```fsharp
let doesNotCompileInFable(x: 'T) =
    typeof<'T>.FullName |> printfn "%s"

let inline doesWork(x: 'T) =
    typeof<'T>.FullName |> printfn "%s"

doesWork 5
```