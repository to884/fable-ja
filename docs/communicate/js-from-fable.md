# Fable から JS を呼び出す

## Fable から JavaScript を呼び出す

相互運用性は、静的に型付けされた F# コードと、型付けされていない動的な JS コードとの間にある信頼の問題です。リスクを和らげるために、Fable はいくつかの方法を用意しており、そのうちの一つがインターフェース契約による型安全性です。場合によっては、もっと動的な方法で JS コードを呼び出す方が便利に思えるかもしれませんが、そうすると潜在的なバグが実行時まで見つからないので、注意が必要です。

ここでは、安全な方法と動的な方法の両方について説明し、あとはあなたの判断にお任せします。それでは始めましょう。

### プロジェクトに JS ライブラリを追加する

まず始めにすることは、プロジェクトにライブラリを追加することです。私たちはいつも `package.json` ファイルを使っているので、`npm install my-awesome-js-library` を使って、必要なライブラリをプロジェクトに追加するだけです。そうすると、ライブラリは `node_modules` フォルダで利用可能になります。

> もし、ライブラリがファイルとして存在する場合は、このステップを省略してください。

### Imports とインターフェースによる型安全性

JS ライブラリのコードを使うには、まず F# でインポートする必要があります。Fable は [ES2015 imports](https://developer.mozilla.org/en/docs/web/JavaScript/reference/statements/import) を使用します。これは後で [Babel](https://babeljs.io/docs/en/plugins#modules) によって `commonjs` や `amd` など他の JS モジュールシステムへ変換することができます。

Fable で ES2015 のインポートを宣言するには、**Import 属性**と**import 式**の 2 つの方法があります。`ImportAttribute` はメンバー、型、またはモジュールを修飾することができ、次のように動作します。

```fsharp
// 名前空間のインポート
[<Import("*", from="my-module")>]
// import * from "my-module"

// メンバーのインポート
[<Import("myFunction", from="my-module")>]
// import { myFunction } from "my-module"

// デフォルトのインポート
[<Import("default", from="express")>]
// import Express from "express"
```

また、次のエイリアス属性も使用できます。

```fsharp
open Fable.Core
open Fable.Core.JsInterop

// Import("*", "my-module") と同じです。
[<ImportAll("my-module")>]
let myModule: obj = jsNative

// ("default", "my-module") と同じです。
[<ImportDefault("my-module")>]
let myModuleDefaultExport: obj = jsNative

// メンバー名は、修飾された値、ここでは `myFunction` から取得されます。
[<ImportMember("my-module")>]
let myFunction(x: int): int = jsNative
```

JS でグローバルにアクセスできる値であれば、代わりにオプションの `name` パラメータで `Global` 属性を使うこともできます。

```fsharp
let [<Global>] console: JS.Console = jsNative

// F# コードで別の名前を使いたい場合は、文字列の引数を渡すこともできます。
let [<Global("console")>] logger: JS.Console = jsNative
```

#### 出力ディレクトリを使った相対パスのインポート

Fable の `-o` オプションを使って生成された JS ファイルを出力ディレクトリに配置しているのであれば、Fable は .fs ソースに対応するファイルのみを移動し、.js や .css などの外部ファイルを移動しない点に注意が必要です。Fable は import の相対パスを生成されたファイルの場所に自動的に調整しますが、生成されたファイルが最終的にどこにあるか正確に知ることができないこともあるでしょう。このような場合は、出力ディレクトリに置き換えられる `${outDir}` マクロを使うと便利です。たとえば、次のような CSS モジュールをインポートする場合です。

```fsharp
[<ImportDefault("${outDir}/../styles/styles.module.css")>]
let styles: CssModule = jsNative
```

たとえば、`-o build` オプションでコンパイルして、`build/Components` ディレクトリにファイルが生成されたとしましょう。生成されたコードは次のようになります。

```js
import styles from "../../styles/styles.module.css";
```

#### オブジェクト指向プログラミング（OOP） クラス定義と継承

JS クラスをインポートする必要があるとしたら、F# は標準的なクラス宣言に `Import` 属性を付けて表現することができます。この場合、実際の実装は JS から提供されるので、メンバーのダミー実装として `jsNative` を使います。jsNative` を用いる場合、メンバーに戻り値の型を追加するのをお忘れなく！

