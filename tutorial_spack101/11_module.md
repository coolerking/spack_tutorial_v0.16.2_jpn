# モジュールファイルチュートリアル

このチュートリアルでは、Spackを使用し、インストールされているソフトウェアのモジュールファイルを生成する方法を解説します。
階層的展開と非階層的展開の両方について詳しく説明し、各モジュールファイルのコンテンツと名前をカスタマイズする方法を示します。

本チュートリアルを実行することで、次のことを理解することができます。

- モジュールファイルとは何か、およびそれらがHPCクラスタでどのように使用されるか
- Spackがインストールするソフトウェアのモジュールファイルを生成する方法
- モジュールファイルの管理に使用できるSpackコマンド
- Spackによって生成されたモジュールファイルをカスタマイズする方法

また Spack は、HPCクラスタへのソフトウェアのインストールを維持管理のすべての一般的なユースケースに対処できることを確認できます。

## チュートリアルのためのセットアップ

チュートリアルの準備として、同じパッケージといくつかの外部パッケージの異なる構成を含む、小さいながらも代表的なソフトウェアのセットをインストールします。
インストールを管理しやすくするために、チュートリアルの前半のすべてをアンインストールすることから始めましょう。

```bash
$ spack uninstall -ay
```

### モジュールツールの作成

最初に必要なのは、モジュールツール自体です。
チュートリアルでは、階層的レイアウトと非階層的レイアウトの両方で機能する `lmod` を使用します。

```bash
$ spack install lmod
```

モジュールツールをインストールしたら、現在のシェルで使用できるようにする必要があります。
Spackのストアのインストールディレクトリは、覚えるのは簡単ではありませんが、`spack location`コマンドで取得できます：

```bash
$ . $(spack location -i lmod)/lmod/lmod/init/bash
```

これで、セットアップファイルを再ソースでき、Spackモジュールがモジュールパスに配置されます。

```bash
$ . share/spack/setup-env.sh
```

### 新しいコンパイラの追加

2番目のステップは、最新コンパイラを構築することです。
Spackは最初の使用時に環境をスキャンし、システムですでに使用可能なコンパイラを自動的に発見します。
ただし、このチュートリアルでは `gcc@8.3.0` を使用することとします。

```bash
$ spack install gcc@8.3.0
```

これは、`spack load gcc@8.3.0` コマンドを使うと環境で取得できます。

```bash
$ spack load gcc@8.3.0
$ which gcc
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gcc
```

これで、`gcc` への `PATH` が通るようになり、`spack compiler add`コマンドを使ってコンパイラのリストに追加することができます：

```bash
$ spack compiler add
==> Added 1 new compiler to /home/spack/.spack/linux/compilers.yaml
    gcc@8.3.0
==> Compilers are defined in the following files:
    /home/spack/.spack/linux/compilers.yaml
```

```bash
$ spack compiler list
==> Available compilers
-- clang ubuntu18.04-x86_64 -------------------------------------
clang@6.0.0

-- gcc ubuntu18.04-x86_64 ---------------------------------------
gcc@8.3.0  gcc@7.5.0  gcc@6.5.0
```

### チュートリアルで使用するソフトウェアをビルド

最後に Spack を使って、例にて使用するパッケージをインストールします：

```bash
$ spack install netlib-scalapack ^openmpi ^openblas
$ spack install netlib-scalapack ^mpich ^openblas
$ spack install netlib-scalapack ^openmpi ^netlib-lapack
$ spack install netlib-scalapack ^mpich ^netlib-lapack
$ spack install py-scipy ^openblas
```

## モジュールファイルとは何か？

モジュールファイルは、シェルセッション中に制御された方法で環境を変更する簡単な方法です。
通常モジュールファイルには、アプリケーション実行やライブラリ使用に必要な情報が含まれています。
この `module` コマンドは、モジュールファイルを解釈して実行するために使用されます。
たとえば `module show` は、モジュールがロードされたときに何をするかを示します：

```bash
$ module show zlib
----------------------------------------------------------------------------------------------------------------------------------------------
   /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/zlib/1.2.11-gcc-8.3.0:
----------------------------------------------------------------------------------------------------------------------------------------------
whatis("A free, general-purpose, legally unencumbered lossless data-compression library. ")
conflict("zlib")
prepend_path("MANPATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/share/man")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/lib")
prepend_path("INCLUDE","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/include")
prepend_path("PKG_CONFIG_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/lib/pkgconfig")
prepend_path("CMAKE_PREFIX_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/")
help([[A free, general-purpose, legally unencumbered lossless data-compression
library.
]])

$ echo $LD_LIBRARY_PATH
```

`module load` を実行すると、上記のすべての変更を実行します：

```bash
$ module load zlib
$ echo $LD_LIBRARY_PATH
/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/zlib-1.2.11-2icaxiyy5rrqdavfv7jbojdq36r6u37n/lib
```

変更を元に戻すには、`module unload` を使います：

```bash
$ module unload zlib
$ echo $LD_LIBRARY_PATH

$
```

### モジュールシステム

HPCで使用されるメインモジュールシステムは2つあり、どちらもSpackでインストールできます。
このチュートリアルでは `lmod` 、TclとLuaの両方を使用して例を示します。

#### 環境モジュール

これはオリジナルのモジュールツールです。
次のコマンドを使用して、Spackをつかってインストールすることができます：

```bash
$ spack install environment-modules
```

