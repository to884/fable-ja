# JavaScript から Fable を呼び出す

JavaScript アプリで Fable の力を借りたいときもあるでしょう。たとえば、新しい
[js node in Node-RED](https://nodered.org/docs/creating-nodes/first-node) や、新しい Node.js サーバーレス関数から便利な F# コードを呼び出したり、強力な json パース処理を JavaScript アプリで呼び出したりするような場合です。

それによって、Fable で動かしながら、機能を 1 つずつ増やしていけるかもしれません。ところで、JavaScript から Fable を呼び出すには何が必要なのでしょうか？まず、生成された JS コードがどのようなものかを少々理解し、正しく呼び出せるようにしておく必要があります。

> [Fable REPL](https://fable.io/repl/) を使って、F# コードの生成された JS を簡単に確認できることをお忘れなく！

## 名前のマングリング

JS はオーバーロードや 1 つのファイルで複数のモジュールをサポートしないため、Fable は衝突を避けるためにいくつかのメンバーの名前をマングリングする必要があり、そのような関数や値を JS から呼び出すことは困難です。しかし、相互運用性を高めるために、Fable が名前を変更しないことを保証しているケースもあります。

- レコードフィールド
- インターフェースと抽象メンバー
- ルートモジュール内の関数と値

ルートモジュールって何？F# は同一ファイル内に複数のモジュールがあっても構わないので、実際のメンバーを含む最初のモジュールをルートモジュールとし、他のモジュールにネストされないようにします。

```fsharp
// 名前空間が長くても関係なく、Fableは実際のコードを見つけたときだけカウントを開始します。
module A.Long.Namespace.RootModule

// この関数の名前は、JSでも同じになります。
let add (x: int) (y: int) = x + y

module Nested =
    // JSのモジュール名の前にプレフィックスが付きます。
    let add (x: int) (y: int) = x * y
```

F# は、1 つのファイルに複数のルートモジュールが存在することがあり、その場合、すべてがマングリングされます。もし、JS にコードを公開したいのであれば、このパターンは避けるべきでしょう。

```fsharp
namespace SharedNamespace

// ここでは、名前の衝突を避けるため、
// 両方の関数をマングリングしています。
module Foo =
    let add x y = x + y

module Bar =
    let add x y = x * y
```

### カスタムの動作

場合によっては、名前のマングリングに関するデフォルトの振る舞いを変更することができます。

- すべてのメンバーを (標準的な JS クラスのように) クラスにアタッチして、マングリングしないようにしたいのであれば、 `AttachMembers` 属性を使います。ただし、この場合 **オーバーロードは動作しない** ので注意してください。
- インターフェースで JS とやり取りする予定がなく、オーバーロードされたメンバーを持ちたいのであれば、 `Mangle` 属性でインターフェース宣言を修飾できます。注意: (System.Collections.IEnumerator のような) .NET BCL から来るインターフェイスは、既定でマングリングされます。

## 一般的な型とオブジェクト

F# と .NET の型の中には、JS に対応するものがあります(/../dotnet/compatibility.html)。Fable はこれを利用して、よりパフォーマンスが高く、バンドルサイズを小さくできるネイティブ型にコンパイルします。また、F# と JS の間でデータをやり取りする際の相互運用性を高めるために利用したりもします。最も重要な共通型は次のとおりです。

- **文字列とブーリアン**は、F# と JS で同じ挙動をします。
- **文字型**は、長さ 1 の JS 文字列としてコンパイルされます。しかし，`int16 '家'`のように明示的に変換すれば，`char` を数値として使用することができます．
- **数値型** は、 `long`, `decimal` と `bigint` を除いて、JS の数値にコンパイルされます。
- **配列** (および `ResizeArray`) は JS 配列にコンパイルされます。数値配列はほとんどの場合 [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray) にコンパイルされますが、インデックス作成、イテレート、マッピングのような一般的な操作では違いはないはずです。この挙動は `typedArrays` オプション](https://www.npmjs.com/package/fable-loader#options) で無効にできます。
- あらゆる **IEnumerable** (または `seq`) は JS でまるで [Iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators#Iterables) のようにトラバースできます。
- **DateTime** は JS の `Date` にコンパイルされます。
- **Regex** は、JS `RegExp` にコンパイルされます。
- Mutable **dictionaries** (F# maps ではありません) は [ES2015 Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) にコンパイルされます。
- Mutable **hashsets** (F# sets ではありません) は、[ES2015 Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) にコンパイルされます。

> 辞書やハッシュセットがカスタムまたは構造的な等式を必要とする場合、Fable はカスタム型を生成しますが、それは JS マップやセットと同じプロパティを共有することになります。

- **オブジェクト**。上記で見たように、レコードフィールドとインターフェースメンバーのみが、名前を変更することなくオブジェクトに添付されます。JS からオブジェクトを送ったり、受け取ったりする際には、この点を理解してください。

```fsharp
type MyRecord =
    { Value: int
      Add: int -> int -> int }
    member this.FiveTimes() =
        this.Value * 5

type IMyInterface =
    abstract Square: unit -> float

type MyClass(value: float) =
    member __.Value = value
    interface IMyInterface with
        member __.Square() = value * value

let createRecord(value: int) =
    { Value = value
      Add = fun x y -> x + y }

let createClass(value: float) =
    MyClass(value)
```

```js
import { createRecord, createClass } from "./Tests.fs";

var record = createRecord(2);

// OK、レコードフィールドを呼び出しています。
record.Add(record.Value, 2); // 4

// ダメ。このメンバーは実際にはオブジェクトにアタッチされていません。
record.FiveTimes();

var myClass = createClass(5);

// ダメ。
myClass.Value;

// Ok, this is an interface member
// OK, これはインタフェース型のメンバーです。
myClass.Square(); // 25
```

## 関数: 自動カリー化の解除

Fable は、関数として渡されたとき、レコードフィールドとして設定されたときなど、多くの場面で自動的に関数のカリー化を解除します。そのため、ほとんどの場合、JS との間で、カリー化された引数を持たない関数のように受け渡しすることができます。

```fsharp
let execute (f: int->int->int) x y =
    f x y
```

```js
import { execute } from "./TestFunctions.fs";

execute(
  function (x, y) {
    return x * y;
  },
  3,
  5
); // 15
```

> F#(および他の関数型言語)における関数のカリー化については、[this](https://fsharpforfunandprofit.com/posts/currying/) を確認してください。

### デリゲートの曖昧さ回避のための使用

Fable のカリー化解除のメカニズムが、特に他の関数を返す関数とこんがらがってしまうような状況がいくつかあります。次のような例を考えてみましょう。

```fsharp
open Fable.Core.JsInterop

let myEffect() =
    printfn "Effect!"
    fun () -> printfn "Cleaning up"

// JSモジュールからのメソッドで、破棄のために別の関数を返すことを期待する。
let useEffect (effect: unit -> (unit -> unit)): unit =
    importMember "my-js-module"

// ダメ。Fableはこれが二項関数だと思い込んでいます。
useEffect myEffect
```

この問題は、コンパイラが `unit -> unit -> unit` と `unit -> (unit -> unit)` を区別できず、二項ラムダ（二つの引数を受け取る関数）としか見れないということです。これは、すべてのコードが F# あれば問題にはなりませんが、このケースのように関数を JS に送信している場合、Fable は間違ってそれをカリー化解除しようとし、予期しない結果を引き起こします。

このようなケースを解消するために、`System.Func`のような [delegate](https://docs.microsoft.com/ja-jp/dotnet/fsharp/language-reference/delegates) を使うと、カリー化されないようにできます。

```fsharp
open System

// デリゲートを使用することで曖昧さを解消できます。
let useEffect (effect: Func<unit, (unit -> unit)>): unit =
    importMember "my-js-module"

// うまくいきます。
useEffect(Func<_,_> myEffect)
```

## JS ファイルから Fable のソースコードを呼び出す

Webpack はローダーを使うことで、異なるプログラミング言語のファイルをプロジェクトにインクルードすることをとても簡単にします。Fable のプロジェクトでは、すでに [fable-loader](https://www.npmjs.com/package/fable-loader) を使っていることが前提なので、次のようなファイルがあった場合、

```fsharp
module HelloFable

let sayHelloFable() = "Hello Fable!"
```

JS からインポートして使うのは、まるで別の JS ファイルのように簡単です。

```js
import { sayHelloFable } from "./HelloFable.fs";

console.log(sayHelloFable());
```

### Typescript から Fable のコードをインポートする

良くも悪くも、Typescript はインポートしたモジュールをチェックしたいのですが、F# のことを全く知らないので、.fs ファイルをインポートしようとすると文句を言われます。

```ts
// ファイルを読み込むことができないので、Typescript は文句を言うでしょう。
import { sayHelloFable } from "./HelloFable.fs";
```

Typescript コンパイラに満足いただけるよう、[宣言ファイル](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html) が必要です。このファイルは、Fable コードから実際に何がエクスポートされるかを Typescript に伝える機会も与えてくれます。次のような `HelloFable.d.ts` ファイルを置くと、上記の import はうまくいくでしょう。

```ts
// 残念ながら、宣言ファイルは相対パスを受け付けないので、
// ワイルドカードの * を使用します。
declare module "*HelloFable.fs" {
  function sayHelloFable(): string;
}
```

## Fable のコンパイルされたコードをライブラリとして呼び出す

### ...あなたのウェブアプリから

プロジェクトが Web アプリで Webpack を使用しているのであれば、Webpack のコンフィギュレーションで `module.exports` の `output` セクションに 2 行のコードを書くだけでよいでしょう。

```js
libraryTarget: 'var',
library: 'MyFableLib'
```

たとえば、

```js
    output: {
        path: path.join(__dirname, "./public"),
        filename: "bundle.js",
        libraryTarget: 'var',
        library: 'MyFableLib'
    },
```

これは、Fable のコードを`MyFableLib`というグローバル変数から呼びたいことを Webpack に伝えています。これでおしまいです。

!!! info
    プロジェクトの **最後のファイル** にあるパブリック関数と値のみが公開されます。

#### 試してみましょう。

上記の HelloFable アプリを、次の内容を含む webpack.config.js でコンパイルしてみましょう。

```js
output: {
    ...
    libraryTarget: 'var',
    library: 'OMGFable'
}
```

では、これを `index.html` ファイルで直接試してみましょう。

```html
<body>
    <script src="bundle.js"></script>
    <script type="text/JavaScript">
      alert( OMGFable.sayHelloFable() );
    </script>
</body>
```

出来上がり！[完全なサンプルはこちら](https://github.com/fable-compiler/fable2-samples/tree/master/interopFableFromJS)

### ...あなたのNode.jsアプリから

ほぼ同じです。もし `var` の代わりに `commonjs` を出力した完全なサンプルを見たい場合は、[このプロジェクトをチェック](https://github.com/fable-compiler/fable2-samples/tree/master/nodejsbundle) してください。そこでは、Webpackのconfigに次の行を追加していることがわかります。

```js
    library:"app",
    libraryTarget: 'commonjs'
```

というように、JavaScriptからコードを呼び出すことができます。

```js
let app = require("./App.js");
```


### Webpack `libraryTarget` についてもっと知る。

オプションの内容を知りたいのであれば、[公式ドキュメント](https://webpack.js.org/configuration/output/#outputlibrarytarget) をご覧ください。