# スタックチュートリアル

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

これまで、統合ユーザ環境または開発環境の文脈での Spack 環境について解説してきました。
ですが Spackの環境にはまだまだはるかに幅広い機能があります。
このチュートリアルでは、Spackを使ったソフトウェアの大規模展開を管理するために、Spackスタックと呼ばれる特殊な種類のSpack環境を使用する方法を検討します。

## スペックマトリクス

シングルユーザ向けの一般的なSpack環境では、仕様の簡単なリストで十分です。
ただし、ソフトウェアの展開では、さまざまなコンパイラにインストールしたいパッケージのセットがあることがよくあります。
これをSpackで表現する最も簡単な方法は、マトリクスを使用することです。
環境チュートリアルからコードディレクトリに戻り、環境を有効化して、`spack.yaml` ファイルを再度編集しましょう。

```bash
$ cd ~/code
$ spack env activate .
$ spack config edit
```

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # add package specs to the `specs` list
  specs:
    - matrix:
        - [boost, trilinos, openmpi]
        - ['%gcc', '%clang']

  view: False
```

ここでは `view` 記述子の解説は避けます（後述）。

インストール時間が長くなってしまうため、時間の都合上、このセクションの残りの部分の具体的な仕様をコンクリート（具現）化して見ていきます。

```bash
$ spack concretize
==> Concretized boost%gcc
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%clang
 -   ovhagtq  boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	  ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef	      ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc
 -   qmjbxf2  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
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
 -   ejanivx	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
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
 -   nndket2	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   zoqjxgk	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
 -   3jmjrzw	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   wsiydak	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   yixejly	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   htdsyvt	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%clang
 -   kedcznr  trilinos@13.0.1%clang@6.0.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
 -   ovhagtq	  ^boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	      ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef		  ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		      ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	      ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   33q7we3	  ^cmake@3.18.4%clang@6.0.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
 -   muufhm7	      ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   p7qety4	      ^openssl@1.1.1h%clang@6.0.0+systemcerts arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		  ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej		      ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx		      ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			  ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   a3zhhmy	  ^glm@0.9.7.1%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   qg3kytq	  ^hdf5@1.10.7%clang@6.0.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   tymzjs6	      ^openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs		  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz		      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn			  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave			      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k				  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw			  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn		      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   suaulg4			  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k		      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha			  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   piicbth			  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   te2k44y	  ^hypre@2.20.0%clang@6.0.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
 -   sirdvzb	      ^openblas@0.3.12%clang@6.0.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   w7of7jn	  ^matio@1.5.17%clang@6.0.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
 -   5dhwwts	  ^metis@5.1.0%clang@6.0.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1 arch=linux-ubuntu18.04-x86_64
 -   bicglpz	  ^mumps@5.3.3%clang@6.0.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   grhzh45	      ^netlib-scalapack@2.1.0%clang@6.0.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   yyboddy	  ^netcdf-c@4.7.4%clang@6.0.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   u4myogl	  ^parmetis@4.0.3%clang@6.0.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
 -   36yyib7	  ^suite-sparse@5.8.1%clang@6.0.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
 -   usfeiki	      ^gmp@6.1.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   g4csges	      ^mpfr@4.0.2%clang@6.0.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
 -   te346cx		  ^autoconf-archive@2019.01.06%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc
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

==> Concretized openmpi%clang
 -   tymzjs6  openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs	  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz	      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn	      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   suaulg4		  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
[+]  5qffmms		  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k	      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		      ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej			  ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx			  ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			      ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   muufhm7				  ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

$ spack find -c
==> In environment /home/spack/code
==> Root specs
-- no arch / clang ----------------------------------------------
boost%clang   openmpi%clang   trilinos%clang

-- no arch / gcc ------------------------------------------------
boost%gcc   openmpi%gcc   trilinos%gcc

==> Concretized roots
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1

==> 33 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     diffutils@3.7	libsigsegv@2.12  numactl@2.0.14   suite-sparse@5.8.1
autoconf-archive@2019.01.06  gdbm@1.18.1	libtool@2.4.6	 openblas@0.3.12  util-macros@1.19.1
automake@1.16.2 	     glm@0.9.7.1	libxml2@2.9.10	 openmpi@3.1.6	  xz@5.2.5
berkeley-db@18.1.40	     gmp@6.1.2		m4@1.4.18	 openssl@1.1.1h   zlib@1.2.11
boost@1.74.0		     hwloc@1.11.11	metis@5.1.0	 perl@5.32.0
bzip2@1.0.8		     libiconv@1.16	mpfr@4.0.2	 pkgconf@1.7.3
cmake@3.18.4		     libpciaccess@0.16	ncurses@6.2	 readline@8.0
```

行列演算は、見た通りに実行されます。
任意の数のリストの仕様制約を取り、それらの内積をとります。

ここでは `boost`、`trilinos` そして `openmpi` が `gcc` 、 `clang` 両方でコンパイルしようとしています。
コンパイラの制約は、コマンドラインの場合と同様に、先頭に `%` 記号が付いていることに注意してください。

行列の制約がどのように解決されるかについて注意すべき特別なことがいくつかあります。
依存関係とバリアントは、マトリックス内のすべてのパッケージに適用されるかどうかに関係なく、マトリックスで使用できます。
ファイルをもう一度編集しましょう。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # add package specs to the `specs` list
  specs:
    - matrix:
        - [boost, trilinos, openmpi]
        - [^mpich, ^mvapich2 fabrics=mrail]
        - ['%gcc', '%clang']

  view: False
```

ここでわかるのは、Spackが `mpi` に依存する `boost` と `trilinos` に `mpi` 制約を適用し、`openmpi` には適用しないことです。

