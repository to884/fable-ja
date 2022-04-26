# Fable とは

Fable は、[F#](https://fsharp.org/) を使用して JavaScript エコシステムで動作するアプリケーションを構築するためのコンパイラです。

```fsharp
type Shape =
    | Square of side: double
    | Rectangle of width: double * length: double

let getArea shape =
    match shape with
    | Square side -> side * side
    | Rectangle (width, length) -> width * length

let square = Square 2.0
printfn $"The area of the square is {getArea square}"
```

## F# とは

F#（エフシャープと発音）は強く型付けされた関数型プログラミング言語であり、堅牢で保守性の高いコードを構築するために、次のような多くの優れた機能を備えています。

-   軽量な構文
-   デフォルトで言語に組み込まれた不変性
-   データやドメインを簡単に表現できる豊富な型
-   複雑な振る舞いを定義するための強力なパターンマッチ
-   などなど...

F# は、Web アプリケーションやクラウドアプリケーションのサーバーですでに使われていますが、データサイエンスや機械学習にもかなり多く使われています。優れた汎用言語であり、ブラウザ上で動作する美しい UI を構築するのにも適しています。

## 新しい JavaScript プロジェクトになんで F# を使うの？

F# はブラウザ上で動作する美しいアプリケーションを構築するのに最適な選択肢です。F# は、

-   軽量な構文による簡潔さ
-   優れた型システムとパターンマッチによる堅牢性
-   言語に組み込まれた不変性による安全性
-   大企業（Microsoft や Jetbrains など）にサポートされ、商用ツールのサポートが受けられる

JavaScript と比較すると、F# より安全で、より堅牢で、読み書きが快適です。

F# は関数型プログラミングやオブジェクトプログラミングの機能を持つ成熟した言語ですが、これらを提供するために読みやすさやシンプルさを犠牲にしているわけではありません。そのため、新しい JavaScript アプリケーションには最適な選択肢だと考えています。
