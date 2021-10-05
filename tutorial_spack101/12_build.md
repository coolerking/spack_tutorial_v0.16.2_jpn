# Spack パッケージビルドシステム

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

いくつかのパッケージテンプレートファイルを書き込んだ後、パッケージのいくつかにパターンが現れることに気付くかもしれません。
たとえば、あなた自身が書いかもしれない `install()` メソッドは `configure` 、 `cmake` 、 `make` 、 `make install` を呼び出しています。
`configure` または `cmake` への引数として `"prefix=" + prefix` を記述していることに気付くかもしれません。
Spackには、すべてのパッケージに対してこれらの行を繰り返すのではなく、これらのパターンを処理できるクラスがあります。
さらに、これらのパッケージファイルを使用すると、これらのビルドシステムをよりきめ細かく制御できます。
このセクションでは、各ビルドシステムについて説明し、これらを操作してパッケージをインストールする方法の例を示します。

## パッケージクラス階層

![パッケージクラス階層](./images/package_class_h.png)

上の図は、クラス階層と各パッケージの関係の概要を示しています。
各サブクラスは親クラス `PackageBaseClass` を継承します。
フェッチ、ステージングディレクトリへの抽出、インストールを含む作業の大部分は、この親クラスで行われます。
次に、各サブクラスは、ビルドシステム固有の機能を追加します。
次のセクションでは、各サブクラスを利用する方法の例と、パッケージ化時にこれらの抽象化がどれほど強力であるかを確認します。

## Package

パッケージファイルを作成するためのウォークスルーで `Package` クラスの例をすでに見てきたので、ここではそれらに多くの時間を費やすことはありません。
簡単に言うと、`Package` クラスではビルドプロセスを任意に制御できますが、サブクラスは特定のパターン（例：`configure` 、 `make` 、 `make install` ）に依存しているのであれば有用です。
`Package` クラスは、パッケージャがSpackのヘルパ関数の一部を利用してパッケージのビルドとインストールをカスタマイズできるため、従来とは異なるビルド方法を持つパッケージに特に役立ちます。

## Autotools

前述したように、Autotools　が `configure` 、 `make` および `make install` コマンドを使用してビルドおよびインストールプロセスを実行するパッケージ。
`Package` クラスでは、あなたの典型的なビルドの呪文は以下で構成されます：

```python
def install(self, spec, prefix):
    configure("--prefix=" + prefix)
    make()
    make("install")
```

これは、[パッケージ作成チュートリアル](04_package_creation.md) で解説したものと似ていることがわかります。

`Autotools` サブクラスの目的は、パッケージファイルを書き込み、`Autotools` ビルドシステムの異なるフェーズごとの操作のための便利なメソッド提供を簡素化することです。

`Autotools` パッケージはつぎの4つのフェーズで構成されています：

1. `autoreconf()`
2. `configure()`
3. `build()`
4. `install()`

これらの各フェーズには、適切なデフォルト処理があります。
`Autotools` クラスの内部を簡単に見てみましょう：

```bash
$ spack edit --build-system autotools
```

これにより、テキストエディタで `AutotoolsPackage` ファイルが開きます。

> **注意**
> これらのクラスのコードを示す例は、長くならないように要約されています。
> ここではパッケージャに関連するもののみを示しています。

```python
    #: system base class
    build_system_class = 'AutotoolsPackage'
    #: Whether or not to update ``config.guess`` on old architectures
    patch_config_guess = True

    #: Targets for ``make`` during the :py:meth:`~.AutotoolsPackage.build`
    #: phase
    build_targets = []
    #: Targets for ``make`` during the :py:meth:`~.AutotoolsPackage.install`
    #: phase
    install_targets = ['install']

    #: Callback names for build-time test
    build_time_test_callbacks = ['check']

    #: Callback names for install-time test
    install_time_test_callbacks = ['installcheck']

    #: Set to true to force the autoreconf step even if configure is present
    force_autoreconf = False
    #: Options to be passed to autoreconf when using the default implementation
    autoreconf_extra_args = []

    def configure(self, spec, prefix):
        """Runs configure with the arguments specified in
        :py:meth:`~.AutotoolsPackage.configure_args`
        and an appropriately set prefix.
        """
        options = getattr(self, 'configure_flag_args', [])
        options += ['--prefix={0}'.format(prefix)]
        options += self.configure_args()

        with working_dir(self.build_directory, create=True):
            inspect.getmodule(self).configure(*options)

    def build(self, spec, prefix):
        """Makes the build targets specified by
        :py:attr:``~.AutotoolsPackage.build_targets``
        """
        with working_dir(self.build_directory):
            inspect.getmodule(self).make(*self.build_targets)

```