```bash
$ spack concretize -f
==> Concretized boost%gcc ^mpich
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%clang ^mpich
 -   ovhagtq  boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	  ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef	      ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%gcc ^mvapich2 fabrics=mrail
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%clang ^mvapich2 fabrics=mrail
 -   ovhagtq  boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	  ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef	      ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mpich
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

==> Concretized trilinos%clang ^mpich
 -   es62tvb  trilinos@13.0.1%clang@6.0.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
 -   ovhagtq	  ^boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	      ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef		  ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		      ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	      ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   33q7we3	  ^cmake@3.18.4%clang@6.0.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
 -   muufhm7	      ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   p7qety4	      ^openssl@1.1.1h%clang@6.0.0+systemcerts arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		  ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej		      ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx		      ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			  ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   a3zhhmy	  ^glm@0.9.7.1%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   aof2vxe	  ^hdf5@1.10.7%clang@6.0.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   6nla6qq	      ^mpich@3.3.2%clang@6.0.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   jsefksx		  ^findutils@4.6.0%clang@6.0.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		      ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yqoblty		      ^texinfo@6.5%clang@6.0.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
 -   qfjizht		  ^hwloc@2.2.0%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz		      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw			  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn		      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   suaulg4			  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
 -   qltkggq	  ^hypre@2.20.0%clang@6.0.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
 -   sirdvzb	      ^openblas@0.3.12%clang@6.0.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   aneq55b	  ^matio@1.5.17%clang@6.0.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
 -   5dhwwts	  ^metis@5.1.0%clang@6.0.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1 arch=linux-ubuntu18.04-x86_64
 -   6ce2pqx	  ^mumps@5.3.3%clang@6.0.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   25hw4j3	      ^netlib-scalapack@2.1.0%clang@6.0.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   dha5nd3	  ^netcdf-c@4.7.4%clang@6.0.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   koc2v2s	  ^parmetis@4.0.3%clang@6.0.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
 -   36yyib7	  ^suite-sparse@5.8.1%clang@6.0.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
 -   usfeiki	      ^gmp@6.1.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   g4csges	      ^mpfr@4.0.2%clang@6.0.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
 -   te346cx		  ^autoconf-archive@2019.01.06%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mvapich2 fabrics=mrail
 -   a4pjwfb  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
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
 -   rqf77du	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   7uoyvsg	      ^mvapich2@2.3.4%gcc@7.5.0~alloca~cuda~debug+regcache+wrapperrpath ch3_rank_bits=32 fabrics=mrail file_systems=auto process_managers=auto threads=multiple arch=linux-ubuntu18.04-x86_64
 -   sdjqt2l		  ^bison@3.6.4%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   pqlmjnb		      ^help2man@1.47.11%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to			  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf			      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm				  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5			      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5		  ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		      ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   2lpslud		  ^rdma-core@32.0%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   ttz66su		      ^libnl@3.3.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   jvt3u4j			  ^flex@2.6.4%gcc@7.5.0+lex patches=09c22e5c6fef327d3e48eb23f0d610dcd3a35ab9207f12e0f875701c677978d3 arch=linux-ubuntu18.04-x86_64
 -   5meza7i		      ^py-docutils@0.15.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   tkhtoma			  ^py-setuptools@50.3.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   szj7juk			      ^python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj				  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw				      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm				  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax				  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b				  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64
 -   sj7igc7	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   kzfn5yl	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
 -   mhfry35	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   3wghjiv	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   wisoa4e	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   3wqgvq6	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%clang ^mvapich2 fabrics=mrail
 -   adggdh4  trilinos@13.0.1%clang@6.0.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
 -   ovhagtq	  ^boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	      ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef		  ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		      ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	      ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   33q7we3	  ^cmake@3.18.4%clang@6.0.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
 -   muufhm7	      ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   p7qety4	      ^openssl@1.1.1h%clang@6.0.0+systemcerts arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		  ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej		      ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx		      ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			  ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   a3zhhmy	  ^glm@0.9.7.1%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   eqbcm4i	  ^hdf5@1.10.7%clang@6.0.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   zayixj6	      ^mvapich2@2.3.4%clang@6.0.0~alloca~cuda~debug+regcache+wrapperrpath ch3_rank_bits=32 fabrics=mrail file_systems=auto process_managers=auto threads=multiple arch=linux-ubuntu18.04-x86_64
 -   43thy45		  ^bison@3.6.4%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   hor2tom		      ^help2man@1.47.11%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   waczh35			  ^gettext@0.21%clang@6.0.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
 -   7p673kn			      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   suaulg4				  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
 -   vuc6wgn			      ^tar@1.32%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   jsefksx		  ^findutils@4.6.0%clang@6.0.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		      ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   piicbth		      ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		      ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yqoblty		      ^texinfo@6.5%clang@6.0.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
 -   nxnlozz		  ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		      ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7bxrpk6		  ^rdma-core@32.0%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   fo7zfqe		      ^libnl@3.3.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   dcwuy4i			  ^flex@2.6.4%clang@6.0.0+lex patches=09c22e5c6fef327d3e48eb23f0d610dcd3a35ab9207f12e0f875701c677978d3 arch=linux-ubuntu18.04-x86_64
 -   bh4u6dn		      ^py-docutils@0.15.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   cozuksz			  ^py-setuptools@50.3.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   flgp2hq			      ^python@3.8.6%clang@6.0.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
 -   cp6c3aa				  ^expat@2.2.10%clang@6.0.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   tm3yohm				      ^libbsd@0.10.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7np5gss				  ^libffi@3.3%clang@6.0.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   bx3okm5				  ^libuuid@1.0.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ofshwkz				  ^sqlite@3.33.0%clang@6.0.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64
 -   egng77n	  ^hypre@2.20.0%clang@6.0.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
 -   sirdvzb	      ^openblas@0.3.12%clang@6.0.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   3tz5rh4	  ^matio@1.5.17%clang@6.0.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
 -   5dhwwts	  ^metis@5.1.0%clang@6.0.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1 arch=linux-ubuntu18.04-x86_64
 -   ukaisd3	  ^mumps@5.3.3%clang@6.0.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   mxbdwkl	      ^netlib-scalapack@2.1.0%clang@6.0.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   bmwq2yy	  ^netcdf-c@4.7.4%clang@6.0.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   6io6o4v	  ^parmetis@4.0.3%clang@6.0.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
 -   36yyib7	  ^suite-sparse@5.8.1%clang@6.0.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
 -   usfeiki	      ^gmp@6.1.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   g4csges	      ^mpfr@4.0.2%clang@6.0.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
 -   te346cx		  ^autoconf-archive@2019.01.06%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mpich
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

==> Concretized openmpi%clang ^mpich
 -   tymzjs6  openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs	  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz	      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn	      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   suaulg4		  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
[+]  5qffmms		  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k	      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		      ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej			  ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx			  ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			      ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   muufhm7				  ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mvapich2 fabrics=mrail
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

==> Concretized openmpi%clang ^mvapich2 fabrics=mrail
 -   tymzjs6  openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs	  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz	      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn	      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   suaulg4		  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
[+]  5qffmms		  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k	      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		      ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej			  ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx			  ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			      ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   muufhm7				  ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

$ spack find -c
==> In environment /home/spack/code
==> Root specs
-- no arch / clang ----------------------------------------------
boost%clang   boost%clang   openmpi%clang   openmpi%clang   trilinos%clang   trilinos%clang

-- no arch / gcc ------------------------------------------------
boost%gcc   boost%gcc	openmpi%gcc   openmpi%gcc   trilinos%gcc   trilinos%gcc

==> Concretized roots
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1  trilinos@13.0.1

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1  trilinos@13.0.1

==> 45 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     gmp@6.1.2		matio@1.5.17		openssl@1.1.1h
autoconf-archive@2019.01.06  hdf5@1.10.7	metis@5.1.0		parmetis@4.0.3
automake@1.16.2 	     hwloc@1.11.11	mpfr@4.0.2		perl@5.32.0
berkeley-db@18.1.40	     hwloc@2.2.0	mpich@3.3.2		pkgconf@1.7.3
boost@1.74.0		     hypre@2.20.0	mumps@5.3.3		readline@8.0
bzip2@1.0.8		     libiconv@1.16	ncurses@6.2		suite-sparse@5.8.1
cmake@3.18.4		     libpciaccess@0.16	netcdf-c@4.7.4		texinfo@6.5
diffutils@3.7		     libsigsegv@2.12	netlib-scalapack@2.1.0	trilinos@13.0.1
findutils@4.6.0 	     libtool@2.4.6	numactl@2.0.14		util-macros@1.19.1
gdbm@1.18.1		     libxml2@2.9.10	openblas@0.3.12 	xz@5.2.5
glm@0.9.7.1		     m4@1.4.18		openmpi@3.1.6		zlib@1.2.11
```

これにより、より一般的な方法で行列を作成することができます。

行列からいくつかの値を除外することも可能です。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # add package specs to the `specs` list
  specs:
    - matrix:
        - [boost, trilinos, openmpi]
        - [^mpich, ^mvapich2 fabrics=mrail]
        - ['%gcc', '%clang']
      exclude:
        - '%clang ^mvapich2'

  view: False
```

これにより、`mvapich2` に依存する `clang` コンパイラで構築されたすべての仕様が除外されます。
`trilions` の3つの構成が表示が表示されます。

```bash
$ spack concretize -f
==> Concretized boost%gcc ^mpich
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%clang ^mpich
 -   ovhagtq  boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	  ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef	      ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%gcc ^mvapich2 fabrics=mrail
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mpich
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

==> Concretized trilinos%clang ^mpich
 -   es62tvb  trilinos@13.0.1%clang@6.0.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
 -   ovhagtq	  ^boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	      ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef		  ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		      ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	      ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   33q7we3	  ^cmake@3.18.4%clang@6.0.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
 -   muufhm7	      ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   p7qety4	      ^openssl@1.1.1h%clang@6.0.0+systemcerts arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		  ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej		      ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx		      ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			  ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   a3zhhmy	  ^glm@0.9.7.1%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   aof2vxe	  ^hdf5@1.10.7%clang@6.0.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   6nla6qq	      ^mpich@3.3.2%clang@6.0.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   jsefksx		  ^findutils@4.6.0%clang@6.0.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		      ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yqoblty		      ^texinfo@6.5%clang@6.0.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
 -   qfjizht		  ^hwloc@2.2.0%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz		      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw			  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn		      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   suaulg4			  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
 -   qltkggq	  ^hypre@2.20.0%clang@6.0.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
 -   sirdvzb	      ^openblas@0.3.12%clang@6.0.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   aneq55b	  ^matio@1.5.17%clang@6.0.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
 -   5dhwwts	  ^metis@5.1.0%clang@6.0.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1 arch=linux-ubuntu18.04-x86_64
 -   6ce2pqx	  ^mumps@5.3.3%clang@6.0.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   25hw4j3	      ^netlib-scalapack@2.1.0%clang@6.0.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   dha5nd3	  ^netcdf-c@4.7.4%clang@6.0.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   koc2v2s	  ^parmetis@4.0.3%clang@6.0.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
 -   36yyib7	  ^suite-sparse@5.8.1%clang@6.0.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
 -   usfeiki	      ^gmp@6.1.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   g4csges	      ^mpfr@4.0.2%clang@6.0.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
 -   te346cx		  ^autoconf-archive@2019.01.06%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mvapich2 fabrics=mrail
 -   a4pjwfb  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
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
 -   rqf77du	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   7uoyvsg	      ^mvapich2@2.3.4%gcc@7.5.0~alloca~cuda~debug+regcache+wrapperrpath ch3_rank_bits=32 fabrics=mrail file_systems=auto process_managers=auto threads=multiple arch=linux-ubuntu18.04-x86_64
 -   sdjqt2l		  ^bison@3.6.4%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   pqlmjnb		      ^help2man@1.47.11%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to			  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf			      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm				  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5			      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5		  ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		      ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   2lpslud		  ^rdma-core@32.0%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   ttz66su		      ^libnl@3.3.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   jvt3u4j			  ^flex@2.6.4%gcc@7.5.0+lex patches=09c22e5c6fef327d3e48eb23f0d610dcd3a35ab9207f12e0f875701c677978d3 arch=linux-ubuntu18.04-x86_64
 -   5meza7i		      ^py-docutils@0.15.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   tkhtoma			  ^py-setuptools@50.3.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   szj7juk			      ^python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj				  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw				      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm				  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax				  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b				  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64
 -   sj7igc7	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   kzfn5yl	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
 -   mhfry35	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   3wghjiv	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   wisoa4e	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   3wqgvq6	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mpich
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

==> Concretized openmpi%clang ^mpich
 -   tymzjs6  openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs	  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz	      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn	      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   suaulg4		  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
[+]  5qffmms		  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k	      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		      ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej			  ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx			  ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			      ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   muufhm7				  ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mvapich2 fabrics=mrail
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

$ spack find -c
==> In environment /home/spack/code
==> Root specs
-- no arch / clang ----------------------------------------------
boost%clang   openmpi%clang   trilinos%clang

-- no arch / gcc ------------------------------------------------
boost%gcc   boost%gcc	openmpi%gcc   openmpi%gcc   trilinos%gcc   trilinos%gcc

==> Concretized roots
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1  trilinos@13.0.1