1990年代初頭に最初にCでコーディングされ、後にTclで完全に書き直されました。
長い間停滞していたこのプロジェクトは、過去数年間にCEAのXavier Delaruelleによって復活し、現在非常に活発に開発されています。
詳細については、 [ドキュメント](https://modules.readthedocs.io/) を参照してください。

#### Lmod

Lmodは、Luaで記述されたモジュールシステムであり、元々はRobertMcLayによって「TexasAdvancedComputing Center」（TACC）で作成されました。
次のコマンドを実行することで取得できます：

```bash
$ spack install lmod
```

チュートリアルセクションのセットアップでも登場しましたが、これは環境モジュールのドロップイン代替品であり、 TclモジュールファイルとLuaモジュールファイルの両方で機能します。
環境モジュールと完全に互換性がありますが、独自の多くの際立った機能も備えています。
主なものは モジュール階層です。
これは、現在ロードされているコンパイラやMPIでビルドされたモジュールのみを表示することにより、モジュールUIを簡素化します。
いくつかのユニークな安全機能もあります。

## Spack はどのようにしてモジュールファイルを生成するのか？

実践的なセクションに入る前に、モジュールファイルがSpackによってどのように生成されるかを解説します。
次の図は、プロセスの概要を示しています。

![Spackがモジュールを生成する流れ](./images/module_spack.png)

Spackのモジュールは、構成ファイル（ `config.yaml` および `modules.yaml` ）、Spackのパッケージレシピからの情報、およびJinja2テンプレートを使用して生成します。
Spackは外部テンプレートエンジン　[Jinja2](http://jinja.pocoo.org/docs/2.9/) から構成されているので、自分でインストールする必要はありません。

### モジュール vs `spack load`

前述の「チュートリアルのセットアップ」セクションで `spack load` 使用していたことに気付いた人もいると思います。
これは Spack の組み込みメカニズムであり、クラスタもしくはラップトップのユーザが簡単にパッケージのパスを通せるように設計されており、Spackの仕様構文を理解します。
モジュールがシステムにセットアップされているかどうかに関係なくSpackが機能する必要があるため、モジュールは必要ありません。

ご想像のとおり、`spack load` や `spack find` を使って何が読み込まれるかを確認することができます：

```bash
$ spack find --loaded
==> 6 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
gcc@8.3.0  gmp@6.1.2  isl@0.18	mpc@1.1.0  mpfr@3.1.6  zlib@1.2.11
```

SpackはHPCシステムで実行するように設計されているため、インストールされているすべてのパッケージのモジュールファイルも生成します。
これにより Spackのインターフェースに慣れていないユーザでも、慣れ親しんだモジュールシステムを通して活用することができます。
次のように、確認してみましょう：

```bash
$ module avail

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   autoconf-2.69-gcc-7.5.0-nwb3bf5	libiconv-1.16-gcc-7.5.0-jearpk4 	       ncurses-6.2-gcc-7.5.0-crhlefo
   automake-1.16.2-gcc-7.5.0-wdflovn	libidn2-2.1.1a-gcc-7.5.0-gcbfo3a	       openssl-1.1.1g-gcc-7.5.0-4dj34yw
   bzip2-1.0.8-gcc-7.5.0-fvfpt26	libsigsegv-2.12-gcc-7.5.0-lbrx7ln	       patchelf-0.10-gcc-7.5.0-2nj3vxx
   curl-7.68.0-gcc-7.5.0-e4rrsab	libtool-2.4.6-gcc-7.5.0-jdxbjft 	       pcre2-10.35-gcc-7.5.0-xyujt3h
   diffutils-3.7-gcc-7.5.0-otkkten	libunistring-0.9.10-gcc-7.5.0-dyq3apq	       perl-5.30.3-gcc-7.5.0-hyrsxn4
   expat-2.2.9-gcc-7.5.0-3f3kkbl	libxml2-2.9.10-gcc-7.5.0-m3l53bh	       pkgconf-1.7.3-gcc-7.5.0-4sh6pym
   gcc-8.3.0-gcc-7.5.0-6hbkzol		lmod-8.3-gcc-7.5.0-kqbszrm		       readline-8.0-gcc-7.5.0-t54jzdy
   gdbm-1.18.1-gcc-7.5.0-4av4gyw	lua-5.3.5-gcc-7.5.0-n37jedg		       tar-1.32-gcc-7.5.0-uwe6tb5
   gettext-0.20.2-gcc-7.5.0-p4xwlyy	lua-luafilesystem-1_7_0_2-gcc-7.5.0-tk7fdik    tcl-8.6.8-gcc-7.5.0-aszidzn
   git-2.27.0-gcc-7.5.0-pzqvpp2 	lua-luaposix-33.4.0-gcc-7.5.0-sas2uzg	       unzip-6.0-gcc-7.5.0-2ozim4d
   gmp-6.1.2-gcc-7.5.0-3ol3tld		m4-1.4.18-gcc-7.5.0-mkc3u4x		       xz-5.2.5-gcc-7.5.0-6nxes4r
   isl-0.18-gcc-7.5.0-4lkvfkv		mpc-1.1.0-gcc-7.5.0-uur2dd6		       zlib-1.2.11-gcc-7.5.0-smoyzzo
   libbsd-0.10.0-gcc-7.5.0-u6ue7vw	mpfr-3.1.6-gcc-7.5.0-nwsubsw

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

これらのどれでも `module load` することができます。
デフォルトでは、Spackは `package-version-compiler-version-hash` 形式で名前が付けられたモジュールを生成しますが、これは少し読みにくいです。
次のセクションでは、これをカスタマイズする方法を説明します。

## 非階層モジュールファイル

ここまでのチュートリアルのとおり実行すると、環境は次のようになっているはずです：

```bash
$ module avail

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   autoconf-2.69-gcc-7.5.0-nwb3bf5	libidn2-2.1.1a-gcc-7.5.0-gcbfo3a	       openssl-1.1.1g-gcc-7.5.0-4dj34yw
   autoconf-2.69-gcc-8.3.0-c7fxiuu	libpciaccess-0.13.5-gcc-8.3.0-rqqclxg	       openssl-1.1.1g-gcc-8.3.0-yygx7ht
   automake-1.16.2-gcc-7.5.0-wdflovn	libsigsegv-2.12-gcc-7.5.0-lbrx7ln	       patchelf-0.10-gcc-7.5.0-2nj3vxx
   automake-1.16.2-gcc-8.3.0-25eawip	libsigsegv-2.12-gcc-8.3.0-oaiujfn	       patchelf-0.10-gcc-8.3.0-35lyjgt
   bzip2-1.0.8-gcc-7.5.0-fvfpt26	libtool-2.4.6-gcc-7.5.0-jdxbjft 	       pcre2-10.35-gcc-7.5.0-xyujt3h
   bzip2-1.0.8-gcc-8.3.0-jx7bite	libtool-2.4.6-gcc-8.3.0-too4ft7 	       perl-5.30.3-gcc-7.5.0-hyrsxn4
   cmake-3.17.3-gcc-8.3.0-7otuah5	libunistring-0.9.10-gcc-7.5.0-dyq3apq	       perl-5.30.3-gcc-8.3.0-xmhlmqr
   curl-7.68.0-gcc-7.5.0-e4rrsab	libxml2-2.9.10-gcc-7.5.0-m3l53bh	       pkgconf-1.7.3-gcc-7.5.0-4sh6pym
   diffutils-3.7-gcc-7.5.0-otkkten	libxml2-2.9.10-gcc-8.3.0-u5khonr	       pkgconf-1.7.3-gcc-8.3.0-oo4jobp
   diffutils-3.7-gcc-8.3.0-5fvw3jh	lmod-8.3-gcc-7.5.0-kqbszrm		       py-cython-0.29.16-gcc-8.3.0-mbnkbhv
   expat-2.2.9-gcc-7.5.0-3f3kkbl	lua-5.3.5-gcc-7.5.0-n37jedg		       py-numpy-1.19.0-gcc-8.3.0-3s3aj6y
   expat-2.2.9-gcc-8.3.0-xrk7vwz	lua-luafilesystem-1_7_0_2-gcc-7.5.0-tk7fdik    py-pybind11-2.5.0-gcc-8.3.0-mjmujjo
   findutils-4.6.0-gcc-8.3.0-gxxeusv	lua-luaposix-33.4.0-gcc-7.5.0-sas2uzg	       py-scipy-1.5.0-gcc-8.3.0-6f5vgp6
   gcc-8.3.0-gcc-7.5.0-6hbkzol		m4-1.4.18-gcc-7.5.0-mkc3u4x		       py-setuptools-46.1.3-gcc-8.3.0-4dcvntv
   gdbm-1.18.1-gcc-7.5.0-4av4gyw	m4-1.4.18-gcc-8.3.0-65odbgi		       python-3.7.7-gcc-8.3.0-4uojybm
   gdbm-1.18.1-gcc-8.3.0-alz6ery	mpc-1.1.0-gcc-7.5.0-uur2dd6		       readline-8.0-gcc-7.5.0-t54jzdy
   gettext-0.20.2-gcc-7.5.0-p4xwlyy	mpfr-3.1.6-gcc-7.5.0-nwsubsw		       readline-8.0-gcc-8.3.0-l33t67c
   gettext-0.20.2-gcc-8.3.0-euuxtvr	mpich-3.3.2-gcc-8.3.0-focol2k		       sqlite-3.31.1-gcc-8.3.0-vz4awtk
   git-2.27.0-gcc-7.5.0-pzqvpp2 	ncurses-6.2-gcc-7.5.0-crhlefo		       tar-1.32-gcc-7.5.0-uwe6tb5
   gmp-6.1.2-gcc-7.5.0-3ol3tld		ncurses-6.2-gcc-8.3.0-klfkayd		       tar-1.32-gcc-8.3.0-fk4qp4i
   hwloc-1.11.11-gcc-8.3.0-lz2i2dd	netlib-lapack-3.8.0-gcc-8.3.0-kbk7dg3	       tcl-8.6.8-gcc-7.5.0-aszidzn
   hwloc-2.2.0-gcc-8.3.0-4otoxex	netlib-scalapack-2.1.0-gcc-8.3.0-h2amar2       texinfo-6.5-gcc-8.3.0-7t62l5t
   isl-0.18-gcc-7.5.0-4lkvfkv		netlib-scalapack-2.1.0-gcc-8.3.0-kt5c5on       unzip-6.0-gcc-7.5.0-2ozim4d
   libbsd-0.10.0-gcc-7.5.0-u6ue7vw	netlib-scalapack-2.1.0-gcc-8.3.0-qsewiuz       util-macros-1.19.1-gcc-8.3.0-obeg5kx
   libbsd-0.10.0-gcc-8.3.0-nr46ju6	netlib-scalapack-2.1.0-gcc-8.3.0-uf4ynvq       xz-5.2.5-gcc-7.5.0-6nxes4r
   libffi-3.3-gcc-8.3.0-lo4lv3d 	numactl-2.0.12-gcc-8.3.0-tusy4nf	       xz-5.2.5-gcc-8.3.0-72wq5od
   libiconv-1.16-gcc-7.5.0-jearpk4	openblas-0.3.10-gcc-8.3.0-x32w5jn	       zlib-1.2.11-gcc-7.5.0-smoyzzo
   libiconv-1.16-gcc-8.3.0-brkkjge	openmpi-3.1.6-gcc-8.3.0-4twpso3 	       zlib-1.2.11-gcc-8.3.0-2icaxiy

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

これまでに生成された非階層モジュールファイルは、モジュール生成に関するSpackのデフォルトルールに従います。
`gcc` モジュールを確認してみましょう。
次に例を示します：

```bash
$ module show gcc-8.3.0-gcc-7.5.0-6hbkzol
----------------------------------------------------------------------------------------------------------------------------------------------
   /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/gcc-8.3.0-gcc-7.5.0-6hbkzol:
----------------------------------------------------------------------------------------------------------------------------------------------
whatis("The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, and Go, as well as libraries for these languages. ")
prepend_path("PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin")
prepend_path("MANPATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/share/man")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib")
prepend_path("LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib64")
prepend_path("LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib64")
prepend_path("C_INCLUDE_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/include")
prepend_path("CPLUS_INCLUDE_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/include")
prepend_path("INCLUDE","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/include")
prepend_path("CMAKE_PREFIX_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/")
setenv("CC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gcc")
setenv("CXX","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/g++")
setenv("FC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
setenv("F77","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
help([[The GNU Compiler Collection includes front ends for C, C++, Objective-C,
Fortran, Ada, and Go, as well as libraries for these languages.
]])
```

予想どおり、パスを表すいくつかの環境変数は、デフォルトのプレフィックス検査ルールに従ってモジュールファイルによって変更されています。

### 環境への不要な変更をフィルタリングする

ここで、サイトが `C_INCLUDE_PATH` 、 `CPLUS_INCLUDE_PATH` 、 `LIBRARY_PATH` の変更をモジュールファイルに含めるべきではないと決定した場合を考えてみましょう。
このルールを順守するために、次の内容の構成ファイル `${SPACK_ROOT}/etc/spack/modules.yaml` を作成することです：

```yaml
modules:
  tcl:
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
```

次に、すべてのモジュールファイルを再生成する必要があります：

```bash
$ spack module tcl refresh
y

==> You are about to regenerate tcl module files for:

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
nwb3bf5 autoconf@2.69
wdflovn automake@1.16.2
fvfpt26 bzip2@1.0.8
e4rrsab curl@7.68.0
otkkten diffutils@3.7
3f3kkbl expat@2.2.9
6hbkzol gcc@8.3.0
4av4gyw gdbm@1.18.1
p4xwlyy gettext@0.20.2
pzqvpp2 git@2.27.0
3ol3tld gmp@6.1.2
4lkvfkv isl@0.18
u6ue7vw libbsd@0.10.0
jearpk4 libiconv@1.16
gcbfo3a libidn2@2.1.1a
lbrx7ln libsigsegv@2.12
jdxbjft libtool@2.4.6
dyq3apq libunistring@0.9.10
m3l53bh libxml2@2.9.10
kqbszrm lmod@8.3
n37jedg lua@5.3.5
tk7fdik lua-luafilesystem@1_7_0_2
sas2uzg lua-luaposix@33.4.0
mkc3u4x m4@1.4.18
uur2dd6 mpc@1.1.0
nwsubsw mpfr@3.1.6
crhlefo ncurses@6.2
4dj34yw openssl@1.1.1g
2nj3vxx patchelf@0.10
xyujt3h pcre2@10.35
hyrsxn4 perl@5.30.3
4sh6pym pkgconf@1.7.3
t54jzdy readline@8.0
uwe6tb5 tar@1.32
aszidzn tcl@8.6.8
2ozim4d unzip@6.0
6nxes4r xz@5.2.5
smoyzzo zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@8.3.0 -------------------------
c7fxiuu autoconf@2.69
25eawip automake@1.16.2
jx7bite bzip2@1.0.8
7otuah5 cmake@3.17.3
5fvw3jh diffutils@3.7
xrk7vwz expat@2.2.9
gxxeusv findutils@4.6.0
alz6ery gdbm@1.18.1
euuxtvr gettext@0.20.2
lz2i2dd hwloc@1.11.11
4otoxex hwloc@2.2.0
nr46ju6 libbsd@0.10.0
lo4lv3d libffi@3.3
brkkjge libiconv@1.16
rqqclxg libpciaccess@0.13.5
oaiujfn libsigsegv@2.12
too4ft7 libtool@2.4.6
u5khonr libxml2@2.9.10
65odbgi m4@1.4.18
focol2k mpich@3.3.2
klfkayd ncurses@6.2
kbk7dg3 netlib-lapack@3.8.0
h2amar2 netlib-scalapack@2.1.0
uf4ynvq netlib-scalapack@2.1.0
qsewiuz netlib-scalapack@2.1.0
kt5c5on netlib-scalapack@2.1.0
tusy4nf numactl@2.0.12
x32w5jn openblas@0.3.10
4twpso3 openmpi@3.1.6
yygx7ht openssl@1.1.1g
35lyjgt patchelf@0.10
xmhlmqr perl@5.30.3
oo4jobp pkgconf@1.7.3
mbnkbhv py-cython@0.29.16
3s3aj6y py-numpy@1.19.0
mjmujjo py-pybind11@2.5.0
6f5vgp6 py-scipy@1.5.0
4dcvntv py-setuptools@46.1.3
4uojybm python@3.7.7
l33t67c readline@8.0
vz4awtk sqlite@3.31.1
fk4qp4i tar@1.32
7t62l5t texinfo@6.5
obeg5kx util-macros@1.19.1
72wq5od xz@5.2.5
2icaxiy zlib@1.2.11

==> Do you want to proceed? [y/n] ==> Regenerating tcl module files
```

`gcc` 用のモジュール情報を参照すると、不要なパスが消えていることが確認できます：

```bash
$ module show gcc-8.3.0-gcc-7.5.0-6hbkzol
----------------------------------------------------------------------------------------------------------------------------------------------
   /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/gcc-8.3.0-gcc-7.5.0-6hbkzol:
----------------------------------------------------------------------------------------------------------------------------------------------
whatis("The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, and Go, as well as libraries for these languages. ")
prepend_path("PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin")
prepend_path("MANPATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/share/man")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib64")
prepend_path("INCLUDE","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/include")
prepend_path("CMAKE_PREFIX_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/")
setenv("CC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gcc")
setenv("CXX","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/g++")
setenv("FC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
setenv("F77","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
help([[The GNU Compiler Collection includes front ends for C, C++, Objective-C,
Fortran, Ada, and Go, as well as libraries for these languages.
]])
```

### 一部のモジュールファイル作成を抑制する

多くのサイトでよくあるもう1つのリクエストとして、新しいスタックを構築する際の中間ステップとしてのみ必要なソフトウェアの公開を避けることがあります。
`gcc@7.5.0` （OSが提供するコンパイラ）でコンパイルされたモジュールファイルが生成されないようにしてみましょう。

実行するには、`${SPACK_ROOT}/etc/spack/modules.yaml` に `blacklist` キーワードを追加する必要があります：

```yaml
modules:
  tcl:
    blacklist:
      -  '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
```

モジュールファイルを再生成します。
今回は、Spackが既存のディレクトリ内のファイルを上書きするのではなく、既存のモジュールツリーを削除し新しいモジュールツリーを再生成するように `--delete-tree` オプションを使います：

```bash
$ spack module tcl refresh --delete-tree -y
==> Regenerating tcl module files
```

```bash
$ module avail

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   autoconf-2.69-gcc-8.3.0-c7fxiuu	    libtool-2.4.6-gcc-8.3.0-too4ft7		pkgconf-1.7.3-gcc-8.3.0-oo4jobp
   automake-1.16.2-gcc-8.3.0-25eawip	    libxml2-2.9.10-gcc-8.3.0-u5khonr		py-cython-0.29.16-gcc-8.3.0-mbnkbhv
   bzip2-1.0.8-gcc-8.3.0-jx7bite	    m4-1.4.18-gcc-8.3.0-65odbgi 		py-numpy-1.19.0-gcc-8.3.0-3s3aj6y
   cmake-3.17.3-gcc-8.3.0-7otuah5	    mpich-3.3.2-gcc-8.3.0-focol2k		py-pybind11-2.5.0-gcc-8.3.0-mjmujjo
   diffutils-3.7-gcc-8.3.0-5fvw3jh	    ncurses-6.2-gcc-8.3.0-klfkayd		py-scipy-1.5.0-gcc-8.3.0-6f5vgp6
   expat-2.2.9-gcc-8.3.0-xrk7vwz	    netlib-lapack-3.8.0-gcc-8.3.0-kbk7dg3	py-setuptools-46.1.3-gcc-8.3.0-4dcvntv
   findutils-4.6.0-gcc-8.3.0-gxxeusv	    netlib-scalapack-2.1.0-gcc-8.3.0-h2amar2	python-3.7.7-gcc-8.3.0-4uojybm
   gdbm-1.18.1-gcc-8.3.0-alz6ery	    netlib-scalapack-2.1.0-gcc-8.3.0-kt5c5on	readline-8.0-gcc-8.3.0-l33t67c
   gettext-0.20.2-gcc-8.3.0-euuxtvr	    netlib-scalapack-2.1.0-gcc-8.3.0-qsewiuz	sqlite-3.31.1-gcc-8.3.0-vz4awtk
   hwloc-1.11.11-gcc-8.3.0-lz2i2dd	    netlib-scalapack-2.1.0-gcc-8.3.0-uf4ynvq	tar-1.32-gcc-8.3.0-fk4qp4i
   hwloc-2.2.0-gcc-8.3.0-4otoxex	    numactl-2.0.12-gcc-8.3.0-tusy4nf		texinfo-6.5-gcc-8.3.0-7t62l5t
   libbsd-0.10.0-gcc-8.3.0-nr46ju6	    openblas-0.3.10-gcc-8.3.0-x32w5jn		util-macros-1.19.1-gcc-8.3.0-obeg5kx
   libffi-3.3-gcc-8.3.0-lo4lv3d 	    openmpi-3.1.6-gcc-8.3.0-4twpso3		xz-5.2.5-gcc-8.3.0-72wq5od
   libiconv-1.16-gcc-8.3.0-brkkjge	    openssl-1.1.1g-gcc-8.3.0-yygx7ht		zlib-1.2.11-gcc-8.3.0-2icaxiy
   libpciaccess-0.13.5-gcc-8.3.0-rqqclxg    patchelf-0.10-gcc-8.3.0-35lyjgt
   libsigsegv-2.12-gcc-8.3.0-oaiujfn	    perl-5.30.3-gcc-8.3.0-xmhlmqr

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

よく見ると、モジュールのブラックリストへの登録が行き過ぎていることがわかります。
`gcc@8.3.0` モジュールは、`gcc@7.5.0` で構築されていたために消えてしまいました。
ブラックリストルールの例外を指定するには、 `whitelist` を使用します：

```yaml
modules:
  tcl:
    whitelist:
      -  gcc
    blacklist:
      -  '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
```

`whitelist` ルールは常に `blacklist` ルールよりも優先されます。
モジュールを再生成するには、次のコマンドを実行します：

```bash
$ spack module tcl refresh -y
==> Regenerating tcl module files
```

`gcc@8.3.0` モジュールが再表示されたことがわかります。

```bash
$ module avail gcc-8.3.0-gcc-7.5.0-6hbkzol

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   gcc-8.3.0-gcc-7.5.0-6hbkzol

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

環境の整理に利用可能な追加機能は、暗黙的にインストールされたパッケージのモジュールファイル生成をスキップすることです。

次の行を `modules.yaml` に追加するだけで済みます：

```yaml
modules:
  tcl:
    blacklist_implicits: true
    whitelist:
      -  gcc
    blacklist:
      -  '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
```

そして、モジュールのファイルツリーを再生成します。

### モジュールファイル名を変更する

モジュールファイルをよりユーザフレンドリにするための次のステップは、モジュールファイルの命名スキームを改善することです。
ハッシュの長さを短くするか、完全に削除するには、構成ファイルで `hash_length` キーワードを使用します：

```yaml
modules:
  tcl:
    hash_length: 0
    whitelist:
      -  gcc
    blacklist:
      -  '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
```

ここでモジュールファイルを再生成しようとすると、エラーが発生します。

```bash
$ spack module tcl refresh --delete-tree -y
==> Error: Name clashes detected in module files:

file: /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/netlib-scalapack-2.1.0-gcc-8.3.0
spec: netlib-scalapack@2.1.0%gcc@8.3.0~pic+shared build_type=RelWithDebInfo patches=f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
spec: netlib-scalapack@2.1.0%gcc@8.3.0~pic+shared build_type=RelWithDebInfo patches=f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
spec: netlib-scalapack@2.1.0%gcc@8.3.0~pic+shared build_type=RelWithDebInfo patches=f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
spec: netlib-scalapack@2.1.0%gcc@8.3.0~pic+shared build_type=RelWithDebInfo patches=f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64

==> Error: Operation aborted
```

> **注意**
> 事前にエラーをチェックしようとしています！
>
> Spackでは、可能な限り事前にエラーをチェックするため、モジュールファイルについて心配する必要はありません。
> 名前の衝突が検出されたため、ディスク上で何も変更されていません。

ここでの問題は、ハッシュなしで4つの異なる `netlib-scalapack` フレーバが同一のモジュールファイル名にマップされることです。
名前を区別するために、名前のフォーマット方法を変更できます：

```yaml
modules:
  tcl:
    hash_length: 0
    whitelist:
      -  gcc
    blacklist:
      -  '%gcc@7.5.0'
    all:
      conflict:
        - '{name}'
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
    projections:
      all:               '{name}/{version}-{compiler.name}-{compiler.version}'
      netlib-scalapack:  '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}-{^mpi.name}'
      ^python^lapack:    '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}'
```

ご覧のとおり、`^mpi^lapack` のような [匿名仕様 anonymous specs](https://spack.readthedocs.io/en/latest/module_file_support.html#anonymous-specs) を使い、制限されたパッケージのセットにのみ適用されるルールを指定することができます。
ここでは、同じ名前の2つのモジュール間の競合を宣言しているため、一緒にロードすることはできません。
また、[spec format 構文](https://spack.readthedocs.io/en/latest/spack.html?highlight=spec%20format#spack.spec.Spec.format) を使用して、コンパイラ、コンパイラバージョン、およびMPIバージョンに従ってモジュールの名前をフォーマットします。
これにより依存関係によって仕様を照合し、DAGに基づいてフォーマットすることができます。

```bash
$ spack module tcl refresh --delete-tree -y
==> Regenerating tcl module files
```

```yaml
$ module avail

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   autoconf/2.69-gcc-8.3.0		libsigsegv/2.12-gcc-8.3.0				      perl/5.30.3-gcc-8.3.0
   automake/1.16.2-gcc-8.3.0		libtool/2.4.6-gcc-8.3.0 				      pkgconf/1.7.3-gcc-8.3.0
   bzip2/1.0.8-gcc-8.3.0		libxml2/2.9.10-gcc-8.3.0				      py-cython/0.29.16-gcc-8.3.0
   cmake/3.17.3-gcc-8.3.0		m4/1.4.18-gcc-8.3.0					      py-numpy/1.19.0-gcc-8.3.0-openblas
   diffutils/3.7-gcc-8.3.0		mpich/3.3.2-gcc-8.3.0					      py-pybind11/2.5.0-gcc-8.3.0
   expat/2.2.9-gcc-8.3.0		ncurses/6.2-gcc-8.3.0					      py-scipy/1.5.0-gcc-8.3.0-openblas
   findutils/4.6.0-gcc-8.3.0		netlib-lapack/3.8.0-gcc-8.3.0				      py-setuptools/46.1.3-gcc-8.3.0
   gcc/8.3.0-gcc-7.5.0			netlib-scalapack/2.1.0-gcc-8.3.0-netlib-lapack-mpich	      python/3.7.7-gcc-8.3.0
   gdbm/1.18.1-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-netlib-lapack-openmpi	      readline/8.0-gcc-8.3.0
   gettext/0.20.2-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-openblas-mpich 	      sqlite/3.31.1-gcc-8.3.0
   hwloc/1.11.11-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-openblas-openmpi      (D)    tar/1.32-gcc-8.3.0
   hwloc/2.2.0-gcc-8.3.0	 (D)	numactl/2.0.12-gcc-8.3.0				      texinfo/6.5-gcc-8.3.0
   libbsd/0.10.0-gcc-8.3.0		openblas/0.3.10-gcc-8.3.0				      util-macros/1.19.1-gcc-8.3.0
   libffi/3.3-gcc-8.3.0 		openmpi/3.1.6-gcc-8.3.0 				      xz/5.2.5-gcc-8.3.0
   libiconv/1.16-gcc-8.3.0		openssl/1.1.1g-gcc-8.3.0				      zlib/1.2.11-gcc-8.3.0
   libpciaccess/0.13.5-gcc-8.3.0	patchelf/0.10-gcc-8.3.0

  Where:
   D:  Default Module

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

> **注意**
> `conflict` ディレクティブはTcl固有であり、設定ファイルの `lmod` セクション中には使用できません。

### カスタム環境の変更を追加する

多くのサイトでは、パッケージがインストールされているフォルダをさす環境変数をパッケージのモジュールファイルに設定するのが通例です。
構成ファイルに `environment` ディレクティブを追加することにより、実現が可能です：

```yaml
modules:
  tcl:
    hash_length: 0
    naming_scheme: '{name}/{version}-{compiler.name}-{compiler.version}'
    whitelist:
      -  gcc
    blacklist:
      -  '%gcc@7.5.0'
    all:
      conflict:
        - '{name}'
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    projections:
      all:               '{name}/{version}-{compiler.name}-{compiler.version}'
      netlib-scalapack:  '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}-{^mpi.name}'
      ^python^lapack:    '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}'
```

内部的には、Spackは `format()` APIを使い、環境変数名または値のいずれかのトークンを置換します。
ただし、2つの注意点があります。

- 変数名で許可されたトークンのセットが `name` 、 `version` 、 `compiler` 、 `compiler.name` 、 `compiler.version` 、 `architecture` に制限されている
- 変数名で展開されたトークンはすべて大文字になる（大文字と小文字は区別される）

モジュールファイルを再生成すると、次のようになります：

```bash
$ spack module tcl refresh -y
==> Regenerating tcl module files
```

```bash
$ module show gcc
----------------------------------------------------------------------------------------------------------------------------------------------
   /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/gcc/8.3.0-gcc-7.5.0:
----------------------------------------------------------------------------------------------------------------------------------------------
whatis("The GNU Compiler Collection includes front ends for C, C++, Objective-C, Fortran, Ada, and Go, as well as libraries for these languages. ")
conflict("gcc")
prepend_path("PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin")
prepend_path("MANPATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/share/man")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/lib64")
prepend_path("INCLUDE","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/include")
prepend_path("CMAKE_PREFIX_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/")
setenv("CC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gcc")
setenv("CXX","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/g++")
setenv("FC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
setenv("F77","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl/bin/gfortran")
setenv("'GCC_ROOT'","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-6hbkzolzshktgws6mz3f4s23v6sbkgnl")
help([[The GNU Compiler Collection includes front ends for C, C++, Objective-C,
Fortran, Ada, and Go, as well as libraries for these languages.
]])
```

ご覧のとおり、gccモジュールには環境変数 `GCC_ROOT` が設定されています。

環境の変更を選択的に適用し、特定のパッケージのみを対象とすることが役立つ場合もあります。
たとえば、次のように `openmpi` モジュールに変更を適用できます：

```yaml
modules:
  tcl:
    hash_length: 0
    naming_scheme: '{name}/{version}-{compiler.name}-{compiler.version}'
    whitelist:
      - gcc
    blacklist:
      - '%gcc@7.5.0'
    all:
      conflict:
        - '{name}'
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    projections:
      all:               '{name}/{version}-{compiler.name}-{compiler.version}'
      netlib-scalapack:  '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}-{^mpi.name}'
      ^python^lapack:    '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}'
```

今回はより選択的になり、`openmpi` モジュールファイルのみを再生成します：

```bash
$ spack module tcl refresh -y openmpi
==> Regenerating tcl module files
```

```bash
$ module show openmpi
----------------------------------------------------------------------------------------------------------------------------------------------
   /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/openmpi/3.1.6-gcc-8.3.0:
----------------------------------------------------------------------------------------------------------------------------------------------
whatis("An open source Message Passing Interface implementation. ")
conflict("openmpi")
prepend_path("PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/bin")
prepend_path("MANPATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/share/man")
prepend_path("LD_LIBRARY_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/lib")
prepend_path("INCLUDE","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/include")
prepend_path("PKG_CONFIG_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/lib/pkgconfig")
prepend_path("CMAKE_PREFIX_PATH","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/")
setenv("MPICC","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/bin/mpicc")
setenv("MPICXX","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/bin/mpic++")
setenv("MPIF77","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/bin/mpif77")
setenv("MPIF90","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky/bin/mpif90")
setenv("'OPENMPI_ROOT'","/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/openmpi-3.1.6-4twpso3kn2f3z6oa527snqvn7fsridky")
setenv("SLURM_MPI_TYPE","pmi2")
setenv("OMPI_MCA_btl_openib_warn_default_git_prefix","0")
help([[An open source Message Passing Interface implementation. The Open MPI
Project is an open source Message Passing Interface implementation that
is developed and maintained by a consortium of academic, research, and
industry partners. Open MPI is therefore able to combine the expertise,
technologies, and resources from all across the High Performance
Computing community in order to build the best MPI library available.
Open MPI offers advantages for system and software vendors, application
developers and computer science researchers.
]])
```

### 依存関係を自動ロードする

Spackは、依存関係を自動ロードするコードを含むモジュールファイルを生成することも可能です。
たとえば、`autoload` ディレクティブを追加して `direct` 値を割り当てることにより、依存関係をロードするPythonモジュールを生成できます：

```yaml
modules:
  tcl:
    verbose: True
    hash_length: 0
    naming_scheme: '{name}/{version}-{compiler.name}-{compiler.version}'
    whitelist:
      - gcc
    blacklist:
      - '%gcc@7.5.0'
    all:
      conflict:
        - '{name}'
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    gcc:
      environment:
        set:
          CC: gcc
          CXX: g++
          FC: gfortran
          F90: gfortran
          F77: gfortran
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    projections:
      all:               '{name}/{version}-{compiler.name}-{compiler.version}'
      netlib-scalapack:  '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}-{^mpi.name}'
      ^python^lapack:    '{name}/{version}-{compiler.name}-{compiler.version}-{^lapack.name}'
    ^python:
      autoload:  direct
```

`python` に依存するすべてのパッケージのモジュールファイルを再生成します：

```bash
$ spack module tcl refresh -y ^python
==> Regenerating tcl module files
```

そして、すべての依存関係を自動ロードするコードが含まれます。

```bash
$ module load py-scipy
Autoloading openblas/0.3.10-gcc-8.3.0
Autoloading py-numpy/1.19.0-gcc-8.3.0-openblas
Autoloading python/3.7.7-gcc-8.3.0
Autoloading python/3.7.7-gcc-8.3.0
```

自動ロード手順中にメッセージが不要な場合は、上記の構成ファイルの `verbose: True` 行設定を省略すれば十分です。

## 階層モジュールファイル

これまでのチュートリアルでは、非階層モジュールファイル、つまりすべて同じルートディレクトリで生成され、動的に `MODULEPATH` の変更を試みないモジュールファイルを使用してきました。
ここからは、すべてのソフトウェアが同時に表示されるフラットなモジュール構造になります：

```bash
$ module avail

----------------------------------------- /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64 -----------------------------------------
   autoconf/2.69-gcc-8.3.0		libsigsegv/2.12-gcc-8.3.0				      perl/5.30.3-gcc-8.3.0
   automake/1.16.2-gcc-8.3.0		libtool/2.4.6-gcc-8.3.0 				      pkgconf/1.7.3-gcc-8.3.0
   bzip2/1.0.8-gcc-8.3.0		libxml2/2.9.10-gcc-8.3.0				      py-cython/0.29.16-gcc-8.3.0
   cmake/3.17.3-gcc-8.3.0		m4/1.4.18-gcc-8.3.0					      py-numpy/1.19.0-gcc-8.3.0-openblas
   diffutils/3.7-gcc-8.3.0		mpich/3.3.2-gcc-8.3.0					      py-pybind11/2.5.0-gcc-8.3.0
   expat/2.2.9-gcc-8.3.0		ncurses/6.2-gcc-8.3.0					      py-scipy/1.5.0-gcc-8.3.0-openblas
   findutils/4.6.0-gcc-8.3.0		netlib-lapack/3.8.0-gcc-8.3.0				      py-setuptools/46.1.3-gcc-8.3.0
   gcc/8.3.0-gcc-7.5.0			netlib-scalapack/2.1.0-gcc-8.3.0-netlib-lapack-mpich	      python/3.7.7-gcc-8.3.0
   gdbm/1.18.1-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-netlib-lapack-openmpi	      readline/8.0-gcc-8.3.0
   gettext/0.20.2-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-openblas-mpich 	      sqlite/3.31.1-gcc-8.3.0
   hwloc/1.11.11-gcc-8.3.0		netlib-scalapack/2.1.0-gcc-8.3.0-openblas-openmpi      (D)    tar/1.32-gcc-8.3.0
   hwloc/2.2.0-gcc-8.3.0	 (D)	numactl/2.0.12-gcc-8.3.0				      texinfo/6.5-gcc-8.3.0
   libbsd/0.10.0-gcc-8.3.0		openblas/0.3.10-gcc-8.3.0				      util-macros/1.19.1-gcc-8.3.0
   libffi/3.3-gcc-8.3.0 		openmpi/3.1.6-gcc-8.3.0 				      xz/5.2.5-gcc-8.3.0
   libiconv/1.16-gcc-8.3.0		openssl/1.1.1g-gcc-8.3.0				      zlib/1.2.11-gcc-8.3.0
   libpciaccess/0.13.5-gcc-8.3.0	patchelf/0.10-gcc-8.3.0

  Where:
   D:  Default Module

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

このレイアウトはデプロイが非常に簡単ですが、上記のスニペットから、ユーザが互換性のないモジュールセットをロードすることを妨げるものは何もないことがわかります。

```bash
$ module purge
$ module load netlib-lapack openblas
$ module list

Currently Loaded Modules:
  1) netlib-lapack/3.8.0-gcc-8.3.0   2) openblas/0.3.10-gcc-8.3.0
```

`conflicts` ディレクティブがモジュールファイルに注意深く配置されている場合でも：

- 一貫した環境を強制せず、エラーだけ報告します。
- たとえば、新しいコンパイラまたはMPIライブラリがインストールされるとすぐに、定期的に更新する必要があります。

階層モジュールファイルは、起動時にシステムで利用できるものの制限されたビューのみを表示することにより、これらの欠点を克服しようとします。
より具体的には、OSが提供するコンパイラとともにインストールされたソフトウェアのみを表示します。
このソフトウェアの中には、他の（通常はより最近の）コンパイラがあり、ロードされると、それらを使用してコンパイルされたすべてのソフトウェアのロックを解除するために新しいディレクトリが `MODULEPATH` に追加されます。
この「ロック解除」のアイデアは、仮想依存関係に任意に拡張できます（次のセクションで解説します）。

### Core/Compiler/MPI

最も広く使用されている階層は、コンパイラに加えて、さまざまなMPIライブラリがそれらにリンクされているソフトウェアのロックを解除する、`Core/Compiler/MPI` と呼ばれる階層です。
以前に使用した `modules.yaml` ファイルを適応させるために必要な手順はいくつかあります：

1. `lmod` ファイルジェネレータを有効にします
2. `tcl` タグを `lmod` に変更します
3. `tcl` に特化した `conflict` ディレクティブを削除します
4. どのコンパイラが使用されるかを `core_compilers` に宣言します 
5. `mpi` プロジェクション内の関連するサフィックスを削除します（階層に置き換えられるため）

これらの変更後、構成ファイルは次のようになります：

```yaml
modules:
  enable::
    - lmod
  lmod:
    core_compilers:
      - 'gcc@7.5.0'
    hierarchy:
      - mpi
    hash_length: 0
    whitelist:
      - gcc
    blacklist:
      - '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    projections:
      all:          '{name}/{version}'
      ^lapack:      '{name}/{version}-{^lapack.name}'
```

> **注意**
> 構成ファイルの二重コロン
>
> `enable` 直後の二重コロン（`:`）は意図的なもので、有効なジェネレータのデフォルトリストをオーバーライドして、`lmod` のみアクティブにする目的で使用されます（詳細は、[全体のオーバーライドセクション](https://spack.readthedocs.io/en/latest/configuration.html#config-overrides) を参照）。

`core_compilers` ディレクティブには、コンパイラのリストを記述します。
これらのコンパイラを使用して構築されたものはすべて階層の `Core` 部にモジュールを作成します。
これは、階層モジュールファイルのエントリポイントです。
OSが提供するコンパイラをリストに入れ、それらを使用して一般的なユーティリティと他のコンパイラのみをビルドするのが一般的な方法です。

モジュールファイルを再生成する場合は、次のように実行し：

```bash
$ spack module lmod refresh --delete-tree -y
==> Regenerating lmod module files
```

`Core`をさすように`MODULEPATH` を更新します：

```bash
$ module purge
$ module unuse $HOME/spack/share/spack/modules/linux-ubuntu18.04-x86_64
$ module use $HOME/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core
```

利用可能なモジュールを要求すると、次のようになります：

```bash
$ module avail

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

当然のことながら、表示されるモジュールは `gcc` だけです。
階層の `Compiler` 部のロック解除をロードします：

```bash
$ module load gcc
$ module avail

------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/gcc/8.3.0 --------------------------------------
   autoconf/2.69      gdbm/1.18.1	    libpciaccess/0.13.5    netlib-lapack/3.8.0	  pkgconf/1.7.3 	      readline/8.0
   automake/1.16.2    gettext/0.20.2	    libsigsegv/2.12	   numactl/2.0.12	  py-cython/0.29.16	      sqlite/3.31.1
   bzip2/1.0.8	      hwloc/1.11.11	    libtool/2.4.6	   openblas/0.3.10	  py-numpy/1.19.0-openblas    tar/1.32
   cmake/3.17.3       hwloc/2.2.0    (D)    libxml2/2.9.10	   openmpi/3.1.6	  py-pybind11/2.5.0	      texinfo/6.5
   diffutils/3.7      libbsd/0.10.0	    m4/1.4.18		   openssl/1.1.1g	  py-scipy/1.5.0-openblas     util-macros/1.19.1
   expat/2.2.9	      libffi/3.3	    mpich/3.3.2 	   patchelf/0.10	  py-setuptools/46.1.3	      xz/5.2.5
   findutils/4.6.0    libiconv/1.16	    ncurses/6.2 	   perl/5.30.3		  python/3.7.7		      zlib/1.2.11

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0 (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

同じことが`MPI`部にも当てはまり、`mpich` または `openmpi` のいずれかをロードすることで有効にできます。
`mpich` のロードから始めましょう：

```bash
$ module load mpich
$ module avail

--------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/mpich/3.3.2-focol2k/gcc/8.3.0 ----------------------------
   netlib-scalapack/2.1.0-netlib-lapack    netlib-scalapack/2.1.0-openblas (D)

------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/gcc/8.3.0 --------------------------------------
   autoconf/2.69      gdbm/1.18.1	    libpciaccess/0.13.5        netlib-lapack/3.8.0    pkgconf/1.7.3		  readline/8.0
   automake/1.16.2    gettext/0.20.2	    libsigsegv/2.12	       numactl/2.0.12	      py-cython/0.29.16 	  sqlite/3.31.1
   bzip2/1.0.8	      hwloc/1.11.11	    libtool/2.4.6	       openblas/0.3.10	      py-numpy/1.19.0-openblas	  tar/1.32
   cmake/3.17.3       hwloc/2.2.0    (D)    libxml2/2.9.10	       openmpi/3.1.6	      py-pybind11/2.5.0 	  texinfo/6.5
   diffutils/3.7      libbsd/0.10.0	    m4/1.4.18		       openssl/1.1.1g	      py-scipy/1.5.0-openblas	  util-macros/1.19.1
   expat/2.2.9	      libffi/3.3	    mpich/3.3.2 	(L)    patchelf/0.10	      py-setuptools/46.1.3	  xz/5.2.5
   findutils/4.6.0    libiconv/1.16	    ncurses/6.2 	       perl/5.30.3	      python/3.7.7		  zlib/1.2.11

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0 (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

```bash
$ module load openblas netlib-scalapack/2.1.0-openblas
$ module list

Currently Loaded Modules:
  1) gcc/8.3.0	 2) mpich/3.3.2   3) openblas/0.3.10   4) netlib-scalapack/2.1.0-openblas
```

この時点で、階層レイアウトが非階層レイアウトよりも改善された一貫性を示すことができます：

```bash
$ export LMOD_AUTO_SWAP=yes
$ module load openmpi

Lmod is automatically replacing "mpich/3.3.2" with "openmpi/3.1.6".


Due to MODULEPATH changes, the following have been reloaded:
  1) netlib-scalapack/2.1.0-openblas
```

`Lmod` がMPIプロバイダの交換を処理し、MPIの変更に準拠するように `netlib-scalapack` モジュールを置き換えました。
このように、誤って2つの異なるMPIプロバイダを同時にプルインしたり、`mpich` もロードされている場合に `openmpi` にリンクされているパッケージのモジュールファイルをロードしたりすることはできません。
コンパイラとMPIの一貫性は、ツールによって保証されます。

### LAPACKを階層に追加する

ここに示した階層は、非階層レイアウトに比べてすでに大幅に改善されていますが、それでも非対称性があります。
`LAPACK` プロバイダは `MPI` プロバイダと同じセマンティックロールをカバーしますが、階層の一部ではありません。

より実用的には、これは `MPI` 環境の一貫性が向上したものの、`LAPACK` 実装に関して以前と同じ問題が発生していることを意味します：

```bash
$ module list

Currently Loaded Modules:
  1) gcc/8.3.0	 2) openblas/0.3.10   3) openmpi/3.1.6	 4) netlib-scalapack/2.1.0-openblas



