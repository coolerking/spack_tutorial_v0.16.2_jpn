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
- パッケージメンテナのリストを指定する方法を掲示（`meintainers`）
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

最初にパッケージを手動でビルドして、問題を解決する方法を見つけられるかどうかを確認してみましょう。

`spack cd` コマンドを使用って、ビルドディレクトリに移動しましょう。

```bash
$ spack cd mpileaks
```

このコマンドを実行すると、最後に試行されたビルドの作業ディレクトリに移動します。
適切なステージディレクトリに移動するはずです。

次に、`spack build-env` コマンドを使って、環境が適切に設定されていることを確認しましょう 。

```bash
$ spack build-env mpileaks bash
```

このコマンドは、Spackが `mpileaks` パッケージのビルドに使用したのと同じ環境を含む新しいシェルを生成しました（`bash` 部は、好きなシェルに自由に置き換えてください）。

ここから、`configure` コマンドを使ってビルドを手動で再実行できます。

```bash
$ ./configure --prefix=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-2e2wmy4spaei2ov4g57hqz7fn3jb43hi
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
checking for /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-wudo6kygednztgzemlnq4l7rlwrwu3zw/bin/mpicc... /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-wudo6kygednztgzemlnq4l7rlwrwu3zw/bin/mpicc
Checking whether /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-wudo6kygednztgzemlnq4l7rlwrwu3zw/bin/mpicc responds to '-showme:compile'... yes
configure: error: unable to locate adept-utils installation
```

以前と同じ結果が得られます。
残念ながら、標準出力にはビルドに役立つ追加情報は含まれていません。

これは `configure` で構築された単純なパッケージであり、インストールディレクトリを指定する必要があることがわかっているので、そのヘルプを使用して、ソフトウェアで使用できるコマンドラインオプションを確認できます。

```bash
$ ./configure --help
`configure' configures mpileaks 1.0 to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.

For better control, use the options below.

Fine tuning of the installation directories:
  --bindir=DIR            user executables [EPREFIX/bin]
  --sbindir=DIR           system admin executables [EPREFIX/sbin]
  --libexecdir=DIR        program executables [EPREFIX/libexec]
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
  --libdir=DIR            object code libraries [EPREFIX/lib]
  --includedir=DIR        C header files [PREFIX/include]
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
  --infodir=DIR           info documentation [DATAROOTDIR/info]
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
  --mandir=DIR            man documentation [DATAROOTDIR/man]
  --docdir=DIR            documentation root [DATAROOTDIR/doc/mpileaks]
  --htmldir=DIR           html documentation [DOCDIR]
  --dvidir=DIR            dvi documentation [DOCDIR]
  --pdfdir=DIR            pdf documentation [DOCDIR]
  --psdir=DIR             ps documentation [DOCDIR]

Program names:
  --program-prefix=PREFIX            prepend PREFIX to installed program names
  --program-suffix=SUFFIX            append SUFFIX to installed program names
  --program-transform-name=PROGRAM   run sed PROGRAM on installed program names

System types:
  --build=BUILD     configure for building on BUILD [guessed]
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]

Optional Features:
  --disable-option-checking  ignore unrecognized --enable/--with options
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
  --disable-dependency-tracking  speeds up one-time build
  --enable-dependency-tracking   do not reject slow dependency extractors
  --enable-shared[=PKGS]  build shared libraries [default=yes]
  --enable-static[=PKGS]  build static libraries [default=yes]
  --enable-fast-install[=PKGS]
                          optimize for fast installation [default=yes]
  --disable-libtool-lock  avoid locking (might break parallel builds)

Optional Packages:
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
  --with-adept-utils=PATH Specify adept-utils path
  --with-callpath=PATH    Specify libcallpath path
  --with-stack-start-c=value
                          Specify integer number for MPILEAKS_STACK_START for
                          C code
  --with-stack-start-fortran=value
                          Specify integer number for MPILEAKS_STACK_START for
                          FORTRAN code
  --with-pic              try to use only PIC/non-PIC objects [default=use
                          both]
  --with-gnu-ld           assume the C compiler uses GNU ld [default=no]

Some influential environment variables:
  CC          C compiler command
  CFLAGS      C compiler flags
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
              nonstandard directory <lib dir>
  LIBS        libraries to pass to the linker, e.g. -l<library>
  CPPFLAGS    C/C++/Objective C preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  CXX         C++ compiler command
  CXXFLAGS    C++ compiler flags
  CPP         C preprocessor
  CXXCPP      C++ preprocessor

