# MkDocs および　Material for MkDocs のインストール

## 要件

- `python3` がインストールされていること
- `pip3` がインストールされていること

## パッケージのインストール

以下のコマンドを実行し、MkDocs および Material for MkDocs をインストールします

### Linux の場合

```console
$ pip3 install --user mkdocs-material
$ pip3 install --user mkdocs-material-extensions
```

### Windows の場合

```pwsh
PS> py -m pip install --user mkdocs-material
PS> py -m pip install --user mkdocs-material-extensions
```

## プラグインのインストール

`mkdocs-git-revision-date-localized` を使用する場合は、以下のコマンドでプラグインをインストールします。

### Linux の場合

```console
$ pip3 install --user mkdocs-git-revision-date-localized-plugin
```

### Windows の場合

```pwsh
PS> py -m pip install --user mkdocs-git-revision-date-localized-plugin 
```

## ドキュメントの参照

MkDocs をブラウズモードで起動するとドキュメントを参照できます。

以下のコマンドをドキュメントのルートディレクトリで実行し、ローカルサーバーを起動します。

### Linux の場合

```shell
$ mkdocs serve
```

### Windows の場合

```pwsh
PS> py -m mkdocs serve
```

### ドキュメントの URL

MkDocs のサーバーが起動し出力された URL にアクセスして、ドキュメントを閲覧します。