==> 45 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     gmp@6.1.2		matio@1.5.17		openssl@1.1.1h
autoconf-archive@2019.01.06  hdf5@1.10.7	metis@5.1.0		parmetis@4.0.3
automake@1.16.2 	     hwloc@1.11.11	mpfr@4.0.2		perl@5.32.0
berkeley-db@18.1.40	     hwloc@2.2.0	mpich@3.3.2		pkgconf@1.7.3
boost@1.74.0		     hypre@2.20.0	mumps@5.3.3		readline@8.0
bzip2@1.0.8		     libiconv@1.16	ncurses@6.2		suite-sparse@5.8.1
cmake@3.18.4		     libpciaccess@0.16	netcdf-c@4.7.4		texinfo@6.5
diffutils@3.7		     libsigsegv@2.12	netlib-scalapack@2.1.0	trilinos@13.0.1
findutils@4.6.0 	     libtool@2.4.6	numactl@2.0.14		util-macros@1.19.1
gdbm@1.18.1		     libxml2@2.9.10	openblas@0.3.12 	xz@5.2.5
glm@0.9.7.1		     m4@1.4.18		openmpi@3.1.6		zlib@1.2.11
```

## Spack 環境での名前付きリスト

Spackでは、環境内の名前付きリストも使用できます。
これらのリストを使って、上記の例をクリーンアップできます。
これらの名前付きリストは、`spack.yaml`ファイルの `definitions` キーで定義されています。
今回のリストは、パッケージまたは制約の単純なリストになりますが、より複雑な例では、名前付きリストに行列を含めることもできます。

ファイルを少しクリーンアップしましょう。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # named lists
  definitions:
    - packages: [boost, trilinos, openmpi]
    - mpis: [mpich, mvapich2 fabrics=mrail]
    - compilers: ['%gcc', '%clang']

  specs:
    - matrix:
        - [$packages]
        - [$^mpis]
        - [$compilers]
      exclude:
        - '%clang ^mvapich2'

  view: false
```

この構文には、慣れるまでに時間がかかる場合があります。
具体的には、行列と名前付きリストへの参照は、yamlのリストオブジェクトとして含まれるのではなく、常に現在の位置に「グチャッ(splatted)」となります。
これは直感に反するように思えるかもしれませんが、リストを組み合わせる場合に重要になります。
`mpi` 制約をパッケージとして宣言し、`$^` 構文を使って依存関係として適用できることに注意してください。
同じことがコンパイラ（`$%`を使用）にも当てはまるので、ここでは両方の構文を示しています。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # named lists
  definitions:
    - packages: [boost, trilinos, openmpi]
    - mpis: [mpich, mvapich2 fabrics=mrail]
    - compilers: ['%gcc', '%clang']
    - singleton_packages: [python, tcl]

  specs:
    - matrix:
        - [$packages]
        - [$^mpis]
        - [$compilers]
      exclude:
        - '%clang ^mvapich2'
    - $singleton_packages

  view: false
```

この例における `specs` リストは、まだ仕様のリスト(環境として必須)です。

このスタックは前の例と同じですが、pythonとtclの単一構成が追加されています。

## 条件付き定義

スペックリストの定義は、`when` 条項を条件とすることもできます。
この `when` 句は、制限された環境で評価されるPython条件文です。
`when` 句で使用できる変数は次のとおりです：

|変数名|値|
|+-----|+-|
|`platform`| このマシンのSpack プラットフォーム名 |
|`os`| このマシンのデフォルトSpack OS 名およびバージョン |
|`target`| このマシンのデフォルトSpackターゲット文字列 |
|`architecture`| このマシンのデフォルトSpackアーキテクチャ文字列 platform-os-target |
|`arch`| `architecture` のエイリアス |
|`env`| ユーザ環境変数辞書 |
|`re`| 正規表現のための　Python `re` モジュール |
|`hostname`| このノードのホスト名 |

`SPACK_STACK_USE_CLANG` 環境変数が設定されている場合にのみclangを使用するように、 `spack.yaml` ファイルを編集することとします。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # named lists
  definitions:
    - packages: [boost, trilinos, openmpi]
    - mpis: [mpich, mvapich2 fabrics=mrail]
    - compilers: ['%gcc']
    - compilers: ['%clang']
      when: 'env.get("SPACK_STACK_USE_CLANG", "") == "1"'
    - singleton_packages: [python, tcl]

  specs:
    - matrix:
        - [$packages]
        - [$^mpis]
        - [$compilers]
      exclude:
        - '%clang ^mvapich2'
    - $singleton_packages

  view: false
```

Spackスタック内の名前付きリストは連結されていることに注意してください。
コンパイラリストを無条件に1か所で定義し、環境変数が適切に設定されている場合は条件付きでコンパイラリストに `clang` を追加できます。

