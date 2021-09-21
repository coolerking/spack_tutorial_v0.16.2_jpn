# 環境 チュートリアル

> **チュートリアル セットアップ**
> 前のセクションを完了していない場合、次のようにSpackを設定してください。
>
> ```bash
> git clone https://github.com/spack/spack
> . spack/share/spack/setup-env.sh
> spack tutorial
> ```
>
> 詳細については前章を参照してください。
> さらにヘルプが必要な場合はSlackチャネル `#tutorial" に参加してください。
> [spackpm.herokuapp.com](spackpm.herokuapp.com) で招待を受けてください。

前章では、Spackを使ってパッケージをインストール、削除、そして一覧表示する方法について学習しました。

- `spack install` コマンドによるパッケージのインストール
- `spack uninstall` コマンドによるパッケージの削除
- `spack find` コマンドによる既存インストレーションの照会

また、設定ファイルを使ったSpackインストレーションのカスタマイズについてもドキュメントの [`packages.yaml`](https://spack.readthedocs.io/en/latest/build_settings.html#build-settings) にて議論してきました。

このセクションでは、パッケージの独立したグループを個別操作を可能にするSpack環境を紹介します。
Spack環境は、他の一般的に使用されるツール(Python venvなど)でサポートされているものと同様の仮想環境をユーザに提供すると同時に一般的なインストレーションをシームレスに共有できるようにすることを目的としています。
また、他のユーザやシステム間で簡単に共有して再利用することも目的としています。

たくさんのパッケージで構成されたソフトウェアの適切な管理や/もしくは、複数のプロジェクトの変化していく要件（例：mpiの異なる実装）に圧倒される可能性があります。
Spack環境は、次のことが簡単におこなうことができます：

- プロジェクトの標準ソフトウェア要件を規定する
- ユーザ実行環境をセットアップする
- 一般的な開発環境をサポートする
- CI/CDのためのパッケージをセットアップする
- 他のマシンでビルドを（ほぼ、もしくは正確に）再現する
- さらにほかにも

このチュートリアルでは、環境作成と使い方の基本を紹介し、次に環境内でソフトウェアを拡張、構成、およびビルドする方法について説明します。
コマンドラインインターフェイスから始め、キー環境ファイルを直接編集する方法についても説明します。
Spackが管理する環境と独立した環境の違いについて説明し、最後に再現性のあるビルドに関するセクションを掲示します。

## 環境の基本

現時点までのチュートリアルの状態を `spack find` コマンドで確認します。

```bash
$ spack find
==> 63 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     hdf5@1.10.7	libxml2@2.9.10	netcdf-c@4.7.4		suite-sparse@5.8.1
autoconf-archive@2019.01.06  hdf5@1.10.7	m4@1.4.18	netlib-scalapack@2.1.0	tcl@8.6.10
automake@1.16.2 	     hdf5@1.10.7	matio@1.5.17	netlib-scalapack@2.1.0	tcl@8.6.10
berkeley-db@18.1.40	     hdf5@1.10.7	matio@1.5.17	numactl@2.0.14		texinfo@6.5
boost@1.74.0		     hwloc@1.11.11	metis@5.1.0	openblas@0.3.12 	trilinos@13.0.1
bzip2@1.0.8		     hwloc@2.2.0	mpc@1.1.0	openmpi@3.1.6		util-macros@1.19.1
cmake@3.18.4		     hypre@2.20.0	mpfr@3.1.6	openssl@1.1.1h		xz@5.2.5
diffutils@3.7		     hypre@2.20.0	mpfr@4.0.2	parmetis@4.0.3		zlib@1.2.8
findutils@4.6.0 	     isl@0.18		mpich@3.3.2	parmetis@4.0.3		zlib@1.2.8
gcc@8.3.0		     libiconv@1.16	mumps@5.3.3	patchelf@0.10		zlib@1.2.11
gdbm@1.18.1		     libpciaccess@0.16	mumps@5.3.3	perl@5.32.0
glm@0.9.7.1		     libsigsegv@2.12	ncurses@6.2	pkgconf@1.7.3
gmp@6.1.2		     libtool@2.4.6	netcdf-c@4.7.4	readline@8.0
```

インストール済みパッケージとそれらの依存関係が完全に、ただし乱雑にリストされています。
これは、 `openmpi` と `mpich` の両方で構築されたパッケージが含まれているだけでなく、`hdf5` や `zlib` のような他のパッケージの複数のバリアントも含まれています。
私たちが学んだ `spack find` コマンドによる照会メカニズムは役に立ちますが、すでにインストールしたものを失うことなく、きれいな状態から始めることができれば素晴らしいでしょう。

### 環境の作成および有効化

`spack env` コマンドはその一助となります。
`myproject` という名前のSpack環境を作成してみましょう。

```bash
$ spack env create myproject
==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
==> Created environment 'myproject' in /home/spack/spack/var/spack/environments/myproject
==> You can activate this environment with:
==>   spack env activate myproject
```

環境は、プロジェクトまたはその他の目的でパッケージのインストールを集約するために使う仮想化 `spack` インスタンスのようなものです。
これには、環境からのすべてのパッケージがリンクされている単一のプレフィックスであるビューが関連付けられています。

`spack env list` コマンドを使用して、これまでに作成した環境を確認できます。

```bash
$ spack env list
==> 1 environments
    myproject