注意すべき重要な点は、強調表示された行です。
これらのプロパティにより、パッケージャは、パッケージに必要なビルドターゲットとインストールターゲットを設定できます。
たとえば、ビルドターゲットとして `foo` を追加したい場合は、`build_targets` プロパティに追加します：

```python
build_targets = ["foo"]
```

これは、パッケージで `make` を呼び出すのと似ています。

```python
make("foo")
```

これは、環境変数を無視し、コマンドライン引数が必要なパッケージがある場合に役立ちます。

注意すべきもう1つのことは `configure()` メソッドにあります。
ここでは、`Autotools` を使用するパッケージ間で共通のパターンであるため、`prefix` 引数がすでに含まれていることがわかります。
`configure_args()` をオーバーライドするだけで、出力が `configure()` に返されます。
次に `configure(` へ)一般的な引数を追加します。

パッケージャには、`autoreconf` パッケージがビルドシステムを更新し、新しい `configure` を生成する必要がある場合に実行するオプションもあります。
ただし、ほとんどの場合これは不要です。

以前に作業した `mpileaks` のpackage.pyファイルを見てみましょう：

```bash
$ spack edit mpileaks
```

mpileaks は `Package` クラスですが、Autotools ビルドシステムを使用していることに注意してください。
このパッケージは受け入れられますが、これを `AutotoolsPackage` クラスにしてさらに単純化してみましょう。

```python
# Copyright 2013-2021 Lawrence Livermore National Security, LLC and other
# Spack Project Developers. See the top-level COPYRIGHT file for details.
#
# SPDX-License-Identifier: (Apache-2.0 OR MIT)

from spack import *


class Mpileaks(AutotoolsPackage):
    """Tool to detect and report leaked MPI objects like MPI_Requests and
       MPI_Datatypes."""

    homepage = "https://github.com/hpc/mpileaks"
    url      = "https://github.com/hpc/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    version('1.0', '8838c574b39202a57d7c2d68692718aa')

    depends_on("mpi")
    depends_on("adept-utils")
    depends_on("callpath")

    def install(self, spec, prefix):
        configure("--prefix=" + prefix,
                  "--with-adept-utils=" + spec['adept-utils'].prefix,
                  "--with-callpath=" + spec['callpath'].prefix)
        make()
        make("install")
```

まず `AutotoolsPackage` クラスを継承します。

`install()` メソッドを保持することはできますが、そのほとんどはベースの `AutotoolsPackage`クラスで処理できます。
実際、オーバーライドする必要があるのは `configure_args()` だけです：

```python
# Copyright 2013-2021 Lawrence Livermore National Security, LLC and other
# Spack Project Developers. See the top-level COPYRIGHT file for details.
#
# SPDX-License-Identifier: (Apache-2.0 OR MIT)

from spack import *


class Mpileaks(AutotoolsPackage):
    """Tool to detect and report leaked MPI objects like MPI_Requests and
       MPI_Datatypes."""

    homepage = "https://github.com/hpc/mpileaks"
    url      = "https://github.com/hpc/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz"

    version('1.0', '8838c574b39202a57d7c2d68692718aa')

    variant("stackstart", values=int, default=0,
            description="Specify the number of stack frames to truncate")

    depends_on("mpi")
    depends_on("adept-utils")
    depends_on("callpath")

    def configure_args(self):
        stackstart = int(self.spec.variants['stackstart'].value)
        args = ["--with-adept-utils=" + spec['adept-utils'].prefix,
                "--with-callpath=" + spec['callpath'].prefix]
        if stackstart:
            args.extend(['--with-stack-start-c=%s' % stackstart,
                         '--with-stack-start-fortran=%s' % stackstart])
        return args
```

Spackがprefix設定を処理するので、`configure` への引数として除外できます。
パッケージが簡素になり、そしてパッケージャは `configure` や `make` が適切に含まれているかどうかを心配する必要はありません。

このバージョンの `mpileaks` パッケージは以前と同じようにインストールされますが、`AutotoolsPackage` クラスを使用すると、見た目がすっきりしたパッケージファイルでインストールできます。

## Makefile

プラットフォームやコンパイラ固有の変数を設定するために通常は `Makefile` を編集する必要があるパッケージ。
これらのパッケージは、これらのタイプのパッケージの作成に役立つ便利なメソッドを提供するサブクラス `Makefile` によって処理されます。

`MakefilePackage` クラスは、次のようなオーバライド可能な3つのフェーズがあります：

1. `edit()`
2. `build()`
3. `install()`

パッケージャは、`Makefile` の編集方法、そしてビルドフェーズまたはインストールフェーズに含めるターゲットを制御できます。

`MakefilePackage` クラスの内部も見てみましょう：

```bash
$ spack edit --build-system makefile
```

次の点に注意してください：