```bash
$ spack concretize -f
==> Concretized boost%gcc ^mpich
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%gcc ^mvapich2 fabrics=mrail
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mpich
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

==> Concretized trilinos%gcc ^mvapich2 fabrics=mrail
 -   a4pjwfb  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
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
 -   rqf77du	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   7uoyvsg	      ^mvapich2@2.3.4%gcc@7.5.0~alloca~cuda~debug+regcache+wrapperrpath ch3_rank_bits=32 fabrics=mrail file_systems=auto process_managers=auto threads=multiple arch=linux-ubuntu18.04-x86_64
 -   sdjqt2l		  ^bison@3.6.4%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   pqlmjnb		      ^help2man@1.47.11%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to			  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf			      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm				  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5			      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5		  ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		      ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   2lpslud		  ^rdma-core@32.0%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   ttz66su		      ^libnl@3.3.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   jvt3u4j			  ^flex@2.6.4%gcc@7.5.0+lex patches=09c22e5c6fef327d3e48eb23f0d610dcd3a35ab9207f12e0f875701c677978d3 arch=linux-ubuntu18.04-x86_64
 -   5meza7i		      ^py-docutils@0.15.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   tkhtoma			  ^py-setuptools@50.3.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   szj7juk			      ^python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj				  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw				      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm				  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax				  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b				  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64
 -   sj7igc7	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   kzfn5yl	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
 -   mhfry35	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   3wghjiv	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   wisoa4e	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   3wqgvq6	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mpich
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

==> Concretized openmpi%gcc ^mvapich2 fabrics=mrail
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

==> Concretized python
 -   szj7juk  python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj	  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw	      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw	  ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy	      ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo		  ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		      ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to	  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf	      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm		  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5	      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm	  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax	  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  es377uq	  ^openssl@1.1.1h%gcc@7.5.0+systemcerts arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j	      ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		  ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b	  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64

==> Concretized tcl
 -   rtz7664  tcl@8.6.10%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

$ spack find -c
==> In environment /home/spack/code
==> Root specs
python	 tcl

-- no arch / gcc ------------------------------------------------
boost%gcc   boost%gcc	openmpi%gcc   openmpi%gcc   trilinos%gcc   trilinos%gcc

==> Concretized roots
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
boost@1.74.0  openmpi@3.1.6  python@3.8.6  tcl@8.6.10  trilinos@13.0.1	trilinos@13.0.1

==> 44 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     gmp@6.1.2		matio@1.5.17		openssl@1.1.1h
autoconf-archive@2019.01.06  hdf5@1.10.7	metis@5.1.0		parmetis@4.0.3
automake@1.16.2 	     hwloc@1.11.11	mpfr@4.0.2		perl@5.32.0
berkeley-db@18.1.40	     hwloc@2.2.0	mpich@3.3.2		pkgconf@1.7.3
boost@1.74.0		     hypre@2.20.0	mumps@5.3.3		readline@8.0
bzip2@1.0.8		     libiconv@1.16	ncurses@6.2		suite-sparse@5.8.1
cmake@3.18.4		     libpciaccess@0.16	netcdf-c@4.7.4		texinfo@6.5
diffutils@3.7		     libsigsegv@2.12	netlib-scalapack@2.1.0	trilinos@13.0.1
findutils@4.6.0 	     libtool@2.4.6	numactl@2.0.14		util-macros@1.19.1
gdbm@1.18.1		     libxml2@2.9.10	openblas@0.3.12 	xz@5.2.5
glm@0.9.7.1		     m4@1.4.18		openmpi@3.1.6		zlib@1.2.11
$ export SPACK_STACK_USE_CLANG=1
$ spack concretize -f
==> Concretized boost%gcc ^mpich
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%clang ^mpich
 -   ovhagtq  boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	  ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef	      ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized boost%gcc ^mvapich2 fabrics=mrail
[+]  hc5atqc  boost@1.74.0%gcc@7.5.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mpich
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

==> Concretized trilinos%clang ^mpich
 -   es62tvb  trilinos@13.0.1%clang@6.0.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
 -   ovhagtq	  ^boost@1.74.0%clang@6.0.0+atomic+chrono~clanglibcpp~container~context~coroutine+date_time~debug+exception~fiber+filesystem+graph~icu+iostreams+locale+log+math~mpi+multithreaded~numpy~pic+program_options~python+random+regex+serialization+shared+signals~singlethreaded+system~taggedlayout+test+thread+timer~versionedlayout+wave cxxstd=98 visibility=hidden arch=linux-ubuntu18.04-x86_64
 -   qxfbjyc	      ^bzip2@1.0.8%clang@6.0.0+shared arch=linux-ubuntu18.04-x86_64
 -   sdvt7ef		  ^diffutils@3.7%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		      ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
[+]  5qffmms	      ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   33q7we3	  ^cmake@3.18.4%clang@6.0.0~doc+ncurses+openssl+ownlibs~qt patches=bf695e3febb222da2ed94b3beea600650e4318975da90e4a71d6f31a6d5d8c3d arch=linux-ubuntu18.04-x86_64
 -   muufhm7	      ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   p7qety4	      ^openssl@1.1.1h%clang@6.0.0+systemcerts arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		  ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej		      ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx		      ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			  ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   a3zhhmy	  ^glm@0.9.7.1%clang@6.0.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   aof2vxe	  ^hdf5@1.10.7%clang@6.0.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   6nla6qq	      ^mpich@3.3.2%clang@6.0.0~argobots+fortran+hwloc+hydra+libxml2+pci+romio~slurm~verbs+wrapperrpath device=ch3 netmod=tcp patches=eb982de3366d48cbc55eb5e0df43373a45d9f51df208abf0835a72dc6c0b4774 pmi=pmi arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   jsefksx		  ^findutils@4.6.0%clang@6.0.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		      ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yqoblty		      ^texinfo@6.5%clang@6.0.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
 -   qfjizht		  ^hwloc@2.2.0%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz		      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw			  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn		      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   suaulg4			  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
 -   qltkggq	  ^hypre@2.20.0%clang@6.0.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
 -   sirdvzb	      ^openblas@0.3.12%clang@6.0.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   aneq55b	  ^matio@1.5.17%clang@6.0.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
 -   5dhwwts	  ^metis@5.1.0%clang@6.0.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1 arch=linux-ubuntu18.04-x86_64
 -   6ce2pqx	  ^mumps@5.3.3%clang@6.0.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   25hw4j3	      ^netlib-scalapack@2.1.0%clang@6.0.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   dha5nd3	  ^netcdf-c@4.7.4%clang@6.0.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   koc2v2s	  ^parmetis@4.0.3%clang@6.0.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
 -   36yyib7	  ^suite-sparse@5.8.1%clang@6.0.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
 -   usfeiki	      ^gmp@6.1.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   g4csges	      ^mpfr@4.0.2%clang@6.0.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
 -   te346cx		  ^autoconf-archive@2019.01.06%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized trilinos%gcc ^mvapich2 fabrics=mrail
 -   a4pjwfb  trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64
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
 -   rqf77du	  ^hdf5@1.10.7%gcc@7.5.0~cxx~debug~fortran+hl~java+mpi+pic+shared~szip~threadsafe api=none arch=linux-ubuntu18.04-x86_64
 -   7uoyvsg	      ^mvapich2@2.3.4%gcc@7.5.0~alloca~cuda~debug+regcache+wrapperrpath ch3_rank_bits=32 fabrics=mrail file_systems=auto process_managers=auto threads=multiple arch=linux-ubuntu18.04-x86_64
 -   sdjqt2l		  ^bison@3.6.4%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   pqlmjnb		      ^help2man@1.47.11%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to			  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf			      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm				  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5			      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mkc3u4x		      ^m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
[+]  lbrx7ln			  ^libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  hrv4dx5		  ^findutils@4.6.0%gcc@7.5.0 patches=84b916c0bf8c51b7e7b28417692f0ad3e7030d1f3c248ba77c42ede5c1c5d11e,bd9e4e5cc280f9753ae14956c4e4aa17fe7a210f55dd6c84aa60b12d106d47a2 arch=linux-ubuntu18.04-x86_64
[+]  mm33a3o		      ^autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  d2krmb5		      ^automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jdxbjft		      ^libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  p46ba5q		      ^texinfo@6.5%gcc@7.5.0 patches=12f6edb0c6b270b8c8dba2ce17998c580db01182d871ee32b7b6e4129bd1d23a,1732115f651cff98989cb0215d8f64da5e0f7911ebf0c13b064920f088f2ffe1 arch=linux-ubuntu18.04-x86_64
[+]  bob4o5m		  ^libpciaccess@0.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  gs6ag7k		      ^util-macros@1.19.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   2lpslud		  ^rdma-core@32.0%gcc@7.5.0~ipo build_type=RelWithDebInfo arch=linux-ubuntu18.04-x86_64
 -   ttz66su		      ^libnl@3.3.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   jvt3u4j			  ^flex@2.6.4%gcc@7.5.0+lex patches=09c22e5c6fef327d3e48eb23f0d610dcd3a35ab9207f12e0f875701c677978d3 arch=linux-ubuntu18.04-x86_64
 -   5meza7i		      ^py-docutils@0.15.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   tkhtoma			  ^py-setuptools@50.3.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   szj7juk			      ^python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj				  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw				      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm				  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax				  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b				  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64
 -   sj7igc7	  ^hypre@2.20.0%gcc@7.5.0~complex~debug~int64~internal-superlu~mixedint+mpi~openmp+shared~superlu-dist patches=6e3336b1d62155f6350dfe42b0f9ea25d4fa0af60c7e540959139deb93a26059 arch=linux-ubuntu18.04-x86_64
[+]  te35tjx	      ^openblas@0.3.12%gcc@7.5.0~consistent_fpcsr~ilp64+pic+shared threads=none arch=linux-ubuntu18.04-x86_64
 -   kzfn5yl	  ^matio@1.5.17%gcc@7.5.0+hdf5+shared+zlib arch=linux-ubuntu18.04-x86_64
[+]  edcrft4	  ^metis@5.1.0%gcc@7.5.0~gdb~int64~real64+shared build_type=Release patches=4991da938c1d3a1d3dea78e49bbebecba00273f98df2a656e38b83d55b281da1,b1225da886605ea558db7ac08dd8054742ea5afe5ed61ad4d0fe7a495b1270d2 arch=linux-ubuntu18.04-x86_64
 -   mhfry35	  ^mumps@5.3.3%gcc@7.5.0+complex+double+float~int64~metis+mpi~parmetis~ptscotch~scotch+shared arch=linux-ubuntu18.04-x86_64
 -   3wghjiv	      ^netlib-scalapack@2.1.0%gcc@7.5.0~ipo~pic+shared build_type=Release patches=1c9ce5fee1451a08c2de3cc87f446aeda0b818ebbce4ad0d980ddf2f2a0b2dc4,f2baedde688ffe4c20943c334f580eb298e04d6f35c86b90a1f4e8cb7ae344a2 arch=linux-ubuntu18.04-x86_64
 -   wisoa4e	  ^netcdf-c@4.7.4%gcc@7.5.0~dap~hdf4~jna+mpi~parallel-netcdf+pic+shared arch=linux-ubuntu18.04-x86_64
 -   3wqgvq6	  ^parmetis@4.0.3%gcc@7.5.0~gdb~int64~ipo+shared build_type=RelWithDebInfo patches=4f892531eb0a807eb1b82e683a416d3e35154a455274cf9b162fb02054d11a5b,50ed2081bc939269689789942067c58b3e522c269269a430d5d34c00edbc5870,704b84f7c7444d4372cb59cca6e1209df4ef3b033bc4ee3cf50f369bce972a9d arch=linux-ubuntu18.04-x86_64
[+]  rdfmg5b	  ^suite-sparse@5.8.1%gcc@7.5.0~cuda~openmp+pic~tbb arch=linux-ubuntu18.04-x86_64
[+]  3ol3tld	      ^gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  mpv2v7v	      ^mpfr@4.0.2%gcc@7.5.0 patches=3f80b836948aa96f8d1cb9cc7f3f55973f19285482a96f9a4e1623d460bcccf0 arch=linux-ubuntu18.04-x86_64
[+]  bdyarrk		  ^autoconf-archive@2019.01.06%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mpich
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

==> Concretized openmpi%clang ^mpich
 -   tymzjs6  openmpi@3.1.6%clang@6.0.0~atomics~cuda~cxx~cxx_exceptions+gpfs~java~legacylaunchers~lustre~memchecker~pmi~singularity~sqlite3+static~thread_multiple+vt+wrapper-rpath fabrics=none schedulers=none arch=linux-ubuntu18.04-x86_64
 -   txvzbcs	  ^hwloc@1.11.11%clang@6.0.0~cairo~cuda~gl~libudev+libxml2~netloc~nvml+pci+shared arch=linux-ubuntu18.04-x86_64
 -   nxnlozz	      ^libpciaccess@0.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   s2dxedn		  ^libtool@2.4.6%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ebsmave		      ^m4@1.4.18%clang@6.0.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64
 -   dfd6u7k			  ^libsigsegv@2.12%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   mghrtpj		  ^pkgconf@1.7.3%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   yzzs2pw		  ^util-macros@1.19.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   7p673kn	      ^libxml2@2.9.10%clang@6.0.0~python arch=linux-ubuntu18.04-x86_64
 -   3fxiwph		  ^libiconv@1.16%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   suaulg4		  ^xz@5.2.5%clang@6.0.0~pic arch=linux-ubuntu18.04-x86_64
[+]  5qffmms		  ^zlib@1.2.11%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   ej2rp4k	      ^numactl@2.0.14%clang@6.0.0 patches=4e1d78cbbb85de625bad28705e748856033eaafab92a66dffd383a3d7e00cc94 arch=linux-ubuntu18.04-x86_64
 -   ew6lwha		  ^autoconf@2.69%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   4ohfgth		      ^perl@5.32.0%clang@6.0.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
 -   ztorlej			  ^berkeley-db@18.1.40%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   ymkkpkx			  ^gdbm@1.18.1%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   b5yhipp			      ^readline@8.0%clang@6.0.0 arch=linux-ubuntu18.04-x86_64
 -   muufhm7				  ^ncurses@6.2%clang@6.0.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
 -   piicbth		  ^automake@1.16.2%clang@6.0.0 arch=linux-ubuntu18.04-x86_64

==> Concretized openmpi%gcc ^mvapich2 fabrics=mrail
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

==> Concretized python
 -   szj7juk  python@3.8.6%gcc@7.5.0+bz2+ctypes+dbm~debug+libxml2+lzma~nis~optimizations+pic+pyexpat+pythoncmd+readline+shared+sqlite3+ssl~tix~tkinter~ucs4+uuid+zlib patches=0d98e93189bc278fbc37a50ed7f183bd8aaf249a8e1670a465f0db6bb4f8cf87 arch=linux-ubuntu18.04-x86_64
[+]  fvfpt26	  ^bzip2@1.0.8%gcc@7.5.0+shared arch=linux-ubuntu18.04-x86_64
[+]  otkkten	      ^diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  jearpk4		  ^libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   ba7brxj	  ^expat@2.2.10%gcc@7.5.0+libbsd arch=linux-ubuntu18.04-x86_64
 -   u6ue7vw	      ^libbsd@0.10.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  4av4gyw	  ^gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  t54jzdy	      ^readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  crhlefo		  ^ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64
[+]  4sh6pym		      ^pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   lbb45to	  ^gettext@0.21%gcc@7.5.0+bzip2+curses+git~libunistring+libxml2+tar+xz arch=linux-ubuntu18.04-x86_64
[+]  yn2r3wf	      ^libxml2@2.9.10%gcc@7.5.0~python arch=linux-ubuntu18.04-x86_64
[+]  komekkm		  ^xz@5.2.5%gcc@7.5.0~pic arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo		  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64
 -   uwe6tb5	      ^tar@1.32%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   hyhbnrm	  ^libffi@3.3%gcc@7.5.0 patches=26f26c6f29a7ce9bf370ad3ab2610f99365b4bdd7b82e7c31df41a3370d685c0 arch=linux-ubuntu18.04-x86_64
 -   wuyj2ax	  ^libuuid@1.0.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  es377uq	  ^openssl@1.1.1h%gcc@7.5.0+systemcerts arch=linux-ubuntu18.04-x86_64
[+]  zfdvt2j	      ^perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64
[+]  4ihuiaz		  ^berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
 -   rhv2o7b	  ^sqlite@3.33.0%gcc@7.5.0+column_metadata+fts~functions~rtree arch=linux-ubuntu18.04-x86_64

==> Concretized tcl
 -   rtz7664  tcl@8.6.10%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64
[+]  smoyzzo	  ^zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64

$ spack find -c
==> In environment /home/spack/code
==> Root specs
python	 tcl

-- no arch / clang ----------------------------------------------
boost%clang   openmpi%clang   trilinos%clang

-- no arch / gcc ------------------------------------------------
boost%gcc   boost%gcc	openmpi%gcc   openmpi%gcc   trilinos%gcc   trilinos%gcc

==> Concretized roots
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
boost@1.74.0  openmpi@3.1.6  trilinos@13.0.1

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
boost@1.74.0  openmpi@3.1.6  python@3.8.6  tcl@8.6.10  trilinos@13.0.1	trilinos@13.0.1

==> 45 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69		     gmp@6.1.2		matio@1.5.17		openssl@1.1.1h
autoconf-archive@2019.01.06  hdf5@1.10.7	metis@5.1.0		parmetis@4.0.3
automake@1.16.2 	     hwloc@1.11.11	mpfr@4.0.2		perl@5.32.0
berkeley-db@18.1.40	     hwloc@2.2.0	mpich@3.3.2		pkgconf@1.7.3
boost@1.74.0		     hypre@2.20.0	mumps@5.3.3		readline@8.0
bzip2@1.0.8		     libiconv@1.16	ncurses@6.2		suite-sparse@5.8.1
cmake@3.18.4		     libpciaccess@0.16	netcdf-c@4.7.4		texinfo@6.5
diffutils@3.7		     libsigsegv@2.12	netlib-scalapack@2.1.0	trilinos@13.0.1
findutils@4.6.0 	     libtool@2.4.6	numactl@2.0.14		util-macros@1.19.1
gdbm@1.18.1		     libxml2@2.9.10	openblas@0.3.12 	xz@5.2.5
glm@0.9.7.1		     m4@1.4.18		openmpi@3.1.6		zlib@1.2.11
```

