# プロジェクトファイル

JS とは異なり、F#プロジェクトでは、すべてのソースを **コンパイル順** に `.fsproj` ファイルに記述する必要があります。一見すると大変な制約のように見えますが、これには[なかなかのメリット]があります(https://fsharpforfunandprofit.com/posts/cyclic-dependencies/)。
F#プロジェクトは .NET エコシステムから発展したものなので、F#プロジェクトにファイルを追加するには、いくらか決まった手順を踏む必要があります。

!!! info
    数ある F# IDE の中には、プロジェクトの作成やファイルの追加・削除といった操作を行うためのコマンドが既に用意されています。以下の手順は、これを手動で行う場合にだけ必要です。

## .fsproj ファイルを作成する

`.fsproj` ファイルは XML 形式です。これはやや古臭く見えるかもしれませんが、幸いなことに、最近のバージョンでは F#プロジェクトの基本スキーマはずっとシンプルになりました。今では、ほとんどのプロジェクトのスケルトンは次のようになっています。

```xml
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>
    <ItemGroup>
        <!-- List your source files here -->
    </ItemGroup>
    <ItemGroup>
        <!-- List your package references here if you don't use Paket -->
    </ItemGroup>
</Project>
```

## .fs ファイルをプロジェクトに追加する

F#のソースファイルは拡張子が `.fs` で終わります。新しいものをプロジェクトに含めるには、`.fsproj` に次のタグを使った相対パスで追加するだけです。

```xml
<ItemGroup>
    <Compile Include="path/to/my/File.fs" />
<ItemGroup>
```

たとえば、`MyAwesomeFeature.fs` と `App.fs` という2つのファイル（最後のファイルにはエントリポイントが含まれています）を持つアプリがあるとすると、次のようになります。

```xml
<ItemGroup>
    <Compile Include="MyAwesomeFeature.fs" />
    <Compile Include="App.fs" />
<ItemGroup>
```

!!! info
    たとえば、`App.fs` が `MyAwesomeFeature.fs` を呼び出す場合、`MyAwesomeFeature.fs` を `App.fs` よりも上に記述しなくてはならないことに気を付けてください。

さらに、`Authentication.fs` というファイルを `Shared` フォルダに追加します。このフォルダは `src` フォルダと同じ階層にあります（例えば、サーバーなどの他のプロジェクトとファイルを共有している場合などに起こります）。それでは、現在のプロジェクトツリーの状態を見てみましょう。

```
myproject
    |_ src
        |_ MyAwesomeFeature.fs
        |_ App.fs
        |_ App.fsproj
    |_ Shared
        |_ Authentication.fs
```

これは、次のようにプロジェクトファイル内で表現できます。

```xml
<ItemGroup>
    <Compile Include="../Shared/Authentication.fs" />
    <Compile Include="MyAwesomeFeature.fs" />
    <Compile Include="App.fs" />
<ItemGroup>
```

注目したいのは、Fable は F#のソースファイルを JS の `import` を使って [ES2015 モジュール](https://exploringjs.com/es6/ch_modules.html)に変換する点です。つまり、他のファイルから参照されていないファイルは、副作用を含めて **無視して実行されます**（最後のファイルを除きます）。これは、プロジェクトの他の部分との関係にかかわらず、すべてのファイルがコンパイルされ実行される.NET とは異なる動作です。