```

`spack env activate` コマンドを使って、環境を **有効化** してみましょう。

```bash
$ spack env activate myproject
```

> **注意**
> `spack env activate` コマンドに `-p` オプションをつけると、環境名をプロンプト先頭に表示させることができます。いまどの環境を使っているのかがわかりやすくなります。

環境を有効化すると、`spack find` コマンドは現在の環境内のものののみ表示します。
このため環境を新規構築した状態では、パッケージが一切表示されません。

```bash
$ spack find
==> In environment myproject
==> No root specs
==> 0 installed packages
```

`spack find` コマンドの出力はわずかに異なっています。
これまでインストールしてきたパッケージが一切利用可能でないのを見ても、慌てる必要はありません（ `myproject` 環境内にいるためです）。
そしてルート仕様がないことも表示されています。
それが何を意味するのかについては後述します。

現在どの環境内にいるのかを確認したい場合は、 `spack env status` コマンドを使います。

```bash
$ spack env status
==> In environment myproject
```

現在の環境から出たい場合は、`spack env deactivate` コマンドもしくは `deactivate` エイリアスを使います。

```bash
$ despacktivate         # short alias for `spack env deactivate`
$ spack env status
==> No active environment
$ spack find
==> 63 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     hdf5@1.10.7	libxml2@2.9.10	netcdf-c@4.7.4		suite-sparse@5.8.1
autoconf-archive@2019.01.06  hdf5@1.10.7	m4@1.4.18	netlib-scalapack@2.1.0	tcl@8.6.10
automake@1.16.2 	     hdf5@1.10.7	matio@1.5.17	netlib-scalapack@2.1.0	tcl@8.6.10
berkeley-db@18.1.40	     hdf5@1.10.7	matio@1.5.17	numactl@2.0.14		texinfo@6.5
boost@1.74.0		     hwloc@1.11.11	metis@5.1.0	openblas@0.3.12 	trilinos@13.0.1
bzip2@1.0.8		     hwloc@2.2.0	mpc@1.1.0	openmpi@3.1.6		util-macros@1.19.1
cmake@3.18.4		     hypre@2.20.0	mpfr@3.1.6	openssl@1.1.1h		xz@5.2.5
diffutils@3.7		     hypre@2.20.0	mpfr@4.0.2	parmetis@4.0.3		zlib@1.2.8
findutils@4.6.0 	     isl@0.18		mpich@3.3.2	parmetis@4.0.3		zlib@1.2.8
gcc@8.3.0		     libiconv@1.16	mumps@5.3.3	patchelf@0.10		zlib@1.2.11
gdbm@1.18.1		     libpciaccess@0.16	mumps@5.3.3	perl@5.32.0
glm@0.9.7.1		     libsigsegv@2.12	ncurses@6.2	pkgconf@1.7.3
gmp@6.1.2		     libtool@2.4.6	netcdf-c@4.7.4	readline@8.0
```

`myproject` 環境から出ると、すべてのインストール済みパッケージが再び `spack find` コマンドで参照できるようになります。

### パッケージのインストール

環境作成と有効化がどのように機能するかを理解したので、`myproject` 環境に戻っていくつかのパッケージをインストールしましょう。
`tcl` 、 `trilinos` をインストールします。

```bash
$ spack env activate myproject
$ spack install tcl
==> All of the packages are already installed
$ spack install trilinos
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openblas-0.3.12-te35tjxctgjpeeuk357xghoinvwf2k7u
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-archive-2019.01.06-bdyarrkdgse5adcjr4kxqc4jns74k7yn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.74.0-hc5atqce4emwrrkxuyfayf5k43r5cst2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-ejanivxt2gsv2qd3so7fil7izp4m3bcd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hypre-2.20.0-nndket2abxbsbuoocew5m643e36waxj3
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/parmetis-4.0.3-htdsyvt5yp6vl3nxbxp3627gohe7ldup
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netlib-scalapack-2.1.0-wsiydak4jm7sms57zcu4rmzmdmgxqb5j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/matio-1.5.17-zoqjxgkw7h3yoxbagripnd6muniihwno
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netcdf-c-4.7.4-yixejlymvzs5aduzazrbs37hugw7sl7g
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mumps-5.3.3-3jmjrzw7kx44fyoltswqx7uhu33zikhm
==> Installing trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2
==> Extracting trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2
==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
```

`tcl` および `trilions` のすべての依存関係は既にインストールされています。
環境ビューが更新されていることに注意してください。

`spack find` コマンドを使って環境の内容を確認します。

```bash
$ spack find
==> In environment myproject
==> Root specs
tcl   trilinos

==> 37 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69	 glm@0.9.7.1	      libtool@2.4.6   netcdf-c@4.7.3	      perl@5.30.3	  xz@5.2.5
automake@1.16.2  hdf5@1.10.6	      libxml2@2.9.10  netlib-scalapack@2.1.0  pkgconf@1.7.3	  zlib@1.2.11
boost@1.73.0	 hwloc@1.11.11	      m4@1.4.18       numactl@2.0.12	      readline@8.0
bzip2@1.0.8	 hypre@2.18.2	      matio@1.5.13    openblas@0.3.10	      suite-sparse@5.7.2
cmake@3.17.3	 libiconv@1.16	      metis@5.1.0     openmpi@3.1.6	      tcl@8.6.8
diffutils@3.7	 libpciaccess@0.13.5  mumps@5.3.3     openssl@1.1.1g	      trilinos@12.18.1
gdbm@1.18.1	 libsigsegv@2.12      ncurses@6.2     parmetis@4.0.3	      util-macros@1.19.1
```

`tcl` と `trilinos` は `myproject` 環境の **ルート仕様** に存在することが確認できます。
これは、`tcl` と `trilinos` を明示的にインストールするように要求したためです。
これにより、環境内のすべてのパッケージを組み合わせたグラフの **ルート** になります。
他のインストール済みパッケージは依存関係を満たすために存在しています。

### パッケージの使用

環境はインストール済みパッケージを使用するための便利な方法を提供します。
`spack env activate` コマンドを実行すると、`PATH` 上の環境にあるすべてのインストレーションが提供されます。
そうでなければ、対象のパッケージのための環境をセットアップするために（依存関係のあるもの含めた）すべてのパッケージを環境内で `spack load` コマンドか `module load` コマンドを使用する必要があります。

パッケージを環境にインストールすると、デフォルトではパッケージは単一のプレフィックスまたはビューにリンクされます。
`spack env activate` コマンドにより環境の有効化をおこなうことで、ビューのサブディレクトリを環境変数 `PATH` 、 `LD_LIBRARY_PATH` 、`CMAKE_PREFIX_PATH` などへ追加します。
これにより、環境が使いやすくなります。

では、試してみましょう。
現時点で環境 `myproject` に `tcl` がインストールされている状態です。
`Tcl` には `tclsh` と呼ばれるシェルのようなアプリケーションが含まれていますが、すでに `tclsh` を使うためのパスが通っていることが確認できます：

```bash
$ which tclsh
/home/spack/spack/var/spack/environments/myproject/.spack-env/view/bin/tclsh
```

パスの中に環境変数名と `view` サブディレクトリが含まれていることに注意してください。

`tclsh` が他のプログラム同様実行可能であることを確認できます：

```bash
$ tclsh
% echo "hello world!"
hello world!
% exit
```

同様に Trilions のプログラムも実行可能になっています。
`algebra` のパスも確認して、実行してみましょう：

```bash
$ which algebra
/home/spack/spack/var/spack/environments/myproject/.spack-env/view/bin/algebra
$ algebra

	   AAAAA   LL	     GGGGG   EEEEEEE  BBBBBB   RRRRRR	 AAAAA
	  AA   AA  LL	    GG	 GG  EE       BB   BB  RR   RR	AA   AA
	  AA   AA  LL	    GG	     EE       BB   BB  RR   RR	AA   AA
	  AAAAAAA  LL	    GG	     EEEEE    BBBBBB   RRRRRR	AAAAAAA
	  AA   AA  LL	    GG	GGG  EE       BB   BB  RRRRR	AA   AA
	  AA   AA  LL	    GG	 GG  EE       BB   BB  RR  RR	AA   AA
	  AA   AA  LLLLLLL   GGGGG   EEEEEEE  BBBBBB   RR   RR	AA   AA


			  *** algebra Version 1.47 ***
			       Revised 2019/01/25

			AN ALGEBRAIC MANIPULATION PROGRAM
		 FOR POST-PROCESSING OF FINITE ELEMENT ANALYSES
				EXODUS II VERSION

			  Run on 2021-04-16 at 07:13:10

	       ==== Email gdsjaar@sandia.gov for support ====

			  +++ Copyright 2008 NTESS +++
		       +++ Under the terms of Contract +++
		      +++ DE-NA0003525 with NTESS, the +++
	+++ U.S. Government retains certain rights in this software. +++



 FATAL ERROR - Filenames not specified.

 FATAL ERROR - Syntax is: "algebra file_in file_out"

 PROGRAM ERROR - Dynamic allocation problem - email code sponsor