Use these variables to override the choices made by `configure' or to help
it to find libraries and programs with nonstandard names/locations.

Report bugs to <moody20@llnl.gov>.
```

次のオプションを使用して、2つの具体的な依存関係のパスを指定できることに注意してください。

- `--with-adept-utils=PATH`
- `--with-callpath=PATH`

それでは、シェルを離れて、Spackリポジトリディレクトリに戻りましょう。

```bash
$ exit
$ cd $SPACK_ROOT
```

提供する引数がわかったので、レシピを更新できます。

## 構成引数の指定

これで、`configure` コマンドに渡す必要のあるオプションがわかりました。
しかし指定するには、パッケージの依存関係のインストールパスがわかっていなくてはなりません。
`package.py` ファイル内からどのように知ることができますか？

幸運にも、パッケージの具現(concrete)化されたSpecインスタンスを照会できます。
`self.spec` プロパティには、その依存関係のパッケージの非循環有向グラフ（DAG）を保持しています。
Spec名前でアクセスされる各依存関係には、インストールパスを含む `prefix` プロパティがあります。

それでは、パッケージの `configure_args` メソッドに2つの具体的な依存関係へのパスを指定するための構成引数を追加しましょう。

`spack edit` コマンドを使って、`$EDITOR` 内から mpileaks の `package.py` ファイルを次のようにバックアップします：

```bash
$ spack edit mpileaks
```

そして、`configure_args` メソッドに引数 `--with-adept-utils` と引数 `--with-callpath` を追加します。

`mpileaks/package.py`（ `tutorial/examples/3.package.py` より）：

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
        args = [
            '--with-adept-utils={0}'.format(self.spec['adept-utils'].prefix),
            '--with-callpath={0}'.format(self.spec['callpath'].prefix),
        ]

        return args
```

`AutotoolsPackage` であるため、メソッドから返された引数はビルド中に自動的に `configure` に渡されます。

それでは、ビルドをもう一度試してみましょう：

```bash
$ spack install mpileaks
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq
==> Installing mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7
==> No binary for mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7 found: installing from source
==> Using cached archive: /home/spack/spack/var/spack/cache/_source-cache/archive/2e/2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825.tar.gz
==> mpileaks: Executing phase: 'autoreconf'
==> mpileaks: Executing phase: 'configure'
==> mpileaks: Executing phase: 'build'
==> mpileaks: Executing phase: 'install'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-dx5qx6s2j7yhuilyvgd3laxakls2i2k7
```

成功しました！

ビルドを成功させるためにやらなくてはならなかったことは、単純な飾り気のないビルドを実行するために構成の際の2つの具現(concrete)化パッケージのパス引数を追加することだけでした。

しかし、他のユーザがソフトウェアを構築するのを助けるために私たちができることはそれだけですか？

## バリアントの追加

パッケージ内のソフトウェアのオプション機能を公開したい場合はどうすればいいのでしょうか？

これには、パッケージバリアントを使用してビルド時オプションを追加します。

`configure` のヘルプ出力を思い出してください。
`mpileaks` ソフトウェアには、Spackにてサポート可能なオプション機能とパッケージがいくつかあります。
この2つは、単に有効または無効にするのではなく、両方とも整数を使用するため、チュートリアルとしては目に付きます。

```text
Optional Features:
  --disable-option-checking  ignore unrecognized --enable/--with options
  --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
  --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
  --disable-dependency-tracking  speeds up one-time build
  --enable-dependency-tracking   do not reject slow dependency extractors
  --enable-shared[=PKGS]  build shared libraries [default=yes]
  --enable-static[=PKGS]  build static libraries [default=yes]
  --enable-fast-install[=PKGS]
                          optimize for fast installation [default=yes]
  --disable-libtool-lock  avoid locking (might break parallel builds)

Optional Packages:
  --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
  --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
  --with-adept-utils=PATH Specify adept-utils path
  --with-callpath=PATH    Specify libcallpath path
  --with-stack-start-c=value
                          Specify integer number for MPILEAKS_STACK_START for
                          C code
  --with-stack-start-fortran=value
                          Specify integer number for MPILEAKS_STACK_START for
                          FORTRAN code
  --with-pic              try to use only PIC/non-PIC objects [default=use
                          both]
  --with-gnu-ld           assume the C compiler uses GNU ld [default=no]
