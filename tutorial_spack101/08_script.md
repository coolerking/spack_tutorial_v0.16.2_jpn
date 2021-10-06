# Spack を使用したスクリプト

このチュートリアルでは、スクリプトに関連する高度なSpack機能を紹介します。
具体的には、`spack find` や `spack python`を使用してスクリプトを作成する方法を紹介します。
チュートリアルの前のセクションでは、`spack find` 使用してインストール済みパッケージを一覧表示および検索する方法を示しました。
`spack python` コマンドを使用すると、Spackのすべての [内部API](https://spack.readthedocs.io/en/latest/spack.html) にアクセスできるため、たとえばより複雑なクエリを記述できます。

Spackには広範なAPIがあるため、ここではさわりを解説します。
少し掘り下げて、独自のスクリプトの作成を開始し、必要なものを見つけるのに十分な情報を提供します。

## チュートリアルのセットアップ

先に進める前に、このセグメントの出力が妥当であることを確認しましょう。
これまでのチュートリアルセクションにてたくさんのパッケージがインストールされている可能性があるため、少しクリーンアップを実行しておきます。

次のコマンドを使用して、 `gcc@8.3.0` を削除し、`hdf5`と `zlib@clang` を再インストールしましょう：

```bash
$ spack uninstall -ay
==> Successfully uninstalled gcc@8.3.0%gcc@7.5.0~binutils~bootstrap~nvptx~piclibs~strip languages=c,c++,fortran patches=49341f7807b12a89750c010a20707598f83121da9829150d4049ed3a1a140564,dc1ca240b7fb70112ae6cc47cd86925adf78d29ed9d0c26b0c51d52e40ceca0e arch=linux-ubuntu18.04-x86_64/7mh4po7
==> Successfully uninstalled automake@1.16.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/d2krmb5
==> Successfully uninstalled pkgconf@1.7.3%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/4sh6pym
==> Successfully uninstalled libtool@2.4.6%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/jdxbjft
==> Successfully uninstalled diffutils@3.7%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/otkkten
==> Successfully uninstalled autoconf@2.69%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/mm33a3o
==> Successfully uninstalled patchelf@0.10%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/2nj3vxx
==> Successfully uninstalled m4@1.4.18%gcc@7.5.0+sigsegv patches=3877ab548f88597ab2327a2230ee048d2d07ace1062efe81fc92e91b7f39cd00,fc9b61654a3ba1a8d6cd78ce087e7c96366c290bc8d2c299f09828d793b853c8 arch=linux-ubuntu18.04-x86_64/mkc3u4x
==> Successfully uninstalled isl@0.18%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/4lkvfkv
==> Successfully uninstalled mpc@1.1.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/uur2dd6
==> Successfully uninstalled libiconv@1.16%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/jearpk4
==> Successfully uninstalled zlib@1.2.11%gcc@7.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64/smoyzzo
==> Successfully uninstalled perl@5.32.0%gcc@7.5.0+cpanm+shared+threads arch=linux-ubuntu18.04-x86_64/zfdvt2j
==> Successfully uninstalled mpfr@3.1.6%gcc@7.5.0 patches=7a6dd71bcda4803d6b89612706a17b8816e1acd5dd9bf1bec29cf748f3b60008 arch=linux-ubuntu18.04-x86_64/nwsubsw
==> Successfully uninstalled berkeley-db@18.1.40%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/4ihuiaz
==> Successfully uninstalled libsigsegv@2.12%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/lbrx7ln
==> Successfully uninstalled gdbm@1.18.1%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/4av4gyw
==> Successfully uninstalled readline@8.0%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/t54jzdy
==> Successfully uninstalled gmp@6.1.2%gcc@7.5.0 arch=linux-ubuntu18.04-x86_64/3ol3tld
==> Successfully uninstalled ncurses@6.2%gcc@7.5.0~symlinks+termlib arch=linux-ubuntu18.04-x86_64/crhlefo
$ spack compiler rm gcc@8.3.0
==> Removed compiler gcc@8.3.0
$ spack install hdf5
==> Installing libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12/linux-ubuntu18.04-x86_64-gcc-7.5.0-libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q from binary cache
==> Installing patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
==> No binary for patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp found: installing from source
==> Using cached archive: /home/spack/spack/var/spack/cache/_source-cache/archive/b2/b2deabce05c34ce98558c0efb965f209de592197b2c88e930298d740ead09019.tar.gz
==> patchelf: Executing phase: 'autoreconf'
==> patchelf: Executing phase: 'configure'
==> patchelf: Executing phase: 'build'
==> patchelf: Executing phase: 'install'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
==> Installing pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
==> Installing util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
==> Installing libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16/linux-ubuntu18.04-x86_64-gcc-7.5.0-libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
==> Installing xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5/linux-ubuntu18.04-x86_64-gcc-7.5.0-xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
==> Installing zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11/linux-ubuntu18.04-x86_64-gcc-7.5.0-zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Installing berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40/linux-ubuntu18.04-x86_64-gcc-7.5.0-berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
==> Installing m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18/linux-ubuntu18.04-x86_64-gcc-7.5.0-m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
==> Installing ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
==> Installing libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10/linux-ubuntu18.04-x86_64-gcc-7.5.0-libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
==> Installing libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6/linux-ubuntu18.04-x86_64-gcc-7.5.0-libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
==> Installing readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
==> Installing libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16/linux-ubuntu18.04-x86_64-gcc-7.5.0-libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
==> Installing gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
==> Installing perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Installing autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69/linux-ubuntu18.04-x86_64-gcc-7.5.0-autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
==> Installing automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Installing numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14/linux-ubuntu18.04-x86_64-gcc-7.5.0-numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
==> Installing hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11/linux-ubuntu18.04-x86_64-gcc-7.5.0-hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
==> Installing openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6/linux-ubuntu18.04-x86_64-gcc-7.5.0-openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
==> Installing hdf5-1.10.7-vedchc5aoqyu3ydbp346qrbpe6kg46rq
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-hdf5-1.10.7-vedchc5aoqyu3ydbp346qrbpe6kg46rq.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hdf5-1.10.7-vedchc5aoqyu3ydbp346qrbpe6kg46rq from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-vedchc5aoqyu3ydbp346qrbpe6kg46rq
$ spack install zlib%clang
==> Installing zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.11/linux-ubuntu18.04-x86_64-clang-6.0.0-zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla
```

これで、`spack find` や `spack python` コマンドを使用してインストール済みパッケージを照会する準備が整いました。

## `spack find` によるスクリプティング

これまでの`spack find` 出力は、人による確認のためのものでした。
ですが、コマンドのいくつかの高度なオプションを利用し、スクリプトへのパイピングに適したマシン可読出力を生成させることが可能です。

### `spack find --format`

`spack find` の主な役割は、インストールされたパッケージに対応する一連の具体的な仕様をユーザに示すことです。
デフォルトでは、出力に表示されるのに慣れている `@version` サフィックスなど、いくつかのデフォルト属性でそれらを表示します。

`--format` オプションを使うと、カスタム形式の文字列を指定することで選択した仕様を表示できます。
フォーマット文字列を使用すると、表示する仕様の特定の部分の名前を指定できます。
最初のオプションの動作を見てみましょう。

Spack インスタンスにインストールされているすべてのパッケージの名前、バージョン、およびハッシュの最初の10文字のみを表示するとします。
次のコマンドを使用して、その出力を生成できます：

```bash
$ spack find --format "{name} {version} {hash:10}"
autoconf 2.69 mm33a3ocsv	hdf5 1.10.7 vedchc5aoq	      libsigsegv 2.12 lbrx7lnfz4  ncurses 6.2 crhlefo3dv     perl 5.32.0 zfdvt2jjua	    xz 5.2.5 komekkmyci
automake 1.16.2 d2krmb5gwe	hwloc 1.11.11 zqwfzhw5k2      libtool 2.4.6 jdxbjfthei	  numactl 2.0.14 wbqbc5vw5s  pkgconf 1.7.3 4sh6pymrm2	    zlib 1.2.11 5qffmms6gw
berkeley-db 18.1.40 4ihuiazsgl	libiconv 1.16 jearpk4xci      libxml2 2.9.10 yn2r3wfhii   openmpi 3.1.6 pmsyupw6w3   readline 8.0 t54jzdy2jj	    zlib 1.2.11 smoyzzo2qh
gdbm 1.18.1 4av4gywgpa		libpciaccess 0.16 bob4o5m3uk  m4 1.4.18 mkc3u4x2p2	  patchelf 0.10 2nj3vxxz5l   util-macros 1.19.1 gs6ag7ktdo
```

`name`、`version`、`hash` は `Spec` オブジェクト内部の属性で、それらを中括弧で囲んたフォーマット文字列に応じて出力されます。

`spack find --format` を使うと、`sort`または`uniq`などの一般的なUNIXコマンドラインツールに出力をパイプするなど、必要な情報だけを取得できます。

### `spack find --json`

`--json` オプションを使うことで、JSON形式でSpecオブジェクトのシリアル化されたバージョンを取得することもできます。
たとえば、次のように入力すると、`zlib`のすべてのインストールの属性を取得できます：

```bash
$ spack find --json zlib
[
 {
  "name": "zlib",
  "hash": "5qffmms6gwykcikh6aag4h3z4scrfdla",
  "version": "1.2.11",
  "arch": {
   "platform": "linux",
   "platform_os": "ubuntu18.04",
   "target": "x86_64"
  },
  "compiler": {
   "name": "clang",
   "version": "6.0.0"
  },
  "namespace": "builtin",
  "parameters": {
   "optimize": true,
   "pic": true,
   "shared": true,
   "cflags": [],
   "cppflags": [],
   "cxxflags": [],
   "fflags": [],
   "ldflags": [],
   "ldlibs": []
  }
 },
 {
  "name": "zlib",
  "hash": "smoyzzo2qhzpn6mg6rd3l2p7b23enshg",
  "version": "1.2.11",
  "arch": {
   "platform": "linux",
   "platform_os": "ubuntu18.04",
   "target": "x86_64"
  },
  "compiler": {
   "name": "gcc",
   "version": "7.5.0"
  },
  "namespace": "builtin",
  "parameters": {
   "optimize": true,
   "pic": true,
   "shared": true,
   "cflags": [],
   "cppflags": [],
   "cxxflags": [],
   "fflags": [],
   "ldflags": [],
   "ldlibs": []
  }
 }
```

`spack find --json` コマンドは、仕様について私たちが知っているすべてのものを構造化された形式で提供します。
出力を `jq` のようなJSONフィルタリングツールにパイプして、必要な部分だけを抽出することができます。

その他の例については、[基本的な使用法](https://spack.readthedocs.io/en/latest/basic_usage.html#machine-readable-output) を確認してください。

## `spack python` コマンドの紹介

より高度な照会を実行する必要がある場合はどうなりますか？

Spackは、インポート可能な Spack の Python モジュールを使い Python インタープリタを起動する `spack python` コマンドを提供します。
残りのコマンドには、基盤となる Python を使用します。
したがって、次のスクリプトを作成できます：

- Spackコマンドを実行
- 抽象的/具体的(concretized)な仕様の検索
- Spack の他の内部コンポーネントに直接アクセス

次のように入力して、Spack対応のPythonインタープリタを起動してみましょう：

```python
$ spack python
exit()

Spack version 0.16.1
Python 3.6.9, Linux x86_64
>>> exit()
```

Pythonインタープリタを使用しているので、ターミナルに戻る場合は `exit()` でセッションを終了します。

### `Spec` オブジェクトへのアクセス

それでは、Spackの `spec` 内部表現を見てみましょう。
すでにご存知のように、仕様は抽象的または具体的(concretized)のいずれかが存在します。
`package.py` ファイル（ `install()` メソッドなど）で確認した仕様は具体的(concretized)で、完全に定義されています。
コマンドラインで入力した仕様は抽象的です。
2つのタイプの違いを理解することは、Spackの内部APIを使用するための鍵です。

`spack python` コマンドで別のPythonインタープリタを開き、 `zlib` 仕様をインスタンス化し、抽象仕様のいくつかのプロパティを確認してみましょう：

```python
  >>> from spack.spec import Spec
  >>> s = Spec('zlib target=ivybridge')
  >>> s.concrete
  False
  >>> s.version
  Traceback (most recent call last):
    File "<console>", line 1, in <module>
    File "/home/spack/spack/lib/spack/spack/spec.py", line 3166, in version
      raise SpecError("Spec version is not concrete: " + str(self))
  SpecError: Spec version is not concrete: zlib arch=linux-None-ivybridge
  >>> s.versions
  [:]
  >>> s.architecture
  linux-None-ivybridge
```

抽象仕様にアクセスできないSpecのプロパティとメソッドがあることに注意してください。
具体的には：

- 例外 – `SpecError` – 対象の`version`にアクセスしようとすると発生
- 関連する `versions` が存在しない
- 仕様のオペレーティングシステムが `None`

ここで、インタープリタを終了せずに、仕様を具体化して再試行してみましょう：

```python
  >>> s.concretize()
  >>> s.concrete
  True
  >>> s.version
  Version('1.2.11')
  >>> s.versions
  [Version('1.2.11')]
  >>> s.architecture
  linux-ubuntu18.04-ivybridge
```

現在、具体化された仕様に注意してください。

- `versions` が存在
- `versions` リストに1つのエントリが存在
- オペレーティングシステムは `ubuntu18.04`

中間抽象仕様を保存する必要はありません。
 `.concretized()` メソッドを省略形として利用できます：

 ```python
  >>> t = Spec('zlib target=ivybridge').concretized()
  >>> s == t
  True
 ```

## Spackデータベース照会

Spackデータベースに保存されている情報を見ると、さらに強力な照会が利用可能です。
Spackの `Database` オブジェクトは変数 `spack.store.db` にあります。
主に `query()` メソッドを介して対話します。
`query()` のために、Pythonのビルトインである `help()` 関数を使い、有効なドキュメントを確認してみましょう：

```python
  >>> import spack.store
  >>> help(spack.store.db.query)
  Help on method query in module spack.database:

  query(*args, **kwargs) method of spack.database.Database instance
      Query the Spack database including all upstream databases.

      Args:
          query_spec: queries iterate through specs in the database and
              return those that satisfy the supplied ``query_spec``. If
              query_spec is `any`, This will match all specs in the
              database.  If it is a spec, we'll evaluate
              ``spec.satisfies(query_spec)``

          known (bool or any, optional): Specs that are "known" are those
              for which Spack can locate a ``package.py`` file -- i.e.,
              Spack "knows" how to install them.  Specs that are unknown may
              represent packages that existed in a previous version of
              Spack, but have since either changed their name or
              been removed

          installed (bool or any, or InstallStatus or iterable of
              InstallStatus, optional): if ``True``, includes only installed
              specs in the search; if ``False`` only missing specs, and if
              ``any``, all specs in database. If an InstallStatus or iterable
              of InstallStatus, returns specs whose install status
              (installed, deprecated, or missing) matches (one of) the
              InstallStatus. (default: True)

          explicit (bool or any, optional): A spec that was installed
              following a specific user request is marked as explicit. If
              instead it was pulled-in as a dependency of a user requested
              spec it's considered implicit.

          start_date (datetime, optional): filters the query discarding
              specs that have been installed before ``start_date``.

          end_date (datetime, optional): filters the query discarding
              specs that have been installed after ``end_date``.

          hashes (container): list or set of hashes that we can use to
              restrict the search

      Returns:
          list of specs that match the query
  (END)
```

私たちは主に引数 `query_spec` を活用します。

`spack find` コマンドは、値を属性や値を所持して_いない_属性の照会に限定されていることを思い出してください。
つまり、特定の基準を満たさないすべてのパッケージに `spack find` コマンドを使用できるわけではありません。

これらのタイプの照会を記述するためにPythonインタフェースを使用します。
たとえば `gcc` でコンパイルされた`mpich`に依存しないすべてのパッケージを見つけようとします。
これは、カスタムPythonコードとSpackデータベース照会を使用して実行できます。
`spack.cmd.display_specs` を出力として使い、`spack find` コマンドと同じ出力機能を実現します：

```python
  >>> gcc_query_spec = Spec('%gcc')
  >>> gcc_specs = spack.store.db.query(gcc_query_spec)
  >>> result = filter(lambda spec: not spec.satisfies('^mpich'), gcc_specs)
  >>> import spack.cmd
  >>> spack.cmd.display_specs(result)
  -- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
  autoconf@2.69    libiconv@1.16        m4@1.4.18       perl@5.30.3         zlib@1.2.11
  automake@1.16.2  libpciaccess@0.13.5  ncurses@6.2     pkgconf@1.7.3
  gdbm@1.18.1      libsigsegv@2.12      numactl@2.0.12  readline@8.0
  hdf5@1.10.6      libtool@2.4.6        openmpi@3.1.6   util-macros@1.19.1
  hwloc@1.11.11    libxml2@2.9.10       patchelf@0.10   xz@5.2.5
```

これで、`spack find` コマンドでは利用できない強力な照会ができました。

再利用のための機能を汎化するまえに、インタプリタを終了して、コマンドラインに戻っておきましょう：

```python
>>> exit()
```

## スクリプトの使用

次に、コマンドラインで引数を受け入れるようにスクリプトをパラメータ化してみましょう。
`include` 仕様と `exclude` 仕様を引数として使用するためのいくつかの一般化により、強力な汎用照会スクリプトを作成できます。

好きなエディタで `find_exclude.py` ファイルを開き、次のコードを追加します：

```python
from spack.spec import Spec
import spack.store
import spack.cmd
import sys

include_spec = Spec(sys.argv[1])
exclude_spec = Spec(sys.argv[2])

all_included = spack.store.db.query(include_spec)
result = filter(lambda spec: not spec.satisfies(exclude_spec), all_included)

spack.cmd.display_specs(result)
```

システムパッケージ `sys` をインポートして使用することで、1番目と2番目のコマンドライン引数にアクセスしていることに注意してください。

これで、次のように入力して新しいスクリプトを実行できます：

```bash
$ spack python find_exclude.py %gcc ^mpich
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69	     gdbm@1.18.1    libiconv@1.16      libtool@2.4.6   ncurses@6.2     patchelf@0.10  readline@8.0	  zlib@1.2.11
automake@1.16.2      hdf5@1.10.7    libpciaccess@0.16  libxml2@2.9.10  numactl@2.0.14  perl@5.32.0    util-macros@1.19.1
berkeley-db@18.1.40  hwloc@1.11.11  libsigsegv@2.12    m4@1.4.18       openmpi@3.1.6   pkgconf@1.7.3  xz@5.2.5
```

@spack python` コマンドの利用・実行を覚えておくことは、素晴らしいことです。

## 実行可能ファイル `spack-python` の使用

スクリプトを他の人が使用できるようにしたい場合は、`spack python` を覚えておく必要はありません。

Python実行可能ファイルの最初の行として通常追加される骨組み行（`#!/usr/bin/env ..`の行のこと）を利用できます。
しかし、すぐにわかるとおもいますが、落とし穴があります。

上記で作成した `find_exclude.py` スクリプトを好きなエディタで開いて、`env`の引数として`spack python`を指定した骨組み行を追加します：

```python
#!/usr/bin/env spack python
from spack.spec import Spec
import spack.store
import spack.cmd
import sys

include_spec = Spec(sys.argv[1])
exclude_spec = Spec(sys.argv[2])

all_included = spack.store.db.query(include_spec)
result = filter(lambda spec: not spec.satisfies(exclude_spec), all_included)

spack.cmd.display_specs(result)
```

次に、エディタを終了して、次のように実行前にスクリプトに実行権限を追加します：

```bash
$ chmod u+x find_exclude.py
$ ./find_exclude.py %gcc ^mpich
/usr/bin/env: 'spack python': No such file or directory
```

運が良ければ、それはあなたのシステムで動作しますが、保証はありません。
一部のシステムは、骨組み行では単一の引数のみサポートします（[ここ](https://www.in-ulm.de/~mascheck/various/shebang/)を参照）。
`spack python` のラッパスクリプトである `spack-python` はこの問題を解決します。

エディタでファイルを再度表示し、`env`引数を次のように `spack-python` に変更します：

```bash
#!/usr/bin/env spack-python
from spack.spec import Spec
import spack.store
import spack.cmd
import sys

include_spec = Spec(sys.argv[1])
exclude_spec = Spec(sys.argv[2])

all_included = spack.store.db.query(include_spec)
result = filter(lambda spec: not spec.satisfies(exclude_spec), all_included)

spack.cmd.display_specs(result)
```

エディタを終了して、スクリプトをもう一度実行してみましょう。

```bash
$ ./find_exclude.py %gcc ^mpich
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
autoconf@2.69	     gdbm@1.18.1    libiconv@1.16      libtool@2.4.6   ncurses@6.2     patchelf@0.10  readline@8.0	  zlib@1.2.11
automake@1.16.2      hdf5@1.10.7    libpciaccess@0.16  libxml2@2.9.10  numactl@2.0.14  perl@5.32.0    util-macros@1.19.1
berkeley-db@18.1.40  hwloc@1.11.11  libsigsegv@2.12    m4@1.4.18       openmpi@3.1.6   pkgconf@1.7.3  xz@5.2.5
```

おめでとう！
これで Spackがインストールされているすべてのシステムで動作するようになりました。

これで、独自のカスタムSpack照会とプロトタイプのアイデアを作成するための基本的なツールができました。
あなたがそれらのうちのいくつかを Spack に寄付してくれることを願っています。