0LAST ERROR/RETURN CODE:     1

0* * * * * E R R O R   C O D E S * * * * *
0OCCURANCES FLAG  MESSAGE TEXT

	  7   1  SUCCESSFUL COMPLETION
	  0   2  UNABLE TO GET REQUESTED SPACE FROM SYSTEM
	  0   3  DATA MANAGER NOT INITIALIZED
	  0   4  DATA MANAGER WAS PREVIOUSLY INITIALIZED
	 10   5  NAME NOT FOUND IN DICTIONARY
	  0   6  NAME ALREADY EXISTS IN DICTIONARY
	  0   7  ILLEGAL LENGTH REQUEST
	  0   8  UNKNOWN DATA TYPE
	  0   9  DICTIONARY IS FULL
	  0  10  VOID TABLE IS FULL
	  0  11  MEMORY BLOCK TABLE IS FULL
	  0  12  OVERLAPPING VOIDS - INTERNAL ERROR
	  0  13  OVERLAPPING MEMORY BLOCKS - INTERNAL ERROR
	  0  14  INVALID MEMORY BLOCK - EXTENSION LIBRARY ERROR
	  0  15  INVALID ERROR CODE
	  0  16  INVALID INPUT NAME
	  0  17  ILLEGAL CALL WHILE IN DEFERRED MODE
	  0  18  NAME IS OF WRONG TYPE FOR OPERATION


 algebra used 0.01 seconds of CPU time
```

ここでも環境ビューの下に実行可能ファイルが表示されます。

### パッケージのアンインストール

他の環境に影響を与えることなく、環境内のパッケージをアンインストールできます。
Spackは共通のインストレーションを共有しますが、環境はそれらのインストールにのみリンクするため（環境内でのアンインストールが）可能です。

別の環境を作成して、この機能をデモンストレーションしましょう。
環境 `myproject` 仮定は `trilinos` が必要です、我々はそれがインストールされていないが、もはやそれを必要とする別のプロジェクトを持っています。

Suppose myproject requires trilinos but we have another project that has it installed but no longer requires it.

`myproject` は `trilions` が必須です。
しかし、既に `trilions` がインストールされていますが必須としていない別のプロジェクトが存在しています。

`hdf5+hl` および `trilions` がインストール済みの`myproject2` 環境の構築から開始します。

```bash
$ spack env create myproject2
==> Updating view at /home/spack/spack/var/spack/environments/myproject2/.spack-env/view
==> Created environment 'myproject2' in /home/spack/spack/var/spack/environments/myproject2
==> You can activate this environment with:
==>   spack env activate myproject2
$ spack env activate myproject2
$ spack install hdf5+hl
==> All of the packages are already installed
$ spack install trilinos
==> All of the packages are already installed
$ spack find
==> In environment myproject2
==> Root specs
hdf5 +hl  trilinos

==> 40 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     gdbm@1.18.1	libsigsegv@2.12  ncurses@6.2		 perl@5.32.0
autoconf-archive@2019.01.06  glm@0.9.7.1	libtool@2.4.6	 netcdf-c@4.7.4 	 pkgconf@1.7.3
automake@1.16.2 	     gmp@6.1.2		libxml2@2.9.10	 netlib-scalapack@2.1.0  readline@8.0
berkeley-db@18.1.40	     hdf5@1.10.7	m4@1.4.18	 numactl@2.0.14 	 suite-sparse@5.8.1
boost@1.74.0		     hwloc@1.11.11	matio@1.5.17	 openblas@0.3.12	 trilinos@13.0.1
bzip2@1.0.8		     hypre@2.20.0	metis@5.1.0	 openmpi@3.1.6		 util-macros@1.19.1
cmake@3.18.4		     libiconv@1.16	mpfr@4.0.2	 openssl@1.1.1h 	 xz@5.2.5
diffutils@3.7		     libpciaccess@0.16	mumps@5.3.3	 parmetis@4.0.3 	 zlib@1.2.11

```

ルート仕様はコマンドラインで指定したとおりに表示されることに注意してください。
`hdf5` パッケージは `+hl` という必要条件を満たしていることを表示しています。

現時点で2つの環境を保持しています。
`myproject` 環境は `tcl` と `trilions` を保持しており、`myproject2` 環境は `hdf5 +hl` と `trilions` を保持しています。

ここで `myproject2` から `trilions` をアンインストールし、環境内の内容を再確認します：

```bash
$ spack uninstall trilinos
y

==> The following packages will be uninstalled:

    -- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
    qmjbxf2 trilinos@13.0.1

==> Do you want to proceed? [y/N] ==> Updating view at /home/spack/spack/var/spack/environments/myproject2/.spack-env/view
$ spack find
==> In environment myproject2
==> Root specs
hdf5 +hl