```

ソフトウェアドキュメント（[https://github.com/LLNL/mpileaks](https://github.com/LLNL/mpileaks)）によると、`--with-stack-start-*` オプションの整数値は、各言語のスタックトレースの先頭から削り取る呼び出し数を表し、生成されたトレースでのmpileaksライブラリ関数内部呼び出しのノイズを効果的に低減します。

簡単にするために、1つのバリアントを使用して、両方の引数の値を指定します。

このオプション機能をサポートするには、パッケージに2つの変更を加える必要があります。

- `variant` ディレクティブを追加する
- 値を使用するように構成オプションを変更する

`int` 値のデフォルトが `0` とするバリアントを追加しましょう。
デフォルト `0` でオプションを効果的に無効にします。
また、値を取得するように `configure_args` を変更 し、ゼロ以外の値がユーザによって指定された際に対応する構成引数を追加します。

`spack edit` コマンドを使用して、`$EDITOR` にて `mpileaks` の `package.py`ファイルをバックアップします：

```bash
$ spack edit mpileaks
```

そして、`variant` ディレクティブと関連する引数を次のように追加します：

`mpileaks/package.py` ( `tutorial/examples/4.package.py` より)：

```python
from spack import *


class Mpileaks(AutotoolsPackage):
    """Tool to detect and report MPI objects like MPI_Requests and
    MPI_Datatypes."""

    homepage = "https://github.com/LLNL/mpileaks"
    url      = "https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    maintainers = ['adamjstewart']

    version('1.0', sha256='2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825')

    variant('stackstart', values=int, default=0,
            description='Specify the number of stack frames to truncate')

    depends_on('mpi')
    depends_on('adept-utils')
    depends_on('callpath')

    def configure_args(self):
        args = [
            '--with-adept-utils={0}'.format(self.spec['adept-utils'].prefix),
            '--with-callpath={0}'.format(self.spec['callpath'].prefix),
        ]

        stackstart = int(self.spec.variants['stackstart'].value)
        if stackstart:
            args.extend([
                '--with-stack-start-c={0}'.format(stackstart),
                '--with-stack-start-fortran={0}'.format(stackstart),
            ])

        return args
