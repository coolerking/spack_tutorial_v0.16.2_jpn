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

`spack.yaml` ファイルに数行追加することで、このソースミラーを使用するようにSpackを構成できます。

```bash
$ spack mirror add mymirror ~/mirror
```

環境内のすべてのパッケージを手動でアップロードするのは面倒な場合があります。
幸いなことに、環境内で `spack mirror create` コマンドを `--all` フラグ付きで実行すると、現在の環境の構築に使用されるすべてのソースが指定されたディレクトリにアップロードされます。

```bash
$ spack mirror create -d ~/mirror -a
==> Adding package autoconf@2.69 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/95/954bd69b391edc12d6a4a51a2dd1476543da5c6bbf05a95b59dc0dd6fd4c2969.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package automake@1.16.2 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b2/b2f361094b410b4acbf4efba7337bdb786335ca09eb2518635a09fb7319ca5c1.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package berkeley-db@18.1.40 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/0c/0cecb2ef0c67b166de93732769abdeba0555086d51de1090df325e18ee8da9c8.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package cmake@3.18.4 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/59/597c61358e6a92ecbfad42a9b5321ddd801fc7e7eca08441307c9138382d4f77.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package dtcmp@1.1.1 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/dd/ddf3c57cbb83515e1b7e4111b8a83f832e66376b40eee5d8a5549dd7b8446bc6.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package gdbm@1.18.1 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/86/86e613527e5dba544e73208f42b78b7c022d4fa5a6d5498bf18c8d6f745b91dc.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package hdf5@1.10.7 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/7a/7a1a0a54371275ce2dfc5cd093775bb025c365846512961e7e5ceaecb437ef15.tar.gz
####################################################################################################################################################################### 100.0%
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/57/57cee5ff1992b4098eda079815c36fc2da9b10e00a9056df054f2384c4fc7523
####################################################################################################################################################################### 100.0%
==> Adding package hwloc@1.11.11 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/74/74329da3be1b25de8e98a712adb28b14e561889244bf3a8138afe91ab18e0b3a.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package json-cwx@0.12 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/3b/3bfae1f23eacba53ee130dbd1a6acf617af4627a9b4e4581d64b20a99b4e2b60.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libiconv@1.16 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/e6/e6a1b1b589654277ee790cce3734f07876ac4ccfaecbee8afa0b649cf529cc04.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libpciaccess@0.16 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/84/84413553994aef0070cf420050aa5c0a51b1956b404920e21b81e96db6a61a27.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libsigsegv@2.12 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/3a/3ae1af359eebaa4ffc5896a1aee3568c052c99879316a1ab57f8fe1789c390b6.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libtool@2.4.6 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/e3/e3bd4d5d3d025a36c21dd6af7ea818a2afcd4dfc1ea5a17b39d7854bcd0c06e3.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libxml2@2.9.10 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/aa/aafee193ffb8fe0c82d4afef6ef91972cbaf5feea100edc2f262750611b4be1f.tar.gz
####################################################################################################################################################################### 100.0%
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/96/96151685cec997e1f9f3387e3626d61e6284d4d6e66e0e440c209286c03e9cc7.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package libyogrt@1.24 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/36/36695030e72b24b1f22bfcfe42bfd1d3c87f9c0eea5e94ce0120782581ea522f.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package lwgrp@1.0.3 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/20/20b2fc3908bfdf04d1c177f86e227a147214cd155c548b3dd75e54c78e1c1c47.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package m4@1.4.18 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/ab/ab2633921a5cd38e48797bf5521ad259bdc4b979078034a3b790d7fec5493fab.tar.gz
####################################################################################################################################################################### 100.0%
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/fc/fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8
####################################################################################################################################################################### 100.0%
==> Adding package macsio@1.1 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/a8/a86249b0f10647c0b631773db69568388094605ec1a0af149d9e61e95e6961ec.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package ncurses@6.2 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/30/30306e0c76e0f9f1f0de987cf1c82a5c21e1ce6568b9227f7da5b71cbea86c9d.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package numactl@2.0.14 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/1e/1ee27abd07ff6ba140aaf9bc6379b37825e54496e01d6f7343330cf1a4487035.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package openmpi@3.1.6 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/50/50131d982ec2a516564d74d5616383178361c2f08fdd7d1202b80bdf66a0d279.tar.bz2
####################################################################################################################################################################### 100.0%
==> Adding package openssl@1.1.1h to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/5c/5c9ca8774bd7b03e5784f26ae9e9e6d749c9da2438545077e6b3d755a06595d9.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package pdsh@2.31 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/0e/0ee066ce395703285cf4f6cf00b54b7097d12457a4b1c146bc6f33d8ba73caa7.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package perl@5.32.0 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/ef/efeb1ce1f10824190ad1cadbcccf6fdb8a5d37007d0100d2d9ae5f2b5900c0b4.tar.gz
####################################################################################################################################################################### 100.0%
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/9d/9da50e155df72bce55cb69f51f1dbb4b62d23740fb99f6178bb27f22ebdf8a46.tar.gz
####################################################################################################################################################################### 100.0%
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/0e/0eac10ed90aeb0459ad8851f88081d439a4e41978e586ec743069e8b059370ac
####################################################################################################################################################################### 100.0%
==> Adding package pkgconf@1.7.3 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b8/b846aea51cf696c3392a0ae58bef93e2e72f8e7073ca6ad1ed8b01c85871f9c0.tar.xz
####################################################################################################################################################################### 100.0%
==> Adding package readline@8.0 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/e3/e339f51971478d369f8a053a330a190781acb9864cf4c541060f12078948e461.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package scr@2.0.0 to mirror
==> Adding package silo@4.10.2 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/3a/3af87e5f0608a69849c00eb7c73b11f8422fa36903dd14610584506e7f68e638.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package util-macros@1.19.1 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/18/18d459400558f4ea99527bc9786c033965a3db45bf4c6a32eefdc07aa9e306a6.tar.bz2
####################################################################################################################################################################### 100.0%
==> Adding package xz@5.2.5 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/51/5117f930900b341493827d63aa910ff5e011e0b994197c3b71c08a20228a42df.tar.bz2
####################################################################################################################################################################### 100.0%
==> Adding package zlib@1.2.11 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/c3/c3e5e9fdd5004dcb542feda5ee4f0ff0744628baf8ed2dd5d66f8ca1197cb1a1.tar.gz
####################################################################################################################################################################### 100.0%
==> Successfully updated mirror in file:///home/spack/mirror
  Archive stats:
    1	 already present
    30	 added
    0	 failed to fetch.
```

