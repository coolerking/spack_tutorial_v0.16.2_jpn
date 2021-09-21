# パッケージ作成チュートリアル

> **チュートリアルのセットアップ**
>
> 過去のセクションを完了していない場合は、Spackを次のように設定する必要があります。
>
> ```bash
> git clone https://github.com/spack/spack
> . spack/share/spack/setup-env.sh
> spack tutorial
> ```
>
> セットアップの詳細については、[基本的なインストールチュートリアル](01_basic.md) を参照してください。
> さらにヘルプが必要な場合は、Slackの `#tutorial` チャンネルに参加してください– [spackpm.herokuapp.com](spackpm.herokuapp.com) で招待状を入手してください。

このチュートリアルでは、単純なSpackパッケージを作成およびデバッグする手順について説明します。
さらなるSpackコマンドの経験を積むために、反復アプローチを使用してパッケージを開発およびデバッグします。
一貫性を保つために、MPIデバッグツールである `mpileaks` パッケージを作成します。

## Spackパッケージとは何？

Spackパッケージはインストールスクリプトであり、基本的にソフトウェアをビルドするためのレシピです。

これらは、ビルドのプロパティと動作を次のように定義します：

- ソフトウェアの場所と取得方法
- 依存関係;
- ソースからソフトウェアをビルドするためのオプション、および
- ビルドコマンド

パッケージのレシピを指定すると、レシピのユーザは、サポートされているシステムのいずれかでさまざまな機能を備えたソフトウェアをビルドするようにSpackに依頼できます。

## 注意事項

このチュートリアルは、Spackの動作バージョンがインストールされていることを前提としています。 Spackのインストール方法については、スタートガイドを参照してください。

コードを作成するので、少なくとも初心者レベルのPythonに精通していることを前提としています 。

チュートリアルであるため、このドキュメントはパッケージングを開始するのに役立ちますが、完全なものではありません。
このトピックに関するより完全なドキュメントについては、Spack公式ドキュメントの [パッケージガイド](https://spack.readthedocs.io/en/latest/packaging_guide.html#packaging-guide) を参照してください。
このセクションで使用されているサンプルコードスニペットは、 [https://github.com/spack/spack-tutorial](https://github.com/spack/spack-tutorial) の `tutorial/examples` にあります。

## 入門

始める前に、次のように3つの環境変数が設定されていることを確認する必要があります。

- `SPACK_ROOT` ：Spackインストール先のパス
- `PATH` ： `$SPACK_ROOT/bin` を含む（spackコマンドの呼び出しは機能します）
- `EDITOR` ：（パッケージを変更したときにSpackが実行できるようにするための）好みのテキストエディタのパス

`setup-env.sh` によって最初の2つの変数は自動的に設定されます。
まだ実行していない場合は、次のようにコマンドを実行します：

```bash
$ . ~/spack/share/spack/setup-env.sh
```

`csh` や `fish` などのシェルを使っている場合は、それに合わせた実行をしてください。

In order to avoid modifying your Spack installation with the package we are creating, add a package repository just for this tutorial by entering the following command:



作成しているパッケージでSpackでインストール済みのものが変更されないようにするには、次のコマンドを入力し、このチュートリアル専用の **パッケージリポジトリ** を追加します。

```bash
$ spack repo add $SPACK_ROOT/var/spack/repos/tutorial/
==> Added repo with namespace 'tutorial'.
```

実行することにより、これからのチュートリアルで行った変更が他の部分に悪影響を与えないことが保証されます。
リポジトリについての詳細については [パッケージリポジトリ](https://spack.readthedocs.io/en/latest/repositories.html) を参照のこと。

## パッケージファイルの作成



TBD


mpileaksに依存するソフトウェアをインストールしたいが、Spackに組み込みパッケージがまだないことがわかったとします。これは、作成する必要があることを意味します。

Spackのcreateコマンドは、パッケージのソースコードの場所を取得し、それを使用して次の目的でテンプレートから新しいパッケージを構築します。

コードをフェッチします。
パッケージスケルトンを作成します。と
選択したエディターでファイルを開きます。