```

`variant` ディレクティブが `self.spec` の `variants` 辞書に変換されていることに注意してください。
また、ユーザが指定した値には、エントリの `value` プロパティからアクセスすることに注意してください。

次に、（ビルド中により多くの出力を取得するための）`--verbose` インストールオプションと新しい `stackstart` パッケージオプションを使って、インストールを再度実行します：

```bash
$ spack install --verbose mpileaks stackstart=4
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiberty-2.33.1-j6hbvqffhsbt5hs7qjvkqu764j622ij4
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tar-1.32-uwe6tb537wlvqfnz3q2f57s77tnlbdsd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gettext-0.21-lbb45tosrs7xp2g6uwwgw4vmn7vay3k7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.72.0-6ubh2sr3co462cmnnf3jxyj6aijpgon6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/elfutils-0.182-23dila3pqmo355ljap76ydo3xmpxpwpk
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libdwarf-20180129-wo2pb2oazz75kmgnuykmapgxspjhga7l
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/intel-tbb-2020.3-5jz3dlyizy4kwgwzhct2ibukw2txxhd7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/dyninst-10.2.1-fqv6osdixfv3mg2zc5jadeqgvjaehfi4
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq
==> Installing mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz
==> No binary for mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz found: installing from source
==> Using cached archive: /home/spack/spack/var/spack/cache/_source-cache/archive/2e/2e34cc4505556d1c1f085758e26f2f8eea0972db9382f051b2dcfb1d7d9e1825.tar.gz
==> mpileaks: Executing phase: 'autoreconf'
==> mpileaks: Executing phase: 'configure'
==> [2021-04-15-21:21:20.022826] '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/configure' '--prefix=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz' '--with-adept-utils=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh' '--with-callpath=/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq' '--with-stack-start-c=4' '--with-stack-start-fortran=4'
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
checking for adept-utils installation... /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh
checking for libcallpath installation... /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq
checking build system type... x86_64-unknown-linux-gnu
checking host system type... x86_64-unknown-linux-gnu
checking for a sed that does not truncate output... /bin/sed
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking for fgrep... /bin/grep -F
checking for ld used by /home/spack/spack/lib/spack/env/gcc/gcc... /home/spack/spack/lib/spack/env/ld
checking if the linker (/home/spack/spack/lib/spack/env/ld) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/nm -B
checking the name lister (/usr/bin/nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 1572864
checking whether the shell understands some XSI constructs... yes
checking whether the shell understands "+="... yes
checking for /home/spack/spack/lib/spack/env/ld option to reload object files... -r
checking for objdump... objdump
checking how to recognize dependent libraries... pass_all
checking for ar... ar
checking for strip... strip
checking for ranlib... ranlib
checking command to parse /usr/bin/nm -B output from /home/spack/spack/lib/spack/env/gcc/gcc object... ok
checking how to run the C preprocessor... /home/spack/spack/lib/spack/env/gcc/gcc -E
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking for dlfcn.h... yes
checking whether we are using the GNU C++ compiler... (cached) yes
checking whether /home/spack/spack/lib/spack/env/gcc/g++ accepts -g... (cached) yes
checking dependency style of /home/spack/spack/lib/spack/env/gcc/g++... (cached) gcc3
checking how to run the C++ preprocessor... /home/spack/spack/lib/spack/env/gcc/g++ -E
checking for objdir... .libs
checking if /home/spack/spack/lib/spack/env/gcc/gcc supports -fno-rtti -fno-exceptions... no
checking for /home/spack/spack/lib/spack/env/gcc/gcc option to produce PIC... -fPIC -DPIC
checking if /home/spack/spack/lib/spack/env/gcc/gcc PIC flag -fPIC -DPIC works... yes
checking if /home/spack/spack/lib/spack/env/gcc/gcc static flag -static works... yes
checking if /home/spack/spack/lib/spack/env/gcc/gcc supports -c -o file.o... yes
checking if /home/spack/spack/lib/spack/env/gcc/gcc supports -c -o file.o... (cached) yes
checking whether the /home/spack/spack/lib/spack/env/gcc/gcc linker (/home/spack/spack/lib/spack/env/ld -m elf_x86_64) supports shared libraries... yes
checking whether -lc should be explicitly linked in... no
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... yes
checking for ld used by /home/spack/spack/lib/spack/env/gcc/g++... /home/spack/spack/lib/spack/env/ld -m elf_x86_64
checking if the linker (/home/spack/spack/lib/spack/env/ld -m elf_x86_64) is GNU ld... yes
checking whether the /home/spack/spack/lib/spack/env/gcc/g++ linker (/home/spack/spack/lib/spack/env/ld -m elf_x86_64) supports shared libraries... yes
checking for /home/spack/spack/lib/spack/env/gcc/g++ option to produce PIC... -fPIC -DPIC
checking if /home/spack/spack/lib/spack/env/gcc/g++ PIC flag -fPIC -DPIC works... yes
checking if /home/spack/spack/lib/spack/env/gcc/g++ static flag -static works... yes
checking if /home/spack/spack/lib/spack/env/gcc/g++ supports -c -o file.o... yes
checking if /home/spack/spack/lib/spack/env/gcc/g++ supports -c -o file.o... (cached) yes
checking whether the /home/spack/spack/lib/spack/env/gcc/g++ linker (/home/spack/spack/lib/spack/env/ld -m elf_x86_64) supports shared libraries... yes
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking for ANSI C header files... (cached) yes
checking whether byte ordering is bigendian... no
configure: creating ./config.status
config.status: creating Makefile
config.status: creating src/Makefile
config.status: creating examples/Makefile
config.status: creating scripts/Makefile
config.status: creating scripts/srun-mpileaks
config.status: creating scripts/srun-mpileaksf
config.status: creating config/config.h
config.status: executing depfiles commands
config.status: executing libtool commands
==> mpileaks: Executing phase: 'build'
==> [2021-04-15-21:21:24.563022] 'make' '-j6'
Making all in scripts
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
Making all in src
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT mpileaks.lo -MD -MP -MF .deps/mpileaks.Tpo -c -o mpileaks.lo mpileaks.cpp
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT comm.lo -MD -MP -MF .deps/comm.Tpo -c -o comm.lo comm.cpp
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT datatype.lo -MD -MP -MF .deps/datatype.Tpo -c -o datatype.lo datatype.cpp
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT errhandler.lo -MD -MP -MF .deps/errhandler.Tpo -c -o errhandler.lo errhandler.cpp
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT fileio.lo -MD -MP -MF .deps/fileio.Tpo -c -o fileio.lo fileio.cpp
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT group.lo -MD -MP -MF .deps/group.Tpo -c -o group.lo group.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT errhandler.lo -MD -MP -MF .deps/errhandler.Tpo -c errhandler.cpp  -fPIC -DPIC -o .libs/errhandler.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT mpileaks.lo -MD -MP -MF .deps/mpileaks.Tpo -c mpileaks.cpp  -fPIC -DPIC -o .libs/mpileaks.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT fileio.lo -MD -MP -MF .deps/fileio.Tpo -c fileio.cpp	-fPIC -DPIC -o .libs/fileio.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT group.lo -MD -MP -MF .deps/group.Tpo -c group.cpp  -fPIC -DPIC -o .libs/group.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT comm.lo -MD -MP -MF .deps/comm.Tpo -c comm.cpp  -fPIC -DPIC -o .libs/comm.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT datatype.lo -MD -MP -MF .deps/datatype.Tpo -c datatype.cpp  -fPIC -DPIC -o .libs/datatype.o
errhandler.cpp: In function 'int MPI_Errhandler_create(void (*)(ompi_communicator_t**, int*, ...), ompi_errhandler_t**)':
errhandler.cpp:33:47: warning: 'int PMPI_Errhandler_create(void (*)(ompi_communicator_t**, int*, ...), ompi_errhandler_t**)' is deprecated: MPI_Errhandler_create is superseded by MPI_Comm_create_errhandler in MPI-2.0 [-Wdeprecated-declarations]
   int rc = PMPI_Errhandler_create(function, eh);
					       ^
In file included from errhandler.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2038:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Errhandler_create(MPI_Handler_function *function,
		    ^~~~~~~~~~~~~~~~~~~~~~
errhandler.cpp: In function 'int MPI_Errhandler_get(MPI_Comm, ompi_errhandler_t**)':
errhandler.cpp:40:40: warning: 'int PMPI_Errhandler_get(MPI_Comm, ompi_errhandler_t**)' is deprecated: MPI_Errhandler_get is superseded by MPI_Comm_get_errhandler in MPI-2.0 [-Wdeprecated-declarations]
   int rc = PMPI_Errhandler_get(comm, eh);
					^
In file included from errhandler.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2043:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Errhandler_get(MPI_Comm comm, MPI_Errhandler *errhandler)
		    ^~~~~~~~~~~~~~~~~~~
datatype.cpp: In function 'int MPI_Type_hvector(int, int, MPI_Aint, MPI_Datatype, ompi_datatype_t**)':
datatype.cpp:60:74: warning: 'int PMPI_Type_hvector(int, int, MPI_Aint, MPI_Datatype, ompi_datatype_t**)' is deprecated: MPI_Type_hvector is superseded by MPI_Type_create_hvector in MPI-2.0 [-Wdeprecated-declarations]
   int rc = PMPI_Type_hvector(count, blocklength, stride, oldtype, newtype);
									  ^
In file included from datatype.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2495:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Type_hvector(int count, int blocklength, MPI_Aint stride,
		    ^~~~~~~~~~~~~~~~~
datatype.cpp: In function 'int MPI_Type_hindexed(int, int*, MPI_Aint*, MPI_Datatype, ompi_datatype_t**)':
datatype.cpp:79:26: warning: 'int PMPI_Type_hindexed(int, int*, MPI_Aint*, MPI_Datatype, ompi_datatype_t**)' is deprecated: MPI_Type_hindexed is superseded by MPI_Type_create_hindexed in MPI-2.0 [-Wdeprecated-declarations]
	  oldtype, newtype);
			  ^
In file included from datatype.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2491:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Type_hindexed(int count, int array_of_blocklengths[],
		    ^~~~~~~~~~~~~~~~~~
datatype.cpp: In function 'int MPI_Type_struct(int, int*, MPI_Aint*, ompi_datatype_t**, ompi_datatype_t**)':
datatype.cpp:89:31: warning: 'int PMPI_Type_struct(int, int*, MPI_Aint*, ompi_datatype_t**, ompi_datatype_t**)' is deprecated: MPI_Type_struct is superseded by MPI_Type_create_struct in MPI-2.0 [-Wdeprecated-declarations]
	array_of_types, newtype);
			       ^
In file included from datatype.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2509:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Type_struct(int count, int array_of_blocklengths[],
		    ^~~~~~~~~~~~~~~~
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT fileio.lo -MD -MP -MF .deps/fileio.Tpo -c fileio.cpp -o fileio.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT mpileaks.lo -MD -MP -MF .deps/mpileaks.Tpo -c mpileaks.cpp -o mpileaks.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT errhandler.lo -MD -MP -MF .deps/errhandler.Tpo -c errhandler.cpp -o errhandler.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT group.lo -MD -MP -MF .deps/group.Tpo -c group.cpp -o group.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT datatype.lo -MD -MP -MF .deps/datatype.Tpo -c datatype.cpp -o datatype.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT comm.lo -MD -MP -MF .deps/comm.Tpo -c comm.cpp -o comm.o >/dev/null 2>&1
mv -f .deps/fileio.Tpo .deps/fileio.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT info.lo -MD -MP -MF .deps/info.Tpo -c -o info.lo info.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT info.lo -MD -MP -MF .deps/info.Tpo -c info.cpp  -fPIC -DPIC -o .libs/info.o
mv -f .deps/mpileaks.Tpo .deps/mpileaks.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT keyval.lo -MD -MP -MF .deps/keyval.Tpo -c -o keyval.lo keyval.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT keyval.lo -MD -MP -MF .deps/keyval.Tpo -c keyval.cpp	-fPIC -DPIC -o .libs/keyval.o
mv -f .deps/errhandler.Tpo .deps/errhandler.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT mem.lo -MD -MP -MF .deps/mem.Tpo -c -o mem.lo mem.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT mem.lo -MD -MP -MF .deps/mem.Tpo -c mem.cpp  -fPIC -DPIC -o .libs/mem.o
keyval.cpp: In function 'int MPI_Keyval_create(int (*)(MPI_Comm, int, void*, void*, void*, int*), int (*)(MPI_Comm, int, void*, void*), int*, void*)':
keyval.cpp:38:70: warning: 'int PMPI_Keyval_create(int (*)(MPI_Comm, int, void*, void*, void*, int*), int (*)(MPI_Comm, int, void*, void*), int*, void*)' is deprecated: MPI_Keyval_create is superseded by MPI_Comm_create_keyval in MPI-2.0 [-Wdeprecated-declarations]
   int rc = PMPI_Keyval_create(copy_fn, delete_fn, keyval, extra_state);
								      ^
In file included from keyval.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2260:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Keyval_create(MPI_Copy_function *copy_fn,
		    ^~~~~~~~~~~~~~~~~~
keyval.cpp: In function 'int MPI_Keyval_free(int*)':
keyval.cpp:93:35: warning: 'int PMPI_Keyval_free(int*)' is deprecated: MPI_Keyval_free is superseded by MPI_Comm_free_keyval in MPI-2.0 [-Wdeprecated-declarations]
   int rc = PMPI_Keyval_free(keyval);
				   ^
In file included from keyval.cpp:10:0:
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include/mpi.h:2264:20: note: declared here
 OMPI_DECLSPEC	int PMPI_Keyval_free(int *keyval)
		    ^~~~~~~~~~~~~~~~
mv -f .deps/group.Tpo .deps/group.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT op.lo -MD -MP -MF .deps/op.Tpo -c -o op.lo op.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT op.lo -MD -MP -MF .deps/op.Tpo -c op.cpp  -fPIC -DPIC -o .libs/op.o
mv -f .deps/datatype.Tpo .deps/datatype.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT request.lo -MD -MP -MF .deps/request.Tpo -c -o request.lo request.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT request.lo -MD -MP -MF .deps/request.Tpo -c request.cpp  -fPIC -DPIC -o .libs/request.o
mv -f .deps/comm.Tpo .deps/comm.Plo
/bin/bash ../libtool --tag=CXX	 --mode=compile /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config   -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include	 -g -O2 -MT win.lo -MD -MP -MF .deps/win.Tpo -c -o win.lo win.cpp
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT win.lo -MD -MP -MF .deps/win.Tpo -c win.cpp  -fPIC -DPIC -o .libs/win.o
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT info.lo -MD -MP -MF .deps/info.Tpo -c info.cpp -o info.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT mem.lo -MD -MP -MF .deps/mem.Tpo -c mem.cpp -o mem.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT keyval.lo -MD -MP -MF .deps/keyval.Tpo -c keyval.cpp -o keyval.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT op.lo -MD -MP -MF .deps/op.Tpo -c op.cpp -o op.o >/dev/null 2>&1
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT win.lo -MD -MP -MF .deps/win.Tpo -c win.cpp -o win.o >/dev/null 2>&1
mv -f .deps/mem.Tpo .deps/mem.Plo
mv -f .deps/info.Tpo .deps/info.Plo
libtool: compile:  /home/spack/spack/lib/spack/env/gcc/g++ -DHAVE_CONFIG_H -I. -I../config -DHAVE_CONFIG_H -I. -I../config -I../src -I../config -g -O2 -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25////////////////////////////include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/include -I/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/include -g -O2 -MT request.lo -MD -MP -MF .deps/request.Tpo -c request.cpp -o request.o >/dev/null 2>&1
mv -f .deps/op.Tpo .deps/op.Plo
mv -f .deps/keyval.Tpo .deps/keyval.Plo
mv -f .deps/win.Tpo .deps/win.Plo
mv -f .deps/request.Tpo .deps/request.Plo
/bin/bash ../libtool --tag=CXX	 --mode=link /home/spack/spack/lib/spack/env/gcc/g++  -g -O2 -avoid-version  -o libmpileaks.la -rpath /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib mpileaks.lo comm.lo datatype.lo errhandler.lo fileio.lo group.lo info.lo keyval.lo mem.lo op.lo request.lo win.lo -L/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/lib -ladept_cutils -ladept_timing -ladept_utils -L/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/lib -lcallpath
libtool: link: /home/spack/spack/lib/spack/env/gcc/g++ -shared -nostdlib /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/7/crtbeginS.o  .libs/mpileaks.o .libs/comm.o .libs/datatype.o .libs/errhandler.o .libs/fileio.o .libs/group.o .libs/info.o .libs/keyval.o .libs/mem.o .libs/op.o .libs/request.o .libs/win.o   -L/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/adept-utils-1.0.1-haidyz4jgiy43bw3jnrtrvdgrsg3uxwh/lib -ladept_cutils -ladept_timing -ladept_utils -L/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/lib -lcallpath -L/usr/lib/gcc/x86_64-linux-gnu/7 -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/7/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/7/../../.. -lstdc++ -lm -lc -lgcc_s /usr/lib/gcc/x86_64-linux-gnu/7/crtendS.o /usr/lib/gcc/x86_64-linux-gnu/7/../../../x86_64-linux-gnu/crtn.o	 -Wl,-soname -Wl,libmpileaks.so -o .libs/libmpileaks.so
libtool: link: ar cru .libs/libmpileaks.a  mpileaks.o comm.o datatype.o errhandler.o fileio.o group.o info.o keyval.o mem.o op.o request.o win.o
ar: `u' modifier ignored since `D' is the default (see `U')
libtool: link: ranlib .libs/libmpileaks.a
libtool: link: ( cd ".libs" && rm -f "libmpileaks.la" && ln -s "../libmpileaks.la" "libmpileaks.la" )
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
Making all in examples
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[1]: Nothing to be done for 'all'.
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
make[1]: Nothing to be done for 'all-am'.
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
==> mpileaks: Executing phase: 'install'
==> [2021-04-15-21:21:31.155262] 'make' '-j6' 'install'
Making install in scripts
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
make[2]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
test -z "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/bin" || /bin/mkdir -p "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/bin"
make[2]: Nothing to be done for 'install-data-am'.
 /usr/bin/install -c srun-mpileaks srun-mpileaksf '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/bin'
make[2]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/scripts'
Making install in src
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
make[2]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
test -z "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib" || /bin/mkdir -p "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib"
make[2]: Nothing to be done for 'install-data-am'.
 /bin/bash ../libtool	--mode=install /usr/bin/install -c   libmpileaks.la '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib'
libtool: install: /usr/bin/install -c .libs/libmpileaks.so /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib/libmpileaks.so
libtool: install: /usr/bin/install -c .libs/libmpileaks.lai /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib/libmpileaks.la
libtool: install: /usr/bin/install -c .libs/libmpileaks.a /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib/libmpileaks.a
libtool: install: chmod 644 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib/libmpileaks.a
libtool: install: ranlib /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib/libmpileaks.a
libtool: finish: PATH="/home/spack/spack/lib/spack/env/gcc:/home/spack/spack/lib/spack/env/case-insensitive:/home/spack/spack/lib/spack/env:/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin:/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/bin:/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25/bin:/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/callpath-1.0.4-czacy2h7swslefbmdxbj6s6tad3mtoxq/bin:/home/spack/spack/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/sbin" ldconfig -n /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib
----------------------------------------------------------------------
Libraries have been installed in:
   /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/lib

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
make[2]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/src'
Making install in examples
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[2]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[2]: Nothing to be done for 'install-exec-am'.
test -z "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks/examples" || /bin/mkdir -p "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks/examples"
 /usr/bin/install -c -m 644 tests.c mpiPing_leaky.f '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks/examples'
make[2]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src/examples'
make[1]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
make[2]: Entering directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
make[2]: Nothing to be done for 'install-exec-am'.
test -z "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks" || /bin/mkdir -p "/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks"
 /usr/bin/install -c -m 644 README '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/share/mpileaks'
make[2]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
make[1]: Leaving directory '/tmp/spack/spack-stage/spack-stage-mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz/spack-src'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpileaks-1.0-twyqbvipfn2ua5s56ninaaurn7htafvz
```

mpileaks の `Executing phase: 'configure'` の後に強調表示された行の最後に表示される `configure` コマンドに2つのスタック開始引数が追加されていることに注意してください 。

ここまで、ドキュメントの更新、依存関係を追加、オプション機能を備えたビルドをサポートするバリアントの追加、といったパッケージの作成方法について説明しました。

機能の変更を反映するようにビルドをカスタマイズする場合はどうすればよいでしょうか？

## Spec オブジェクトの参照

パッケージが発展し、さまざまなシステムに移植されると、ビルドを変更する必要が生じることがよくあります。
ここでは、パッケージの `Spec` 出番となります。

これまで、バリアントの依存関係と値のパスを `Spec` から取得することを検討してきましたが、それ以上の方法があります。
パッケージの `self.spec` プロパティを使用すると、次のようなパッケージビルドに関する情報を参照できます。

- パッケージの依存関係がどのように構築されたか
- どのコンパイラが使用されていたか
- インストールされているパッケージのバージョン
- どのバリアントが指定されたか（暗黙的または明示的に）

完全なドキュメントは [Packaging Guide](https://spack.readthedocs.io/en/latest/packaging_guide.html#packaging-guide) にありますが、ここでは一般的な参照例を紹介します。

### Spec バージョンの参照

パッケージのバージョン、コンパイラ、および依存関係に基づいてビルドをカスタマイズできます。
それぞれの参照例は次のとおりです：

- `mpileaks` バージョン `1.1` 以降を構築していますか？

```python
if self.spec.satisfies('@1.1:'):
    # Do things needed for version 1.1 or newer
```

- ビルドに使用する `gcc` はバージョン `5.0` に到達していますか？

```python
if self.spec.satisfies('%gcc@:5.0'):
    # Add arguments specific to gcc's up to 5.0
```

- `dyninst` の依存関係は少なくともバージョン `8.0` ですか？

```python
if self.spec['dyninst'].satisfies('@8.0:'):
    # Use newest dyninst options
```

### Spec 名の参照

抽象 `Spec` の具体的なバージョンにカスタマイズする必要があるビルドの場合は、`name` プロパティを使用できます。
例えば：

- ビルドに使用するMPI実装は `openmpi` ですか？

```python
if self.spec['mpi'].name == 'openmpi':
    # Do openmpi things
```

### バリアントの参照

有効なバリアントに基づいてビルドオプションを調整するには、次のように `Spec` 自体を参照します。

- `debug` バリアントを使用してビルドしていますか？

```python
if '+debug' in self.spec:
    # Add -g option to configure flags
```

これらは `Spec` 参照のほんの一例です。
Spackには、パッケージの開発をガイドするための例として役立つ、何千もの組み込みパッケージがあります。
これらのパッケージは `$SPACK_ROOT/var/spack/repos/builtin/packages` にあります。

グッドラック！

## クリーンアップ

このチュートリアルを終了する前に、Spackインスタンスまたはチュートリアルの今後のセクションに干渉しないことを確認しましょう。
次のコマンドを入力して、ここで行った作業を元に戻します：

```bash
$ spack uninstall -ay mpileaks
==> Successfully uninstalled mpileaks@1.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/dx5qx6s
==> Successfully uninstalled mpileaks@1.0%gcc@7.5.0 stackstart=4 arch=linux-ubuntu18.04-x86_64/twyqbvi
$ spack repo remove tutorial
==> Removed repository /home/spack/spack/var/spack/repos/tutorial with namespace 'tutorial'.
$ rm -rf $SPACK_ROOT/var/spack/repos/tutorial/packages/mpileaks
```