$ module avail

-------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/openmpi/3.1.6-4twpso3/gcc/8.3.0 ---------------------------
   netlib-scalapack/2.1.0-netlib-lapack    netlib-scalapack/2.1.0-openblas (L,D)

------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/gcc/8.3.0 --------------------------------------
   autoconf/2.69      gdbm/1.18.1	    libpciaccess/0.13.5    netlib-lapack/3.8.0	      pkgconf/1.7.3		  readline/8.0
   automake/1.16.2    gettext/0.20.2	    libsigsegv/2.12	   numactl/2.0.12	      py-cython/0.29.16 	  sqlite/3.31.1
   bzip2/1.0.8	      hwloc/1.11.11	    libtool/2.4.6	   openblas/0.3.10     (L)    py-numpy/1.19.0-openblas	  tar/1.32
   cmake/3.17.3       hwloc/2.2.0    (D)    libxml2/2.9.10	   openmpi/3.1.6       (L)    py-pybind11/2.5.0 	  texinfo/6.5
   diffutils/3.7      libbsd/0.10.0	    m4/1.4.18		   openssl/1.1.1g	      py-scipy/1.5.0-openblas	  util-macros/1.19.1
   expat/2.2.9	      libffi/3.3	    mpich/3.3.2 	   patchelf/0.10	      py-setuptools/46.1.3	  xz/5.2.5
   findutils/4.6.0    libiconv/1.16	    ncurses/6.2 	   perl/5.30.3		      python/3.7.7		  zlib/1.2.11

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0 (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".


$ module load netlib-scalapack/2.1.0-netlib-lapack

The following have been reloaded with a version change:
  1) netlib-scalapack/2.1.0-openblas => netlib-scalapack/2.1.0-netlib-lapack

