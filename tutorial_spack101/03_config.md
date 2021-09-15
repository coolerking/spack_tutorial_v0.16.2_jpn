> **チュートリアルセットアップ**
> これより前のチュートリアルセクションを実行していない場合は、次の手順でSpackをセットアップしてください。
> ```bash
> git clone https://github.com/spack/spack
> . spack/share/spack/setup-env.sh
> spack tutorial
> ```
> セットアップの詳細は [基本的なインストールチュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_basics.html#basics-tutorial) を参照のこと。
> さらに支援が欲しい場合はSlackチャネル `#tutorial` に参加してくださいー [spackpm.herokuapp.com](spackpm.herokuapp.com) で招待を受けてください

# 構成チュートリアル

このチュートリアルでは、ソフトウェアのインストールに関するSpackの動作をカスタマイズできるさまざまな構成オプションについて説明します。
最初に、構成ファイルの階層について説明します。
次に、コンパイラの構成オプションについて説明し、Spackのコンパイラ自動検出を拡張するためにそれらを使用する方法に焦点を当てます。
そして、パッケージ構成ファイルについて説明し、デフォルトのビルドオプションをオーバーライドする方法と、使用する外部パッケージインストールを指定する方法に焦点を当てます。
最後に、より高レベルのSpack構成オプションを管理する構成ファイルの設定方法について簡単に触れます。

これらすべての機能について、完全な構成ファイルを作成する方法を示します。
次に、構成がinstallコマンドにどのように影響するかを示すものもあれば、 `spack spec` コマンドを使用して、構成の変更がSpackの具体化アルゴリズムにどのように影響するかを示すものもあります。
ここでの出力表示はすべて Ubuntuバージョン18.04 Server のものです。

## 構成スコープ

チーム全員に共通の構成設定を提供したり、単一のユーザアカウントに固有のデフォルトの動作を設定するなど、ユースケースに対応することができます。
Spackは、このカスタマイズを処理するための6つの構成スコープを提供します。
これらのスコープは、優先度の高い順に次のとおりです：

|範囲|ディレクトリ|
|----|------------|
|コマンドライン|該当なし|
|環境|(`spack.yaml` の)環境ベースディレクトリ内|
|カスタム|`--config-scope` で指定されたカスタムディレクトリ|
|ユーザ|`~/.spack/`|
|サイト|`$SPACK_ROOT/etc/spack/`|
|システム|`/etc/spack`|
|デフォルト|`$SPACK_ROOT/etc/spack/defaults/`|

Spackのデフォルトの構成設定は `$SPACK_ROOT/etc/spack/defaults` に配置されています。
これらは参照はかまいませんが、直接編集はしないでください。
これらの設定を上書きするには、優先度の高い構成スコープのいずれかに新しい構成ファイルを作成します。

特定のクラスタには、異なるプロジェクトに関連付けられた複数のSpackインストールがある場合があります。
すべてのSpackインストールに共通の設定を提供するには、構成ファイルを `/etc/spack` に配置します。
特定のSpackインストールに固有の設定を提供するには、 `$SPACK_ROOT/etc/spack` ディレクトリを使用できます。

特定ユーザに固有設定を定義したい場合は、構成ファイルを `~/.spack` ディレクトリに追加する必要があります。
Spackが最初にシステム上のコンパイラをチェックしたとき、コンパイラ構成がこのディレクトリに配置されていることに気付いたかもしれません。

構成設定は、カスタムの場所に配置することもできます。
カスタムの場所は、コマンドラインで `--config-scope` オプションを使って指定します。
ユースケースの例は、2セットの構成を管理することです。
1つは開発用で、もう1つは本番環境用です。

コマンドラインで指定された設定は、他のすべての構成スコープよりも優先されます。

有効な構成を表示するために `spack config blame <config>` コマンドを使用することもできます。
Spackは、構成がどのスコープからアセンブルされたかを示します。

### プラットフォーム固有のスコープ

施設によっては、単一の共有ファイルシステムから複数のプラットフォームを管理する場合があります。
これを処理するために、上記の各構成スコープには、プラットフォーム固有とプラットフォーム非依存の2つのサブスコープがあります。
たとえば、コンパイラ設定は次の場所に保存できます。

- `environment-root-dir/spack.yaml`
- `~/.spack/<platform>/compilers.yaml`
- `~/.spack/compilers.yaml`
- `$SPACK_ROOT/etc/spack/<platform>/compilers.yaml`
- `$SPACK_ROOT/etc/spack/compilers.yaml`
- `/etc/spack/<platform>/compilers.yaml`
- `/etc/spack/compilers.yaml`
- `$SPACK_ROOT/etc/defaults/<platform>/compilers.yaml`
- `$SPACK_ROOT/etc/defaults/compilers.yaml`

これらのファイルは優先順位の高い順に列挙しています。`~/.spack/<platform>` ディレクトリのファイルは `~/.spack/` ディレクトリの設定をオーバライドします。

## YAML形式

Spack構成は、固有のスキーマを持つネストされたYAMLディクショナリです。
構成はテーマに基づいてセクション（ `compilers` など）に編成され、辞書の最上位キーがセクションを指定します。
Spackは通常、セクションごとに個別のファイルを保持しますが、環境ではセクションをまとめて( `spack.yaml` 内に)保持します。

Spackが構成をチェックすると、構成スコープが優先順位の高い順に辞書として更新され、優先順位の高いファイルが低い優先順位のファイルを上書きできるようになります。
YAMLディクショナリは、コロン `：` を使用してキーと値のペアを指定します。
SpackはYAML構文をわずかに拡張して、二重コロン `::` でキーと値のペアを指定できるようにします。
二重コロンが使用する場合、そのセクションを追加する代わりに、Spackはそのセクションにあったものを新しい値に置き換えます。
たとえば、次のようなユーザのコンパイラ構成ファイルがあります：

```yaml
compilers::
- compiler:
    spec: gcc@7.5.0
    paths:
      cc: /usr/bin/gcc
      cxx: /usr/bin/g++
      f77: /usr/bin/gfortran
      fc: /usr/bin/gfortran
    flags: {}
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
```

ユーザ構成スコープが最後に検索されたスコープであり、その `compilers::` 行が以前のすべての構成ファイル情報を置き換えるため、他のコンパイラが使用されないことが保証されます。
同じ構成ファイルに二重コロンではなく単一コロンがある場合、他の構成ファイルにリストされている他のコンパイラにGCCバージョン7.5.0コンパイラが追加されます。

構成セクションは、環境の `spack.yaml` ファイルで管理されている場合、セクションが最上位の `spack` キーの1レベル下にネストされていることを除いて、ほぼ同じように表示されます。
たとえば、上記の `compilers.yaml` は次に示す `spack.yaml` 環境ファイルに組み込むことができます。

```yaml
spack:
  specs: []
  view: true
  compilers::
  - compiler:
      spec: gcc@7.5.0
      paths:
        cc: /usr/bin/gcc
        cxx: /usr/bin/g++
        f77: /usr/bin/gfortran
        fc: /usr/bin/gfortran
      flags: {}
      operating_system: ubuntu18.04
      target: x86_64
      modules: []
      environment: {}
      extra_rpaths: []
```

## コンパイラ構成

ほとんどのタスクでは、システムで最初にSpackを実行したときに自動検出されたコンパイラがSpackで使用可能になります。
これまでのチュートリアルで説明したように、 `spack compiler add` コマンドを使用してコンパイラがどこにあるかをSpackに伝えることもできます。
ただし、状況によっては使用可能なコンパイラをさらにきめ細かく制御したい場合があります。
このセクションでは、コンパイラ構成ファイルを使用してその制御を実行する方法を説明します。

まずコンパイラ構成ファイルを開きます：

```bash
$ spack config edit compilers
```

有効な環境がない状態から始めるので、`compilers.yaml` が編集可能な状態で開きます（有効な環境でも可能）：

```yaml
compilers:
- compiler:
    spec: clang@6.0.0
    paths:
      cc: /usr/bin/clang-6.0
      cxx: /usr/bin/clang++-6.0
      f77:
      fc:
    flags: {}
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
- compiler:
    spec: gcc@6.5.0
    paths:
      cc: /usr/bin/gcc-6
      cxx:
      f77:
      fc:
    flags: {}
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
- compiler:
    spec: gcc@7.5.0
    paths:
      cc: /usr/bin/gcc-7
      cxx: /usr/bin/g++-7
      f77: /usr/bin/gfortran-7
      fc: /usr/bin/gfortran-7
    flags: {}
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
```

GCCコンパイラの2バージョンと、FlangコンパイラのないClangコンパイラの1バージョンを指定しています。
ここで、C/C++コード用のClangコンパイラでコンパイルしたいが、Fortranコンポーネントはgfortranでコンパイルしたいコードがあるとします。
これを行うには、`compilers.yaml` ファイルに別のエントリを追加します：

```yaml
- compiler:
    spec: clang@6.0.0-gfortran
    paths:
      cc: /usr/bin/clang-6.0
      cxx: /usr/bin/clang++-6.0
      f77: /usr/bin/gfortran-7
      fc: /usr/bin/gfortran-7
    flags: {}
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
```


変更を加えた `compiler` エントリのセクションについて説明しましょう。
最大の変更は、`paths` セクションです。
ここでは、それぞれのプログラミング言語およびその仕様（バージョン）に使用するコンパイラへのパスを列挙します。
この場合、C/C++用のClangコンパイラと、Fortranの両方の仕様( `f77` と `fc` )用の `gfortran` コンパイラを指定しています。
このコンパイラの `spec` エントリも変更しました。
`spec` エントリでは Spack内で使用するコンパイラ名を指定しており、名前とバージョン番号で構成され `@` で区切って記述します。
名前は　Spackでサポートされているコンパイラ名（arm、cce、clang、fj、gcc、intel、nag、pgi、xl、xl_r）のいずれかである必要があります。
バージョン番号は、任意の英数字ならびに `-` 、 `.` 、`-` を含む文字列でなくてはなりません。
`target` と `operating_system` セクションは変更しません。
これらの2つのセクションでは、Spackがさまざまなコンパイラを使用できることを定義し、主に複数システムで使用される構成ファイルに役立ちます。

次のように呼び出すことで、新しいコンパイラが機能することを確認できます：

```bash
$ spack install --no-cache zlib %clang@6.0.0-gfortran
...
```

この新しいコンパイラは、Fortranコードでも機能します。
次のように、バイナリキャッシュ内で既に利用可能なビルド依存関係 `cmake%gcc@7.5.0` をを使って、小さなパッケージ（`json-fortran`）をコンパイルします：

```bash
$ spack install cmake %gcc@7.5.0
...
$ spack install --no-cache json-fortran %clang@6.0.0-gfortran ^cmake%gcc@7.5.0
...
```

### コンパイラフラグ

コンパイラの中には、特定のコンピューティング環境で正しく機能するために特定のコンパイラフラグを必要とする場合があります。
Spackは、特定のコンパイラが呼び出されるたびにコンパイラフラグを設定するための構成オプションを提供します。
これらのフラグはパッケージ仕様の一部になり、したがってビルドの出自の一部になります。
ビルド時のコマンドライン上のオプションとして、フラグが暗黙のビルド変数（`cflags` 、 `cxxflags` 、 `cppflags` 、 `fflags` 、 `ldflags` そして `ldlibs` ）を介して設定されます。

コンパイラ構成ファイルを再び開いて、コンパイラフラグを次のように追加してみましょう：

```yaml
- compiler:
    spec: clang@6.0.0-gfortran
    paths:
      cc: /usr/bin/clang-6.0
      cxx: /usr/bin/clang++-6.0
      f77: /usr/bin/gfortran-7
      fc: /usr/bin/gfortran-7
    flags:
      cppflags: -g
    operating_system: ubuntu18.04
    target: x86_64
    modules: []
    environment: {}
    extra_rpaths: []
```

`spack spec` コマンドでテストし、仕様がどのように具体（コンクリート）化されているかを確認することができます：

```bash
$ spack spec json-fortran%clang@6.0.0-gfortran
Input spec
--------------------------------
json-fortran%clang@6.0.0-gfortran

Concretized
--------------------------------
json-fortran@7.1.0%clang@6.0.0-gfortran cppflags="-g" ~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
    ^cmake@3.18.4%clang@6.0.0-gfortran cppflags="-g" ~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
        ^ncurses@6.2%clang@6.0.0-gfortran cppflags="-g" ~symlinks+termlib arch=linux-ubuntu18.04-x86_64
            ^pkgconf@1.7.3%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
        ^openssl@1.1.1h%clang@6.0.0-gfortran cppflags="-g" +systemcerts arch=linux-ubuntu18.04-x86_64
            ^perl@5.32.0%clang@6.0.0-gfortran cppflags="-g" +cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
                ^berkeley-db@18.1.40%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^gdbm@1.18.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                    ^readline@8.0%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^zlib@1.2.11%clang@6.0.0-gfortran cppflags="-g" +optimize+pic+shared arch=linux-ubuntu18.04-x86_64
```

DAGのすべてのノードに `cppflags="-g"` が追加されていることが確認できます。

## 高度なコンパイラ構成

コンパイラ構成ファイルのエントリには、まだ説明していない3つのフィールドがあります。

`compiler` の `modules` フィールドは主にCrayシステムで使用されますが、特定のモジュールがロードされた場合にのみ役立つコンパイラを備えたシステムでも役に立ちます。
コンパイラ構成の `modules` フィールドにあるすべてのモジュールは、 そのコンパイラーを使用するパッケージのビルド環境の一部としてロードされます：

```yaml
- compiler:
    ...
    modules:
    - PrgEnv-gnu
    - gcc/5.3.0
    ...
```

コンパイラ構成の `environment` フィールドは、ビルド時に環境変数を設定する必要があるコンパイラに使用されます。
たとえば、インテル(R)コンパイラ・スイートが適切なライセンス・サーバを指定するように環境変数 `INTEL_LICENSE_FILE` を必要とする場合、`compilers.yaml` を次のように設定します：

```yaml
- compiler:
    ...
    environment:
      set:
        INTEL_LICENSE_FILE: 1713@license4
    ...
```

`set` に加えて `environment` もまた `unset` 、 `prepend_path` そして `append_path` をサポートします。

コンパイラ構成の `extra_rpaths` フィールドは、デフォルトですべての依存関係をrpathしないコンパイラに使用されます。
コンパイラはSpackの外部にインストールされることが多いため、Spackはコンパイラの依存関係を管理して rpath の使用を強制させることができません。
これにより、パッケージがコンパイラによって課されたリンク依存関係を適切に検出できない場合があります。
実行可能ファイルに自動的にrpathされない結果の実行可能ファイルにリンク依存関係を課すコンパイラの場合、

コンパイラ構成の `extra_rpaths` フィールドは、そのコンパイラによって作成されたすべての実行可能ファイルに rpath する依存関係をSpackに指示します。
実行可能ファイルは、コンパイラによって課されたリンクの依存関係を見つけることができます。
`extra_rpaths` フィールドの設定例は次の通りです：

```yaml
- compiler:
    ...
    extra_rpaths:
    - /apps/intel/ComposerXE2017/compilers_and_libraries_2017.5.239/linux/compiler/lib/intel64_lin
    ...
```

> 訳者注：
> リンクする動的ライブラリがさらに別の動的ライブラリを必要とし、それがSpack管理下にないパスにある場合に指定ればいいらしい。

## パッケージ設定の構成

Spackのパッケージ設定は、 `packages` 構成セクションで管理されます。まず、デフォルトの `packages.yaml` ファイルを見てみましょう：

```bash
$ spack config --scope defaults edit packages
```

```yaml
# -------------------------------------------------------------------------
# This file controls default concretization preferences for Spack.
#
# Settings here are versioned with Spack and are intended to provide
# sensible defaults out of the box. Spack maintainers should edit this
# file to keep it current.
#
# Users can override these settings by editing the following files.
#
# Per-spack-instance settings (overrides defaults):
#   $SPACK_ROOT/etc/spack/packages.yaml
#
# Per-user settings (overrides default and site settings):
#   ~/.spack/packages.yaml
# -------------------------------------------------------------------------
packages:
  all:
    compiler: [gcc, intel, pgi, clang, xl, nag, fj]
    providers:
      D: [ldc]
      awk: [gawk]
      blas: [openblas]
      daal: [intel-daal]
      elf: [elfutils]
      fftw-api: [fftw]
      gl: [mesa+opengl, opengl]
      glx: [mesa+glx, opengl]
      glu: [mesa-glu, openglu]
      golang: [gcc]
      ipp: [intel-ipp]
      java: [openjdk, jdk, ibm-java]
      jpeg: [libjpeg-turbo, libjpeg]
      lapack: [openblas]
      mariadb-client: [mariadb-c-client, mariadb]
      mkl: [intel-mkl]
      mpe: [mpe2]
      mpi: [openmpi, mpich]
      mysql-client: [mysql, mariadb-c-client]
      opencl: [pocl]
      pil: [py-pillow]
      pkgconfig: [pkgconf, pkg-config]
      scalapack: [netlib-scalapack]
      szip: [libszip, libaec]
      tbb: [intel-tbb]
      unwind: [libunwind]
    permissions:
      read: world
      write: user
```

ここではコンパイラおよび仮想パッケージのプロバイダのデフォルトが設定されています。
これがどのように機能するかを説明するために、Clangコンパイラを優先し、OpenMPIよりもMPICHを優先する設定変更をおこなうこととします。
現在の設定では、GCCとOpenMPIを優先しています。

```bash
$ spack spec hdf5
Input spec
--------------------------------
hdf5

Concretized
--------------------------------
hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran~hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^openmpi@3.1.6%gcc@7.5.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
        ^hwloc@1.11.11%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
            ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                    ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
                        ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
            ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
                ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
                ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
            ^numactl@2.0.14%gcc@7.5.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
                ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                    ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
                        ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                        ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                            ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
                                ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
                ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
```

環境内でデフォルト設定をオーバーライドしてみましょう。
有効化した環境の場合、関連する構成を `spack config edit` コマンドで編集できます（セクション名を指定する必要はありません）。

```bash
$ spack env create config-env
$ spack env activate config-env
$ spack config edit
```

> **警告**
> 環境を使用せずにこれらの変更を行った場合もまったく同じ効果が得られますが、構成チュートリアルの後で関連する `packages.yaml` ファイルを削除する必要があります。
> そうしないと、（ここで実行されるはずの構成変更が実行されなかったため、）後のチュートリアルセクションで実行するコマンドが同じ出力を生成しません。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
```

前に説明した構成スコープのため、これら2つの項目のデフォルト設定をオーバーライドします。

```bash
$ spack spec hdf5
Input spec
--------------------------------
hdf5

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^mpich@3.3.2%clang@6.0.0-gfortran cppflags="-g" ~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
        ^autoconf@2.69%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^m4@1.4.18%clang@6.0.0-gfortran cppflags="-g" +sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
                ^libsigsegv@2.12%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^perl@5.32.0%clang@6.0.0-gfortran cppflags="-g" +cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
                ^berkeley-db@18.1.40%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^gdbm@1.18.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                    ^readline@8.0%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                        ^ncurses@6.2%clang@6.0.0-gfortran cppflags="-g" ~symlinks+termlib arch=linux-ubuntu18.04-x86_64
                            ^pkgconf@1.7.3%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
        ^automake@1.16.2%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
        ^findutils@4.6.0%clang@6.0.0-gfortran cppflags="-g"  patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
            ^libtool@2.4.6%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^texinfo@6.5%clang@6.0.0-gfortran cppflags="-g"  patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
        ^hwloc@2.2.0%clang@6.0.0-gfortran cppflags="-g" ~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
            ^libpciaccess@0.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^util-macros@1.19.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^libxml2@2.9.10%clang@6.0.0-gfortran cppflags="-g" ~python arch=linux-ubuntu18.04-x86_64
                ^libiconv@1.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^xz@5.2.5%clang@6.0.0-gfortran cppflags="-g" ~pic arch=linux-ubuntu18.04-x86_64
                ^zlib@1.2.11%clang@6.0.0-gfortran cppflags="-g" +optimize+pic+shared arch=linux-ubuntu18.04-x86_64
```

### バリアント設定

パッケージ構成ファイルは、パッケージ改訂版のバリアント（改訂箇所）を設定することもできます。
たとえば、ここで共有ライブラリなしですべてのパッケージをビルドするように設定変更することとします。
これを実現するには、 `shared` バリアントを含むすべてのパッケージでバリアントをオフにします。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
```

次のように `spack spec hdf5` コマンドを実行することで確認できます：

```bash
$ spack spec hdf5
Input spec
--------------------------------
hdf5

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java+mpi+pic~shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^mpich@3.3.2%clang@6.0.0-gfortran cppflags="-g" ~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
        ^autoconf@2.69%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^m4@1.4.18%clang@6.0.0-gfortran cppflags="-g" +sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
                ^libsigsegv@2.12%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^perl@5.32.0%clang@6.0.0-gfortran cppflags="-g" +cpanm~shared+threads arch=linux-ubuntu18.04-x86_64
                ^berkeley-db@18.1.40%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^gdbm@1.18.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                    ^readline@8.0%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                        ^ncurses@6.2%clang@6.0.0-gfortran cppflags="-g" ~symlinks+termlib arch=linux-ubuntu18.04-x86_64
                            ^pkgconf@1.7.3%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
        ^automake@1.16.2%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
        ^findutils@4.6.0%clang@6.0.0-gfortran cppflags="-g"  patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
            ^libtool@2.4.6%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^texinfo@6.5%clang@6.0.0-gfortran cppflags="-g"  patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
        ^hwloc@2.2.0%clang@6.0.0-gfortran cppflags="-g" ~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci~shared arch=linux-ubuntu18.04-x86_64
            ^libpciaccess@0.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^util-macros@1.19.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^libxml2@2.9.10%clang@6.0.0-gfortran cppflags="-g" ~python arch=linux-ubuntu18.04-x86_64
                ^libiconv@1.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^xz@5.2.5%clang@6.0.0-gfortran cppflags="-g" ~pic arch=linux-ubuntu18.04-x86_64
                ^zlib@1.2.11%clang@6.0.0-gfortran cppflags="-g" +optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

これまでのところ、パッケージ設定にグローバルな変更を加えただけです。
このチュートリアル全体で見てきたように、HDF5は、Spackのデフォルト設定であるMPIが有効になっている状態でビルドされます。
シリアルHDF5を日常的に必要とするプロジェクトに取り組んでいると、すぐに煩わしくなり、常に `hdf5~mpi` を入力する必要があります。
その代わりに、HDF5の設定を更新します。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
    hdf5:
      variants: ~mpi
```

これで hdf5 は、デフォルト設定としてMPI依存関係なしでコンクリート化されます。

```bash
$ spack spec hdf5
Input spec
--------------------------------
hdf5

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java~mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^zlib@1.2.11%clang@6.0.0-gfortran cppflags="-g" +optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

一般に、すべてのパッケージに設定できるすべての属性は、個々のパッケージに個別に設定できます。

### 外部パッケージ

Spack外部でインストールされたパッケージを使ったビルドのタイミングもパッケージ構成ファイルは制御します。
Spackには、 外部にインストールされたパッケージを自動的に検出して登録できる `spack external find` コマンドがあります。
これは多くの一般的なビルド依存関係で機能しますが、Spackがまだ検出できないパッケージに対してこれを手動で行う方法を知ることも重要です。

これらのシステムには、zlibがプリインストールされています。
このパッケージとその場所についてSpackに伝えましょう。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
    hdf5:
      variants: ~mpi
    zlib:
      externals:
      - spec: zlib@1.2.8%gcc@7.5.0
        prefix: /usr
```

上記の記述では、zlib1.2.8がシステムにインストールされていることをSpackに伝えました。
また、zlibが見つかるインストールプレフィックスについても説明しました。
どのバリアントで構築されたかは正確にはわかりませんが、問題ありません。

```bash
$ spack spec hdf5
Input spec
--------------------------------
hdf5

Concretized
--------------------------------
hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran~hl~java~mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^zlib@1.2.8%gcc@7.5.0+optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

Spackが外部のzlibインストールを使用していることに気付くでしょうが、zlibのビルドに使用されたコンパイラは、コンパイラ設定であるclangをオーバーライドしています。
Clangを明示的に指定した場合：

```bash
$ spack spec hdf5%clang
Input spec
--------------------------------
hdf5%clang

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java~mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^zlib@1.2.11%clang@6.0.0-gfortran cppflags="-g" +optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

Spackは、Clangで構築されているHDF5とzlibの両方に具体化しています。
これには、zlibを再構築するという副作用があります。
Spackにシステムzlibの使用を強制する場合は、2つの選択肢があります。
コマンドラインで指定するか、独自のzlibをビルドすることは許可されていないことをSpackに伝えることができます。
ここでは後者を使用します：

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
    hdf5:
      variants: ~mpi
    zlib:
      externals:
      - spec: zlib@1.2.8%gcc@7.5.0
        prefix: /usr
      buildable: false
```

これで、Spackは外部でインストールされた zlib を選択するように強制されます。

```bash
$ spack spec hdf5%clang
Input spec
--------------------------------
hdf5%clang

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java~mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^zlib@1.2.8%gcc@7.5.0+optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

これは、仮想の依存関係によって少し複雑になります。
独自のMPIを構築したくないが、HDF5の並列バージョンが必要である、とします。
幸い、これらのシステムにはMPICHがインストールされています。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
    hdf5:
      variants: ~mpi
    zlib:
      externals:
      - spec: zlib@1.2.8%gcc@7.5.0
        prefix: /usr
      buildable: false
    mpich:
      externals:
      - spec: mpich@3.3%gcc@7.5.0
        prefix: /usr
      buildable: false
```

この構成ファイルで `hdf5+mpi` をコンクリート化する場合は、代替のMPI実装でビルドするだけです。

```bash
$ spack spec hdf5+mpi %clang
Input spec
--------------------------------
hdf5%clang+mpi

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^openmpi@3.1.6%clang@6.0.0-gfortran cppflags="-g" ~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
        ^hwloc@1.11.11%clang@6.0.0-gfortran cppflags="-g" ~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci~shared arch=linux-ubuntu18.04-x86_64
            ^libpciaccess@0.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^libtool@2.4.6%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                    ^m4@1.4.18%clang@6.0.0-gfortran cppflags="-g" +sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
                        ^libsigsegv@2.12%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^pkgconf@1.7.3%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^util-macros@1.19.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
            ^libxml2@2.9.10%clang@6.0.0-gfortran cppflags="-g" ~python arch=linux-ubuntu18.04-x86_64
                ^libiconv@1.16%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                ^xz@5.2.5%clang@6.0.0-gfortran cppflags="-g" ~pic arch=linux-ubuntu18.04-x86_64
                ^zlib@1.2.8%gcc@7.5.0+optimize+pic~shared arch=linux-ubuntu18.04-x86_64
            ^numactl@2.0.14%clang@6.0.0-gfortran cppflags="-g"  patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
                ^autoconf@2.69%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                    ^perl@5.32.0%clang@6.0.0-gfortran cppflags="-g" +cpanm~shared+threads arch=linux-ubuntu18.04-x86_64
                        ^berkeley-db@18.1.40%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                        ^gdbm@1.18.1%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                            ^readline@8.0%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
                                ^ncurses@6.2%clang@6.0.0-gfortran cppflags="-g" ~symlinks+termlib arch=linux-ubuntu18.04-x86_64
                ^automake@1.16.2%clang@6.0.0-gfortran cppflags="-g"  arch=linux-ubuntu18.04-x86_64
```

私たちは他のMPI実装よりもMPICHを優先することを設定しただけであり、Spackはビルドを禁止していないパッケージで問題なくビルドします。
`hdf5+mpi%clang^mpich` と明示的にリクエストすることでこれを解決することも、他のMPI実装を使用しないようにSpackを構成することもできます。
このセクションでは構成に重点を置いているので（、しかも前者は退屈なので）、パッケージ構成を再度変更する必要があります。

デフォルトで再びMPIを使用してビルドするようにHDF5を構成が可能です。

```yaml
spack:
  specs: []
  view: true
  packages:
    all:
      compiler: [clang, gcc, intel, pgi, xl, nag, fj]
      providers:
        mpi: [mpich, openmpi]
      variants: ~shared
    zlib:
      externals:
      - spec: zlib@1.2.8%gcc@7.5.0
        prefix: /usr
      buildable: false
    mpich:
      externals:
      - spec: mpich@3.3%gcc@7.5.0
        prefix: /usr
    mpi:
      buildable: false
```

MPIプロバイダを構築しないようにSpackを構成したので、再試行します。

```bash
$ spack spec hdf5%clang
Input spec
--------------------------------
hdf5%clang

Concretized
--------------------------------
hdf5@1.10.7%clang@6.0.0-gfortran cppflags="-g" ~cxx~debug~fortran~hl~java+mpi+pic~shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
    ^mpich@3.3%gcc@7.5.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=c7d4ecf865dccff5b764d9c66b6a470d11b0b1a5b4f7ad1ffa61079ad6b5dede,eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
    ^zlib@1.2.8%gcc@7.5.0+optimize+pic~shared arch=linux-ubuntu18.04-x86_64
```

`packages.yaml` 内でパッケージ設定のほとんどを構成することにより、コマンドラインで仕様を指定するときに実行する必要のある作業量を削減できます。
コンパイラとバリアントの設定に加えて、バージョンの設定も指定できます。
`^` を介して依存関係を指定することを除いて、コマンドラインで指定できるものはすべて `packages.yaml` 内にまったく同じ仕様構文指定することができます。

### インストール権限

`packages` 構成では、パッケージのインストール時に使用するデフォルトのアクセス許可も制御されます。
デフォルトでは、インストールプレフィックスは誰でも読み取り可能ですが、ユーザが書き込み可能であることに気付くでしょう。

ライセンスされたソフトウェアパッケージ `converge` をインストールする必要があるとします。
特定の研究グループ `fluid_dynamics` がこのライセンスの料金を支払うため、このグループのメンバのみがソフトウェアにアクセスできるようにする必要があります。
次のようにして実現することができます：

```yaml
packages:
  converge:
    permissions:
      read: group
      group: fluid_dynamics
```

これで `fluid_dynamics` グループのメンバのみが任意の `converge` インストールを使用できます。

では、このチュートリアルセクションで行った構成の変更を破棄して、環境を無効化できるようにします。

```bash
$ spack env deactivate
```

> **警告**
> 環境を無効化しない場合、仕様は後のチュートリアルセクションで異なる方法でコンクリート化され、結果は一致しなくなります。

## 高度な設定

コンパイラとパッケージの設定に加えて、Spackではいくつかの高レベルの設定をカスタマイズできます。
これらの設定は、 `config` セクションで（環境外で個別のファイルとして保存される場合は `config.yaml` 内で）管理されます。
次のように実行すると、デフォルト設定を確認できます。

```bash
$ spack config --scope defaults edit config
```

```yaml
# -------------------------------------------------------------------------
# This is the default spack configuration file.
#
# Settings here are versioned with Spack and are intended to provide
# sensible defaults out of the box. Spack maintainers should edit this
# file to keep it current.
#
# Users can override these settings by editing the following files.
#
# Per-spack-instance settings (overrides defaults):
#   $SPACK_ROOT/etc/spack/config.yaml
#
# Per-user settings (overrides default and site settings):
#   ~/.spack/config.yaml
# -------------------------------------------------------------------------
config:
  # This is the path to the root of the Spack install tree.
  # You can use $spack here to refer to the root of the spack instance.
  install_tree: $spack/opt/spack


  # Locations where templates should be found
  template_dirs:
    - $spack/share/spack/templates


  # Default directory layout
  install_path_scheme: "${ARCHITECTURE}/${COMPILERNAME}-${COMPILERVER}/${PACKAGE}-${VERSION}-${HASH}"


  # Locations where different types of modules should be installed.
  module_roots:
    tcl:    $spack/share/spack/modules
    lmod:   $spack/share/spack/lmod


  # Temporary locations Spack can try to use for builds.
  #
  # Recommended options are given below.
  #
  # Builds can be faster in temporary directories on some (e.g., HPC) systems.
  # Specifying `$tempdir` will ensure use of the default temporary directory
  # (i.e., ``$TMP` or ``$TMPDIR``).
  #
  # Another option that prevents conflicts and potential permission issues is
  # to specify `~/.spack/stage`, which ensures each user builds in their home
  # directory.
  #
  # A more traditional path uses the value of `$spack/var/spack/stage`, which
  # builds directly inside Spack's instance without staging them in a
  # temporary space.  Problems with specifying a path inside a Spack instance
  # are that it precludes its use as a system package and its ability to be
  # pip installable.
  #
  # In any case, if the username is not already in the path, Spack will append
  # the value of `$user` in an attempt to avoid potential conflicts between
  # users in shared temporary spaces.
  #
  # The build stage can be purged with `spack clean --stage` and
  # `spack clean -a`, so it is important that the specified directory uniquely
  # identifies Spack staging to avoid accidentally wiping out non-Spack work.
  build_stage:
    - $tempdir/$user/spack-stage
    - ~/.spack/stage
  # - $spack/var/spack/stage


  # Cache directory for already downloaded source tarballs and archived
  # repositories. This can be purged with `spack clean --downloads`.
  source_cache: $spack/var/spack/cache


  # Cache directory for miscellaneous files, like the package index.
  # This can be purged with `spack clean --misc-cache`
  misc_cache: ~/.spack/cache


  # If this is false, tools like curl that use SSL will not verify
  # certifiates. (e.g., curl will use use the -k option)
  verify_ssl: true


  # If set to true, Spack will attempt to build any compiler on the spec
  # that is not already available. If set to False, Spack will only use
  # compilers already configured in compilers.yaml
  install_missing_compilers: False


  # If set to true, Spack will always check checksums after downloading
  # archives. If false, Spack skips the checksum step.
  checksum: true


  # If set to true, `spack install` and friends will NOT clean
  # potentially harmful variables from the build environment. Use wisely.
  dirty: false


  # The language the build environment will use. This will produce English
  # compiler messages by default, so the log parser can highlight errors.
  # If set to C, it will use English (see man locale).
  # If set to the empty string (''), it will use the language from the
  # user's environment.
  build_language: C


  # When set to true, concurrent instances of Spack will use locks to
  # avoid modifying the install tree, database file, etc. If false, Spack
  # will disable all locking, but you must NOT run concurrent instances
  # of Spack.  For filesystems that don't support locking, you should set
  # this to false and run one Spack at a time, but otherwise we recommend
  # enabling locks.
  locks: true


  # The maximum number of jobs to use when running `make` in parallel,
  # always limited by the number of cores available. For instance:
  # - If set to 16 on a 4 cores machine `spack install` will run `make -j4`
  # - If set to 16 on a 18 cores machine `spack install` will run `make -j16`
  # If not set, Spack will use all available cores up to 16.
  # build_jobs: 16


  # If set to true, Spack will use ccache to cache C compiles.
  ccache: false


  # How long to wait to lock the Spack installation database. This lock is used
  # when Spack needs to manage its own package metadata and all operations are
  # expected to complete within the default time limit. The timeout should
  # therefore generally be left untouched.
  db_lock_timeout: 120


  # How long to wait when attempting to modify a package (e.g. to install it).
  # This value should typically be 'null' (never time out) unless the Spack
  # instance only ever has a single user at a time, and only if the user
  # anticipates that a significant delay indicates that the lock attempt will
  # never succeed.
  package_lock_timeout: null
```

ご覧のとおり、Spackが使用するディレクトリの多くはカスタマイズできます。
たとえば、 `$SPACK_ROOT` 階層外のプレフィックスにパッケージをインストールするようにSpackに指示できます。
複数のSpackインスタンスを使用している場合は、モジュールファイルを中央の場所に書き込むことができます。
高速なスクラッチファイルシステムを使用している場合は、次のような `config.yaml` でこのファイルシステムからビルドを実行できます。

```yaml
config:
  build_stage:
    - /scratch/$user/spack-stage
```

> **注意**
> ビルドステージディレクトリをスクラッチスペース内の他のディレクトリと区別して、`spack clean` により無関係なファイルが誤って削除されないようにすることが重要です。
> これは、デフォルト設定と文書化された例に示されているように、`spack` と `stage` の組み合わせを各パスに含めることで実現できます。
> 詳細については、 [基本設定](https://spack.readthedocs.io/en/latest/config_yaml.html#config-yaml) を参照してください。

`LD_LIBRARY_PATH` のような環境変数を絶対に必要とするコンパイラを備えたシステムでは、Spackが次の `dirty` 設定でビルド環境をクリーンアップするのを防ぐことができます：

```yaml
config:
  dirty: true
```

ただし、不要なライブラリをビルドに取り込む可能性があるため、これは強くお勧めしません。

多くのユーザが関心を持つ可能性のある最後の設定は、Spackビルドの並列処理をカスタマイズする機能です。
デフォルトでは、Spackはノード上のコアの数に等しいジョブの数（最大16）並行してすべてのパッケージをインストールします。
たとえば16コアのノードでは、次のようになります：

```bash
$ spack install --no-cache --verbose --overwrite --yes-to-all zlib
==> Installing zlib
==> Executing phase: 'install'
==> './configure' '--prefix=/home/user/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg'
...
==> 'make' '-j16'
...
==> 'make' '-j16' 'install'
...
[+] /home/user/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
```

上記では、ノード上16個のコアすべてを使用してビルドしています。
共有ログインノードを使用している場合、これにより他のユーザのシステムの速度が低下する可能性があります。
使用可能なライセンスの数に厳しい制限または制限がある場合、これほど多くのコアでビルドできない可能性があります。
ビルドで使用するコアの数を制限するには、次のように `build_jobs` を設定します：

```bash
$ spack config edit config
```

```yaml
config:
  build_jobs: 2
```

zlibをアンインストールして再インストールすると、2つのコアのみが使用されていることがわかります。

```bash
$ spack install --no-cache --verbose --overwrite --yes-to-all zlib
==> Installing zlib
==> Executing phase: 'install'
==> './configure' '--prefix=/home/user/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg'
...
==> 'make' '-j2'
...
==> 'make' '-j2' 'install'
...
[+] /home/user/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
```

何らかの理由ですべてをシリアルでビルドする場合は、build_jobsを1に設定します。

## まとめ

このチュートリアルでは、`compilers.yaml` と `packages.yaml` を使用して基本的なSpack設定をカバーしました。
Spackには、[モジュールファイルチュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_modules.html#modules-tutorial) で説明する `modules.yaml` を含む、さらに多くの構成ファイルがあります。
Spackの多くの構成設定の詳細については、Spackのメインドキュメントの [構成セクション](https://spack.readthedocs.io/en/latest/configuration.html) を参照してください 。

他のサイトでSpackを構成する方法のサンプルについては、 [https://github.com/spack/spack-configs](https://github.com/spack/spack-configs) を参照して ください。
サイトでSpackを使用していて、構成ファイルを共有したい場合は、プルリクエストを送信してください。