==> 21 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69	     hdf5@1.10.7	libsigsegv@2.12  ncurses@6.2	 pkgconf@1.7.3	     zlib@1.2.11
automake@1.16.2      hwloc@1.11.11	libtool@2.4.6	 numactl@2.0.14  readline@8.0
berkeley-db@18.1.40  libiconv@1.16	libxml2@2.9.10	 openmpi@3.1.6	 util-macros@1.19.1
gdbm@1.18.1	     libpciaccess@0.16	m4@1.4.18	 perl@5.32.0	 xz@5.2.5
```

アンインストールしたあと、ルート仕様は1つパッケージ `hdf5 +hl` だけになり、依存関係が少なくなります。

`myproject` 環境のほうはまだ `trilions` パッケージが必須となっています。
では `myproject` 環境に戻って、パッケージがまだ残っていることを確認してみましょう：

```bash
$ despacktivate
$ spack env activate myproject
$ spack find
==> In environment myproject
==> Root specs
tcl   trilinos

==> 41 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     glm@0.9.7.1	libxml2@2.9.10		numactl@2.0.14	    tcl@8.6.10
autoconf-archive@2019.01.06  gmp@6.1.2		m4@1.4.18		openblas@0.3.12     trilinos@13.0.1
automake@1.16.2 	     hdf5@1.10.7	matio@1.5.17		openmpi@3.1.6	    util-macros@1.19.1
berkeley-db@18.1.40	     hwloc@1.11.11	metis@5.1.0		openssl@1.1.1h	    xz@5.2.5
boost@1.74.0		     hypre@2.20.0	mpfr@4.0.2		parmetis@4.0.3	    zlib@1.2.11
bzip2@1.0.8		     libiconv@1.16	mumps@5.3.3		perl@5.32.0
cmake@3.18.4		     libpciaccess@0.16	ncurses@6.2		pkgconf@1.7.3
diffutils@3.7		     libsigsegv@2.12	netcdf-c@4.7.4		readline@8.0
gdbm@1.18.1		     libtool@2.4.6	netlib-scalapack@2.1.0	suite-sparse@5.8.1
```

フー！`myproject` 環境にはまだ `trilions` パッケージがルート仕様上に残っていましたね。
Spackは参照カウントを使用して、 `myproject` に `trilinos` がまだインストールされていることを認識します。

> **注意**
> Trilinosは、環境やパッケージの依存関係で不要になった場合にのみ Spack によってアンインストールされます。

## 同時に複数の仕様に対応

これまで、個々のパッケージを処理するために `install` や `uninstall` を使ってきました。
環境はパッケージの集合を定義するため、それらを一緒にインストールする前に、それらの仕様を環境に追加できます。
仕様は、コマンドラインで追加するか、環境構成ファイルに直接入力できます。

インストールする前に環境に仕様を追加することで、環境全体を一度にインストールできます。
個々のパッケージは、発展するにつれて環境に追加したり、環境から削除したりできます。

環境のすべての仕様を一度に処理することには、いくつかの利点があります。
まず、Spackの外部でカスタムインストールスクリプトを作成する必要はありません。
次に、Spackのインストールレベルのビルド並列処理を利用することで、多数のパッケージの大規模ビルドを並行して起動できます。

このセクションでは、仕様をインストールする前に環境に仕様を追加する2つの方法に焦点を当てました。

### 仕様の追加

まず `myproject` 環境にいくつかの仕様を追加することから始めていきましょう：

```bash
$ spack env activate myproject
$ spack add hdf5+hl
==> Adding hdf5+hl to environment myproject
==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
$ spack add gmp
==> Adding gmp to environment myproject
==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
```

それでは、`spack find` コマンドを使用して何が起こったのかを見てみましょう：

```bash
$ spack find
==> In environment myproject
==> Root specs
gmp   hdf5 +hl	tcl   trilinos

==> 37 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69	 glm@0.9.7.1	      libtool@2.4.6   netcdf-c@4.7.3	      perl@5.30.3	  xz@5.2.5
automake@1.16.2  hdf5@1.10.6	      libxml2@2.9.10  netlib-scalapack@2.1.0  pkgconf@1.7.3	  zlib@1.2.11
boost@1.73.0	 hwloc@1.11.11	      m4@1.4.18       numactl@2.0.12	      readline@8.0
bzip2@1.0.8	 hypre@2.18.2	      matio@1.5.13    openblas@0.3.10	      suite-sparse@5.7.2
cmake@3.17.3	 libiconv@1.16	      metis@5.1.0     openmpi@3.1.6	      tcl@8.6.8
diffutils@3.7	 libpciaccess@0.13.5  mumps@5.3.3     openssl@1.1.1g	      trilinos@12.18.1
gdbm@1.18.1	 libsigsegv@2.12      ncurses@6.2     parmetis@4.0.3	      util-macros@1.19.1
```

追加した2つの仕様、`hdf5 +hl` と `gmp` が、 **ルート仕様** としてリストされていることに注意してください。
`spack add` コマンドはルートを環境に追加するだけなので、実際にはまだ環境にインストールされていません。

まだインストールされていないすべてのパッケージは、`spack install` コマンドを引数なしで実行するだけで、有効な環境にインストールできます。

```bash
$ spack install
==> Concretized hdf5+hl
[+]  ejanivx  hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
[+]  pmsyupw	  ^openmpi@3.1.6%gcc@7.5.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
[+]  zqwfzhw	      ^hwloc@1.11.11%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x			  ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			      ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		      ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf		  ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  komekkm		      ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		      ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  wbqbc5v		  ^numactl@2.0.14%gcc@7.5.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j			  ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz			      ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw			      ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy				  ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo				      ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized gmp
[+]  3ol3tld  gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o	  ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x	      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln		  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j	      ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		  ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		  ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy		      ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo			  ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym			      ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5	  ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft	  ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Installing environment myproject
==> All of the packages are already installed
==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
```

Spackは、関連するすべてのパッケージがインストールする前に、新しいルート仕様をコンクリート化します。
これは、`spack find` コマンドを使用して確認できます 。

```bash
$ spack find
==> In environment myproject
==> Root specs
gmp   hdf5 +hl	tcl   trilinos

