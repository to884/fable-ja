# let キーワード

`#!fshap let` は、何か値を名前に割り当てるために使われる F# のキーワードで、文字列や整数などのいわゆるプリミティブ型や、関数、配列やレコードなどの複雑な構造体に結びつけるために使います。

ここでは，文字列を `x` という名前の識別子に結びつける方法を紹介します。

```fsharp
let x = "some text"
```

たとえば JavaScript のような他の言語では、このようなスニペットは定数とみなされます。F# は `#!javascript var` や `#!javascript const` はなく、`#!fsharp let` しかありません。F# はあらゆる値が既定ではイミュータブルなので、このスニペットは定数と同じものです。

F# の `#!fsharp let` は JavaScript の `#!fsharp let` とは違うので、このページで後ほど説明します。

関数については後ほど詳しく説明しますが、ここでは関数を `add` という名前の識別子に結びつける方法を説明します。

```fsharp
let add x y =
    x + y
```

上の例では、2 つの整数を足す関数が `add` という名前の識別子に結び付けられ、2 つの値が `x` と `y` という名前の識別子に結び付けられます。

## シャドウイング

F# では、通常のバインディングはイミュータブルで、名前を付けたバインディングに値を再割り当てすることはできません。

たとえば、次のような例です。

```fsharp
let x = "the answer"
let x = 42
```

この場合、 `x` は新たに定義されるわけでも、`#!fsharp string` から `#!fsharp int` に型が変更されるわけでもなく、同じ名前の `x` を違う値にバインドしていることになります。
もちろん、この例では、1 つのバインディングが他のバインディングのすぐ後に起こっているので、あまり役に立ちませんが、次のように考えてみてください。

```fsharp
let printName name =
    let stripLastName name =
        if (String.exists (fun c -> c = ' ') name) then
            name.Split([|' '|]).[0]
        else
            name
    printfn "%s" name // Will print "John Doe"
    printfn "%s" (stripLastName name) // Will print "John"

printName "John Doe"
```

このスニペットの主な目的は、関数 `printName` が `name` という名前の引数を受け取り、その本体で別の関数 `stripLastName` を定義して、その引数も `name` を受け取ることを確認することなので、上記のコードを完全に理解できなくてもあまり気にしないでください。
`stripLastName` のスコープ内では、 `name` 引数は新しいバインディングを作成し、 `printName` 関数で受け取った `name` 引数を上書きしています。
そして、このことは `printName` 関数の最後にある 2 つの出力で確かめることができます。

## JavaScript との比較

F# の `#!fsharp let` キーワードが JavaScript のそれと大きく違う点は、次のとおりです。

- JavaScript では、`#!fsharp let` を使って名前の付いた変数を定義し、その値を再代入することができますが、F# はそのようなことはありません。
- JavaScript では、`#!fsharp let` はスコープにバインドされているので、別のスコープであれば、すでに使われている名前で新しい変数を宣言することができますが、F# は、そのような変更は同じスコープの中だけであればできます。

## 参考情報

- [`#!fsharp let` バインディング](https://docs.microsoft.com/ja-jp/dotnet/fsharp/language-reference/functions/let-bindings)