```fsharp
[<Import("DataManager", from="library/data")>]
type DataManager<'Model> (conf: Config) =
    member _.delete(data: 'Model): Promise<'Model> = jsNative
    member _.insert(data: 'Model): Promise<'Model> = jsNative
    member _.update(data: 'Model): Promise<'Model> = jsNative
```

この時点で、通常の F# で行われているように、メンバーを使ったり、継承したりすることが可能になります [F# 公式ドキュメントを参照](https://docs.microsoft.com/ja-jp/dotnet/fsharp/language-reference/inheritance)。F# は、メンバーを継承してオーバーライドする場合、基底クラスでメンバーを `#!fsharp abstract` と宣言することが要求され、基底クラスから直接使用できるようにする場合は、既定の実装を用いることが必要になります。たとえば、次のようになります。

```fsharp
// このクラスはJSに存在します。
[<Import("DataManager", from="library/data")>]
type DataManager<'T> (conf: Config) =
    abstract update: data: 'T -> Promise<'T>
    default _.update(data: 'T): Promise<'T> = jsNative

// このクラスはわたしたちのコードに存在します。
type MyDataManager<'T>(config) =
    inherit DataManager<'T>(config)
    // データを基底クラスに送る前に、データに対して何かをすることもできます。
    override _.update(data) =
        base.update(data)

let test (data: 'T) (manager: DataManager<'T>) =
    manager.update(data) |> ignore

// これは、テストがDataManagerを期待していても、MyDataManager.updateが呼び出されます。
MyDataManager(myConfig) |> test myData
```