==> 41 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     glm@0.9.7.1	libxml2@2.9.10		numactl@2.0.14	    tcl@8.6.10
autoconf-archive@2019.01.06  gmp@6.1.2		m4@1.4.18		openblas@0.3.12     trilinos@13.0.1
automake@1.16.2 	     hdf5@1.10.7	matio@1.5.17		openmpi@3.1.6	    util-macros@1.19.1
berkeley-db@18.1.40	     hwloc@1.11.11	metis@5.1.0		openssl@1.1.1h	    xz@5.2.5
boost@1.74.0		     hypre@2.20.0	mpfr@4.0.2		parmetis@4.0.3	    zlib@1.2.11
bzip2@1.0.8		     libiconv@1.16	mumps@5.3.3		perl@5.32.0
cmake@3.18.4		     libpciaccess@0.16	ncurses@6.2		pkgconf@1.7.3
diffutils@3.7		     libsigsegv@2.12	netcdf-c@4.7.4		readline@8.0
gdbm@1.18.1		     libtool@2.4.6	netlib-scalapack@2.1.0	suite-sparse@5.8.1
```

### 仕様の構成

環境は、単なるルート仕様のリストではありません。
ルート仕様には、環境が有効化されたときのSpackの動作に影響を与える構成設定が含まれています。
これまでのところ、`myproject` はオーバーライドできる構成のデフォルトに依存しています。
ここでは、仕様を追加し、`mpich` を使った `mpi` ビルドに依存するすべてのパッケージを確認する方法を見ていきます。

`spack spec` コマンドを実行すると、環境外で実行したものと同じコンクリート化を表示します。

```bash
$ spack spec hypre
Input spec
--------------------------------
hypre

Concretized
--------------------------------
hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
    ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
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