$ module list

Currently Loaded Modules:
  1) gcc/8.3.0	 2) openblas/0.3.10   3) openmpi/3.1.6	 4) netlib-scalapack/2.1.0-netlib-lapack
```

モジュールファイルが手動で書き込まれ、複数のプロバイダ間の組み合わせを追跡することがすぐに非常に複雑になるため、`Core` / `Compiler` / `MPI` よりも深い階層は、多くのサイトで「異常」または「非実用的」と見なされる可能性があります。

例えば、階層内に `MPI` と `LAPACK` の両方を持つことは、つぎの4つのカテゴリの1つにソフトウェアを分類して、

1. `MPI` または `LAPACK` に依存しないソフトウェア
2. `MPI` のみに依存するソフトウェア
3. `LAPACK` のみに依存するソフトウェア
4. 両方に依存するソフトウェア

いつユーザに表示するかを決定する必要があることを意味します。
階層内の仮想依存関係の数が増えると、状況はさらに複雑になります。

Spackがインストールされたソフトウェア用に維持しているDAGを利用して、この組み合わせ問題をクリーンで自動化された方法で解決できます。
ある意味で、この組み合わせの複雑さを管理するSpackの機能により、より深い階層が実現可能になります。

例に戻って、階層に `lapack` を追加し、`lapack`の残りの接尾辞の射影を削除しましょう：

```yaml
modules:
  enable::
    - lmod
  lmod:
    core_compilers:
      - 'gcc@7.5.0'
    hierarchy:
      - mpi
      - lapack
    hash_length: 0
    whitelist:
      - gcc
    blacklist:
      - '%gcc@7.5.0'
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    projections:
      all:          '{name}/{version}'
