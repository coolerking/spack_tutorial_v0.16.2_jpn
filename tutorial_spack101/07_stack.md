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

今のところは、viewディレクティブは避けます（後述にて解説）。

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

TBD