!!! info
    インポートしたい JS コードの Typescript 宣言があれば、[ts2fable](https://github.com/fable-compiler/ts2fable) を使って、F# パインディングを記述することができます。

#### 練習してみましょう! 最初の挑戦

さて、ここまで見てきたところで、[interop](https://github.com/fable-compiler/fable3-samples/tree/master/interop) のサンプルのコードを確認してみましょう。

Fable プロジェクトで使いたい `alert.js` ファイルがあるとします。

```js
function triggerAlert(message) {
  alert(message);
}

const someString = "And I Like that!";

export { triggerAlert, someString };
```

ご覧のように、1 つの関数 `triggerAlert` と 1 つの定数 `someString` が用意されています。両方とも ES6 の `export` キーワードを使ってエクスポートされています。

これを Fable のコードで使用するために、これを模倣した `interface` を作成してみましょう。

```fsharp
  open Fable.Core.JsInterop // interop tools を呼ぶのに必要です。

  type IAlert =
    abstract triggerAlert : message:string -> unit
    abstract someString: string
```

ご覧の通り、処理はいたって簡単です。`IAlert` の `I` は必須ではありませんが、インターフェイスを使うという意味で重要なヒントになります。`abstract` キーワードは F# に具体的な実装がないことを表しているに過ぎません。
そしてそれは、実際その通りです。私たちは JavaScript の実装に頼っているのですから。

では、これを使ってみましょう。

```fsharp
  [<ImportAll("path/to/alert.js")>]
  let mylib: IAlert = jsNative
```

ここでは、`ImportAll` 属性を使います。これは、先ほど説明したように `Import("*", "path/to/alert.js")` と同じ意味合いです。

- step 1: 使いたい要素を指定します。ここで `*` というのは 「エクスポートされたものをすべて取り込む」という意味です。
- step 2: js ライブラリのパスを設定します。
- step 3: js ライブラリをマッピングするために、`mylib` という `let` バインディングを作成します。
- step 4: `jsNative` キーワードを使用して、`mylib` が JavaScript ネイティブ実装のプレースホルダであることを示します。

これで使えるようになりましたね。

```fsharp
mylib.triggerAlert ("Hey I'm calling my js library from Fable > " + mylib.someString)
```

すべてが正しく動作していれば、ブラウザにアラートポップアップが作成されるはずです! もちろん、このサンプルは Web アプリを対象としていますが、Node.js アプリでも同じことができます。

#### 練習してみましょう! 2 回目の挑戦

`Canvas.js` ファイルからエクスポートされた 2 つの関数があると仮定します。それぞれ、`drawSmiley` と `drawBubble` です。

```js
export function drawSmiley(id) {
  // 何かする
}
export function drawBubble(id) {
  // 何かする
}
```

これは、`alert.js`で使ったのと同じ方法が使えます。

```fsharp
  open Fable.Core.JsInterop // interop tools を呼ぶのに必要です。

  type ICanvas =
    abstract drawSmiley: (id:string) -> unit
    abstract drawBubble: (id:string) -> unit

  [<ImportAll("path/to/Canvas.js")>]
  let mylib: ICanvas = jsNative

  mylib.drawSmiley "smiley" // など。
```

または、`importMember` ヘルパー関数を 使って、js 関数と F# 関数を直接対応させることもできます。

```fsharp
open Fable.Core.JsInterop // interop tools を呼ぶのに必要です。

module Canvas =
  // ここでは、canvas.jsからdrawSmileyというメンバ関数をインポートしています。
  let drawSmiley(id:string): unit = importMember "path/to/Canvas.js"
  let drawBubble(id:string): unit = importMember "path/to/Canvas.js"

Canvas.drawSmiley "smiley"
```

結果は同じでしょうが、考え方が微妙に違うのです。それは要するにあなたの選択次第です 😉。

#### その他のインポートヘルパー

Fable.Core.JsInterop` のおかげで、他にも使える相互運用ヘルパーが用意されています。

```fsharp
open Fable.Core.JsInterop

let buttons: obj = importAll "my-lib/buttons"
// JS: import * as buttons from "my-lib/buttons"

// 関数宣言でも使えます
let getTheme(x: int): IInterface = importDefault "my-lib"
// JS: import getTheme from "my-lib"

let myString: string = importMember "my-lib"
// JS: import { myString } from "my-lib"

// ImportAttribute と同様に、メンバー名を指定するには、
// `import` とだけ記述します。
let aDifferentName: string = import "myString" "my-lib"
// import { myString } from "my-lib"
```

ときには、純粋にその副作用のために JS をインポートすることが必要な場合があります。たとえば、ブラウザポリフィルの中には、関数をエクスポートせず、実行時にブラウザの組み込み DOM 型を拡張するものがあります。

その場合は、`importSideEffects` を使います。

```fsharp
open Fable.Core.JsInterop

importSideEffects("my-polyfill-library") // npmパッケージから。

importSideEffects("./my-polyfill.js") // ローカル.jsファイルより
```

### Emit, F# では不十分な場合

`Emit` 属性を使うと、関数を修飾することができます。関数を呼び出すと、インラインでその属性の内容に置き換えられ、プレースホルダー `$0, $1, $2...` が引数に置き換わります。たとえば、次のようなコードを書くと、次のような JavaScript が生成されます。

```fsharp
open Fable.Core

[<Emit("$0 + $1")>]
let add (x: int) (y: string): string = jsNative

let result = add 1 "2"
```

正確な引数の数がわからないときは、次のような構文を使えます。

```fsharp
type Test() =
    [<Emit("$0($1...)")>]
    member __.Invoke([<ParamArray>] args: int[]): obj = jsNative
```

また、オプションのパラメータに条件付きのシンタックスを渡すことも可能です。

```fsharp
type Test() =
    [<Emit("$0[$1]{{=$2}}")>]
    member __.Item with get(): float = jsNative and set(v: float): unit = jsNative

    // この構文は、JSで2番目の引数がtrueになったら'i'を表示し、
    // そうでなければ何も表示しないことを意味します。
    [<Emit("new RegExp($0,'g{{$1?i:}}')")>]
    member __.ParseRegex(pattern: string, ?ignoreCase: bool): Regex = jsNative
```

`Emit` の内容は実際には [Babel](https://babeljs.io/) によってパースされるので、やはり何らかの形で検証が行われます。しかし、この方法を乱用することはお勧めしません。なぜなら、F# コンパイラによってコードがチェックされないからです。

#### やってみよう！ Emit を使う

では、Emit を使って、次の `MyClass.js` のような新しい例を見てみましょう。

```js
export default class MyClass {
  // コンストラクタには、 `value` と `awesomeness` フィールドを持つ
  // オブジェクトを渡すことに注意してください。
  constructor({ value, awesomeness }) {
    this._value = value;
    this._awesomeness = awesomeness;
  }

  get value() {
    return this._value;
  }

  set value(newValue) {
    this._value = newValue;
  }

  isAwesome() {
    return this._value === this._awesomeness;
  }

  static getPI() {
    return Math.PI;
  }
}
```

そのメンバーを列挙してみましょう。

- 現在の値をゲッターとセッターで返す `value` メンバー。
- 現在の値がすげぇ値と等しいかどうかをチェックする `isAwesome` メソッド。
- 静的メソッド `getPi()` は、単に `Math.PI` の値を返します。

以下は、Fable の実装です。まず、メンバーから見ていきましょう。

```fsharp
type MyClass<'T> =
  // このプロパティはセッターも持っていることに注意してください。
  abstract value: 'T with get, set
  abstract isAwesome: unit -> bool
```

さて、コンストラクタを含む静的関数を呼び出せるようにする必要があります。なので、そのために別のインターフェイスを書きます。

```fsharp
type MyClassStatic =
  [<Emit("new $0({ value: $1, awesomeness: $2 })")>]
  abstract Create: 'T * 'T -> MyClass<'T>
  abstract getPI : unit-> float
```

!!! info
    前述の DataManager のようにダミー実装を含むクラス宣言を使うこともできますが、F# 型システムの制限を克服したり、JS クラスを値として扱えるようにするために、Fable バインディングでは JS 型のインスタンスと静的部分を二つのインターフェースに分けることが一般的であることがわかるでしょう。この場合、慣習的に `Create` はコンストラクタを表します。

ここでは、JS の `new` キーワードを適用するために `Emit` 属性を使用し、MyClass のコンストラクタが受け入れる引数で JS オブジェクトを作成しています。ここで `$0` はインターフェースオブジェクト（この場合、MyClass static）を表している点に注意してください。

最後になりましたが、MyClass をインポートしましょう。

```fsharp
[<ImportDefault("../public/MyClass.js")>]
let MyClass : MyClassStatic = jsNative
```

これで、私たちの JS クラスを使用することができるようになりました。それでは、コードの全体像を見てみましょう。

```fsharp

type MyClass<'T> =
  abstract value: 'T with get, set
  abstract isAwesome: unit -> bool

type MyClassStatic =
  [<Emit("new $0({ value: $1, awesomeness: $2 })")>]
  abstract Create: 'T * 'T -> MyClass<'T>
  abstract getPI : unit-> float

[<ImportDefault("../public/MyClass.js")>]
let MyClass : MyClassStatic = jsNative

let myObject = MyClass.Create(40, 42)

// ゲッターを使います。
let whatDoIget = myObject.value
mylib.triggerAlert ("Hey I'm calling my js class from Fable. It gives " + (string whatDoIget))

// セッターを使います。
myObject.value <- 42
mylib.triggerAlert ("Now it's better. It gives " + (string myObject.value))

// メンバー関数を呼び出します。
mylib.triggerAlert ("Isn't it awesome? " + (string (myObject.isAwesome())))

// 静的関数を呼び出します。
mylib.triggerAlert ("PI is " + (string (MyClass.getPI())))
```

`Import` と `Emit` 属性を組み合わせることもできます。つまり、MyClass のインポートとビルドを一度に行えるわけです。この場合、 `$0` はインポートされた要素に置き換えられる点に注意してください。

```fsharp
[<ImportDefault("../public/MyClass.js")>]
[<Emit("new $0({ value: $1, awesomeness: $2 })")>]
let createMyClass(value: 'T, awesomeness: 'T) : MyClass<'T> = jsNative
```

### その他の特別な属性

`Fable.Core` には、JS の相互運用のために、次のような属性があります。

#### Erase 属性

TypeScript には [共用型](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)という考え方がありますが、これは **F# の共用型とは違います**。前者は、関数の引数が異なる型を受け入れるかどうかを静的にチェックするために使われるだけです。Fable では、それらは **Erased Union Types** と呼ばれており、そのケースは 1 つだけのデータフィールドを持たなければなりません。コンパイル後、ラップは消去され、データフィールドだけが残ります。消去された共用型を定義するには、その型に `Erase` 属性をつければいいだけです。たとえば、次のようになります。

```fsharp
open Fable.Core

[<Erase>]
type MyErasedType =
    | String of string
    | Number of int

myLib.myMethod(String "test")
```

```js
myLib.myMethod("test");
```

`Fable.Core` には、あらかじめ定義された消去型が含まれており、次のように使うことができます。

```fsharp
open Fable.Core

type Test = abstract Value: string
let myMethod(arg: U3<string, int, Test>): unit = importMember "./myJsLib"

let testValue = { new Test with member __.Value = "Test" }
myMethod(U3.Case3 testValue)
```

`U2`、`U3` `...` を受け入れるメソッドに引数を渡す場合、シンタックスシュガーとして `!^` を使うことができるので、大文字小文字を正確にタイプする必要がありません (引数は型チェックされます)。

```fsharp
open Fable.Core.JsInterop

myMethod !^5 // myMethod(U3.Case2 5)と同じ意味です。
myMethod !^testValue

// これはコンパイルできません。myMethod は浮動小数点数を受け付けません。
myMethod !^2.3
```

!!! info
    消去された共用体は主にインポートされた JS 関数のシグネチャを型付けすることを目的としたものであり、`Choice` の安易な置き換えでは無いことに注意してください。消去された共用体の型に対してパターンマッチングを行うことはできますが、これは型検査としてコンパイルされ、**型検査は Fable では非常に弱いので**、消去された共用体の一般的な引数が JS ランタイムで簡単に区別できる型（文字列、数値、配列など）になっているときのみお勧めします。

```fsharp
let test(arg: U3<string, int, float[]>) =
    match arg with
    | U3.Case1 x -> printfn "A string %s" x
    | U3.Case2 x -> printfn "An int %i" x
    | U3.Case3 xs -> Array.sum xs |> printfn "An array with sum %f"

// JSではおおよそ次のように変換されます。

// function test(arg) {
//   if (typeof arg === "number") {
//     toConsole(printf("An int %i"))(arg);
//   } else if (isArray(arg)) {
//     toConsole(printf("An array with sum %f"))(sum(arg));
//   } else {
//     toConsole(printf("A string %s"))(arg);
//   }
// }
```

#### StringEnum 属性

TypeScript では [String Enum](https://www.typescriptlang.org/docs/handbook/enums.html#string-enums) や [String Literal Types](https://mariusschulz.com/blog/string-literal-types-in-typescript) といった、文字列の値を持つ列挙型のようなものを定義することができます。Fable では、共用型と `StringEnum` 属性を使用することで、同じ機能を使えます。これらの共用型は、共用型の名前にマッチした文字列にコンパイルされるため、データフィールドを持ってはいけません。

既定では、コンパイルされた文字列は、最初の文字が小文字になります。これを防ぎたいとか、共用ケース名とは異なる文字列を使いたい場合には、 `CompiledName` 属性を使います。

```fsharp
open Fable.Core

[<StringEnum>]
type MyStrings =
    | Vertical
    | [<CompiledName("Horizontal")>] Horizontal

myLib.myMethod(Vertical, Horizontal)
```

```js
// js output
myLib.myMethod("vertical", "Horizontal");
```

#### TypeScriptTaggedUnion 属性

!!! info
    この機能を使用するには、Fable Tool >=3.6.2 と Fable.Core >=3.6.1 が必要です。

TypeScript には [判別共用体](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions) という考え方もあり、これは F# の共用型と似たような働きをしますが、アプローチが若干違います。

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

// 使い方
function describeShape(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return "circle of radius ${shape.radius}";
    case "square":
      return "square of length ${shape.sideLength}";
  }
}
```

ここで、`Shape` は `kind` という共通のフィールドを持っており、これが共用ケースを判別するための「タグ」として働きます。

F# の共用体とはまったく違った動きをしますが、特別な属性 `TypeScriptTaggedUnion` によって、まるで F# の共用体であるかのように扱うことができます。

```fsharp
type Circle =
    abstract kind: string
    abstract radius: float

type Square =
    abstract kind: string
    abstract sideLength: float

[<TypeScriptTaggedUnion("kind")>]
type Shape =
    | Circle of Circle
    | Square of Square

// 使い方
let describeShape (shape: Shape) =
    match shape with
    | Circle c -> $"circle of radius {c.radius}"
    | Square s -> $"square of length {s.sideLength}"
```

まず、`"!fsharp [<TypeScriptTaggedUnion("kind")>]` でタグフィールドの名前 (この場合は `kind`) を指定し、次に、その名前が "tag" フィールドの値と等しくなるように共用型の各ケースを定義し、最初の文字を大文字にします（例えば `"circle"` は `Circle` ）。もし、タグの値とはまるっきり違う名前を使いたい場合には、 `CompiledName` 属性を使用します。

```fsharp
[<TypeScriptTaggedUnion("kind")>]
type Shape =
    | [<CompiledName("circle")>] C of Circle
    | [<CompiledName("square")>] S of Square
```

"tag" フィールドが文字列ではなく、数値や列挙型になっていることもあります。

```typescript
enum ShapeKind {
  Circle = 1,
  Square = 2,
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

type Shape = Circle | Square;
```

このような場合には、 `CompiledName` の代わりに `CompiledValue` 属性を使います。

```fsharp
type ShapeKind =
    | Circle = 1
    | Square = 2

type Circle =
    abstract kind: ShapeKind
    abstract radius: float

type Square =
    abstract kind: ShapeKind
    abstract sideLength: float

[<TypeScriptTaggedUnion("kind")>]
type Shape =
    | [<CompiledValue(ShapeKind.Circle)>] Circle of Circle
    | [<CompiledValue(ShapeKind.Square)>] Square of Square
```

これらの共用型は 1 つのデータフィールドを 1 つだけ持たなければなりません。TypeScript の判別可能な共用体の多くはインターフェースの共用体であるため、まず対応する F# ンターフェースを定義し、それらをまとめて `TypeScriptTaggedUnion` にしたいところです。

!!! info
    F# の共用体型を `TypeScriptTaggedUnion` で置き換えようとしてはいけません。これは、`match` が効率の良い `switch` ステートメントにコンパイルされるためです。そのため、`TypeScriptTaggedUnion` を使用すると、無駄なキーストロークが増えるだけで、バグが発生する可能性があります。TypeScript から来る共用型にバインドする場合のみ、この機能を使うようにしましょう。

### プレーンオールド JavaScript オブジェクト

プレーンな JS オブジェクト(別名 POJO)を作成するには、`createObj` を使います。

```fsharp
open Fable.Core.JsInterop

let data =
    createObj [
        "todos" ==> Storage.fetch()
        "editedTodo" ==> Option<Todo>.None
        "visibility" ==> "all"
    ]
```

同様の効果は、新しい F# の[匿名レコード](https://devblogs.microsoft.com/dotnet/announcing-f-4-6-preview/) でも行えます。

```fsharp
let data =
    {| todos = Storage.fetch()
       editedTodo = Option<Todo>.None
       visibility = "all" |}
```

!!! info
    fable-compiler 2.3.6 以降、ダイナミックキャスト演算子 `!!! ` を使って匿名レコードをインターフェースにキャストするとき、匿名レコードのフィールドがインターフェースのものと一致しないと、Fable は警告を出します。この機能は JS との相互運用にのみ使います。F# コードでは、実装型を持たないインターフェイスをインスタンス化するための正しい方法は [オブジェクト式](https://fsharpforfunandprofit.com/posts/object-expressions/) になっています。

```fsharp
type IMyInterface =
    abstract foo: string with get, set
    abstract bar: float with get, set
    abstract baz: int option with get, set

// 警告："foo "は文字列でなければなりません。
let x: IMyInterface = !!{| foo = 5; bar = 4.; baz = Some 0 |}

// Warning, "bar" フィールドがありません
let y: IMyInterface = !!{| foo = "5"; bAr = 4.; baz = Some 0 |}

// OK, "baz "はオプションなので、なくても大丈夫です。
let z: IMyInterface = !!{| foo = "5"; bar = 4. |}
```

また、`createEmpty` を使ってインターフェースから JS オブジェクトを作成し、フィールドを手動で割り当てるといったこともできます。

```fsharp
let x = createEmpty<IMyInterface> // var x = {}
x.foo <- "abc"                    // x.foo = "abc"
x.bar <- 8.5                      // val.bar = 8.5
```

コンパイル時に Fable が直接 JS オブジェクトに最適化することもできる似たような解決策は、`jsOptions`ヘルパーを使うことです。

```fsharp
let x = jsOptions<IMyInterface>(fun x ->
    x.foo <- "abc"
    x.bar <- 8.5)
```

もうひとつの方法は、 `keyValueList` ヘルパーと組み合わせて、共用ケースのリスト (あるいは任意のシーケンス) を使うことです。これは、React の prop を表現するためによく使われます。ケース名の変換には大文字小文字のルールを指定できます（通常は最初の文字を小文字にします）。また、必要であれば、一部のケースを `CompiledName` 属性で装飾して、JS ランタイムでその名前を変更することもできます。

```fsharp
open Fable.Core.JsInterop

type JsOption =
    | Flag1
    | Name of string
    | [<CompiledName("quantity")>] QTY of int

let inline sendToJs (opts: JsOption list) =
    keyValueList CaseRules.LowerFirst opts |> aNativeJsFunction

sendToJs [
    Flag1
    Name "foo"
    QTY 5
]
// JS: { flag1: true, name: "foo", quantity: 5 }
```

!!! info
    Fable は、リストリテラルを直接 `keyValueList` に当てはめるときに、コンパイル時に変換を行うことができます。そのため、通常はヘルパーを含む関数をインライン化するのがよいでしょう。

### 動的型付け: これを読んじゃダメ!

先に説明したツールを使うことで、Fable はすべてのコードがコンパイラによってチェックされるため、（インターフェイス契約が正しい限り）やっかいなバグに陥らないよう保証してくれます。もしコンパイルできない場合は、JS ライブラリが存在しないか、そのパスが良くないか、F# の実装で足りないものがあるかのどれかです。私たちは、24 時間 365 日使われるシステム、Web アプリや Node.js アプリで、Fable を利用しています。コンパイルできれば、99%の確率で何の問題もなく動作することが分かっています。

私たちのモットーは 「コンパイルできれば、動く！」です。

しかし、私たちが述べたように、**相互運用は信頼の問題です**。もしあなたが JS のコードと F# コードを信用しているならば、さらにチェックすることなく進めてもいいのかもしれませんね。知らんけど。

!!! caution
    免責事項: 自己責任で使ってください。

#### 動的型付けって何？

`Fable.Core.JsInterop` は F# のダイナミック演算子を実装しているので、以下のようにオブジェクトのプロパティに名前で（静的チェックなしで）簡単にアクセスできます。

```fsharp
open Fable.Core.JsInterop

printfn "Value: %O" jsObject?myProperty
// JS: jsObject?myProperty

let pname = "myProperty"

printfn "Value: %O" jsObject?(pname) //  リファレンスによるアクセス
// JS: jsObject[pname]

jsObject?myProperty <- 5 // 代入もも可能
```

ダイナミック演算子とアプリケーションを組み合わせると、Fable は通常のメソッド呼び出しと同じようにタプル引数を分解します。また、これらの演算は連鎖して JS fluent API を再現できます。

```fsharp
let result = jsObject?myMethod(1, 2)
// JS: jsObject.myMethod(1, 2)

chart
    ?width(768.)
    ?height(480.)
    ?group(speedSumGroup)
    ?on("renderlet", fun chart ->
        chart?selectAll("rect")?on("click", fun sender args ->
            Browser.console.log("click!", args))

// chart
//     .width(768)
//     .height(480)
//     .group(speedSumGroup)
//     .on("renderlet", function (chart) {
//         return chart.selectAll("rect").on("click", function (sender, args) {
//             return console.log("click!", args);
//         });
//      });
```

JS で new キーワードを使って関数を呼び出す必要がある場合は、`createNew` を使います。

```fsharp
open Fable.Core.JsInterop

let instance = createNew jsObject(1, 2)
// JS: new jsObject(1, 2)
```

もし、動的な型付けのための演算子ではなく、拡張メンバを使いたいのであれば、 `Fable.Core.DynamicExtensions` を開いて、 `.Item` と `.Invoke` というメソッドを任意のオブジェクトで使えるようにできます。

```fsharp
open Fable.Core.DynamicExtensions

let foo = obj()
let bar1 = foo.["b"]  // Same as foo.Item("b")
foo.["c"] <- 14
let bar2 = foo.Invoke(4, "a")
```

#### ダイナミック・キャスト

JS から型付けされていないオブジェクトを受け取るとき、それを特定の型にキャストしたいこともあるでしょう。そういった場合には、F#  `unbox` 関数や `Fable.Core.JsInterop` の `!!!` 演算子が使えます。これは F# 型チェッカーをバイパスしますが、**Fable はキャストが正しいかどうかを確認するための実行時チェック**を追加しない点に注意してください。