このディレクトリは、共有ファイルシステム上に配置してユーザ間で共有することも可能で、通常のUNIXファイルのアクセス許可で保護できます。
共有ファイルシステムでSpackミラーを作成する場合は、ミラーを更新するたびにファイルのアクセス許可を修正するか、作成する新しいファイルが適切なアクセス許可を持つように [umask](https://man7.org/linux/man-pages/man2/umask.2.html) 設定を更新することを忘れないでください。
次の実行例内の単語 `spack` を適切なUNIXグループに置き換えてください。

```bash
$ umask 027
$ chmod -R g+rs ~/mirror
$ chgrp -R spack ~/mirror
```

Spackがミラーディレクトリから読み取り可能な状態である限りは、Spackはインターネットにアクセスする代わりに、ミラーからソースパッケージを読み取ろうとします。
これは、外部インターネットにはアクセス不可な環境にあり、共有ファイルシステムにはアクセス可能なコンピュータにとっては大きな恩恵となる可能性があります。
外部インターネットから分離されたシステムで Spack を使用する必要がある場合は、Spackミラーディレクトリ全体をバンドルし、分離されたシステムでバンドルを解除する必要があります。
そこから、外部インターネットにアクセスできないコンピュータで使用するのと同じ手順に従って、Spackミラーを使用します。

ミラーにソースを追加する必要がある場合は、ミラーの作成に使用したコマンドを再実行できます。
たとえば、bzip2を環境に追加する場合は、次のように実行します。

```bash
$ spack add bzip2
==> Adding bzip2 to environment /home/spack/cache-env
==> Updating view at /home/spack/cache-env/.spack-env/view
$ spack install
==> Concretized bzip2
 -   fvfpt26  bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
 -   otkkten	  ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4	      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Installing environment /home/spack/cache-env
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Installing diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr.spack
####################################################################################################################################################################### 100.0%
==> Extracting diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Installing bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8/linux-ubuntu18.04-x86_64-gcc-7.5.0-bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui.spack
####################################################################################################################################################################### 100.0%
==> Extracting bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Updating view at /home/spack/cache-env/.spack-env/view
==> Updating view at /home/spack/cache-env/.spack-env/view
```

ここで bzip2 を追加したので、次にミラーを更新する必要があります。

```bash
$ spack mirror create -d ~/mirror -a
==> Adding package autoconf@2.69 to mirror
==> Adding package automake@1.16.2 to mirror
==> Adding package berkeley-db@18.1.40 to mirror
==> Adding package bzip2@1.0.8 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/ab/ab5a03176ee106d3f0fa90e381da478ddae405918153cca248e682cd0c4a2269.tar.gz
####################################################################################################################################################################### 100.0%
==> Adding package cmake@3.18.4 to mirror
==> Adding package diffutils@3.7 to mirror
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b3/b3a7a6221c3dc916085f0d205abf6b8e1ba443d4dd965118da364a1dc1cb3a26.tar.xz
####################################################################################################################################################################### 100.0%
==> Adding package dtcmp@1.1.1 to mirror
==> Adding package gdbm@1.18.1 to mirror
==> Adding package hdf5@1.10.7 to mirror
==> Adding package hwloc@1.11.11 to mirror
==> Adding package json-cwx@0.12 to mirror
==> Adding package libiconv@1.16 to mirror
==> Adding package libpciaccess@0.16 to mirror
==> Adding package libsigsegv@2.12 to mirror
==> Adding package libtool@2.4.6 to mirror
==> Adding package libxml2@2.9.10 to mirror
==> Adding package libyogrt@1.24 to mirror
==> Adding package lwgrp@1.0.3 to mirror
==> Adding package m4@1.4.18 to mirror
==> Adding package macsio@1.1 to mirror
==> Adding package ncurses@6.2 to mirror
==> Adding package numactl@2.0.14 to mirror
==> Adding package openmpi@3.1.6 to mirror
==> Adding package openssl@1.1.1h to mirror
==> Adding package pdsh@2.31 to mirror
==> Adding package perl@5.32.0 to mirror
==> Adding package pkgconf@1.7.3 to mirror
==> Adding package readline@8.0 to mirror
==> Adding package scr@2.0.0 to mirror
==> Adding package silo@4.10.2 to mirror
==> Adding package util-macros@1.19.1 to mirror
==> Adding package xz@5.2.5 to mirror
==> Adding package zlib@1.2.11 to mirror
==> Successfully updated mirror in file:///home/spack/mirror
  Archive stats:
    31	 already present
    2	 added
    0	 failed to fetch.
```

Spackは、Spackミラーにすでに含まれているソースコードパッケージのアップロードをスキップします。
ミラーはさまざまな環境で共有できます。
つまり、1つのミラーに、チームの依存関係を構築するために必要なすべてのソースコードを格納できます。

## バイナリキャッシュミラーの設定

開発プラクティスの一部として Spack を使用するようにチームを設定する場合、Spackを使用することの最大の欠点に直面します。
Spackですべてのパッケージを最初から作成するのは時間がかかります。
大規模なプロジェクトのソフトウェア依存関係の再コンパイルは、完了するまでに数時間かかる場合があります。
すべての開発者が独自のソフトウェアスタックを再構築している場合、計算リソースが大量に浪費され、開発者の生産性が低下します。

Spackには、この問題を軽減するのに役立つ2つの方法があります。
1つめはチェーンされた Spack インスタンス、そして2つめは Spack バイナリキャッシュです。
ここではこの問題を解決する方法として Spack バイナリキャッシュについて解説します。

Spackバイナリキャッシュは、Spackバイナリパッケージで構成されています。
`*.spack` 拡張子で終わる各Spackバイナリパッケージは、gpg署名されたインストール済みの Spack パッケージのtarballです。
バイナリキャッシュを備えたミラーからパッケージをインストールする場合、Spack は

- ビルドする仕様のハッシュと完全に一致する Spack バイナリパッケージがあるかどうかを確認します。
- バイナリパッケージが見つかった場合、Spack は Spackバイナリパッケージの署名が信頼できるかどうかを確認します。署名が信頼されていないか、パッケージが見つからなかった場合、Spackはソースからパッケージをビルドします。
- 署名が信頼できる場合、Spack は Spack パッケージを解凍して再配置します。

ユーザにとって、Spack バイナリキャッシュは透過的に使用できます。
これは、バイナリパッケージとソースパッケージに別々のワークフローを選択する必要がある conda のようなシステムとは明らかに異なります。
実は既にチュートリアルの前半で、Spack バイナリミラーを使用するようにSpackを設定して使用する方法をすでに示していました。
そのときに実行したコマンドは次のとおりです：

```bash
$ spack mirror add tutorial /mirror
$ spack buildcache keys --install --trust
==> Fetching file:///mirror/build_cache/_pgp/95C717877AC00FFDAA8FD6E99CFA4A453B7C69B2.pub
 ###################################################################################################################################################################### 100.0%
gpg: keybox '/home/spack/spack/opt/spack/gpg/pubring.kbx' created
gpg: /home/spack/spack/opt/spack/gpg/trustdb.gpg: trustdb created
gpg: key 9CFA4A453B7C69B2: public key "sc-tutorial (GPG created for Spack) <becker33@llnl.gov>" imported
gpg: Total number processed: 1
gpg:		   imported: 1
```

Spackバイナリキャッシュの構築にはいくつかの落とし穴がありますが、ソースミラーの構築とほぼ同じくらい簡単です。
まず、自分たちのために新しい環境を作ることから始めましょう。
バイナリキャッシュに公開する予定なので、これらすべてのパッケージを自分でコンパイルする必要があります。
これには時間がかかる場合があるため、ここではすばやくコンパイルできるパッケージを使った新しい環境を作成します。

```bash
$ cd ~
$ mkdir cache-binary
$ cd cache-binary
$ spack env create -d .
==> Updating view at /home/spack/cache-binary/.spack-env/view
==> Created environment in /home/spack/cache-binary
==> You can activate this environment with:
==>   spack env activate /home/spack/cache-binary
$ spacktivate .
$ spack add bzip2
==> Adding bzip2 to environment /home/spack/cache-binary
==> Updating view at /home/spack/cache-binary/.spack-env/view
$ spack add zlib
==> Adding zlib to environment /home/spack/cache-binary
==> Updating view at /home/spack/cache-binary/.spack-env/view
```

ビルド前に Spack 構成ファイルの次の行を変更する必要があります。
その後で、念のためキャッシュを使用しないように強制して環境ビルドを開始します。

```bash
$ spack config add "config:install_tree:padded_length:128"
$ spack install --no-cache
==> Concretized bzip2
 -   fvfpt26  bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
 -   otkkten	  ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   jearpk4	      ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized zlib
 -   smoyzzo  zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Installing environment /home/spack/cache-binary
==> Installing libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/e6/e6a1b1b589654277ee790cce3734f07876ac4ccfaecbee8afa0b649cf529cc04.tar.gz
####################################################################################################################################################################### 100.0%
==> libiconv: Executing phase: 'autoreconf'
==> libiconv: Executing phase: 'configure'
==> libiconv: Executing phase: 'build'
==> libiconv: Executing phase: 'install'
==> Warning: Module file already exists : skipping creation
file : /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/libiconv-1.16-gcc-7.5.0-jearpk4
spec : libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+] /home/spack/spack/opt/spack/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_pa/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Installing zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/c3/c3e5e9fdd5004dcb542feda5ee4f0ff0744628baf8ed2dd5d66f8ca1197cb1a1.tar.gz
####################################################################################################################################################################### 100.0%
==> zlib: Executing phase: 'install'
==> Warning: Module file already exists : skipping creation
file : /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/zlib-1.2.11-gcc-7.5.0-smoyzzo
spec : zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
[+] /home/spack/spack/opt/spack/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_pa/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Installing diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b3/b3a7a6221c3dc916085f0d205abf6b8e1ba443d4dd965118da364a1dc1cb3a26.tar.xz
####################################################################################################################################################################### 100.0%
==> diffutils: Executing phase: 'autoreconf'
==> diffutils: Executing phase: 'configure'
==> diffutils: Executing phase: 'build'
==> diffutils: Executing phase: 'install'
==> Warning: Module file already exists : skipping creation
file : /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/diffutils-3.7-gcc-7.5.0-otkkten
spec : diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64 ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+] /home/spack/spack/opt/spack/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_pa/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Installing bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/ab/ab5a03176ee106d3f0fa90e381da478ddae405918153cca248e682cd0c4a2269.tar.gz
####################################################################################################################################################################### 100.0%
==> bzip2: Executing phase: 'install'
==> Warning: Module file already exists : skipping creation
file : /home/spack/spack/share/spack/modules/linux-ubuntu18.04-x86_64/bzip2-1.0.8-gcc-7.5.0-fvfpt26
spec : bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64 ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64 ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+] /home/spack/spack/opt/spack/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_path_placeholder/spack_pa/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Updating view at /home/spack/cache-binary/.spack-env/view
==> Updating view at /home/spack/cache-binary/.spack-env/view
```

この構成変更により、Spackはすべてのパッケージを128文字以上のパスにインストールします。
Spack によるパッケージ再配置する手段のために、この変更が必要としています。
再配置は、次の手順で構成されます。

- すべてのテキストファイルを検索して、パッケージビルダーのパスを特定のローカルインストールパスに置き換えます。
- `patchelf` を使って、バイナリ内のすべてのRPATHを特定のローカルインストールパスを指すように置き換えます。
- すべてのバイナリを検索して、パッケージビルダーのパスのハードコードされた C 文字列を特定のローカルインストールパスに置き換えます。

パディングを追加すると、バイナリで C 文字列としてハードコーディングされたパスが、ユーザの最終的なインストールパスを保持するのに十分な大きさになります。
文字列が長いと、一部のソフトウェアパッケージでコンパイルの問題が発生することがあるため、128を選択することを推奨します。
ユーザのインストールパスが長すぎる場合、Spackは警告を表示します。
すべてのスクリプトとRPATHは引き続き適切に再配置されますが、バイナリ内のC文字列は変更されません。
パッケージによっては、ソフトウェアを試して使用するときに問題が発生する場合があります。

また、すべてのパッケージに署名するためにgpgキーを作成する必要があります。
秘密鍵と公開鍵は、将来再利用できるように安全な場所にバックアップする必要があります。

```bash
$ spack gpg create "My Name" "<my.email@my.domain.com>"
gpg: key 070F30B4CDF253A9 marked as ultimately trusted
gpg: directory '/home/spack/spack/opt/spack/gpg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/spack/spack/opt/spack/gpg/openpgp-revocs.d/ABC74A4103675F76561A607E070F30B4CDF253A9.rev'
$ mkdir ~/private_gpg_backup
$ cp ~/spack/opt/spack/gpg/*.gpg ~/private_gpg_backup
$ cp ~/spack/opt/spack/gpg/pubring.* ~/mirror
```

この設定が完了すると、バイナリキャッシュをバイナリパッケージで埋める準備が整います。
バイナリパッケージは、既存のソースミラーにアタッチされます。
ソースコードキャッシュミラーと同じ手順➖環境の作成、ソースミラーの作成、およびパディング済みパスを使用したSpackインストールへのパッケージのビルド➖に従います。
仕様をバイナリキャッシュにアップロードするには、`spack buildcache create --only=package spec` コマンドを使います。
ここでは、 for ループで使って環境内のすべての非外部パッケージのバイナリパッケージを作成します。
ループにてこの環境の各仕様のハッシュを返す `spack find` コマンドを記述し、外部パッケージを除外したものに対して `spack buildcache create` コマンドを実行します。

```bash
$
for ii in $(spack find --format "yyy {version} /{hash}" |
	    grep -v -E "^(develop^master)" |
	    grep "yyy" |
	    cut -f3 -d" ")
do
  spack buildcache create -af -d ~/mirror --only=package $ii
done
==> Buildcache files will be output to file:///home/spack/mirror/build_cache
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:	1  signed:   0	trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: using "ABC74A4103675F76561A607E070F30B4CDF253A9" as default secret key for signing
==> Buildcache files will be output to file:///home/spack/mirror/build_cache
gpg: using "ABC74A4103675F76561A607E070F30B4CDF253A9" as default secret key for signing
==> Buildcache files will be output to file:///home/spack/mirror/build_cache
gpg: using "ABC74A4103675F76561A607E070F30B4CDF253A9" as default secret key for signing
==> Buildcache files will be output to file:///home/spack/mirror/build_cache
gpg: using "ABC74A4103675F76561A607E070F30B4CDF253A9" as default secret key for signing
```

出来上がりです！
これで Spack ミラーはバイナリキャッシュで拡張されました。
このキャッシュは、Spackソースミラーと同様、外部インターネットアクセスのないシステムで使用できます。
いつものように、ミラーを更新した後、ファイルのアクセス許可を更新することを忘れないでください。

```bash
$ umask 027
$ chmod -R g+rs ~/mirror
$ chgrp -R spack ~/mirror
```

今回のチュートリアルの範囲からは外れますが、Spack ミラーおよびビルドキャッシュも同様に  `https://` や `s3://` にホスト可能です。
方法の詳細については、Spack ドキュメントを参照してください。

ユーザがこのバイナリキャッシュを使用する前に、バイナリキャッシュにリストされているすべてのパッケージを信頼していることを確認する必要があります。
ファイルシステム上の信頼できるユーザ間でファイルを共有している場合は、次のコマンドを使用してこれを行うことができます：

```bash
$ spack buildcache keys --install --trust --force
==> Fetching file:///mirror/build_cache/_pgp/95C717877AC00FFDAA8FD6E99CFA4A453B7C69B2.pub
####################################################################################################################################################################### 100.0%
gpg: key 9CFA4A453B7C69B2: "sc-tutorial (GPG created for Spack) <becker33@llnl.gov>" not changed
gpg: Total number processed: 1
gpg:		  unchanged: 1
```

これはバイナリキャッシュ上のすべてのキーをダウンロードしてそれらを信頼することを意味します。
ビルド開始前に、ユーザに新しいSpackインスタンスで上記のコマンドを実行してもらいます。

## キャッシュの概念

開発チーム内でspackを使用している場合は、バイナリキャッシュを使用してソースミラーを設定することを検討してください。
ソースミラーを使用すると、外部インターネットアクセスのないマシンでSpack環境を複製できます。
バイナリミラーを使用すると、すべてを最初から再コンパイルする負担がなくなり、開発時間を節約できます。