## view 記述子

単純なビューはスタックでは機能しないため、ここまでのチュートリアルではスタックのビューを作成しないようにSpackを設定していました。

同じ名前の複数のパッケージを具体化してきました➖同じビューにリンクすると競合します。

これを回避するために、`view` 記述子を使用します。
これにより、各パッケージがビューにリンクされる方法、ビューにリンクされるパッケージ、またはその両方を定義できます。

最後にもう一度 `spack.yaml` ファイルを編集しましょう。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # named lists
  definitions:
    - packages: [boost, trilinos, openmpi]
    - mpis: [mpich, mvapich2 fabrics=mrail]
    - compilers: ['%gcc']
    - compilers: ['%clang']
      when: 'env.get("SPACK_STACK_USE_CLANG", "") == "1"'
    - singleton_packages: [python, tcl]

  specs:
    - matrix:
        - [$packages]
        - [$^mpis]
        - [$compilers]
      exclude:
        - '%clang ^mvapich2'
    - $singleton_packages

  view:
    default:
      root: views/default
      select: ['%gcc']
      exclude: [^mvapich2, 'hwloc@:1.99']
    full:
      root: views/full
      projections:
        ^mpi: '{name}/{name}-{version}-{^mpi.name}-{^mpi.version}-{compiler.name}-{compiler.version}'
        all: '{name}/{name}-{version}-{compiler.name}-{compiler.version}'