```

モジュールファイルが通常どおり再生成された後に：

```bash
$ module purge
$ spack module lmod refresh --delete-tree -y
==> Regenerating lmod module files
```

これで、階層に追加のコンポーネントがあることがわかります。

```bash
$ module load gcc

Inactive Modules:
  1) netlib-scalapack/2.1.0-netlib-lapack

$ module load openblas
$ module avail

-------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/openmpi/3.1.6-4twpso3/openblas/0.3.10-x32w5jn/gcc/8.3.0 ---------------
   netlib-scalapack/2.1.0

------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/openblas/0.3.10-x32w5jn/gcc/8.3.0 --------------------------
   py-numpy/1.19.0    py-scipy/1.5.0

------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/gcc/8.3.0 --------------------------------------
   autoconf/2.69      gdbm/1.18.1	    libpciaccess/0.13.5    netlib-lapack/3.8.0	      pkgconf/1.7.3	      tar/1.32
   automake/1.16.2    gettext/0.20.2	    libsigsegv/2.12	   numactl/2.0.12	      py-cython/0.29.16       texinfo/6.5
   bzip2/1.0.8	      hwloc/1.11.11	    libtool/2.4.6	   openblas/0.3.10     (L)    py-pybind11/2.5.0       util-macros/1.19.1
   cmake/3.17.3       hwloc/2.2.0    (D)    libxml2/2.9.10	   openmpi/3.1.6       (L)    py-setuptools/46.1.3    xz/5.2.5
   diffutils/3.7      libbsd/0.10.0	    m4/1.4.18		   openssl/1.1.1g	      python/3.7.7	      zlib/1.2.11
   expat/2.2.9	      libffi/3.3	    mpich/3.3.2 	   patchelf/0.10	      readline/8.0
   findutils/4.6.0    libiconv/1.16	    ncurses/6.2 	   perl/5.30.3		      sqlite/3.31.1

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0 (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".


$ module load openmpi
$ module avail

-------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/openmpi/3.1.6-4twpso3/openblas/0.3.10-x32w5jn/gcc/8.3.0 ---------------
   netlib-scalapack/2.1.0

------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/openblas/0.3.10-x32w5jn/gcc/8.3.0 --------------------------
   py-numpy/1.19.0    py-scipy/1.5.0

------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/gcc/8.3.0 --------------------------------------
   autoconf/2.69      gdbm/1.18.1	    libpciaccess/0.13.5    netlib-lapack/3.8.0	      pkgconf/1.7.3	      tar/1.32
   automake/1.16.2    gettext/0.20.2	    libsigsegv/2.12	   numactl/2.0.12	      py-cython/0.29.16       texinfo/6.5
   bzip2/1.0.8	      hwloc/1.11.11	    libtool/2.4.6	   openblas/0.3.10     (L)    py-pybind11/2.5.0       util-macros/1.19.1
   cmake/3.17.3       hwloc/2.2.0    (D)    libxml2/2.9.10	   openmpi/3.1.6       (L)    py-setuptools/46.1.3    xz/5.2.5
   diffutils/3.7      libbsd/0.10.0	    m4/1.4.18		   openssl/1.1.1g	      python/3.7.7	      zlib/1.2.11
   expat/2.2.9	      libffi/3.3	    mpich/3.3.2 	   patchelf/0.10	      readline/8.0
   findutils/4.6.0    libiconv/1.16	    ncurses/6.2 	   perl/5.30.3		      sqlite/3.31.1

---------------------------------------- /home/spack/spack/share/spack/lmod/linux-ubuntu18.04-x86_64/Core ----------------------------------------
   gcc/8.3.0 (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

`MPI` と `LAPACK` プロバイダの両方が同じ安全機能の恩恵を受けるようになります：

```bash
$ module load py-numpy netlib-scalapack

Activating Modules:
  1) netlib-scalapack/2.1.0

$ module load mpich

Lmod is automatically replacing "openmpi/3.1.6" with "mpich/3.3.2".


Due to MODULEPATH changes, the following have been reloaded:
  1) netlib-scalapack/2.1.0

$ module load netlib-lapack

Lmod is automatically replacing "openblas/0.3.10" with "netlib-lapack/3.8.0".


Inactive Modules:
  1) py-numpy

Due to MODULEPATH changes, the following have been reloaded:
  1) netlib-scalapack/2.1.0
```

`openblas` モジュールで `py-numpy` をコンパイルしただけなので、`LAPACK` プロバイダを切り替えると非アクティブになります。
ユーザ環境は設計上一貫しています！

## テンプレートの操作

導入で簡単に述べたように、Spackは [Jinja2](http://jinja.pocoo.org/docs/2.9/) を使用して、個々のモジュールファイルを生成します。
これは、生成されたものに対するカスタマイズに関して、その柔軟性とパワーのすべてを持っていることを意味します！

### モジュールファイルテンプレート

Spackがモジュールファイルの生成に使用するテンプレートは、Spackプレフィックス内の `share/spack/templates/module` ディレクトリに保存され、すべて同じ共通の構造を共有します。
通常、これらは生成されるモジュールのタイプを識別するヘッダで始まります。
階層モジュールファイルの場合は次のとおりです：

```lua
-- -*- lua -*-
-- Module file created by spack (https://github.com/spack/spack) on {{ timestamp }}
--
-- {{ spec.short_spec }}
--
```

二重中括弧 `{{ ... }}` 内のステートメントは 、モジュールの生成時に評価および置換される [式](http://jinja.pocoo.org/docs/2.9/templates/#expressions) を示し ます。
次に、ファイルの残りの部分は、必要に応じてユーザがオーバーライドまたは拡張できる [ブロック](http://jinja.pocoo.org/docs/2.9/templates/#template-inheritance) に分割されます。
`{% ... %}` で区切られた制御構造は、テンプレート言語でも許可されます：

```lua
{% block environment %}
{% for command_name, cmd in environment_modifications %}
{% if command_name == 'PrependPath' %}
prepend_path("{{ cmd.name }}", "{{ cmd.value }}", "{{ cmd.separator }}")
{% elif command_name == 'AppendPath' %}
append_path("{{ cmd.name }}", "{{ cmd.value }}", "{{ cmd.separator }}")
{% elif command_name == 'RemovePath' %}
remove_path("{{ cmd.name }}", "{{ cmd.value }}", "{{ cmd.separator }}")
{% elif command_name == 'SetEnv' %}
setenv("{{ cmd.name }}", "{{ cmd.value }}")
{% elif command_name == 'UnsetEnv' %}
unsetenv("{{ cmd.name }}")
{% endif %}
{% endfor %}
{% endblock %}
```

Spackがテンプレートを検索する場所は、`config.yaml` にて指定されており、

```yaml
  # Locations where templates should be found
  template_dirs:
    - $spack/share/spack/templates
```

次に説明するように、ユーザがカスタムテンプレートを使用するように拡張することができます。

### デフォルトのテンプレートを拡張する

ソフトウェアの1つがグループメンバーシップによって保護されていると仮定しましょう。
許可されたユーザは同じLinuxグループに属し、アクセスはグループレベルで許可されます。
まだ使用する資格がない人が、モジュールの読み込み時に、組織内の誰に連絡してグループに参加するかを伝える役立つメッセージを受け取ることができたら素晴らしいと思いませんか？

このようなサイト固有の動作でモジュールファイルの生成を自動化するために、Spackがモジュールファイルを検索する場所のリストを拡張することから始めます。
次の内容で `${SPACK_ROOT}/etc/spack/config.yaml` ファイルを作成しましょう：

```yaml
config:
  template_dirs:
    - $HOME/.spack/templates
```

これにより、Spackは、テンプレートファイルを探すときに別の場所も検索するようになります。
次に、上記のフォルダにカスタムテンプレート拡張機能を作成する必要があります：

```lua
{% extends "modules/modulefile.lua" %}
{% block footer %}
-- Access is granted only to specific groups
if not isDir("{{ spec.prefix }}") then
    LmodError (
        "You don't have the necessary rights to run \"{{ spec.name }}\".\n\n",
        "\tPlease write an e-mail to 1234@foo.com if you need further information on how to get access to it.\n"
    )
end
{% endblock %}
```

このファイルに `group-restricted.lua` と名前を付けましょう。
この行は、

```lua
{% extends "modules/modulefile.lua" %}
```

階層モジュールファイルの標準テンプレートを再利用していることをJinja2に通知します。
このセクションは、フッタブロックをオーバーライドします。
最後に、`modules.yaml` に数行を追加して、新しいカスタムテンプレートを使用する必要がある仕様をSpackに通知する必要があります。
説明のために、それが `netlib-scalapack` であると仮定しましょう：

```yaml
modules:
  enable::
    - lmod
  lmod:
    core_compilers:
      - 'gcc@7.5.0'
    hierarchy:
      - mpi
      - lapack
    hash_length: 0
    whitelist:
      - gcc
    blacklist:
      - '%gcc@7.5.0'
      - readline
    all:
      filter:
        environment_blacklist:
          - "C_INCLUDE_PATH"
          - "CPLUS_INCLUDE_PATH"
          - "LIBRARY_PATH"
      environment:
        set:
          '{name}_ROOT': '{prefix}'
    openmpi:
      environment:
        set:
          SLURM_MPI_TYPE: pmi2
          OMPI_MCA_btl_openib_warn_default_gid_prefix: '0'
    netlib-scalapack:
      template: 'group-restricted.lua'
```

最後にモジュールファイルを再生成すると、次のようになります：

```bash
$ spack module lmod refresh -y netlib-scalapack
==> Regenerating lmod module files
```

それぞれの `netlib-scalapack` モジュールファイルの最後に次の記述があります：

```lua
-- Access is granted only to specific groups
if not isDir("/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-8.3.0/netlib-scalapack-2.0.2-2p75lzqjbsnev7d2j2osgpkz7ib33oca") then
    LmodError (
        "You don't have the necessary rights to run \"netlib-scalapack\".\n\n",
        "\tPlease write an e-mail to 1234@foo.com if you need further information on how to get access to it.\n"
    )
end
```

また、ソフトウェアにアクセスできないすべてのユーザは、ソフトウェアを要求する正しい電子メールアドレスにリダイレクトされるようになります。

### 次のセクションのためのリストア設定

チュートリアルの今後のセクションでは、`gcc@8.3.0` コンパイラは使用しません。
これは現在デフォルトのコンパイラであるため（現在のデフォルトは利用可能な `gcc` の最新バージョン）、ここで削除します。

```bash
$ spack compiler rm gcc@8.3.0
```

これにより、チュートリアルの残りの部分がスムーズに進むようになります。
