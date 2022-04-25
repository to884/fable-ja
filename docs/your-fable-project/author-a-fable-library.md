# Fable ライブラリを作ろう

Fable で使えるライブラリを書くには、次のような条件をクリアするだけです。

- [Fable と互換性のない](https://fable.io/docs/dotnet/compatibility.html) FSharp.Core/BCL の API を使わない。
- シンプルな `.fsproj` ファイルにすること: MSBuild の条件などは使用しないこと。
- NuGet パッケージに含まれるソースファイルは、`fable` という名前のフォルダーに入れる。

最後の項目はややこしく思えるかもしれませんが、プロジェクトファイルに数行追加するだけで、あとは全て `dotnet pack` コマンドがやってくれます。

```xml
<!-- NuGet パッケージの "fable "フォルダにソースファイルを追加します -->
<ItemGroup>
    <Content Include="*.fsproj; **\*.fs; **\*.fsi" PackagePath="Fable Filter" />
    <Content Include="*.fsproj; **\*.fs" PackagePath="fable Filter" />
</ItemGroup>
```

これだけで、あなたのライブラリが Fable と互換性を持つようになります。NuGet にパッケージを公開するには、[Microsoft のドキュメント](https://docs.microsoft.com/en-us/nuget/quickstart/create-and-publish-a-package-using-the-dotnet-cli) をチェックするか、または [Fake](https://fake.build/dotnet-nuget.html#Creating-NuGet-packages) を使ってもいいでしょう。

## テスト

公開する前に、あなたのライブラリのユニットテストを書いて、すべてが期待通りに動くことを確認するのは良いことです。そのための最も簡単な方法は、[このサンプル](https://github.com/fable-compiler/fable2-samples/tree/master/mocha) のように、[Mocha](https://mochajs.org/) のような JS テストランナーを使用することです。また、[Fable.Mocha](https://github.com/Zaid-Ajaj/Fable.Mocha)のような Fable プロジェクト用のツールを含むライブラリを使うことも可能です。
