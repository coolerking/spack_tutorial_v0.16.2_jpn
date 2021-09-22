# ミラーチュートリアル

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

このチュートリアルでは、バイナリキャッシュを使用してソースミラーを設定するプロセスについて説明します。
ソースミラーとバイナリキャッシュは、インターネットにアクセスできないマシンでSpackを使用する場合に非常に便利です。
ソースミラーを使用すると、外部のインターネットにアクセスする代わりに、ファイルシステム上のディレクトリからソースコードをフェッチできます。
バイナリキャッシュを使用すると、コンパイル済みのバイナリをSpackインストールパスにインストールできます。
これら2つの機能を組み合わせることで、大規模な開発チーム内でSpackを使用する際のビルドを高速化できます。

このチュートリアルではミラーのファイルシステムを使用しますが、ミラーはWebサーバまたはs3バケットにセットアップすることもできます。
`curl` でアクセス可能な任意のURLをSpackミラーとしてセットアップできます。

デフォルトでは、ダウンロードの信頼性を高めるために、Spackにはクラウド内のソースミラーが構成されています。
このチュートリアルでは、バイナリキャッシュ用のミラーもすでに設定しています。
次のようにコマンド実行して、これらのミラーが構成されていることを確認することができます：

```bash
$ spack mirror list
tutorial	file:///mirror
spack-public	https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/
```

## ソースキャッシュミラーの設定

`spack install` コマンドを実行すると、Spackはインターネットにアクセスして、パッケージをビルドするためのソースコードを取得します。
これはほとんどのクラスタで正常に機能しますが、問題のクラスタが外部インターネットにアクセスできない場合はどうでしょうか？
さまざまな理由でこのようなケースが発生する可能性があります。
大規模なインターネットに接続されていない計算ノード上に構築している場合もあれば、クラスタ全体がインターネットから分離されている場合もあります。

```bash
$ cd ~
$ mkdir cache-env
$ cd cache-env
$ spack env create -d .
==> Updating view at /home/spack/cache-env/.spack-env/view
==> Created environment in /home/spack/cache-env
==> You can activate this environment with:
==>   spack env activate /home/spack/cache-env
$ spacktivate .
$ # for now, disable fortran support in all packages
$ spack config add "packages:all:variants: ~fortran"
$ spack add macsio+scr
==> Adding macsio+scr to environment /home/spack/cache-env
$ spack install
==> Concretized macsio+scr
[+]  lbkg732  macsio@1.1%gcc@7.5.0~exodus~hdf5~ipo+mpi~pdb+scr+silo~szip~typhonio~zfp~zlib build_type=RelWithDebInfo patches=59479b946e5bbf677e814dc1cde12b38dd3b083fec8c543fc6d3abf9f73dbbfa arch=linux-ubuntu18.04-x86_64
[+]  bltycqw	  ^cmake@3.18.4%gcc@7.5.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
[+]  crhlefo	      ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		  ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  es377uq	      ^openssl@1.1.1h%gcc@7.5.0+systemcerts arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j		  ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		      ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw		      ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy			  ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+]  7tkgwjv	  ^json-cwx@0.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o	      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		  ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln		      ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5	      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft	      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  pmsyupw	  ^openmpi@3.1.6%gcc@7.5.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
[+]  zqwfzhw	      ^hwloc@1.11.11%gcc@7.5.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf		  ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  komekkm		      ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  wbqbc5v		  ^numactl@2.0.14%gcc@7.5.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
[+]  sqm6ik6	  ^scr@2.0.0%gcc@7.5.0+dtcmp~fortran~ipo+libyogrt async_api=NONE build_type=RelWithDebInfo cache_base=/tmp cntl_base=/tmp copy_config=none file_lock=FLOCK resource_manager=SLURM scr_config=scr.conf arch=linux-ubuntu18.04-x86_64
[+]  fxiami4	      ^dtcmp@1.1.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  tzf27nc		  ^lwgrp@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  ld352tm	      ^libyogrt@1.24%gcc@7.5.0~static scheduler=system arch=linux-ubuntu18.04-x86_64
[+]  pv3zp64	      ^pdsh@2.31%gcc@7.5.0+ssh+static_modules arch=linux-ubuntu18.04-x86_64
[+]  vfrf7as	  ^silo@4.10.2%gcc@7.5.0~fortran+mpi+pic+shared~silex patches=7b5a1dc2a0e358e667088d77e7caa780967fa8ea60be89c44986605df9990abe arch=linux-ubuntu18.04-x86_64
[+]  vedchc5	      ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran~hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64

==> Installing environment /home/spack/cache-env
==> All of the packages are already installed
==> Updating view at /home/spack/cache-env/.spack-env/view
```

この環境を作成してインストールすると、このビルドをミラーに再現するために必要なソースコードを簡単にアップロードできます。
次のコマンドは、ミラーを作成し、環境に含まれている `scr` パッケージのソースコードをアップロードします。
`-d` フラグ（`--directory`の短縮形）は、ミラーリングされたソースコードファイルを配置する場所をSpackに伝達します。

```bash
$ spack mirror create -d ~/mirror scr
==> Adding package scr@2.0.0 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/47/471978ae0afb56a20847d3989b994fbd680d1dea21e77a5a46a964b6e3deed6b.tar.gz
####################################################################################################################################################################### 100.0%
==> Successfully created mirror in file:///home/spack/mirror
  Archive stats:
    0	 already present
    1	 added
    0	 failed to fetch.
```

TBD
