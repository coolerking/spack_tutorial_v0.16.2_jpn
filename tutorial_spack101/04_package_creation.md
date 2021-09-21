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

mpileaksに依存するソフトウェアをインストールをしたいけれど、Spackに組み込みパッケージがまだないことがわかったとします。
この場合、パッケージを作成する必要があります。

Spackの `create` コマンドは、パッケージのソースコードの場所を取得し、それを使って次の手順でテンプレートから新しいパッケージを構築します：

- コードをフェッチ
- パッケージスケルトンを作成
- 選択したエディタでファイルを開く

> **注意**
> 多くのバージョンが利用可能なソフトウェアからパッケージを作成する例は、[パッケージの作成と編集](https://spack.readthedocs.io/en/latest/packaging_guide.html#creating-editing-packages) にあります。

`mpileaks` ソースコードは、ソフトウェアのリポジトリ（[https://github.com/LLNL/mpileaks](https://github.com/LLNL/mpileaks) ）上にtar＋圧縮形式で提供されています。
Spackは、次のURLを引数にして `spack create` コマンドを実行すると、tarballの内容を確認し、パッケージを生成します。

```bash
$ spack create https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz
==> This looks like a URL for mpileaks
==> Found 1 version of mpileaks:

  1.0  https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz

==> Fetching https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz
 ######################################################################## 100.0%##O#- #
==> This package looks like it uses the autotools build system
==> Created template for mpileaks package
==> Created package file: /home/spack/spack/var/spack/repos/tutorial/packages/mpileaks/package.py
```

選択したテキストエディタが起動して、 `package.py` ファイルを編集するために開きます。

`package.py` ファイルがチュートリアルリポジトリ `packages` の `mpileaks` サブディレクトリに存在している必要があります（つまり、 `$SPACK_ROOT/var/spack/repos/tutorial/packages/mpileaks/package.py` ）。

時間を取ってファイルを確認してください。

Spackテンプレートは、次に示すようなスケルトン構成になっています：

- パッケージをSpackリポジトリに提供する方法についての説明（先頭コメント２セクション目の記述）
- 対象ソフトウェアが `Autotools` で構築されていることを示している（継承元クラスから）
- docstring テンプレートを提供（Pythonコメントサンプル部分）
- ホームページのURL指定例を掲示（`https://www.example.com`）
- パッケージメンテナのリストを指定する方法を掲示（``meintainers`）
- ソフトウェアのバージョンディレクティブをチェックサム付きで指定（`version`）
- 依存関係ディレクティブの例を掲示（コメント内の `depends_on('foo')`）
- `configure_args` メソッドのスケルトンを提供

`mpileaks/package.py` (`tutorial/examples/0.package.py`より)：

```python
# Copyright 2013-2021 Lawrence Livermore National Security, LLC and other
# Spack Project Developers. See the top-level COPYRIGHT file for details.
#
# SPDX-License-Identifier: (Apache-2.0 OR MIT)

# ----------------------------------------------------------------------------
# If you submit this package back to Spack as a pull request,
# please first remove this boilerplate and all FIXME comments.
#
# This is a template package file for Spack.  We've put "FIXME"
# next to all the things you'll want to change. Once you've handled
# them, you can save this file and test your package like this:
#
#     spack install mpileaks
#
# You can edit this file again by typing:
#
#     spack edit mpileaks
#
# See the Spack documentation for more information on packaging.
# ----------------------------------------------------------------------------

from spack import *


class Mpileaks(AutotoolsPackage):
    """FIXME: Put a proper description of your package here."""

    # FIXME: Add a proper url for your package's homepage here.
    homepage = "https://www.example.com"
    url      = "https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    # FIXME: Add a list of GitHub accounts to
    # notify when the package is updated.
    # maintainers = ['github_user1', 'github_user2']

    version('1.0', sha256='2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825')

    # FIXME: Add dependencies if required.
    # depends_on('foo')

    def configure_args(self):
        # FIXME: Add arguments other than --prefix
        # FIXME: If not needed delete this function
        args = []
        return args
```

> **注意**
> `maintainers` フィールドは、パッケージに変更が加えられたときに通知を受け取っても構わない方々のGitHubユーザ名をコンマ区切りで記述したリストです。
> この情報は、自分のソフトウェア用にSpackパッケージを保守している開発者、または他の人が保守しているソフトウェアに依存している開発者に役立ちます。

プレースホルダ（後で書き換える必要のある項目）を次のように入力します：

- 該当パッケージに関するいくつかの情報を文書化
- 依存関係のあるパッケージを追加
- パッケージのビルドに必要な構成を指定するための引数を追加

とりあえず、Spackがスケルトンで何をするか見てみましょう。

エディタを終了し、 `spack install` コマンドを使用してパッケージをビルドしてみてください：

```bash
$ spack install mpileaks
==> Installing mpileaks-1.0-2zeskz65ktjgqj6u57venocpndampdt3
==> No binary for mpileaks-1.0-2zeskz65ktjgqj6u57venocpndampdt3 found: installing from source
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/2e/2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825.tar.gz
 ######################################################################## 100.0%
==> mpileaks: Executing phase: 'autoreconf'
==> mpileaks: Executing phase: 'configure'
==> Error: ProcessError: Command exited with status 1:
    '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-2zeskz65ktjgqj6u57venocpndampdt3/spack-src/configure' '--prefix=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-2zeskz65ktjgqj6u57venocpndampdt3'

1 error found in build log:
     30    checking for mpipgcc... no
     31    Checking whether not-found responds to '-showme:compile'... no
     32    Checking whether not-found responds to '-showme'... no
     33    Checking whether not-found responds to '-compile-info'... no
     34    Checking whether not-found responds to '-show'... no
     35    /tmp/spack/spack-stage/spack-stage-mpileaks-1.0-2zeskz65ktjgqj6u57ve
	   nocpndampdt3/spack-src/configure: line 4931: Echo: command not found
  >> 36    configure: error: unable to locate adept-utils installation

See build log for details:
  /tmp/spack/spack-stage/spack-stage-mpileaks-1.0-2zeskz65ktjgqj6u57venocpndampdt3/spack-build-out.txt
```

出力メッセージから、明らかにビルドできなかったことがわかります。
`configure` において依存関係のインストール場所が見つからないことを示しています。

それでは、パッケージをソフトウェアに合わせてカスタマイズしてみましょう：

## パッケージドキュメントの追加

まず、ドキュメントに記入しましょう。

`spack edit` コマンドを使って、 `$EDITOR` にて `mpileaks` の `package.py` ファイルを自分のファイルに戻します：

```bash
$ spack edit mpileaks
```

次の変更を加えましょう：

- 上部コメント部の破線間の指示を削除する
- 最初の `FIXME` コメントを `mpileaks` のdocstringによる説明に置き換える
- ホームページのプロパティを正しいリンクに置き換える
- `maintainers` プロパティのコメントを解除し、GitHubユーザ名を追加する

> **注意**
> Copyrightチュートリアルドキュメントの長さを減らすために、ここではパッケージスニペットの残りの部分の句を除外します。ただし、公開されたパッケージでは保持する必要があります。

次に、`package.py` ファイルに変更と追加を行います。

編集後のパッケージには、次の情報が含まれている必要があります。

`mpileaks/package.py`（ `tutorial/examples /1.package.py` より）：

```python
from spack import *


class Mpileaks(AutotoolsPackage):
    """Tool to detect and report MPI objects like MPI_Requests and
    MPI_Datatypes."""

    homepage = "https://github.com/LLNL/mpileaks"
    url      = "https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    maintainers = ['adamjstewart']

    version('1.0', sha256='2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825')

    # FIXME: Add dependencies if required.
    # depends_on('foo')

    def configure_args(self):
        # FIXME: Add arguments other than --prefix
        # FIXME: If not needed delete this function
        args = []
        return args
```

上記の時点では、パッケージ内の主要なドキュメントのみを更新しました。
ソフトウェアの構築には役立ちませんが、情報を確認できるようになりました。

パッケージの情報を確認するために `spack info` コマンドを実行してみましょう：

```bash
$ spack info mpileaks
AutotoolsPackage:   mpileaks

Description:
    Tool to detect and report MPI objects like MPI_Requests and
    MPI_Datatypes.

Homepage: https://github.com/LLNL/mpileaks

Maintainers: @adamjstewart

Tags:
    None

Preferred version:
    1.0    https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz

Safe versions:
    1.0    https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz

Variants:
    None

Installation Phases:
    autoreconf	  configure    build	install

Build Dependencies:
    None

Link Dependencies:
    None

Run Dependencies:
    None

Virtual Packages:
    None
```

時間を取って `spack info` コマンドの出力を確認してください。
出力されたメッセージには、パッケージから派生した次の情報が表示されます：

- `Autotools` パッケージであること
- パッケージの説明、ホームページ、およびメンテナが掲示されている
- コマンドに指定したURLが含まれています。spack create
- 優先バージョンはもとのソースコードからのものになっている
- デフォルトの `Autotools` パッケージインストールフェーズが一覧表示されている

手動で追加する必要があるため、 さまざまなタイプの依存関係については `None` で示されています。

パッケージに関する詳細情報を入力すると、`spack info` コマンドの出力メッセージの情報が増えます。

> **注意**
> `Autotools` パッケージの使用に関する詳細は、[AutotoolsPackage](https://spack.readthedocs.io/en/latest/build_systems/autotoolspackage.html#phases) を参照のこと。
> Spackが認識しているビルドシステムの完全なリストは [Build Systems](https://spack.readthedocs.io/en/latest/build_systems.html) にて確認できます。

これで、ビルドレシピの入力を開始するための準備が整いました。

## 依存関係の追加

最初にソフトウェアのリポジトリ（ [https://github.com/LLNL/mpileaks](https://github.com/LLNL/mpileaks) ）のドキュメントを確認して、確定した依存関係を追加します。
この `mpileaks` ソフトウェアは、次の3つのサードパーティライブラリに依存しています：

- `mpi`
- `adept-utils`
- `callpath`

> **注意**
> 幸運なことに、上記の依存関係はすべて Spack の組み込みパッケージです。そうでなければ、それらのパッケージも作成する必要がありました。

`spack edit` コマンドを使って `$EDITOR` を開き、`mpileaks` の `package.py` ファイルをバックアップします：

```bash
$ spack edit mpileaks
```

次に示すように、`depends_on` ディレクティブを使って依存関係を指定して追加します：

`mpileaks/package.py` ( `tutorial/examples/2.package.py` より)：

```python
from spack import *


class Mpileaks(AutotoolsPackage):
    """Tool to detect and report MPI objects like MPI_Requests and
    MPI_Datatypes."""

    homepage = "https://github.com/LLNL/mpileaks"
    url      = "https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    maintainers = ['adamjstewart']

    version('1.0', sha256='2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825')

    depends_on('mpi')
    depends_on('adept-utils')
    depends_on('callpath')

    def configure_args(self):
        # FIXME: Add arguments other than --prefix
        # FIXME: If not needed delete this function
        args = []
        return args
```

依存関係を追加すると、Spackはパッケージビルド前にこれらのパッケージがインストールされていることを確認する必要があることを通知します。

> **注意**
> `mpi` の依存関係には、他に2つの異なる仮想依存関係が存在します。つまり、Spackは、 `openmpi`または`mvapich2` などの `mpi` インターフェイスを提供するパッケージで依存関係を満たす必要があります。
> このようなパッケージを **プロバイダ** と呼びます。仮想依存関係の詳細については、[Packaging Guide](https://spack.readthedocs.io/en/latest/packaging_guide.html#packaging-guide) を参照してください。

`spack install` コマンドを使ってソフトウェアを再度ビルドしようとすると、さらに多くのインストレーションが実行されます。

```bash
$ spack install mpileaks
==> Installing libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16/linux-ubuntu18.04-x86_64-gcc-7.5.0-libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl.spack
 ######################################################################## 100.0%
==> Extracting libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl from binary cache
==> Installing patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
==> No binary for patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp found: installing from source
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b2/b2deabce05c34ce98558c0efb965f209de592197b2c88e930298d740ead09019.tar.gz
 ######################################################################## 100.0%
==> patchelf: Executing phase: 'autoreconf'
==> patchelf: Executing phase: 'configure'
==> patchelf: Executing phase: 'build'
==> patchelf: Executing phase: 'install'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Installing zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11/linux-ubuntu18.04-x86_64-gcc-7.5.0-zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg.spack
 ######################################################################## 100.0%
==> Extracting zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Installing pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7.spack
 ######################################################################## 100.0%
==> Extracting pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
==> Installing berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40/linux-ubuntu18.04-x86_64-gcc-7.5.0-berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn.spack
 ######################################################################## 100.0%
==> Extracting berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
==> Installing libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12/linux-ubuntu18.04-x86_64-gcc-7.5.0-libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q.spack
 ######################################################################## 100.0%
==> Extracting libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
==> Installing util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2.spack
 ######################################################################## 100.0%
==> Extracting util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
==> Installing xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5/linux-ubuntu18.04-x86_64-gcc-7.5.0-xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v.spack
 ######################################################################## 100.0%
==> Extracting xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
==> Installing libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiberty-2.33.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4.spack
 ######################################################################## 100.0%
==> Extracting libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4
==> Installing diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr.spack
 ######################################################################## 100.0%
==> Extracting diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Installing tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/tar-1.32/linux-ubuntu18.04-x86_64-gcc-7.5.0-tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd.spack
 ######################################################################## 100.0%
==> Extracting tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd
==> Installing ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm.spack
 ######################################################################## 100.0%
==> Extracting ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
==> Installing m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18/linux-ubuntu18.04-x86_64-gcc-7.5.0-m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps.spack
 ######################################################################## 100.0%
==> Extracting m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
==> Installing libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10/linux-ubuntu18.04-x86_64-gcc-7.5.0-libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d.spack
 ######################################################################## 100.0%
==> Extracting libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
==> Installing bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8/linux-ubuntu18.04-x86_64-gcc-7.5.0-bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui.spack
 ######################################################################## 100.0%
==> Extracting bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Installing readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v.spack
 ######################################################################## 100.0%
==> Extracting readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
==> Installing libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6/linux-ubuntu18.04-x86_64-gcc-7.5.0-libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl.spack
 ######################################################################## 100.0%
==> Extracting libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
==> Installing boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.72.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6.spack
 ######################################################################## 100.0%
==> Extracting boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6
==> Installing gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/gettext-0.21/linux-ubuntu18.04-x86_64-gcc-7.5.0-gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7.spack
 ######################################################################## 100.0%
==> Extracting gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7
==> Installing gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb.spack
 ######################################################################## 100.0%
==> Extracting gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
==> Installing libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16/linux-ubuntu18.04-x86_64-gcc-7.5.0-libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7.spack
 ######################################################################## 100.0%
==> Extracting libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
==> Installing elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/elfutils-0.182/linux-ubuntu18.04-x86_64-gcc-7.5.0-elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk.spack
 ######################################################################## 100.0%
==> Extracting elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk
==> Installing perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t.spack
 ######################################################################## 100.0%
==> Extracting perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Installing libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libdwarf-20180129/linux-ubuntu18.04-x86_64-gcc-7.5.0-libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l.spack
 ######################################################################## 100.0%
==> Extracting libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l
==> Installing openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h/linux-ubuntu18.04-x86_64-gcc-7.5.0-openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52.spack
 ######################################################################## 100.0%
==> Extracting openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
==> Installing autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69/linux-ubuntu18.04-x86_64-gcc-7.5.0-autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd.spack
 ######################################################################## 100.0%
==> Extracting autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
==> Installing cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4/linux-ubuntu18.04-x86_64-gcc-7.5.0-cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j.spack
 ######################################################################## 100.0%
==> Extracting cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j from binary cache
==> Warning: patchelf --print-rpath /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in produced an error [Command exited with status 1:
    '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp/bin/patchelf' '--print-rpath' '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in'
patchelf: not an ELF executable
]
==> Warning: patchelf --force-rpath --set-rpath /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in failed with error Command exited with status 1:
    '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp/bin/patchelf' '--force-rpath' '--set-rpath' '' '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in'
patchelf: not an ELF executable

[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
==> Installing automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6.spack
 ######################################################################## 100.0%
==> Extracting automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Installing intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7.spack
 ######################################################################## 100.0%
==> Extracting intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7
==> Installing numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14/linux-ubuntu18.04-x86_64-gcc-7.5.0-numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh.spack
 ######################################################################## 100.0%
==> Extracting numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
==> Installing dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/dyninst-10.2.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4.spack
 ######################################################################## 100.0%
==> Extracting dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4
==> Installing hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11/linux-ubuntu18.04-x86_64-gcc-7.5.0-hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu.spack
 ######################################################################## 100.0%
==> Extracting hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
==> Installing openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6/linux-ubuntu18.04-x86_64-gcc-7.5.0-openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25.spack
 ######################################################################## 100.0%
==> Extracting openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
==> Installing adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh.spack
 ######################################################################## 100.0%
==> Extracting adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh
==> Installing callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4/linux-ubuntu18.04-x86_64-gcc-7.5.0-callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq.spack
 ######################################################################## 100.0%
==> Extracting callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq
==> Installing mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7
==> No binary for mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7 found: installing from source
==> Using cached archive: /home/spack/spack/var/spack/cache/_source-cache/archive/2e/2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825.tar.gz
==> mpileaks: Executing phase: 'autoreconf'
==> mpileaks: Executing phase: 'configure'
==> Error: ProcessError: Command exited with status 1:
    '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7/spack-src/configure' '--prefix=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7'

1 error found in build log:
     23    checking whether /home/spack/spack/lib/spack/env/gcc/gcc and cc unde
	   rstand -c and -o together... yes
     24    checking whether we are using the GNU C++ compiler... yes
     25    checking whether /home/spack/spack/lib/spack/env/gcc/g++ accepts -g.
	   .. yes
     26    checking dependency style of /home/spack/spack/lib/spack/env/gcc/g++
	   ... gcc3
     27    checking for /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gc
	   c-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc...
	   /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openm
	   pi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc
     28    Checking whether /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_6
	   4/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc
	    responds to '-showme:compile'... yes
  >> 29    configure: error: unable to locate adept-utils installation

See build log for details:
  /tmp/spack/spack-stage/spack-stage-mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7/spack-build-out.txt
```

> **注意**
> SpackにMPIをインストールまたは構成していない場合、このコマンドの実行には時間がかかり、より多くの出力が生成される可能性があります。

これで、Spackがすべての依存関係を識別して構築したことがわかります。それは次のことを発見しました：

- `openmpi` パッケージで `mpi` 依存関係を満たしている
- `callpath` は具体的な依存関係である
- `adept-utils` は具体的な依存関係である

しかし、まだパッケージをビルドできていません。

## パッケージビルドのデバッグ

`adept-utils` パッケージの `configure` によるエラーにより、`mpileaks` パッケージはまだビルドできていません。
経験豊富な `Autotools` 開発者は、問題とその解決策をすでに理解しているかもしれません。

しかし、ここではこの機会にSpack機能を使用して問題を調査してみましょう。
続行するためのオプションは次のとおりです：

- ビルドログの確認
- パッケージの手動ビルド

### ビルドログの確認

ビルドログからいくつかの手がかりが得られる可能性があるので、インストール失敗によって参照を推奨されたパスにある `spack-build-out.txt` ファイルの内容を見てみましょう：

```bash
$ cat /tmp/spack/spack-stage/spack-stage-mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7/spack-build-out.txt
==> mpileaks: Executing phase: 'autoreconf'
==> mpileaks: Executing phase: 'configure'
==> [2021-04-15-21:20:51.712716] '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7/spack-src/configure' '--prefix=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7'
checking metadata... no
checking installation directory variables... yes
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /bin/mkdir -p
checking for gawk... no
checking for mawk... mawk
checking whether make sets $(MAKE)... yes
checking for gcc... /home/spack/spack/lib/spack/env/gcc/gcc
checking for C compiler default output file name... a.out
checking whether the C compiler works... yes
checking whether we are cross compiling... no
checking for suffix of executables...
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether /home/spack/spack/lib/spack/env/gcc/gcc accepts -g... yes
checking for /home/spack/spack/lib/spack/env/gcc/gcc option to accept ISO C89... none needed
checking for style of include used by make... GNU
checking dependency style of /home/spack/spack/lib/spack/env/gcc/gcc... gcc3
checking whether /home/spack/spack/lib/spack/env/gcc/gcc and cc understand -c and -o together... yes
checking whether we are using the GNU C++ compiler... yes
checking whether /home/spack/spack/lib/spack/env/gcc/g++ accepts -g... yes
checking dependency style of /home/spack/spack/lib/spack/env/gcc/g++... gcc3
checking for /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc... /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc
Checking whether /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin/mpicc responds to '-showme:compile'... yes
configure: error: unable to locate adept-utils installation
```

都合がいいことに、このケースのエラーでは、_spack install_ によるログの最後の行に表示されています。

ここでは、`configure` コマンドによって実行されるいくつかのチェックも確認できます。
最も重要なことは、最後の行が非常に明確であるということです：`adept-utils` の依存関係のインストールパスが見つかりませんでした。

> **注意**
> Spackは、標準のインクルードディレクトリとライブラリディレクトリをコンパイラの検索パスに自動的に追加しますが、この情報が取得されないことも珍しくありません。
> `mpileaks` のような一部のソフトウェアでは、コマンドラインでパスを明示的に指定する必要があります。

それでは、ステージングされたビルドディレクトリからさらに調査してみましょう。

### パッケージの手動ビルド

TBD
