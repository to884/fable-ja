# Fable ライブラリを使おう

.NET のパッケージマネージャのデファクトである [NuGet](https://www.nuget.org/) を使ってライブラリを利用することが少なくありません。

そのため、ライブラリが必要になります。Fable では、数多くのライブラリが提供されており、すぐに利用することができます。

- [Fable.Core](https://www.nuget.org/packages/Fable.Core/) は、すべての Fable プロジェクトに必要なライブラリです。
- [Fable.Browser.Dom](https://www.nuget.org/packages/Fable.Browser.Dom): すべての DOM に関する必要なものです。（windows、document）
- [Fable.Elmish.React](https://www.nuget.org/packages/Fable.Elmish.React) Elm アーキテクチャと React をレンダリングエンジンとして使用した Web アプリケーションを作成するためのツールです。
- [Thoth.Json](https://www.nuget.org/packages/Thoth.Json): JSON シリアライゼーションに使います。

> すべての NuGet ライブラリが Fable で動作するわけではないことに注意してください。Fable に対応しているかどうかを調べるには、ライブラリのドキュメントを参照してください。

Fable のライブラリを呼び出すには、2 つの方法があります。

1. プロジェクトファイル内で直接参照する
2. [Paket](https://fsprojects.github.io/Paket/) を使う。

## Option 1: プロジェクトファイルからライブラリを手動で参照する

`.fs` ファイルと同じように、`.fsproj` ファイルで直接ライブラリを参照することができます。

その際、どのライブラリを使いたいのか、また、どのバージョンを使いたいのかを伝える必要があります。たとえば、`Fable.Browser.Dom` のバージョン `1.0.0` の場合、`.fsproj` ファイルに次のノードを追加します。

```xml
<PackageReference Include="Fable.Browser.Dom" Version="1.0.0" />
```

このように、ライブラリの標準的なフォーマットになっています。

```xml
<PackageReference Include="[PACKAGE_ID]" Version="[PACKAGE_VERSION]" />
```

dotnet SDK では、.fsproj を手動で編集せずにこの操作を行うための CLI コマンドが用意されています。また、バージョン番号を省略した場合は、自動的に最新の安定版を採用してくれます。たとえば、

`dotnet add package Fable.Elmish.React [-v 3.0.1]` といった具合です。

> Visual Studio や Rider のような IDE でも、グラフィックインターフェイスに NuGet パッケージを管理するためのオプションが含まれています。

基本的に必要なのはこれだけです。ビルドプロセスが自動的にライブラリをダウンロードし、それに対してコードをコンパイルします。

> もし、ビルドの前にパッケージをダウンロードする必要がある場合（たとえば、IDE でのエラーを取り除くため）、.fsproj ファイルがあるフォルダで `dotnet restore` コマンドを実行します。

## オプション 2: Paket を使う

ライブラリを追加する 2 つ目の方法は、[Paket](https://fsprojects.github.io/Paket/) ライブラリマネージャを使った方法です。これは強制ではありませんが、たいていの場合、大規模なプロジェクトでは推奨される方法です。

Paket の使用方法は、[公式ドキュメント](https://fsprojects.github.io/Paket/get-started.html) に従えば、一目瞭然です。

さらに、より簡単に使えるよう、私たちは [サンプル](https://github.com/fable-compiler/fable2-samples/tree/master/withpaket) を作りました。これは、あなたが paket のドキュメントを読んでいる間、役に立つことでしょう。

たいていは、Paket を使い始めるには数分しかかかりません。