```

チュートリアル中にスタックにすべてをインストールするための時間がないので、ビューが完全に入力されていることは確認しませんが、既にインストール済みのパッケージはビューにリンクされます。

```bash
$ spack concretize
==> Updating view at views/default
==> Updating view at views/full
$ ls views/default
TrilinosRepoVersion.txt  bin  etc  include  lib  man  sbin  share
$ ls views/default/lib
cmake					   libhdf5.so
engines-1.1				   libhdf5.so.103
exodus2.py				   libhdf5.so.103.3.0
exodus3.py				   libhdf5_hl.a
exomerge2.py				   libhdf5_hl.so
exomerge3.py				   libhdf5_hl.so.100
libHYPRE-2.20.0.so			   libhdf5_hl.so.100.1.4
libHYPRE.so				   libhistory.a
libIoex.so				   libhistory.so
libIoex.so.13				   libhistory.so.8
libIoex.so.13.0 			   libhistory.so.8.0
libIogn.so				   libhwloc.so
libIogn.so.13				   libhwloc.so.15
libIogn.so.13.0 			   libhwloc.so.15.2.0
libIogs.so				   libiconv.so
libIogs.so.13				   libiconv.so.2
libIogs.so.13.0 			   libiconv.so.2.6.1
libIohb.so				   libifpack.so
libIohb.so.13				   libifpack.so.13
libIohb.so.13.0 			   libifpack.so.13.0
libIonit.so				   libifpack2.so
libIonit.so.13				   libifpack2.so.13
libIonit.so.13.0			   libifpack2.so.13.0
libIoss.so				   libio_info_lib.so
libIoss.so.13				   libio_info_lib.so.13
libIoss.so.13.0 			   libio_info_lib.so.13.0
libIotr.so				   libklu.so
libIotr.so.13				   libklu.so.1
libIotr.so.13.0 			   libklu.so.1.3.8
libIovs.so				   libkokkosalgorithms.so
libIovs.so.13				   libkokkosalgorithms.so.13
libIovs.so.13.0 			   libkokkosalgorithms.so.13.0
libModeLaplace.so			   libkokkoscontainers.so
libModeLaplace.so.13			   libkokkoscontainers.so.13
libModeLaplace.so.13.0			   libkokkoscontainers.so.3.1.1
libamd.so				   libkokkoscore.so
libamd.so.2				   libkokkoscore.so.13
libamd.so.2.4.6 			   libkokkoscore.so.3.1.1
libamesos.so				   libkokkoskernels.so
libamesos.so.13 			   libkokkoskernels.so.13
libamesos.so.13.0			   libkokkoskernels.so.13.0
libamesos2.so				   libkokkostsqr.so
libamesos2.so.13			   libkokkostsqr.so.13
libamesos2.so.13.0			   libkokkostsqr.so.13.0
libanasazi.so				   libldl.so
libanasazi.so.13			   libldl.so.2
libanasazi.so.13.0			   libldl.so.2.2.6
libanasaziepetra.so			   liblzma.a
libanasaziepetra.so.13			   liblzma.so
libanasaziepetra.so.13.0		   liblzma.so.5
libanasazitpetra.so			   liblzma.so.5.2.5
libanasazitpetra.so.13			   libmapvarlib.so
libanasazitpetra.so.13.0		   libmapvarlib.so.13
libaprepro_lib.so			   libmapvarlib.so.13.0
libaprepro_lib.so.13			   libmatio.a
libaprepro_lib.so.13.0			   libmatio.so
libaztecoo.so				   libmatio.so.9
libaztecoo.so.13			   libmatio.so.9.1.2
libaztecoo.so.13.0			   libmenu.a
libbelos.so				   libmenu.so
libbelos.so.13				   libmenu.so.6
libbelos.so.13.0			   libmenu.so.6.2
libbelosepetra.so			   libmenu_g.a
libbelosepetra.so.13			   libmenuw.a
libbelosepetra.so.13.0			   libmenuw.so
libbelostpetra.so			   libmenuw.so.6
libbelostpetra.so.13			   libmenuw.so.6.2
libbelostpetra.so.13.0			   libmenuw_g.a
libbelosxpetra.so			   libmetis.so
libbelosxpetra.so.13			   libml.so
libbelosxpetra.so.13.0			   libml.so.13
libboost_atomic-mt.a			   libml.so.13.0
libboost_atomic-mt.so			   libmongoose.a
libboost_atomic-mt.so.1.74.0		   libmongoose.so
libboost_atomic.a			   libmongoose.so.2
libboost_atomic.so			   libmongoose.so.2.0.4
libboost_atomic.so.1.74.0		   libmpfr.a
libboost_chrono-mt.a			   libmpfr.so
libboost_chrono-mt.so			   libmpfr.so.6
libboost_chrono-mt.so.1.74.0		   libmpfr.so.6.0.2
libboost_chrono.a			   libmpi.a
libboost_chrono.so			   libmpi.so
libboost_chrono.so.1.74.0		   libmpi.so.12
libboost_date_time-mt.a 		   libmpi.so.12.1.8
libboost_date_time-mt.so		   libmpi.so.40
libboost_date_time-mt.so.1.74.0 	   libmpi.so.40.10.4
libboost_date_time.a			   libmpi_mpifh.a
libboost_date_time.so			   libmpi_mpifh.so
libboost_date_time.so.1.74.0		   libmpi_mpifh.so.40
libboost_exception-mt.a 		   libmpi_mpifh.so.40.11.3
libboost_exception.a			   libmpi_usempi_ignore_tkr.a
libboost_filesystem-mt.a		   libmpi_usempi_ignore_tkr.so
libboost_filesystem-mt.so		   libmpi_usempi_ignore_tkr.so.40
libboost_filesystem-mt.so.1.74.0	   libmpi_usempi_ignore_tkr.so.40.10.2
libboost_filesystem.a			   libmpi_usempif08.a
libboost_filesystem.so			   libmpi_usempif08.so
libboost_filesystem.so.1.74.0		   libmpi_usempif08.so.40
libboost_graph-mt.a			   libmpi_usempif08.so.40.10.3
libboost_graph-mt.so			   libmpich.so
libboost_graph-mt.so.1.74.0		   libmpichcxx.so
libboost_graph.a			   libmpichf90.so
libboost_graph.so			   libmpicxx.a
libboost_graph.so.1.74.0		   libmpicxx.so
libboost_iostreams-mt.a 		   libmpicxx.so.12
libboost_iostreams-mt.so		   libmpicxx.so.12.1.8
libboost_iostreams-mt.so.1.74.0 	   libmpifort.a
libboost_iostreams.a			   libmpifort.so
libboost_iostreams.so			   libmpifort.so.12
libboost_iostreams.so.1.74.0		   libmpifort.so.12.1.8
libboost_locale-mt.a			   libmpl.so
libboost_locale-mt.so			   libmuelu-adapters.so
libboost_locale-mt.so.1.74.0		   libmuelu-adapters.so.13
libboost_locale.a			   libmuelu-adapters.so.13.0
libboost_locale.so			   libmuelu-interface.so
libboost_locale.so.1.74.0		   libmuelu-interface.so.13
libboost_log-mt.a			   libmuelu-interface.so.13.0
libboost_log-mt.so			   libmuelu.so
libboost_log-mt.so.1.74.0		   libmuelu.so.13
libboost_log.a				   libmuelu.so.13.0
libboost_log.so 			   libmumps_common.so
libboost_log.so.1.74.0			   libncurses++.a
libboost_log_setup-mt.a 		   libncurses++.so
libboost_log_setup-mt.so		   libncurses++.so.6
libboost_log_setup-mt.so.1.74.0 	   libncurses++.so.6.2
libboost_log_setup.a			   libncurses++_g.a
libboost_log_setup.so			   libncurses++w.a
libboost_log_setup.so.1.74.0		   libncurses++w.so
libboost_math_c99-mt.a			   libncurses++w.so.6
libboost_math_c99-mt.so 		   libncurses++w.so.6.2
libboost_math_c99-mt.so.1.74.0		   libncurses++w_g.a
libboost_math_c99.a			   libncurses.a
libboost_math_c99.so			   libncurses.so
libboost_math_c99.so.1.74.0		   libncurses.so.6
libboost_math_c99f-mt.a 		   libncurses.so.6.2
libboost_math_c99f-mt.so		   libncurses_g.a
libboost_math_c99f-mt.so.1.74.0 	   libncursesw.a
libboost_math_c99f.a			   libncursesw.so
libboost_math_c99f.so			   libncursesw.so.6
libboost_math_c99f.so.1.74.0		   libncursesw.so.6.2
libboost_math_c99l-mt.a 		   libncursesw_g.a
libboost_math_c99l-mt.so		   libnemesis.so
libboost_math_c99l-mt.so.1.74.0 	   libnemesis.so.13
libboost_math_c99l.a			   libnemesis.so.13.0
libboost_math_c99l.so			   libnetcdf.a
libboost_math_c99l.so.1.74.0		   libnetcdf.settings
libboost_math_tr1-mt.a			   libnetcdf.so
libboost_math_tr1-mt.so 		   libnetcdf.so.18
libboost_math_tr1-mt.so.1.74.0		   libnetcdf.so.18.0.0
libboost_math_tr1.a			   libnuma.a
libboost_math_tr1.so			   libnuma.so
libboost_math_tr1.so.1.74.0		   libnuma.so.1
libboost_math_tr1f-mt.a 		   libnuma.so.1.0.0
libboost_math_tr1f-mt.so		   libompitrace.a
libboost_math_tr1f-mt.so.1.74.0 	   libompitrace.so
libboost_math_tr1f.a			   libompitrace.so.40
libboost_math_tr1f.so			   libompitrace.so.40.10.1
libboost_math_tr1f.so.1.74.0		   libopa.so
libboost_math_tr1l-mt.a 		   libopen-pal.a
libboost_math_tr1l-mt.so		   libopen-pal.so
libboost_math_tr1l-mt.so.1.74.0 	   libopen-pal.so.40
libboost_math_tr1l.a			   libopen-pal.so.40.10.6
libboost_math_tr1l.so			   libopen-rte.a
libboost_math_tr1l.so.1.74.0		   libopen-rte.so
libboost_prg_exec_monitor-mt.a		   libopen-rte.so.40
libboost_prg_exec_monitor-mt.so 	   libopen-rte.so.40.10.5
libboost_prg_exec_monitor-mt.so.1.74.0	   libopenblas-r0.3.12.a
libboost_prg_exec_monitor.a		   libopenblas-r0.3.12.so
libboost_prg_exec_monitor.so		   libopenblas.a
libboost_prg_exec_monitor.so.1.74.0	   libopenblas.so
libboost_program_options-mt.a		   libopenblas.so.0
libboost_program_options-mt.so		   liboshmem.a
libboost_program_options-mt.so.1.74.0	   liboshmem.so
libboost_program_options.a		   liboshmem.so.40
libboost_program_options.so		   liboshmem.so.40.10.2
libboost_program_options.so.1.74.0	   libpanel.a
libboost_random-mt.a			   libpanel.so
libboost_random-mt.so			   libpanel.so.6
libboost_random-mt.so.1.74.0		   libpanel.so.6.2
libboost_random.a			   libpanel_g.a
libboost_random.so			   libpanelw.a
libboost_random.so.1.74.0		   libpanelw.so
libboost_regex-mt.a			   libpanelw.so.6
libboost_regex-mt.so			   libpanelw.so.6.2
libboost_regex-mt.so.1.74.0		   libpanelw_g.a
libboost_regex.a			   libparmetis.so
libboost_regex.so			   libpciaccess.a
libboost_regex.so.1.74.0		   libpciaccess.so
libboost_serialization-mt.a		   libpciaccess.so.0
libboost_serialization-mt.so		   libpciaccess.so.0.11.1
libboost_serialization-mt.so.1.74.0	   libpord.so
libboost_serialization.a		   librbio.so
libboost_serialization.so		   librbio.so.2
libboost_serialization.so.1.74.0	   librbio.so.2.2.6
libboost_system-mt.a			   libreadline.a
libboost_system-mt.so			   libreadline.so
libboost_system-mt.so.1.74.0		   libreadline.so.8
libboost_system.a			   libreadline.so.8.0
libboost_system.so			   libsacado.so
libboost_system.so.1.74.0		   libsacado.so.13
libboost_test_exec_monitor-mt.a 	   libsacado.so.13.0
libboost_test_exec_monitor.a		   libscalapack.so
libboost_thread-mt.a			   libsliplu.so
libboost_thread-mt.so			   libsliplu.so.1
libboost_thread-mt.so.1.74.0		   libsliplu.so.1.0.2
libboost_thread.a			   libsmumps.so
libboost_thread.so			   libspqr.so
libboost_thread.so.1.74.0		   libspqr.so.2
libboost_timer-mt.a			   libspqr.so.2.0.9
libboost_timer-mt.so			   libssl.a
libboost_timer-mt.so.1.74.0		   libssl.so
libboost_timer.a			   libssl.so.1.1
libboost_timer.so			   libsuitesparseconfig.so
libboost_timer.so.1.74.0		   libsuitesparseconfig.so.5
libboost_unit_test_framework-mt.a	   libsuitesparseconfig.so.5.8.1
libboost_unit_test_framework-mt.so	   libsupes.so
libboost_unit_test_framework-mt.so.1.74.0  libsupes.so.13
libboost_unit_test_framework.a		   libsupes.so.13.0
libboost_unit_test_framework.so 	   libsuplib.so
libboost_unit_test_framework.so.1.74.0	   libsuplib.so.13
libboost_wave-mt.a			   libsuplib.so.13.0
libboost_wave-mt.so			   libsuplib_c.so
libboost_wave-mt.so.1.74.0		   libsuplib_c.so.13
libboost_wave.a 			   libsuplib_c.so.13.0
libboost_wave.so			   libsuplib_cpp.so
libboost_wave.so.1.74.0 		   libsuplib_cpp.so.13
libboost_wserialization-mt.a		   libsuplib_cpp.so.13.0
libboost_wserialization-mt.so		   libteuchoscomm.so
libboost_wserialization-mt.so.1.74.0	   libteuchoscomm.so.13
libboost_wserialization.a		   libteuchoscomm.so.13.0
libboost_wserialization.so		   libteuchoscore.so
libboost_wserialization.so.1.74.0	   libteuchoscore.so.13
libbtf.so				   libteuchoscore.so.13.0
libbtf.so.1				   libteuchoskokkoscomm.so
libbtf.so.1.2.6 			   libteuchoskokkoscomm.so.13
libbz2.a				   libteuchoskokkoscomm.so.13.0
libbz2.so				   libteuchoskokkoscompat.so
libbz2.so.1				   libteuchoskokkoscompat.so.13
libbz2.so.1.0				   libteuchoskokkoscompat.so.13.0
libbz2.so.1.0.8 			   libteuchosnumerics.so
libcamd.so				   libteuchosnumerics.so.13
libcamd.so.2				   libteuchosnumerics.so.13.0
libcamd.so.2.4.6			   libteuchosparameterlist.so
libccolamd.so				   libteuchosparameterlist.so.13
libccolamd.so.2 			   libteuchosparameterlist.so.13.0
libccolamd.so.2.9.6			   libteuchosparser.so
libcharset.a				   libteuchosparser.so.13
libcharset.so				   libteuchosparser.so.13.0
libcharset.so.1 			   libteuchosremainder.so
libcharset.so.1.0.0			   libteuchosremainder.so.13
libcholmod.so				   libteuchosremainder.so.13.0
libcholmod.so.3 			   libtinfo.a
libcholmod.so.3.0.14			   libtinfo.so
libcmumps.so				   libtinfo.so.6
libcolamd.so				   libtinfo.so.6.2
libcolamd.so.2				   libtinfo_g.a
libcolamd.so.2.9.6			   libtinfow.a
libcrypto.a				   libtinfow.so
libcrypto.so				   libtinfow.so.6
libcrypto.so.1.1			   libtinfow.so.6.2
libcurses.a				   libtinfow_g.a
libcurses.so				   libtpetra.so
libcxsparse.so				   libtpetra.so.13
libcxsparse.so.3			   libtpetra.so.13.0
libcxsparse.so.3.2.0			   libtpetraclassic.so
libdmumps.so				   libtpetraclassic.so.13
libepetra.so				   libtpetraclassic.so.13.0
libepetra.so.13 			   libtpetraclassiclinalg.so
libepetra.so.13.0			   libtpetraclassiclinalg.so.13
libepetraext.so 			   libtpetraclassiclinalg.so.13.0
libepetraext.so.13			   libtpetraclassicnodeapi.so
libepetraext.so.13.0			   libtpetraclassicnodeapi.so.13
libexoIIv2for32.so			   libtpetraclassicnodeapi.so.13.0
libexoIIv2for32.so.13			   libtpetraext.so
libexoIIv2for32.so.13.0 		   libtpetraext.so.13
libexodus.so				   libtpetraext.so.13.0
libexodus.so.13 			   libtpetrainout.so
libexodus.so.13.0			   libtpetrainout.so.13
libexodus_for.so			   libtpetrainout.so.13.0
libexodus_for.so.13			   libtrilinosss.so
libexodus_for.so.13.0			   libtrilinosss.so.13
libfmpich.so				   libtrilinosss.so.13.0
libform.a				   libtriutils.so
libform.so				   libtriutils.so.13
libform.so.6				   libtriutils.so.13.0
libform.so.6.2				   libumfpack.so
libform_g.a				   libumfpack.so.5
libformw.a				   libumfpack.so.5.7.9
libformw.so				   libxml2.a
libformw.so.6				   libxml2.so
libformw.so.6.2 			   libxml2.so.2
libformw_g.a				   libxml2.so.2.9.10
libgaleri-epetra.so			   libxpetra-sup.so
libgaleri-epetra.so.13			   libxpetra-sup.so.13
libgaleri-epetra.so.13.0		   libxpetra-sup.so.13.0
libgaleri-xpetra.so			   libxpetra.so
libgaleri-xpetra.so.13			   libxpetra.so.13
libgaleri-xpetra.so.13.0		   libxpetra.so.13.0
libgdbm.a				   libz.a
libgdbm.so				   libz.so
libgdbm.so.6				   libz.so.1
libgdbm.so.6.0.0			   libz.so.1.2.11
libgdbm_compat.a			   libzmumps.so
libgdbm_compat.so			   libzoltan.so
libgdbm_compat.so.4			   libzoltan.so.13
libgdbm_compat.so.4.0.0 		   libzoltan.so.13.0
libgmp.a				   libzoltan2.so
libgmp.so				   libzoltan2.so.13
libgmp.so.10				   libzoltan2.so.13.0
libgmp.so.10.3.2			   mpi.mod
libgmpxx.a				   mpi_ext.mod
libgmpxx.so				   mpi_f08.mod
libgmpxx.so.4				   mpi_f08_callbacks.mod
libgmpxx.so.4.5.2			   mpi_f08_ext.mod
libgraphblas.so 			   mpi_f08_interfaces.mod
libgraphblas.so.3			   mpi_f08_interfaces_callbacks.mod
libgraphblas.so.3.3.3			   mpi_f08_types.mod
libgtest.so				   openmpi
libgtest.so.13				   pkgconfig
libgtest.so.13.0			   pmpi_f08_interfaces.mod
libh5bzip2.so				   terminfo
libhdf5.a				   x86_64-linux-gnu
libhdf5.settings			   xml2Conf.sh
$ ls views/full
boost  glm   hwloc     libpciaccess  metis  mumps     netlib-scalapack	openmpi   readline	xz
bzip2  gmp   hypre     libxml2	     mpfr   ncurses   numactl		openssl   suite-sparse	zlib
gdbm   hdf5  libiconv  matio	     mpich  netcdf-c  openblas		parmetis  trilinos
$ ls views/full/zlib
zlib-1.2.11-clang-6.0.0  zlib-1.2.11-gcc-7.5.0
$ ls views/full/zlib/zlib-1.2.11-gcc-7.5.0
include  lib  share
$ ls views/full/zlib/zlib-1.2.11-gcc-7.5.0/lib
libz.a	libz.so  libz.so.1  libz.so.1.2.11  pkgconfig
```

`view` 記述子には、`all` もしくは `roots`のいずれかである `link`キーも含まれています。
これまで見てきたように、デフォルトの動作は、暗黙の依存関係を含むすべてのパッケージをビューにリンクすることです。
`roots` オプションは、ルートパッケージのみをビューにリンクします。

```yaml
# This is a Spack Environment file.
#
# It describes a set of packages to be installed, along with
# configuration setings.
spack:
  # named lists
  definitions:
    - packages: [boost, trilinos, openmpi]
    - mpis: [mpich, mvapich2 fabrics=mrail]
    - compilers: ['%gcc']
    - compilers: ['%clang']
      when: 'env.get("SPACK_STACK_USE_CLANG", "") == "1"'
    - singleton_packages: [python, tcl]

  specs:
    - matrix:
        - [$packages]
        - [$^mpis]
        - [$compilers]
      exclude:
        - '%clang ^mvapich2'
    - $singleton_packages

  view:
    default:
      root: views/default
      select: ['%gcc']
      exclude: [^mvapich2]
      link: roots
    full:
      root: views/full
      projections:
        ^mpi: '{name}/{name}-{version}-{^mpi.name}-{^mpi.version}-{compiler.name}-{compiler.version}'
        all: '{name}/{name}-{version}-{compiler.name}-{compiler.version}'