```python
class MakefilePackage(PackageBase):
    #: Phases of a package that is built with an hand-written Makefile
    phases = ['edit', 'build', 'install']
    #: This attribute is used in UI queries that need to know the build
    #: system base class
    build_system_class = 'MakefilePackage'

    #: Targets for ``make`` during the :py:meth:`~.MakefilePackage.build`
    #: phase
    build_targets = []
    #: Targets for ``make`` during the :py:meth:`~.MakefilePackage.install`
    #: phase
    install_targets = ['install']

    #: Callback names for build-time test
    build_time_test_callbacks = ['check']

    #: Callback names for install-time test
    install_time_test_callbacks = ['installcheck']

    def edit(self, spec, prefix):
        """Edits the Makefile before calling make. This phase cannot
        be defaulted.
        """
        tty.msg('Using default implementation: skipping edit phase.')

    def build(self, spec, prefix):
        """Calls make, passing :py:attr:`~.MakefilePackage.build_targets`
        as targets.
        """
        with working_dir(self.build_directory):
            inspect.getmodule(self).make(*self.build_targets)

    def install(self, spec, prefix):
        """Calls make, passing :py:attr:`~.MakefilePackage.install_targets`
        as targets.
        """
        with working_dir(self.build_directory):
            inspect.getmodule(self).make(*self.install_targets)
```

`Autotools` と同様に、 `MakefilePackage` クラスにはパッケージャが設定できるプロパティがあります。
強調表示されているさまざまなメソッドをオーバーライドすることもできます。

[Bowtie](http://bowtie-bio.sourceforge.net/index.shtml) パッケージを再作成してみましょう：

```bash
$ spack create -f https://downloads.sourceforge.net/project/bowtie-bio/bowtie/1.2.1.1/bowtie-1.2.1.1-src.zip
==> This looks like a URL for bowtie
==> Found 1 version of bowtie:

1.2.1.1  https://downloads.sourceforge.net/project/bowtie-bio/bowtie/1.2.1.1/bowtie-1.2.1.1-src.zip

==> How many would you like to checksum? (default is 1, q to abort) 1
==> Downloading...
==> Fetching https://downloads.sourceforge.net/project/bowtie-bio/bowtie/1.2.1.1/bowtie-1.2.1.1-src.zip
######################################################################## 100.0%
==> Checksummed 1 version of bowtie
==> This package looks like it uses the makefile build system
==> Created template for bowtie package
==> Created package file: /Users/mamelara/spack/var/spack/repos/builtin/packages/bowtie/package.py
```

フェッチが完了すると、Spackは通常の方法でテキストエディタを開き、`MakefilePackage` package.pyのテンプレートを作成します：

```python
# Copyright 2013-2021 Lawrence Livermore National Security, LLC and other
# Spack Project Developers. See the top-level COPYRIGHT file for details.
#
# SPDX-License-Identifier: (Apache-2.0 OR MIT)

from spack import *


class Bowtie(MakefilePackage):
    """FIXME: Put a proper description of your package here."""

    # FIXME: Add a proper url for your package's homepage here.
    homepage = "http://www.example.com"
    url      = "https://downloads.sourceforge.net/project/bowtie-bio/bowtie/1.2.1.1/bowtie-1.2.1.1-src.zip"

    version('1.2.1.1', 'ec06265730c5f587cd58bcfef6697ddf')

    # FIXME: Add dependencies if required.
    # depends_on('foo')

    def edit(self, spec, prefix):
        # FIXME: Edit the Makefile if necessary
        # FIXME: If not needed delete this function
        # makefile = FileFilter('Makefile')
        # makefile.filter('CC = .*', 'CC = cc')
        return
```

Spackは、`Bowtie` が `Make` を使用することを正常に検出できました。
パッケージの残りの詳細を追加しましょう：

```python
# Copyright 2013-2021 Lawrence Livermore National Security, LLC and other
# Spack Project Developers. See the top-level COPYRIGHT file for details.
#
# SPDX-License-Identifier: (Apache-2.0 OR MIT)

from spack import *


class Bowtie(MakefilePackage):
    """Bowtie is an ultrafast, memory efficient short read aligner
    for short DNA sequences (reads) from next-gen sequencers."""

    homepage = "https://sourceforge.net/projects/bowtie-bio/"
    url      = "https://downloads.sourceforge.net/project/bowtie-bio/bowtie/1.2.1.1/bowtie-1.2.1.1-src.zip"

    version('1.2.1.1', 'ec06265730c5f587cd58bcfef6697ddf')

    variant("tbb", default=False, description="Use Intel thread building block")

    depends_on("tbb", when="+tbb")

    def edit(self, spec, prefix):
        # FIXME: Edit the Makefile if necessary
        # FIXME: If not needed delete this function
        # makefile = FileFilter('Makefile')
        # makefile.filter('CC = .*', 'CC = cc')
        return
```

TBD