[コンクリート化の設定](https://spack.readthedocs.io/en/latest/build_settings.html#concretization-preferences) を使用して `mpi` プロバイダの選択を構成して、コンクリート化時の動作を変更できます。

`spack config get` を実行して、環境の構成を確認することから始めましょう：

```bash
$ spack config get
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs: [tcl, trilinos, hdf5+hl, gmp]
  view: true
```

`spack config get` コマンドの結果はSpackが環境構成を保存する際に使用する特別なYMAL形式で表示します。
現在のファイルは、単一の `spack` ヘッダと **ルート** `specs` (仕様)のリストが記述されています。

> **注意**
> 次に進む前に、環境変数 `EDITOR` へ、お好みのテキストエディタのパスを設定していることを確認してください。

`spack config edit` コマンドを使って、 `mpich` を `mpi` プロバイダとして優先するようにファイルを編集しましょう。

上記のファイルがエディタで開いた状態となっているはずです。
`packages:all:providers:mpi:` 以下のエントリを含むように変更します。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  packages:
    all:
      providers:
        mpi: [mpich]

  # add package specs to the `specs` list
  specs: [tcl, trilinos, hdf5, gmp]
```

> **注意**
>
> この設定は、**デフォルトの** `mpi` プロバイダのみを定義します。
> （ `spack install hdf5^openmpi` などを実行して）コマンドラインでプロバイダ設定をオーバーライドすることも可能です。
>
> ここでは、環境構成が具体化にどのように影響するかを示すために、これを紹介します。
>構成オプションについては、[構成チュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_configuration.html#configs-tutorial) で詳しく説明しています。

`spack spec` コマンドを使用して、この変更がパッケージに与える影響を見てみましょう。

```bash
$ spack spec hypre
Input spec
--------------------------------
hypre

Concretized
--------------------------------
hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
    ^mpich@3.3.2%gcc@7.5.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
	^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	    ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
		^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	    ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
		^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
		^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
		    ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
			^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
			    ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
	    ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	    ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
	^hwloc@2.2.0%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
	    ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
		^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
	    ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
		^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
		^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
		^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
    ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
```

`mpich` は具体化された仕様の `mpi` の依存関係であることが確認できます。
また `mpich` のコンクリート化された依存関係もすべて確認できます。

ここで表面を削るだけで、 `mpi` に依存するパッケージの `mpi` プロバイダのデフォルトを変更します。
環境に対して行うことができる他の多くのカスタマイズがあります。
詳細については、このセクションの最後にあるリンクを参照してください。

### 環境の再コンクリート化

仮想プロバイダの変更など構成に大幅な変更を加えたあとは、環境にパッケージを再インストールする必要がある場合があります。
これは Spack に環境を再コンクリート化し、仕様を再インストールするよう強制することで実現可能です。

たとえば環境 `myproject` の一部を `openmpi` でインストールしたため、環境にいんすとーるされたパッケージは新しい構成と同期しなくなりました。
`myproject` のすべてを `mpich` でインストールしたいとします。

`spack concretize --force` コマンドを実行して、Spack環境の仕様すべてを再コンクリート化しましょう：

```bash
$ spack concretize --force
==> Concretized tcl
[+]  rtz7664  tcl@8.6.10%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos
[+]  xvzpvxd  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
[+]  hc5atqc	  ^boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	      ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten		  ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	      ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  bltycqw	  ^cmake@3.18.4%gcc@7.5.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
[+]  crhlefo	      ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		  ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  es377uq	      ^openssl@1.1.1h%gcc@7.5.0+systemcerts arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j		  ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		      ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		      ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy			  ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  n4xti2g	  ^glm@0.9.7.1%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
[+]  cq4k4cv	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
[+]  vbelg5s	      ^mpich@3.3.2%gcc@7.5.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		  ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		  ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5		  ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		      ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  a4cxlu7		  ^hwloc@2.2.0%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		      ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k			  ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf		      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm			  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  us6giii	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
[+]  usmrkqu	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
[+]  rmlof2f	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
[+]  h7suxc3	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
[+]  spkgtxp	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  i7otoak	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized hdf5+hl
[+]  cq4k4cv  hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
[+]  vbelg5s	  ^mpich@3.3.2%gcc@7.5.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o	      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		  ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln		      ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j		  ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		      ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		      ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy			  ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo			      ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym				  ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5	      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5	      ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		  ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		  ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  a4cxlu7	      ^hwloc@2.2.0%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf		  ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  komekkm		      ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		      ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized gmp
[+]  3ol3tld  gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o	  ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x	      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln		  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j	      ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		  ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		  ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy		      ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo			  ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym			      ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5	  ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft	  ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Updating view at /home/spack/spack/var/spack/environments/myproject/.spack-env/view
```

これですべての仕様がコンクリート化されMPI実装として `mpich` を使ったインストールの準備ができました。
`spack install` コマンドを再実行することで処理を完了することができます。

## 環境内でのビルド

有効化された環境では、システムにインストールされているかのように、インストールされているプログラムを呼び出すことができます。
このセクションでは、その機能を利用します。

いくつかのMPIプログラムをコンパイルします。
`myproject` 環境にMPI実装がインストールされているため、`mpicc` がパスとして有効になっています。
これは `which` を使用して確認できます：

```bash
$ spack env status
==> In environment myproject
$ which mpicc
/home/spack/spack/var/spack/environments/myproject/.spack-env/view/bin/mpicc
```

前述したとおり、環境を有効化するといくつかの環境変数が設定されます。
`CPATH` 、 `LIBRARY_PATH` 、 `LD_LIBRARY_PATH` のような環境変数を含んでおり、環境にインストール済みのパッケージのヘッダやライブラリを見つけることができます。

`env|grep PATH` を実行して、パス関連の環境変数を見てみましょう：

```bash
$ env | grep PATH=
LIBRARY_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib64
C_INCLUDE_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/include
LD_LIBRARY_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib64
CPLUS_INCLUDE_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/include
PKG_CONFIG_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib/pkgconfig:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/share/pkgconfig:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib64/pkgconfig
ACLOCAL_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/share/aclocal
PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/bin:/home/spack/spack/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PYTHONPATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib
SPACK_LD_LIBRARY_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/lib64
MANPATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view/share/man:/home/spack/spack/var/spack/environments/myproject/.spack-env/view/man
MODULEPATH=/home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64
CMAKE_PREFIX_PATH=/home/spack/spack/var/spack/environments/myproject/.spack-env/view
```

とても単純なMPIプログラムを作成することで、これらの環境設定の使用法を示すことができます。

次のコードが記述されている `mpi-hello.c` というプログラムを作成しましょう：

```c
#include <stdio.h>
#include <mpi.h>
#include <zlib.h>

int main(int argc, char **argv) {
  int rank;
  MPI_Init(&argc, &argv);

  MPI_Comm_rank(MPI_COMM_WORLD, &rank);
  printf("Hello world from rank %d\n", rank);

  if (rank == 0) {
    printf("zlib version: %s\n", ZLIB_VERSION);
  }

  MPI_Finalize();
}
```

このプログラムは `mpi` および `zlib` のヘッダファイルをincludeしています。
このプログラムを実行すると各MPIランクおよび `zlib` のバージョンも表示します。

ビルドして実行してみましょう：

```bash
$ mpicc ./mpi-hello.c
$ mpirun -n 4 ./a.out
Hello world from rank 3
Hello world from rank 0
zlib version: 1.2.11
Hello world from rank 1
Hello world from rank 2
```

include パスやライブラリパスなど、特別な引数をコンパイラに渡していないことに注意してください。
また、`Hello world` という表示がランクごとに出力され、プログラムのビルドに使用された `zlib` バージョンが出力されていることもわかります。

`spack find` コマンドを実行すると、プログラムのビルドに使用された `zlib` のバージョンが、環境内に存在することを確認できます：

```bash
$ spack find zlib
==> In environment myproject
==> Root specs
gmp   hdf5 +hl  tcl   trilinos

==> 1 installed package
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 ----------------
zlib@1.2.11
```

表示されたバージョンが、インストールに使用されたバージョンと一致することに注意してください。

## ビルドの再現

Spack 環境は [Python venv](https://docs.python.org/3/library/venv.html) や [Conda 環境](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#) ににた仮想環境を提供します。
ある環境のパッケージを別の環境のパッケージから分離しておくことが、Spack環境の目的です。
これらの環境は spackにより管理され、独立しています。
いずれの場合も、それらの環境ファイルを使用して、他のユーザや他のマシンでビルドを再現できます。
これらのファイルはビルドを再現するための鍵となりますので、ここからはじめていきましょう。

### 環境ファイル

`spack.yaml` と `spack.lock` は、環境の内容をトラッキングする2つのファイルです。
`spack.yaml` ファイルは、既に `spack config edit` コマンドにより編集した環境構成を保持します。
`spack.lock` ファイルはコンクリート化中に自動的に生成されます。

2つのファイルは、2つの基本的な概念をあらわしています。

- `spack.yaml` ：インストールする仕様の概要および構成、と
- `spack.lock` ：完全にコンクリート化された仕様、です。

これらのファイルは、開発者および管理者が再現可能な手段で環境を管理するために使用することを目的としています。
それらの再利用については後で説明します。

> **注意**
> 両方の環境ファイルをリポジトリでバージョン管理、共有することで、異なるユーザや他のマシンに同じソフトウェアセットをインストールすることができます。

### Spackにより管理された環境と独立した環境

環境は、Spackにより管理されているか独立しているかのどちらかです。
どちらのタイプの環境も、環境ファイルによって定義されます。
これまでのチュートリアルでは、Spackにより管理された環境のみを作成しました。
このセクションでは、それらの違いについて説明します。

Spackにより管理されている環境は、 `spack env create <名前>` コマンドを使用して作成します。
Spackにより管理されている環境は、`var/spack/environments` サブディレクトリに自動的に作成され、名前で参照することができます。

独立した環境は、2つの方法のいずれかで作成できます。
１つめの方法は、Spackの環境ファイルを（ `var/spack/environments` 以外の）任意のディレクトリに配置します。
２つめの方法は、 `spack env create -d <ディレクトリ>` を使用してファイルが存在するディレクトリを指定します。
独立した環境には名前はありません。

### Spackにより管理された環境のレビュー

これまでのセクションでは有効な環境 `myproject` を作成しているので、このセクションでは主にその環境ファイルに焦点を当てましょう。

`spack config edit` コマンドを使用して環境の構成を変更した際に、実際に `spack.yaml` ファイルを編集していました。
`spack cd` コマンドを使用して、ファイルを含むディレクトリに移動できます：

```bash
$ spack cd -e myproject
$ pwd
/home/spack/spack/var/spack/environments/myproject
$ ls
spack.lock  spack.yaml
```

`myproject` は、Spackにより管理されている環境の範囲内 `var/spack/environments` のサブディレクトリに存在しています。
したがって、環境名で参照できます。
`spack env list` コマンドを実行時することでも、表示されます：

```bash
$ spack env list
==> 2 environments
    myproject  myproject2
```

緑色で強調表示されていることから　`myproject` が有効状態であることがわかります。

また環境ディレクトリに `spack.yaml` と `spack.lock` の両方の環境ファイルが含まれていることも確認できます。
これは、環境をコンクリート化した際に`spack.lock` ファイルが作成されるからです。

`spack.yaml` ファイルを `cat` すると、 `spack config get` コマンドで前に確認した内容と同じであることがわかります：

```bash
$ cat spack.yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs: [tcl, trilinos, hdf5+hl, gmp]
  view: true
```

### 独立した環境の作成

環境は、作成またはSpackインスタンスによって管理する必要はありません。
むしろ、それらの環境ファイルは任意のディレクトリに配置できます。
この機能は、環境ベースのソフトウェアリリースやCI/CDなどのユースケースに非常に役立ちます。

単純なプロジェクトのために、独立した環境を最初から作成してみましょう。

```bash
$ cd
$ mkdir code
$ cd code
$ spack env create -d .
==> Updating view at /home/spack/code/.spack-env/view
==> Created environment in /home/spack/code
==> You can activate this environment with:
==>   spack env activate /home/spack/code
```

`spack env cretae -d .` コマンドにより、Spackが環境を作成し、ビューを更新し、有効化化に必要なコマンド記述を出力していることに注意してください。
有効化するためのコマンドの内容からわかるように、環境は独立しているため、ディレクトリパスを指定する必要があります。

ディレクトリの内容を一覧表示し、構成ファイルを確認して、このコマンドで実際に何が起こったかを見てみましょう：

```bash
$ ls
spack.yaml
$ cat spack.yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs: []
  view: true
```

codeディレクトリに `spack.yaml` ファイルが作成されていることに注意してください。
また、構成ファイルのspecsが空のリスト（つまり `[]` ）となっていることに注意してください。
この空のリストは、環境はルート仕様だけ含んでいることを意図しています。

`spack env list` コマンドを実行することで、管理された環境ではないことを確認できます：

```bash
$ spack env list
==> 2 environments
    myproject  myproject2
```

codeディレクトリパスが表示されていないことに注意してください。

それでは、環境にいくつかの仕様を追加しましょう。
プロジェクトが `boost` 、 `trilions` 、 `openmpi` に依存するとします。
お気に入りのテキストエディタを使用して、これらのパッケージを仕様リストに追加します。
ここでの例では、YAMLリストのダッシュ構文（つまり `-` ）が使用されています。
次のような記述でパッケージが含まれるようになるはずです：

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs:
  - boost
  - trilinos
  - openmpi
  view: true
```

では、環境を有効化し、パッケージをインストールしてみましょう：

```bash
$ spack env activate .
$ spack install
==> Concretized boost
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos
[+]  qmjbxf2  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
[+]  hc5atqc	  ^boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	      ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten		  ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	      ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  bltycqw	  ^cmake@3.18.4%gcc@7.5.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
[+]  crhlefo	      ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		  ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  es377uq	      ^openssl@1.1.1h%gcc@7.5.0+systemcerts arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j		  ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		      ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		      ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy			  ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  n4xti2g	  ^glm@0.9.7.1%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
[+]  ejanivx	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
[+]  pmsyupw	      ^openmpi@3.1.6%gcc@7.5.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
[+]  zqwfzhw		  ^hwloc@1.11.11%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		      ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft			  ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x			      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln				  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k			  ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf		      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm			  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  wbqbc5v		      ^numactl@2.0.14%gcc@7.5.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o			  ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5			  ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  nndket2	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
[+]  zoqjxgk	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
[+]  3jmjrzw	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
[+]  wsiydak	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
[+]  yixejly	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  htdsyvt	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi
[+]  pmsyupw  openmpi@3.1.6%gcc@7.5.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
[+]  zqwfzhw	  ^hwloc@1.11.11%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m	      ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		  ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		  ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		  ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf	      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  komekkm		  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  wbqbc5v	      ^numactl@2.0.14%gcc@7.5.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		  ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j		      ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz			  ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw			  ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy			      ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo				  ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		  ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Installing environment /home/spack/code
==> All of the packages are already installed
==> Updating view at /home/spack/code/.spack-env/view
```

Spackは、仕様とその依存関係をインストールする前に、仕様をコンクリート化したことに注意してください。
また、環境のビューも更新されました。
これらのパッケージはすべて環境外にすでにインストールされているため、それらのリンクが環境に追加されました。

### インストールされた環境の更新

Spackは、初期仕様がインストールされた後でも、環境の微調整をサポートします。
前にコマンドラインインターフェイスを使った環境の外で行うのと同様、仕様を自由に追加削除できます。

たとえば `hdf5` をファイルに追加してみましょう：

```bash
$ spack add hdf5@5.5.1
==> Adding hdf5@5.5.1 to environment /home/spack/code
==> Updating view at /home/spack/code/.spack-env/view
$ cat spack.yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs:
  - boost
  - trilinos
  - openmpi
  - hdf5@5.5.1
  view: true
```

`spack add` こまんどにより有効化されている環境にパッケージが追加され、構成ファイル内に仕様リストが追加されていることに注意してください。

> **注意**
> `spack add` コマンドだけでは設定ファイルに追加されるだけなので、環境へパッケージをインストールするには `spack install` コマンドを実行する必要があります。

ここで `spack remove` コマンドを使い、設定ファイルから仕様を削除します：

```bash
$ spack remove hdf5
==> Removing hdf5 from environment /home/spack/code
==> Updating view at /home/spack/code/.spack-env/view
$ cat spack.yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration settings.
spack:
  # add package specs to the `specs` list
  specs: [boost, trilinos, openmpi]
  view: true
```

仕様が環境の使用リストから削除されたことがわかります。

> **注意**
> `spack add` や `spack remove` を使用する代わりに `spack.yaml` ファイルを直接編集することも可能です。

### `spack.lock` のレビュー

それでは、抽象仕様からコンクリート化に注意を向けましょう。

これまで焦点を当ててきたのは `spack.yaml` ファイルによって表される抽象的な環境構成にありました。
`spack.yaml` ファイルがコンクリート化されると、Spackは環境の完全にコンクリート化（具現化・具体化）された状態を表すファイル `spack.lock` を作成します。

`spack.lock` ファイルは、環境構築を再現するために必要な情報をマシンで読み取れるように表現することを目的としています。
そのため `json` 記述されており、 `yaml` よりも読みにくくなっています。

現在の環境の `spack.lock` ファイルの先頭30行を見てみましょう：

```bash
$ head -30 spack.lock
{
 "_meta": {
  "file-type": "spack-lockfile",
  "lockfile-version": 2
 },
 "roots": [
  {
   "hash": "iwji2vty6m7hp27zbzufewdhq6pty3fx",
   "spec": "boost"
  },
  {
   "hash": "246z4kkzk6vsg2ot3rzlz5jwwimxj63g",
   "spec": "trilinos"
  },
  {
   "hash": "7o7eidaryeh72cizcbpvvjnq7ojjya6f",
   "spec": "openmpi"
  }
 ],
 "concrete_specs": {
  "iwji2vty6m7hp27zbzufewdhq6pty3fx": {
   "boost": {
    "version": "1.74.0",
    "arch": {
     "platform": "linux",
     "platform_os": "ubuntu18.04",
     "target": "x86_64"
    },
    "compiler": {
     "name": "gcc",
```

現在の `spack.lock` ファイルは読み取り可能ではありますが、環境の各パッケージの実際の構成を表す約2000行の情報が記述されています。

### 環境の再現

環境ファイルの内容について説明したので、それらを使用して環境を再現する方法について説明します。
別のマシンで自身が行うか、自分以外の誰かが構築した環境を利用することをお勧めします。
プロセスはどちらの場合も同じです。

いずれかの環境ファイルを `spack env create` コマンドに渡すことで、環境を再作成できます。
選択するファイルは、抽象仕様を使用してビルドを概算するか、コンクリート化された仕様に基づいて正確なビルドを概算するかによって異なります。

### `spack.yaml` の使用

`spack.yaml` ファイルを使うと、おおよそのビルドが作成されます。
このアプローチは、たとえば新しいプラットフォームで同じ仕様を構築する場合に関係します。
ファイル内の抽象的な要件を保持することにより、環境を再現できます。
ただし、コンクリート化担当者が異なる依存関係を選択する可能性があるため、ソフトウェアのビルドが実際には異なる場合があります。

`spack env create` コマンドを使用して、ファイルから抽象的な環境（`abstract`という環境名で）を作成してみましょう：

```bash
$ spack env create abstract spack.yaml
==> Updating view at /home/spack/spack/var/spack/environments/abstract/.spack-env/view
==> Created environment 'abstract' in /home/spack/spack/var/spack/environments/abstract
==> You can activate this environment with:
==>   spack env activate abstract
```

ここでは、Spackが指定した環境名でSpackにより管理された環境を作成したことがわかります。

また、これは新しく作成された環境であるため、環境を有効化した後に `spack find` を実行することでもわかるように、まだ仕様がインストールされていません。

```bash
$ spacktivate abstract
$ spack find
==> In environment abstract
==> Root specs
boost   openmpi   trilinos 

==> 0 installed packages
```

`spack.yaml` ファイルに列挙されているものと同じルート仕様が存在していることに注意してください。

### `spack.lock` の使用

`spack.lock` ファイルは、元のビルドを正確に複製するために使用されます。
ビルド中に行われたすべての決定に関する情報が含まれているので、ビルドを複製することができます。

ファイルから、`concrete` という名前の具現化（コンクリート化）されたな環境を作成しましょう：

```bash
$ spack env create concrete spack.lock
==> Updating view at /home/spack/spack/var/spack/environments/concrete/.spack-env/view
==> Created environment 'concrete' in /home/spack/spack/var/spack/environments/concrete
==> You can activate this environment with:
==>   spack env activate concrete
```

Spackにより管理された環境 `concrete` を再度作成したことがわかります。

`spack.lock` ファイルから環境を作成したので、同じルート仕様を取得するだけでなく、環境を有効化した後に `spack find` コマンドを呼び出すことでわかるように、すべてのパッケージが環境にインストールされます。

```bash
$ spacktivate concrete
$ spack find
==> In environment concrete
==> Root specs
boost   openmpi   trilinos

==> 40 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69                libiconv@1.16           openblas@0.3.12
autoconf-archive@2019.01.06  libpciaccess@0.16       openmpi@3.1.6
automake@1.16.2              libsigsegv@2.12         openssl@1.1.1h
berkeley-db@18.1.40          libtool@2.4.6           parmetis@4.0.3
boost@1.74.0                 libxml2@2.9.10          perl@5.32.0
bzip2@1.0.8                  m4@1.4.18               pkgconf@1.7.3
cmake@3.18.4                 matio@1.5.17            readline@8.0
diffutils@3.7                metis@5.1.0             suite-sparse@5.8.1
gdbm@1.18.1                  mpfr@4.0.2              trilinos@13.0.1
glm@0.9.7.1                  mumps@5.3.3             util-macros@1.19.1
gmp@6.1.2                    ncurses@6.2             xz@5.2.5
hdf5@1.10.7                  netcdf-c@4.7.4          zlib@1.2.11
hwloc@1.11.11                netlib-scalapack@2.1.0
hypre@2.20.0                 numactl@2.0.14
```

> **注意**
> `spack.lock` を使ってのビルド再構成には、（一般的には）同じタイプのマシンを用意しておく必要があります。

## さらなる詳細

この（環境）チュートリアルでは、環境のさわりとその機能について説明しました。
詳細については、以下のSpackリソースをご覧ください。

### 環境のセットアップと構築

- [環境](https://spack.readthedocs.io/en/latest/environments.html) ：リファレンスドキュメント
- [構成チュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_configuration.html) ：環境をカスタマイズする
- [Spack スタックチュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_stacks.html) ：組み合わせ環境を構成する（たとえば、複数コンパイラに対応する同一パッケージ）
- [インストールレベルの並列ビルド](https://spack.readthedocs.io/en/latest/packaging_guide.html#install-level-build-parallelism) ：環境の並列ビルドをするための `spack install` 実行のしかたについて


### 環境の使用

- [開発者ワークフロー](https://spack-tutorial.readthedocs.io/en/latest/tutorial_developer_workflows.html)：環境上でコードを開発する
- [Spack環境でのGitLab CI パイプライン](https://spack.readthedocs.io/en/latest/pipelines.html) ：環境を使ってCIパイプラインを作成する
- [コンテナイメージ](https://spack.readthedocs.io/en/latest/containers.html) ：環境からコンテナを作成する
- [Spack スタックチュートリアル](https://spack-tutorial.readthedocs.io/en/latest/tutorial_stacks.html) ：ソフトウェアの大規模展開を管理する

### 環境のサンプル

- [Spack スタックカタログ](https://spack.github.io/spack-stack-catalog/) ：GitHubで探索可能な環境を見つける