```

```bash
$ spack concretize
==> Updating view at views/default
==> Updating view at views/full
$ ls views/default
TrilinosRepoVersion.txt  bin  etc  include  lib  share
$ ls views/default/lib
cmake					libboost_timer.so
exodus2.py				libboost_timer.so.1.74.0
exodus3.py				libboost_unit_test_framework-mt.a
exomerge2.py				libboost_unit_test_framework-mt.so
exomerge3.py				libboost_unit_test_framework-mt.so.1.74.0
libIoex.so				libboost_unit_test_framework.a
libIoex.so.13				libboost_unit_test_framework.so
libIoex.so.13.0 			libboost_unit_test_framework.so.1.74.0
libIogn.so				libboost_wave-mt.a
libIogn.so.13				libboost_wave-mt.so
libIogn.so.13.0 			libboost_wave-mt.so.1.74.0
libIogs.so				libboost_wave.a
libIogs.so.13				libboost_wave.so
libIogs.so.13.0 			libboost_wave.so.1.74.0
libIohb.so				libboost_wserialization-mt.a
libIohb.so.13				libboost_wserialization-mt.so
libIohb.so.13.0 			libboost_wserialization-mt.so.1.74.0
libIonit.so				libboost_wserialization.a
libIonit.so.13				libboost_wserialization.so
libIonit.so.13.0			libboost_wserialization.so.1.74.0
libIoss.so				libepetra.so
libIoss.so.13				libepetra.so.13
libIoss.so.13.0 			libepetra.so.13.0
libIotr.so				libepetraext.so
libIotr.so.13				libepetraext.so.13
libIotr.so.13.0 			libepetraext.so.13.0
libIovs.so				libexoIIv2for32.so
libIovs.so.13				libexoIIv2for32.so.13
libIovs.so.13.0 			libexoIIv2for32.so.13.0
libModeLaplace.so			libexodus.so
libModeLaplace.so.13			libexodus.so.13
libModeLaplace.so.13.0			libexodus.so.13.0
libamesos.so				libexodus_for.so
libamesos.so.13 			libexodus_for.so.13
libamesos.so.13.0			libexodus_for.so.13.0
libamesos2.so				libgaleri-epetra.so
libamesos2.so.13			libgaleri-epetra.so.13
libamesos2.so.13.0			libgaleri-epetra.so.13.0
libanasazi.so				libgaleri-xpetra.so
libanasazi.so.13			libgaleri-xpetra.so.13
libanasazi.so.13.0			libgaleri-xpetra.so.13.0
libanasaziepetra.so			libgtest.so
libanasaziepetra.so.13			libgtest.so.13
libanasaziepetra.so.13.0		libgtest.so.13.0
libanasazitpetra.so			libifpack.so
libanasazitpetra.so.13			libifpack.so.13
libanasazitpetra.so.13.0		libifpack.so.13.0
libaprepro_lib.so			libifpack2.so
libaprepro_lib.so.13			libifpack2.so.13
libaprepro_lib.so.13.0			libifpack2.so.13.0
libaztecoo.so				libio_info_lib.so
libaztecoo.so.13			libio_info_lib.so.13
libaztecoo.so.13.0			libio_info_lib.so.13.0
libbelos.so				libkokkosalgorithms.so
libbelos.so.13				libkokkosalgorithms.so.13
libbelos.so.13.0			libkokkosalgorithms.so.13.0
libbelosepetra.so			libkokkoscontainers.so
libbelosepetra.so.13			libkokkoscontainers.so.13
libbelosepetra.so.13.0			libkokkoscontainers.so.3.1.1
libbelostpetra.so			libkokkoscore.so
libbelostpetra.so.13			libkokkoscore.so.13
libbelostpetra.so.13.0			libkokkoscore.so.3.1.1
libbelosxpetra.so			libkokkoskernels.so
libbelosxpetra.so.13			libkokkoskernels.so.13
libbelosxpetra.so.13.0			libkokkoskernels.so.13.0
libboost_atomic-mt.a			libkokkostsqr.so
libboost_atomic-mt.so			libkokkostsqr.so.13
libboost_atomic-mt.so.1.74.0		libkokkostsqr.so.13.0
libboost_atomic.a			libmapvarlib.so
libboost_atomic.so			libmapvarlib.so.13
libboost_atomic.so.1.74.0		libmapvarlib.so.13.0
libboost_chrono-mt.a			libml.so
libboost_chrono-mt.so			libml.so.13
libboost_chrono-mt.so.1.74.0		libml.so.13.0
libboost_chrono.a			libmpi.a
libboost_chrono.so			libmpi.so
libboost_chrono.so.1.74.0		libmpi.so.40
libboost_date_time-mt.a 		libmpi.so.40.10.4
libboost_date_time-mt.so		libmpi_mpifh.a
libboost_date_time-mt.so.1.74.0 	libmpi_mpifh.so
libboost_date_time.a			libmpi_mpifh.so.40
libboost_date_time.so			libmpi_mpifh.so.40.11.3
libboost_date_time.so.1.74.0		libmpi_usempi_ignore_tkr.a
libboost_exception-mt.a 		libmpi_usempi_ignore_tkr.so
libboost_exception.a			libmpi_usempi_ignore_tkr.so.40
libboost_filesystem-mt.a		libmpi_usempi_ignore_tkr.so.40.10.2
libboost_filesystem-mt.so		libmpi_usempif08.a
libboost_filesystem-mt.so.1.74.0	libmpi_usempif08.so
libboost_filesystem.a			libmpi_usempif08.so.40
libboost_filesystem.so			libmpi_usempif08.so.40.10.3
libboost_filesystem.so.1.74.0		libmuelu-adapters.so
libboost_graph-mt.a			libmuelu-adapters.so.13
libboost_graph-mt.so			libmuelu-adapters.so.13.0
libboost_graph-mt.so.1.74.0		libmuelu-interface.so
libboost_graph.a			libmuelu-interface.so.13
libboost_graph.so			libmuelu-interface.so.13.0
libboost_graph.so.1.74.0		libmuelu.so
libboost_iostreams-mt.a 		libmuelu.so.13
libboost_iostreams-mt.so		libmuelu.so.13.0
libboost_iostreams-mt.so.1.74.0 	libnemesis.so
libboost_iostreams.a			libnemesis.so.13
libboost_iostreams.so			libnemesis.so.13.0
libboost_iostreams.so.1.74.0		libompitrace.a
libboost_locale-mt.a			libompitrace.so
libboost_locale-mt.so			libompitrace.so.40
libboost_locale-mt.so.1.74.0		libompitrace.so.40.10.1
libboost_locale.a			libopen-pal.a
libboost_locale.so			libopen-pal.so
libboost_locale.so.1.74.0		libopen-pal.so.40
libboost_log-mt.a			libopen-pal.so.40.10.6
libboost_log-mt.so			libopen-rte.a
libboost_log-mt.so.1.74.0		libopen-rte.so
libboost_log.a				libopen-rte.so.40
libboost_log.so 			libopen-rte.so.40.10.5
libboost_log.so.1.74.0			liboshmem.a
libboost_log_setup-mt.a 		liboshmem.so
libboost_log_setup-mt.so		liboshmem.so.40
libboost_log_setup-mt.so.1.74.0 	liboshmem.so.40.10.2
libboost_log_setup.a			libsacado.so
libboost_log_setup.so			libsacado.so.13
libboost_log_setup.so.1.74.0		libsacado.so.13.0
libboost_math_c99-mt.a			libsupes.so
libboost_math_c99-mt.so 		libsupes.so.13
libboost_math_c99-mt.so.1.74.0		libsupes.so.13.0
libboost_math_c99.a			libsuplib.so
libboost_math_c99.so			libsuplib.so.13
libboost_math_c99.so.1.74.0		libsuplib.so.13.0
libboost_math_c99f-mt.a 		libsuplib_c.so
libboost_math_c99f-mt.so		libsuplib_c.so.13
libboost_math_c99f-mt.so.1.74.0 	libsuplib_c.so.13.0
libboost_math_c99f.a			libsuplib_cpp.so
libboost_math_c99f.so			libsuplib_cpp.so.13
libboost_math_c99f.so.1.74.0		libsuplib_cpp.so.13.0
libboost_math_c99l-mt.a 		libteuchoscomm.so
libboost_math_c99l-mt.so		libteuchoscomm.so.13
libboost_math_c99l-mt.so.1.74.0 	libteuchoscomm.so.13.0
libboost_math_c99l.a			libteuchoscore.so
libboost_math_c99l.so			libteuchoscore.so.13
libboost_math_c99l.so.1.74.0		libteuchoscore.so.13.0
libboost_math_tr1-mt.a			libteuchoskokkoscomm.so
libboost_math_tr1-mt.so 		libteuchoskokkoscomm.so.13
libboost_math_tr1-mt.so.1.74.0		libteuchoskokkoscomm.so.13.0
libboost_math_tr1.a			libteuchoskokkoscompat.so
libboost_math_tr1.so			libteuchoskokkoscompat.so.13
libboost_math_tr1.so.1.74.0		libteuchoskokkoscompat.so.13.0
libboost_math_tr1f-mt.a 		libteuchosnumerics.so
libboost_math_tr1f-mt.so		libteuchosnumerics.so.13
libboost_math_tr1f-mt.so.1.74.0 	libteuchosnumerics.so.13.0
libboost_math_tr1f.a			libteuchosparameterlist.so
libboost_math_tr1f.so			libteuchosparameterlist.so.13
libboost_math_tr1f.so.1.74.0		libteuchosparameterlist.so.13.0
libboost_math_tr1l-mt.a 		libteuchosparser.so
libboost_math_tr1l-mt.so		libteuchosparser.so.13
libboost_math_tr1l-mt.so.1.74.0 	libteuchosparser.so.13.0
libboost_math_tr1l.a			libteuchosremainder.so
libboost_math_tr1l.so			libteuchosremainder.so.13
libboost_math_tr1l.so.1.74.0		libteuchosremainder.so.13.0
libboost_prg_exec_monitor-mt.a		libtpetra.so
libboost_prg_exec_monitor-mt.so 	libtpetra.so.13
libboost_prg_exec_monitor-mt.so.1.74.0	libtpetra.so.13.0
libboost_prg_exec_monitor.a		libtpetraclassic.so
libboost_prg_exec_monitor.so		libtpetraclassic.so.13
libboost_prg_exec_monitor.so.1.74.0	libtpetraclassic.so.13.0
libboost_program_options-mt.a		libtpetraclassiclinalg.so
libboost_program_options-mt.so		libtpetraclassiclinalg.so.13
libboost_program_options-mt.so.1.74.0	libtpetraclassiclinalg.so.13.0
libboost_program_options.a		libtpetraclassicnodeapi.so
libboost_program_options.so		libtpetraclassicnodeapi.so.13
libboost_program_options.so.1.74.0	libtpetraclassicnodeapi.so.13.0
libboost_random-mt.a			libtpetraext.so
libboost_random-mt.so			libtpetraext.so.13
libboost_random-mt.so.1.74.0		libtpetraext.so.13.0
libboost_random.a			libtpetrainout.so
libboost_random.so			libtpetrainout.so.13
libboost_random.so.1.74.0		libtpetrainout.so.13.0
libboost_regex-mt.a			libtrilinosss.so
libboost_regex-mt.so			libtrilinosss.so.13
libboost_regex-mt.so.1.74.0		libtrilinosss.so.13.0
libboost_regex.a			libtriutils.so
libboost_regex.so			libtriutils.so.13
libboost_regex.so.1.74.0		libtriutils.so.13.0
libboost_serialization-mt.a		libxpetra-sup.so
libboost_serialization-mt.so		libxpetra-sup.so.13
libboost_serialization-mt.so.1.74.0	libxpetra-sup.so.13.0
libboost_serialization.a		libxpetra.so
libboost_serialization.so		libxpetra.so.13
libboost_serialization.so.1.74.0	libxpetra.so.13.0
libboost_system-mt.a			libzoltan.so
libboost_system-mt.so			libzoltan.so.13
libboost_system-mt.so.1.74.0		libzoltan.so.13.0
libboost_system.a			libzoltan2.so
libboost_system.so			libzoltan2.so.13
libboost_system.so.1.74.0		libzoltan2.so.13.0
libboost_test_exec_monitor-mt.a 	mpi.mod
libboost_test_exec_monitor.a		mpi_ext.mod
libboost_thread-mt.a			mpi_f08.mod
libboost_thread-mt.so			mpi_f08_callbacks.mod
libboost_thread-mt.so.1.74.0		mpi_f08_ext.mod
libboost_thread.a			mpi_f08_interfaces.mod
libboost_thread.so			mpi_f08_interfaces_callbacks.mod
libboost_thread.so.1.74.0		mpi_f08_types.mod
libboost_timer-mt.a			openmpi
libboost_timer-mt.so			pkgconfig
libboost_timer-mt.so.1.74.0		pmpi_f08_interfaces.mod
libboost_timer.a
$ ls views/full
boost  glm   hwloc     libpciaccess  metis  mumps     netlib-scalapack	openmpi   readline	xz
bzip2  gmp   hypre     libxml2	     mpfr   ncurses   numactl		openssl   suite-sparse	zlib
gdbm   hdf5  libiconv  matio	     mpich  netcdf-c  openblas		parmetis  trilinos
```

これで、デフォルトビューにルートライブラリ（ `boost` 、 `trilinos` 、 `openmpi` ）のみが表示されます。
残りは非表示になっていますが、引き続きフルビューで利用可能です。
