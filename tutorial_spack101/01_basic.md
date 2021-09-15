# 基本的なインストールチュートリアル

このチュートリアルでは Spackをつかってソフトウェアをインストールするプロセスについて解説します。
最初に `spack install` コマンドについて説明し、仕様構文の能力とそれが利用者に与える柔軟性にフォーカスします。
また、`spack find` コマンドによるインストールされたパッケージ表示や `spack uninstall` コマンドをカバーします。
最後に、Spackがコンパイラを管理する方法、特にSpack内でビルドコンパイラの使用に関連する方法について説明します。

すべてのコマンドの完全な出力を掲示していますが、その出力のごく一部(または単に成功したという事)にのみ注意を喚起することがよくあります。
なお、出力例はすべて AWS上のUbuntu18.04インスタンスのものです。

## Spack インストール

Spackはボックス外で動作します。Spackのクローンを作成してはじめます。次のように実行し、最新版(2021年8月30日時点v0.16)をチェックアウトします：

```bash
$ git clone https://github.com/spack/spack ~/spack
Cloning into '/home/spack/spack'...
remote: Enumerating objects: 275937, done.K
remote: Counting objects: 100% (923/923), done.K
remote: Compressing objects: 100% (570/570), done.K
remote:nTotale2759379(delta3309),7reused4738.(delta|205),2pack-reused 275014K
Receiving objects: 100% (275937/275937), 111.88 MiB | 2.29 MiB/s, done.
Resolving deltas: 100% (116670/116670), done.
```

```bash
$ cd ~/spack
$ git checkout releases/v0.16
Branch 'releases/v0.16' set up to track remote branch 'releases/v0.16' from 'origin'.
Switched to a new branch 'releases/v0.16'
```

次に、Spackをパスに追加します。
Spackにはいくつかの優れたコマンドライン統合ツールがあり、環境変数 PATH に単に追加するのではなく、次のようにSpackセットアップスクリプトを実行します：

```bash
. share/spack/setup-env.sh
```

## Spack とは？

`spack list` コマンドを実行すると、有効なパッケージを表示します。

```bash
$ spack list
==> 5063 packages.
3proxy			   libisal				     py-cython				r-geor
abduco			   libiscsi				     py-cyvcf2				r-geosphere
abi-compliance-checker	   libjpeg				     py-d2to1				r-getopt
abi-dumper		   libjpeg-turbo			     py-dask				r-getoptlong
```

`spack list` コマンドは、照会文字列も引数にとることができます。
Spackは文字列両端にワイルドカードを自動的に追加しますが、明示的に指定することもできます。例えば、有効なPythonパッケージをすべて参照するには、次のように実行します：

```bash
$ spack list 'py-*'
==> 1246 packages.
py-3to2 				  py-cheroot		       py-hypothesis		  py-planar			py-scp
py-4suite-xml				  py-cherrypy		       py-ics			  py-planet			py-scs
py-abipy				  py-chronyk		       py-identify		  py-plotly			py-seaborn
py-absl-py				  py-cinema-lib 	       py-idna			  py-pluggy			py-secretstorage
```

## パッケージのインストール

Spackを使ったパッケージのインストールはとても簡単です。
１つのパッケージをインストールするには単に `spack install <package_name>` とタイプします。

```bash
$ spack install zlib
==> Installing zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> No binary for zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg found: installing from source
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/c3/c3e5e9fdd5004dcb542feda5ee4f0ff0744628baf8ed2dd5d66f8ca1197cb1a1.tar.gz
 ###################################################################################################################################################################### 100.0%
==> zlib: Executing phase: 'install'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
```

Spackは、ソースもしくはバイナリキャッシュからソフトウェアをインストールすることができます。
バイナリキャッシュ内のパッケージは、セキュリティのためにGPGで署名されています。
このチュートリアルでは、バイナリキャッシュを用意しているので、ソースからの遅いコンパイルを待つ必要はありません。
バイナリキャッシュからインストールできるようにするには、バイナリキャッシュの場所を使用してSpackを構成し、バイナリキャッシュが署名されたGPGキーを信頼する必要があります。

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

同じ `spack install` コマンドを使用してバイナリキャッシュからチュートリアルの残りのパッケージをインストールできます（チュートリアルの後半でSpackの構成について詳しく学習します）。
デフォルトでは、バイナリキャッシュバージョンが存在する場合はそちらからインストールされ、存在しない場合はソースからのインストールにフォールバックします。

Spackの仕様構文は、パッケージの特定の構成を指定可能にするインターフェースです。
`%` はコンパイラを指定するために使用されます。

```bash
$ spack install zlib %clang
==> Installing zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.11/linux-ubuntu18.04-x86_64-clang-6.0.0-zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla from binary cache
==> Installing patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
==> No binary for patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp found: installing from source
==> Fetching https://spack-llnl-mirror.s3-us-west-2.amazonaws.com/_source-cache/archive/b2/b2deabce05c34ce98558c0efb965f209de592197b2c88e930298d740ead09019.tar.gz
 ###################################################################################################################################################################### 100.0%
==> patchelf: Executing phase: 'autoreconf'
==> patchelf: Executing phase: 'configure'
==> patchelf: Executing phase: 'build'
==> patchelf: Executing phase: 'install'
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla
```

このインストレーションは、既存のインストレーションとは別に配置されていることに注意してください。
これについては後で詳しく説明しますが、これはSpackが任意のバージョンのソフトウェアをサポートできるようにするための機能の一部です。

インストールを指示する前に、特定のバージョンを確認できます。 `spack versions` コマンドを使用して使用可能な `zlib` のバージョンを確認してから、別のバージョンをインストールします。

```bash
$ spack versions zlib
==> Safe versions (already checksummed):
  1.2.11  1.2.8  1.2.3
==> Remote versions (not yet checksummed):
  1.2.10   1.2.7.1  1.2.5.3  1.2.4.5  1.2.4.1  1.2.3.7	1.2.3.3  1.2.2.3  1.2.1.2  1.2.0.7  1.2.0.3  1.1.4  1.1.0  1.0.6  1.0.1    0.94  0.79
  1.2.9    1.2.7    1.2.5.2  1.2.4.4  1.2.4    1.2.3.6	1.2.3.2  1.2.2.2  1.2.1.1  1.2.0.6  1.2.0.2  1.1.3  1.0.9  1.0.5  1.0-pre  0.93  0.71
  1.2.7.3  1.2.6.1  1.2.5.1  1.2.4.3  1.2.3.9  1.2.3.5	1.2.3.1  1.2.2.1  1.2.1    1.2.0.5  1.2.0.1  1.1.2  1.0.8  1.0.4  0.99	   0.92  0.9
  1.2.7.2  1.2.6    1.2.5    1.2.4.2  1.2.3.8  1.2.3.4	1.2.2.4  1.2.2	  1.2.0.8  1.2.0.4  1.2.0    1.1.1  1.0.7  1.0.2  0.95	   0.91  0.8
```

```bash
$ spack install zlib %gcc@6.5.0
==> Installing zlib-1.2.11-qtrzwovqizyaw2clz5rkhoxr3j5mbrxy
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-6.5.0/zlib-1.2.11/linux-ubuntu18.04-x86_64-gcc-6.5.0-zlib-1.2.11-qtrzwovqizyaw2clz5rkhoxr3j5mbrxy.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.11-qtrzwovqizyaw2clz5rkhoxr3j5mbrxy from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-6.5.0/zlib-1.2.11-qtrzwovqizyaw2clz5rkhoxr3j5mbrxy
```

`@` はパッケージのとコンパイラの両方のバージョンを指定するために使用されます。

```bash
$ spack install zlib@1.2.8
==> Installing zlib-1.2.8-t4p5jnme6csjrx4vbnla5za7ghcctg76
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8/linux-ubuntu18.04-x86_64-gcc-7.5.0-zlib-1.2.8-t4p5jnme6csjrx4vbnla5za7ghcctg76.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.8-t4p5jnme6csjrx4vbnla5za7ghcctg76 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8-t4p5jnme6csjrx4vbnla5za7ghcctg76
```

コンパイラフラグも指定可能です。Spack は `cppflags` 、 `cflags` 、 `cxxflags` 、 `fflags` 、 `ldflags` 、および `ldlibs` パラメータを指定することができます。
コマンドラインでこれらの値にスペースが含めたい場合は、引用符で囲む必要があります。
これらの値は Spack コンパイララッパによって自動的にコンパイル行に挿入されます。

```bash
$ spack install zlib@1.2.8 cppflags=-O3
==> Installing zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8/linux-ubuntu18.04-x86_64-gcc-7.5.0-zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps
```

`spack find` コマンドは、インストールされているパッケージを照会するために使います。
一部のパッケージは一見するとデフォルト出力と同じように見えることに注意してください。
`-l` フラグを追加すると、各パッケージのハッシュを表示します。
`-f` グラグを追加すると、各パッケージのインストール時に指定したコンパイラフラグを表示します。

```bash
$ spack find
==> 6 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@6.5.0 -------------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
patchelf@0.10  zlib@1.2.8  zlib@1.2.8  zlib@1.2.11
```

Spackは、仕様ごとにハッシュを生成します。
このハッシュはパッケージの完全な来歴の関数であるため、仕様を変更するとハッシュに影響します。
Spackはこの値を使用して、仕様を比較し、すべての組み合わせバージョンに固有のインストールディレクトリを生成します。
ソフトウェアの依存関係がより複雑なパッケージをインストールする際は、既存パッケージのハッシュが目的の仕様に一致する場合にのみ、Spackは既存パッケージを再利用が可能になります。


```bash
$ spack install tcl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Installing tcl-8.6.10-rtz76647rejri5fkpdmnthttncv2vgkx
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10/linux-ubuntu18.04-x86_64-gcc-7.5.0-tcl-8.6.10-rtz76647rejri5fkpdmnthttncv2vgkx.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting tcl-8.6.10-rtz76647rejri5fkpdmnthttncv2vgkx from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10-rtz76647rejri5fkpdmnthttncv2vgkx
```

依存関係は `^` を指定します。
なお、指定は再帰的であることに注意してください。
トップレベルパッケージについては `^` を使うことで依存関係を指定することができます。

```bash
$ spack install tcl ^zlib@1.2.8 %clang
==> Installing zlib-1.2.8-pdfmc5xxsopitln5y7wcirbmfz47o5tp
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.8/linux-ubuntu18.04-x86_64-clang-6.0.0-zlib-1.2.8-pdfmc5xxsopitln5y7wcirbmfz47o5tp.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting zlib-1.2.8-pdfmc5xxsopitln5y7wcirbmfz47o5tp from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.8-pdfmc5xxsopitln5y7wcirbmfz47o5tp
==> Installing tcl-8.6.10-yqtoq2ezfg5v4xfhgdujilfxkcfjbjx7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/clang-6.0.0/tcl-8.6.10/linux-ubuntu18.04-x86_64-clang-6.0.0-tcl-8.6.10-yqtoq2ezfg5v4xfhgdujilfxkcfjbjx7.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting tcl-8.6.10-yqtoq2ezfg5v4xfhgdujilfxkcfjbjx7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/clang-6.0.0/tcl-8.6.10-yqtoq2ezfg5v4xfhgdujilfxkcfjbjx7
```

パッケージは、ハッシュを使ってコマンドラインから参照することも可能です。
コマンド `spack find -lf` を使うと、zlib（ `cppflags="-O3"` ）の最適なインストレーションのハッシュが `h6i53if` から始まることを確認できます。
`/` を付加したハッシュで参照することにより、仕様全体を入力せずに対象のパッケージを明示的にビルドできます。
Gitのような他のツール同様、コマンドラインでハッシュ全体を指定する必要はありません。
ハッシュを一意に識別するのに十分な桁数を指定できます。
ハッシュプレフィックスがあいまいな場合（インストールされている2つ以上のパッケージがプレフィックスを共有している場合）、Spackはエラーを報告します。

```bash
$ spack install tcl ^/h6i
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps
==> Installing tcl-8.6.10-m2hc42k5bmqzqwudra5a3qx3fgyplnjr
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10/linux-ubuntu18.04-x86_64-gcc-7.5.0-tcl-8.6.10-m2hc42k5bmqzqwudra5a3qx3fgyplnjr.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting tcl-8.6.10-m2hc42k5bmqzqwudra5a3qx3fgyplnjr from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10-m2hc42k5bmqzqwudra5a3qx3fgyplnjr
```

`spack find` コマンドに `-d` フラグを指定すると、依存関係情報を参照することができます。
依存関係情報を参照するパッケージはトップレベルエントリであることに注意してください。

```bash
$ spack find -ldf
==> 10 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
yqtoq2e tcl@8.6.10%clang
pdfmc5x     zlib@1.2.8%clang

pdfmc5x zlib@1.2.8%clang

5qffmms zlib@1.2.11%clang


-- linux-ubuntu18.04-x86_64 / gcc@6.5.0 -------------------------
qtrzwov zlib@1.2.11%gcc


-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
2nj3vxx patchelf@0.10%gcc

m2hc42k tcl@8.6.10%gcc
h6i53if     zlib@1.2.8%gcc  cppflags="-O3"

rtz7664 tcl@8.6.10%gcc
smoyzzo     zlib@1.2.11%gcc

t4p5jnm zlib@1.2.8%gcc

h6i53if zlib@1.2.8%gcc	cppflags="-O3"

smoyzzo zlib@1.2.11%gcc
```

もう少し複雑なパッケージ操作に移りましょう。
HDF5は、MPIに依存するより複雑なパッケージの良いサンプルです。
「ボックス外」でインストールすると OpenMPI でビルドされます。

```bash
$ spack install hdf5
==> Installing libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12/linux-ubuntu18.04-x86_64-gcc-7.5.0-libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q from binary cache
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
```

Spackパッケージにはバリアントとよばれるビルドオプションを指定することもできます。
バリアントで真偽値を指定する場合、 接頭文字に `+` （真値）、 `~` （偽値）もしくは `-` （偽値）を指定します。
2つの `False` をあらわす接頭文字があるのは、ことなるシチュエーションにおけるシェル構文解析で衝突を避けるためです。
バリアント（真偽値もしくはその他）はコンパイラフラグとしておなじ構文を使って指定することも可能です。
次のサンプルでは、MPIサポート無しでHDF5をインストールしています。

```bash
$ spack install hdf5~mpi
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
==> Installing hdf5-1.10.7-sldl7zxtufal5qvh2zrggnxtrtava7a7
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-hdf5-1.10.7-sldl7zxtufal5qvh2zrggnxtrtava7a7.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hdf5-1.10.7-sldl7zxtufal5qvh2zrggnxtrtava7a7 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-sldl7zxtufal5qvh2zrggnxtrtava7a7
```

別のMPI実装でHDF5をSpackでインストールすることもできます。
MPI自体はパッケージではありませんが、パッケージはMPIのような抽象インターフェイスに依存しています。
Spackは、"仮想依存関係（virtual dependencies）"を通じてこれらを処理します。
HDF5などのパッケージは、MPIインターフェイスに依存する場合があります。
HDF5はその他のパッケージ（ `openmpi` 、 `mpich` 、 `mvapich2` など）MPIインターフェースを使用するようにインストールすることもできます。
これらの傾向パッケージはいずれも、MPIの依存関係を要求できます。
たとえば、依存関係 `mpich` を指定することで、MPICHが提供するMPIサポートを使用してHDF5をビルドすることができます。
Spackは、仮想依存関係のバージョン管理もサポートしています。
パッケージはMPIインターフェイスバージョン3に依存でき、提供パッケージは実装するインターフェイスのバージョンを指定します。
仕様の一部指定 `^mpi@3` は、いくつかの提供パッケージのいずれかによって満たされる可能性があります。

```bash
$ spack install hdf5+hl+mpi ^mpich
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
==> Installing hwloc-2.2.0-a4cxlu767bxxceahypqpj2gjdt7hmhvm
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-2.2.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-hwloc-2.2.0-a4cxlu767bxxceahypqpj2gjdt7hmhvm.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hwloc-2.2.0-a4cxlu767bxxceahypqpj2gjdt7hmhvm from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-2.2.0-a4cxlu767bxxceahypqpj2gjdt7hmhvm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Installing texinfo-6.5-p46ba5qb5wvx6a2bneh4vlurqusz35bj
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/texinfo-6.5/linux-ubuntu18.04-x86_64-gcc-7.5.0-texinfo-6.5-p46ba5qb5wvx6a2bneh4vlurqusz35bj.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting texinfo-6.5-p46ba5qb5wvx6a2bneh4vlurqusz35bj from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/texinfo-6.5-p46ba5qb5wvx6a2bneh4vlurqusz35bj
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Installing findutils-4.6.0-hrv4dx5txigi2la7siu3qsa7cervn7tv
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/findutils-4.6.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-findutils-4.6.0-hrv4dx5txigi2la7siu3qsa7cervn7tv.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting findutils-4.6.0-hrv4dx5txigi2la7siu3qsa7cervn7tv from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/findutils-4.6.0-hrv4dx5txigi2la7siu3qsa7cervn7tv
==> Installing mpich-3.3.2-vbelg5sthuytvkxbrvue2alphviggiur
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpich-3.3.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-mpich-3.3.2-vbelg5sthuytvkxbrvue2alphviggiur.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mpich-3.3.2-vbelg5sthuytvkxbrvue2alphviggiur from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpich-3.3.2-vbelg5sthuytvkxbrvue2alphviggiur
==> Installing hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars
```

既存インストレーションを簡単に記録表示します。

```bash
$ spack find -ldf
==> 36 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
yqtoq2e tcl@8.6.10%clang
pdfmc5x     zlib@1.2.8%clang

pdfmc5x zlib@1.2.8%clang

5qffmms zlib@1.2.11%clang


-- linux-ubuntu18.04-x86_64 / gcc@6.5.0 -------------------------
qtrzwov zlib@1.2.11%gcc


-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
mm33a3o autoconf@2.69%gcc
mkc3u4x     m4@1.4.18%gcc
lbrx7ln 	libsigsegv@2.12%gcc
zfdvt2j     perl@5.32.0%gcc
4ihuiaz 	berkeley-db@18.1.40%gcc
4av4gyw 	gdbm@1.18.1%gcc
t54jzdy 	    readline@8.0%gcc
crhlefo 		ncurses@6.2%gcc

d2krmb5 automake@1.16.2%gcc
zfdvt2j     perl@5.32.0%gcc
4ihuiaz 	berkeley-db@18.1.40%gcc
4av4gyw 	gdbm@1.18.1%gcc
t54jzdy 	    readline@8.0%gcc
crhlefo 		ncurses@6.2%gcc

4ihuiaz berkeley-db@18.1.40%gcc

hrv4dx5 findutils@4.6.0%gcc

4av4gyw gdbm@1.18.1%gcc
t54jzdy     readline@8.0%gcc
crhlefo 	ncurses@6.2%gcc

sldl7zx hdf5@1.10.7%gcc
smoyzzo     zlib@1.2.11%gcc

vedchc5 hdf5@1.10.7%gcc
pmsyupw     openmpi@3.1.6%gcc
zqwfzhw 	hwloc@1.11.11%gcc
bob4o5m 	    libpciaccess@0.16%gcc
yn2r3wf 	    libxml2@2.9.10%gcc
jearpk4 		libiconv@1.16%gcc
komekkm 		xz@5.2.5%gcc
smoyzzo 		zlib@1.2.11%gcc
wbqbc5v 	    numactl@2.0.14%gcc

cq4k4cv hdf5@1.10.7%gcc
vbelg5s     mpich@3.3.2%gcc
a4cxlu7 	hwloc@2.2.0%gcc
bob4o5m 	    libpciaccess@0.16%gcc
yn2r3wf 	    libxml2@2.9.10%gcc
jearpk4 		libiconv@1.16%gcc
komekkm 		xz@5.2.5%gcc
smoyzzo 		zlib@1.2.11%gcc

zqwfzhw hwloc@1.11.11%gcc
bob4o5m     libpciaccess@0.16%gcc
yn2r3wf     libxml2@2.9.10%gcc
jearpk4 	libiconv@1.16%gcc
komekkm 	xz@5.2.5%gcc
smoyzzo 	zlib@1.2.11%gcc
wbqbc5v     numactl@2.0.14%gcc

a4cxlu7 hwloc@2.2.0%gcc
bob4o5m     libpciaccess@0.16%gcc
yn2r3wf     libxml2@2.9.10%gcc
jearpk4 	libiconv@1.16%gcc
komekkm 	xz@5.2.5%gcc
smoyzzo 	zlib@1.2.11%gcc

jearpk4 libiconv@1.16%gcc

bob4o5m libpciaccess@0.16%gcc

lbrx7ln libsigsegv@2.12%gcc

jdxbjft libtool@2.4.6%gcc

yn2r3wf libxml2@2.9.10%gcc
jearpk4     libiconv@1.16%gcc
komekkm     xz@5.2.5%gcc
smoyzzo     zlib@1.2.11%gcc

mkc3u4x m4@1.4.18%gcc
lbrx7ln     libsigsegv@2.12%gcc

vbelg5s mpich@3.3.2%gcc
a4cxlu7     hwloc@2.2.0%gcc
bob4o5m 	libpciaccess@0.16%gcc
yn2r3wf 	libxml2@2.9.10%gcc
jearpk4 	    libiconv@1.16%gcc
komekkm 	    xz@5.2.5%gcc
smoyzzo 	    zlib@1.2.11%gcc

crhlefo ncurses@6.2%gcc

wbqbc5v numactl@2.0.14%gcc

pmsyupw openmpi@3.1.6%gcc
zqwfzhw     hwloc@1.11.11%gcc
bob4o5m 	libpciaccess@0.16%gcc
yn2r3wf 	libxml2@2.9.10%gcc
jearpk4 	    libiconv@1.16%gcc
komekkm 	    xz@5.2.5%gcc
smoyzzo 	    zlib@1.2.11%gcc
wbqbc5v 	numactl@2.0.14%gcc

2nj3vxx patchelf@0.10%gcc

zfdvt2j perl@5.32.0%gcc
4ihuiaz     berkeley-db@18.1.40%gcc
4av4gyw     gdbm@1.18.1%gcc
t54jzdy 	readline@8.0%gcc
crhlefo 	    ncurses@6.2%gcc

4sh6pym pkgconf@1.7.3%gcc

t54jzdy readline@8.0%gcc
crhlefo     ncurses@6.2%gcc

rtz7664 tcl@8.6.10%gcc
smoyzzo     zlib@1.2.11%gcc

m2hc42k tcl@8.6.10%gcc
h6i53if     zlib@1.2.8%gcc  cppflags="-O3"

p46ba5q texinfo@6.5%gcc
zfdvt2j     perl@5.32.0%gcc
4ihuiaz 	berkeley-db@18.1.40%gcc
4av4gyw 	gdbm@1.18.1%gcc
t54jzdy 	    readline@8.0%gcc
crhlefo 		ncurses@6.2%gcc

gs6ag7k util-macros@1.19.1%gcc

komekkm xz@5.2.5%gcc

t4p5jnm zlib@1.2.8%gcc

h6i53if zlib@1.2.8%gcc	cppflags="-O3"

smoyzzo zlib@1.2.11%gcc
```

Spackは、パッケージの依存関係を有向非巡回グラフ（DAG）としてモデル化します。
`spack find -d` コマンドは、DAGをツリー形式で表示します。
`spack graph` コマンドを使うことで、DAG全体をグラフとして表示することもできます。

```bash
$ spack graph hdf5+hl+mpi ^mpich
o  hdf5
|\
| o  mpich
| |\
| | |\
| | | |\
| | | | |\
| | | | | |\
| | | | | | |\
| | | | | | | |\
| | | | | | | | |\
| | | | | | o | | |  hwloc
| | |_|_|_|/| | | |
| |/| | |_|/| | | |
| | | |/| |/ / / /
| | | o | | | | |  libxml2
| |_|/| | | | | |
|/| |/| | | | | |
| |/| | | | | | |
| | | |\ \ \ \ \ \
o | | | | | | | | |  zlib
 / / / / / / / / /
| | o | | | | | |  xz
| |  / / / / / /
| | | | o | | |  libpciaccess
| |_|_|/| | | |
|/| | |/| | | |
| | | | o | | |  util-macros
| | | |  / / /
| | | | o | |  findutils
| | |_|/| | |
| |/| |/| | |
| | | | |\| |
| | | | |\ \ \
| | | | | | |/
| | | | | |/|
| | | | o | |  texinfo
| | | | | | o  automake
| | | | | |/|
| | | | |/|/
| | | | | o  autoconf
| | |_|_|/|
| |/| | |/
| | | | o  perl
| | | | |\
| | | | o |  gdbm
| | | | o |  readline
| | | | o |  ncurses
| |_|_|/ /
|/| | | |
o | | | |  pkgconf
 / / / /
| | o |  libtool
| |/ /
|/| |
o | |  m4
o | |  libsigsegv
 / /
o |  libiconv
 /
o  berkeley-db
```

You may also have noticed that 
there are some packages shown in the spack find -d output that 
we didn’t install explicitly.

`spack find -d` コマンドの出力には、明示的にインストールしなかったパッケージもいくつか表示されていることに気付いたかもしれません。
これらは暗黙的にインストールされた依存関係です。
暗黙的にインストールされたいくつかのパッケージは、 `spack find -d` コマンド出力に依存関係として表示されませんが、これらはビルド時の依存関係です。

For example, libpciaccess is a dependency of openmpi and requires m4 to build.
たとえば、`libpciaccess` は `openmpi` の依存関係で、ビルドするには `m4` が必要となります。
Spackはのインストールの一部として `m4` がビルドされますが、実行時にはリンクされていないため、DAGの一部にはなりません。
Spackは、一貫性要件が異なる（それほど厳密ではない）ため、ビルドの依存関係を異なる方法で処理します。
異なるバージョンの依存関係を使用して2つのパッケージをビルドすることは完全に可能ですが、リンクされた依存関係では当然実行できません。

HDF5は、zlib や Tcl の基本的なサンプルよりも複雑ですが、経験豊富な HPC ユーザであれば少しの時間を与えられてインストールすることを合理的に期待できるソフトウェアの領域内にあります。
次に、さらに複雑なパッケージを見てみましょう。

```bash
$ spack install trilinos
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/util-macros-1.19.1-gs6ag7ktdoiirb62t7bcagjw62szrrg2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/xz-5.2.5-komekkmyciga3kl24edjmredhj3uyt7v
==> Installing openblas-0.3.12-te35tjxctgjpeeuk357xghoinvwf2k7u
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/openblas-0.3.12/linux-ubuntu18.04-x86_64-gcc-7.5.0-openblas-0.3.12-te35tjxctgjpeeuk357xghoinvwf2k7u.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting openblas-0.3.12-te35tjxctgjpeeuk357xghoinvwf2k7u from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openblas-0.3.12-te35tjxctgjpeeuk357xghoinvwf2k7u
==> Installing autoconf-archive-2019.01.06-bdyarrkdgse5adcjr4kxqc4jns74k7yn
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-archive-2019.01.06/linux-ubuntu18.04-x86_64-gcc-7.5.0-autoconf-archive-2019.01.06-bdyarrkdgse5adcjr4kxqc4jns74k7yn.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting autoconf-archive-2019.01.06-bdyarrkdgse5adcjr4kxqc4jns74k7yn from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-archive-2019.01.06-bdyarrkdgse5adcjr4kxqc4jns74k7yn
==> Installing diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libxml2-2.9.10-yn2r3wfhiilelyulh5toteicdtxjhw7d
==> Installing bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8/linux-ubuntu18.04-x86_64-gcc-7.5.0-bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/bzip2-1.0.8-fvfpt26p5divvq6344vojyn64enojlui
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
==> Installing boost-1.74.0-hc5atqce4emwrrkxuyfayf5k43r5cst2
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.74.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-boost-1.74.0-hc5atqce4emwrrkxuyfayf5k43r5cst2.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting boost-1.74.0-hc5atqce4emwrrkxuyfayf5k43r5cst2 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/boost-1.74.0-hc5atqce4emwrrkxuyfayf5k43r5cst2
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libpciaccess-0.16-bob4o5m3uku6vtdil5imasprgy775zg7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
==> Installing openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h/linux-ubuntu18.04-x86_64-gcc-7.5.0-openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
==> Installing cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4/linux-ubuntu18.04-x86_64-gcc-7.5.0-cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j from binary cache
==> Warning: patchelf --print-rpath /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in produced an error [Command exited with status 1:
    '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp/bin/patchelf' '--print-rpath' '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in'
patchelf: not an ELF executable
]
==> Warning: patchelf --force-rpath --set-rpath /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in failed with error Command exited with status 1:
    '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp/bin/patchelf' '--force-rpath' '--set-rpath' '' '/home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j/share/cmake-3.18/Modules/Internal/CPack/CPack.OSXScriptLauncher.in'
patchelf: not an ELF executable

[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
==> Installing metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/metis-5.1.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr
==> Installing glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/glm-0.9.7.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg
==> Installing gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/gmp-6.1.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/numactl-2.0.14-wbqbc5vw5sxzwhvu56p6x5nd5n4abrvh
==> Installing mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-4.0.2/linux-ubuntu18.04-x86_64-gcc-7.5.0-mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-1.11.11-zqwfzhw5k2ollygh6nrjpsi7u4d4g6lu
==> Installing suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/suite-sparse-5.8.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openmpi-3.1.6-pmsyupw6w3gql4loaor25gfumlmvkl25
==> Installing hypre-2.20.0-nndket2abxbsbuoocew5m643e36waxj3
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hypre-2.20.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-hypre-2.20.0-nndket2abxbsbuoocew5m643e36waxj3.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hypre-2.20.0-nndket2abxbsbuoocew5m643e36waxj3 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hypre-2.20.0-nndket2abxbsbuoocew5m643e36waxj3
==> Installing parmetis-4.0.3-htdsyvt5yp6vl3nxbxp3627gohe7ldup
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/parmetis-4.0.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-parmetis-4.0.3-htdsyvt5yp6vl3nxbxp3627gohe7ldup.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting parmetis-4.0.3-htdsyvt5yp6vl3nxbxp3627gohe7ldup from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/parmetis-4.0.3-htdsyvt5yp6vl3nxbxp3627gohe7ldup
==> Installing hdf5-1.10.7-ejanivxt2gsv2qd3so7fil7izp4m3bcd
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7/linux-ubuntu18.04-x86_64-gcc-7.5.0-hdf5-1.10.7-ejanivxt2gsv2qd3so7fil7izp4m3bcd.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hdf5-1.10.7-ejanivxt2gsv2qd3so7fil7izp4m3bcd from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-ejanivxt2gsv2qd3so7fil7izp4m3bcd
==> Installing netlib-scalapack-2.1.0-wsiydak4jm7sms57zcu4rmzmdmgxqb5j
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/netlib-scalapack-2.1.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-netlib-scalapack-2.1.0-wsiydak4jm7sms57zcu4rmzmdmgxqb5j.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting netlib-scalapack-2.1.0-wsiydak4jm7sms57zcu4rmzmdmgxqb5j from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netlib-scalapack-2.1.0-wsiydak4jm7sms57zcu4rmzmdmgxqb5j
==> Installing matio-1.5.17-zoqjxgkw7h3yoxbagripnd6muniihwno
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/matio-1.5.17/linux-ubuntu18.04-x86_64-gcc-7.5.0-matio-1.5.17-zoqjxgkw7h3yoxbagripnd6muniihwno.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting matio-1.5.17-zoqjxgkw7h3yoxbagripnd6muniihwno from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/matio-1.5.17-zoqjxgkw7h3yoxbagripnd6muniihwno
==> Installing netcdf-c-4.7.4-yixejlymvzs5aduzazrbs37hugw7sl7g
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/netcdf-c-4.7.4/linux-ubuntu18.04-x86_64-gcc-7.5.0-netcdf-c-4.7.4-yixejlymvzs5aduzazrbs37hugw7sl7g.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting netcdf-c-4.7.4-yixejlymvzs5aduzazrbs37hugw7sl7g from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netcdf-c-4.7.4-yixejlymvzs5aduzazrbs37hugw7sl7g
==> Installing mumps-5.3.3-3jmjrzw7kx44fyoltswqx7uhu33zikhm
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mumps-5.3.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-mumps-5.3.3-3jmjrzw7kx44fyoltswqx7uhu33zikhm.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mumps-5.3.3-3jmjrzw7kx44fyoltswqx7uhu33zikhm from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mumps-5.3.3-3jmjrzw7kx44fyoltswqx7uhu33zikhm
==> Installing trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1-qmjbxf2kamskoplhqlejy3esksrquni2
```


ここまでで、だんだんと Spackの力を理解し始めていると思います。
デフォルト構成の Trilinos には23のトップレベルの依存関係があり、その多くには独自の依存関係があります。
より複雑なパッケージのインストールは、経験豊富なユーザでも数日から数週間かかる場合があります。
チュートリアルのバイナリインストールを実行しましたが、Spackを使用したTrilinosのソースインストールには約3時間かかります（システムによって異なります）が、プログラマの時間はわずか20秒です（コマンドを入力して実行開始するだけ）。

Spackは、DAG全体の一貫性を管理します。
すべてのMPI依存関係は、MPIの同じ構成などによって満たされます。
Trilinos を再度インストールする場合、MPICHで構築された以前のHDF5への依存関係を指定します。

```bash
$ spack install trilinos+hdf5 ^hdf5+hl+mpi ^mpich
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
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hwloc-2.2.0-a4cxlu767bxxceahypqpj2gjdt7hmhvm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/openssl-1.1.1h-es377uqsqougfc67jyg7yfjyyuukin52
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/texinfo-6.5-p46ba5qb5wvx6a2bneh4vlurqusz35bj
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/cmake-3.18.4-bltycqwh5oofai4f6o42q4uuj4w5zb3j
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/glm-0.9.7.1-n4xti2gmzctc2oskcjphmruidrna4vfg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/metis-5.1.0-edcrft4l4szpqji2kowdkzq2ofueh3dr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/findutils-4.6.0-hrv4dx5txigi2la7siu3qsa7cervn7tv
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-4.0.2-mpv2v7v7dasis5nbrrnmci5tc2rrca2y
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpich-3.3.2-vbelg5sthuytvkxbrvue2alphviggiur
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/suite-sparse-5.8.1-rdfmg5b5lobcgmho4yqzd7zq7wibbelk
==> Installing parmetis-4.0.3-i7otoakvxjvvg4vy3pu6sckkx2sw5jys
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/parmetis-4.0.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-parmetis-4.0.3-i7otoakvxjvvg4vy3pu6sckkx2sw5jys.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting parmetis-4.0.3-i7otoakvxjvvg4vy3pu6sckkx2sw5jys from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/parmetis-4.0.3-i7otoakvxjvvg4vy3pu6sckkx2sw5jys
==> Installing netlib-scalapack-2.1.0-h7suxc3q3jvwpdka2e2zlmyqg2qn6nri
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/netlib-scalapack-2.1.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-netlib-scalapack-2.1.0-h7suxc3q3jvwpdka2e2zlmyqg2qn6nri.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting netlib-scalapack-2.1.0-h7suxc3q3jvwpdka2e2zlmyqg2qn6nri from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netlib-scalapack-2.1.0-h7suxc3q3jvwpdka2e2zlmyqg2qn6nri
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars
==> Installing hypre-2.20.0-us6giiixlzh6algkngco2m3sp5qt42xb
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/hypre-2.20.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-hypre-2.20.0-us6giiixlzh6algkngco2m3sp5qt42xb.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting hypre-2.20.0-us6giiixlzh6algkngco2m3sp5qt42xb from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hypre-2.20.0-us6giiixlzh6algkngco2m3sp5qt42xb
==> Installing mumps-5.3.3-rmlof2fllv34xs6slcd5aiqfxwwfhcx3
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mumps-5.3.3/linux-ubuntu18.04-x86_64-gcc-7.5.0-mumps-5.3.3-rmlof2fllv34xs6slcd5aiqfxwwfhcx3.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mumps-5.3.3-rmlof2fllv34xs6slcd5aiqfxwwfhcx3 from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mumps-5.3.3-rmlof2fllv34xs6slcd5aiqfxwwfhcx3
==> Installing matio-1.5.17-usmrkquiytrw7ckm6tvygexfcoc3emjq
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/matio-1.5.17/linux-ubuntu18.04-x86_64-gcc-7.5.0-matio-1.5.17-usmrkquiytrw7ckm6tvygexfcoc3emjq.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting matio-1.5.17-usmrkquiytrw7ckm6tvygexfcoc3emjq from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/matio-1.5.17-usmrkquiytrw7ckm6tvygexfcoc3emjq
==> Installing netcdf-c-4.7.4-spkgtxp7pq6beoz4uzrpmntvk3fmjnvo
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/netcdf-c-4.7.4/linux-ubuntu18.04-x86_64-gcc-7.5.0-netcdf-c-4.7.4-spkgtxp7pq6beoz4uzrpmntvk3fmjnvo.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting netcdf-c-4.7.4-spkgtxp7pq6beoz4uzrpmntvk3fmjnvo from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/netcdf-c-4.7.4-spkgtxp7pq6beoz4uzrpmntvk3fmjnvo
==> Installing trilinos-13.0.1-xvzpvxdt7x66bifz4kg3t7jvsqewjytw
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1/linux-ubuntu18.04-x86_64-gcc-7.5.0-trilinos-13.0.1-xvzpvxdt7x66bifz4kg3t7jvsqewjytw.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting trilinos-13.0.1-xvzpvxdt7x66bifz4kg3t7jvsqewjytw from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1-xvzpvxdt7x66bifz4kg3t7jvsqewjytw
```

MPI に依存する Trilinos DAG のすべてのパッケージが MPICH を使用するようになりました。

```bash
$ spack find -d trilinos
==> 2 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
trilinos@13.0.1
    boost@1.74.0
	bzip2@1.0.8
	zlib@1.2.11
    glm@0.9.7.1
    hdf5@1.10.7
	mpich@3.3.2
	    hwloc@2.2.0
		libpciaccess@0.16
		libxml2@2.9.10
		    libiconv@1.16
		    xz@5.2.5
    hypre@2.20.0
	openblas@0.3.12
    matio@1.5.17
    metis@5.1.0
    mumps@5.3.3
	netlib-scalapack@2.1.0
    netcdf-c@4.7.4
    parmetis@4.0.3
    suite-sparse@5.8.1
	gmp@6.1.2
	mpfr@4.0.2

trilinos@13.0.1
    boost@1.74.0
	bzip2@1.0.8
	zlib@1.2.11
    glm@0.9.7.1
    hdf5@1.10.7
	openmpi@3.1.6
	    hwloc@1.11.11
		libpciaccess@0.16
		libxml2@2.9.10
		    libiconv@1.16
		    xz@5.2.5
		numactl@2.0.14
    hypre@2.20.0
	openblas@0.3.12
    matio@1.5.17
    metis@5.1.0
    mumps@5.3.3
	netlib-scalapack@2.1.0
    netcdf-c@4.7.4
    parmetis@4.0.3
    suite-sparse@5.8.1
	gmp@6.1.2
	mpfr@4.0.2
```

既に解説しましたが、 `spack find -d` コマンドは依存関係情報をツリーとして表示します。
多くの場合これで十分ですが、 Trilinos を含む多くの複雑なパッケージには、ツリーとして完全に表現できない依存関係があります。
この場合も、 ``spack graph` コマンドは依存関係情報の完全なDAGを表示します。

```bash
$ spack graph trilinos
o  trilinos
|\
| |\
| | |\
| | | |\
| | | | |\
| | | | | |\
| | | | | | |\
| | | | | | | |\
| | | | | | | | |\
| | | | | | | | | |\
| | | | | | | | | | |\
| | | | | | | | | | | |\
| | | | | | | | | | | | |\
o | | | | | | | | | | | | |  suite-sparse
|\ \ \ \ \ \ \ \ \ \ \ \ \ \
| |_|_|/ / / / / / / / / / /
|/| | | | | | | | | | | | |
| |\ \ \ \ \ \ \ \ \ \ \ \ \
| | |\ \ \ \ \ \ \ \ \ \ \ \ \
| | | |_|_|_|_|_|/ / / / / / /
| | |/| | | | | | | | | | | |
| | | |\ \ \ \ \ \ \ \ \ \ \ \
| | | | |\ \ \ \ \ \ \ \ \ \ \ \
| | | | | | |_|_|_|_|_|_|_|_|/ /
| | | | | |/| | | | | | | | | |
| | | | | | o | | | | | | | | |  parmetis
| | | |_|_|/| | | | | | | | | |
| | |/| | |/| | | | | | | | | |
| | | | | | |/ / / / / / / / /
| | | | | | | | | o | | | | |  mumps
| |_|_|_|_|_|_|_|/| | | | | |
|/| | | | | | |_|/| | | | | |
| | | | | | |/| |/ / / / / /
| | | | | | | |/| | | | | |
| | | | | | | o | | | | | |  netlib-scalapack
| |_|_|_|_|_|/| | | | | | |
|/| | | | | |/| | | | | | |
| | | | | |/|/ / / / / / /
| | o | | | | | | | | | |  metis
| | | |_|/ / / / / / / /
| | |/| | | | | | | | |
| | | | | | | | | | o |  glm
| | | |_|_|_|_|_|_|/ /
| | |/| | | | | | | |
| | o | | | | | | | |  cmake
| | |\ \ \ \ \ \ \ \ \
| | o | | | | | | | | |  openssl
| | |\ \ \ \ \ \ \ \ \ \
| | | | | | | | o | | | |  netcdf-c
| | | |_|_|_|_|/| | | | |
| | |/| | | |_|/| | | | |
| | | | | |/| |/| | | | |
| | | | | | | | | |_|/ /
| | | | | | | | |/| | |
| | | | | | | | | o | |  matio
| | | |_|_|_|_|_|/| | |
| | |/| | | | | |/ / /
| | | | | | | | | o |  hypre
| |_|_|_|_|_|_|_|/| |
|/| | | | | | | |/ /
| | | | | | | |/| |
| | | | | | | | o |  hdf5
| | | |_|_|_|_|/| |
| | |/| | | | |/ /
| | | | | | | o |  openmpi
| | | |_|_|_|/| |
| | |/| | | | | |
| | | | | | | |\ \
| | | | | | | | |\ \
| | | | | | | | | o |  hwloc
| | | | | | | | |/| |
| | | | | | | |/|/| |
| | | | | | | | | |\ \
| | | | | | | | | o | |  libxml2
| | | |_|_|_|_|_|/| | |
| | |/| | | | | |/| | |
| | | | | | | |/| | | |
| | | | | | | | | |\ \ \
| | | | | | | | | | | | o  boost
| | | |_|_|_|_|_|_|_|_|/|
| | |/| | | | | | | | | |
| | o | | | | | | | | | |  zlib
| |  / / / / / / / / / /
| | | | | | | | o | | |  xz
| | | | | | | |  / / /
| | | | | | | | | o |  libpciaccess
| | | | | | | |_|/| |
| | | | | | |/| | | |
| | | | | | | | | |\ \
| | | | | | | | | o | |  util-macros
| | | | | | | | |  / /
| | | | | | | o | | |  numactl
| | | | | |_|/| | | |
| | | | |/| | | | | |
| | | | | | | |\ \ \ \
| | | | | | | | |_|/ /
| | | | | | | |/| | |
| | | | | | | | |\ \ \
| o | | | | | | | | | |  mpfr
| |\ \ \ \ \ \ \ \ \ \ \
| | |_|_|/ / / / / / / /
| |/| | | | | | | | | |
| | |\ \ \ \ \ \ \ \ \ \
| | | |_|_|_|_|/ / / / /
| | |/| | | | | | | | |
| | | |\ \ \ \ \ \ \ \ \
| | | | |_|_|/ / / / / /
| | | |/| | | | | | | |
| | | | |\ \ \ \ \ \ \ \
| | | | | |_|_|_|/ / / /
| | | | |/| | | | | | |
| | | | | |\ \ \ \ \ \ \
| | | | | | | |_|_|/ / /
| | | | | | |/| | | | |
| | | o | | | | | | | |  gmp
| | |/| | | | | | | | |
| |/|/| | | | | | | | |
| | | |\| | | | | | | |
| | | | |_|/ / / / / /
| | | |/| | | | | | |
| | | | o | | | | | |  automake
| | | |/| | | | | | |
| | | | | |/ / / / /
| | | | |/| | | | |
| | | o | | | | | |  autoconf
| | |/| | | | | | |
| |/| | | | | | | |
| | | |/ / / / / /
| | | o | | | | |  perl
| | | |\ \ \ \ \ \
| | | o | | | | | |  gdbm
| | | o | | | | | |  readline
| | | | |_|/ / / /
| | | |/| | | | |
| | | o | | | | |  ncurses
| | | | |_|/ / /
| | | |/| | | |
| | | o | | | |  pkgconf
| | |  / / / /
o | | | | | |  openblas
 / / / / / /
| o | | | |  libtool
|/ / / / /
o | | | |  m4
o | | | |  libsigsegv
 / / / /
| | | o  bzip2
| | | o  diffutils
| | |/
| | o  libiconv
| |
o |  berkeley-db
 /
o  autoconf-archive
```

いくつかのオプションを使用して、出力の表示方法を制御できます。

`spack graph` コマンドのASCII出力は、複雑なパッケージだと解析が難しい場合があります。
`--dot` オプションを使うと、出力をGraphviz `.dot` 形式に変更できます 。


## パッケージのアンインストール

ここまでのチュートリアルでは zlib と Tcl についてたくさんの構成をインストールしてきました。
次に、実際には必要のないパッケージのいくつかを調べてアンインストールします。

```bash
$ spack find -d tcl
==> 3 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
tcl@8.6.10
    zlib@1.2.8


-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
tcl@8.6.10
    zlib@1.2.8

tcl@8.6.10
    zlib@1.2.11
```

```bash
$ spack find zlib
==> 6 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.8  zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@6.5.0 -------------------------
zlib@1.2.11

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
zlib@1.2.8  zlib@1.2.8	zlib@1.2.11
```

installと同じ構文を使って、仕様ごとにパッケージをアンインストールできます。


```bash
$ spack uninstall zlib %gcc@6.5.0
==> The following packages will be uninstalled:

    -- linux-ubuntu18.04-x86_64 / gcc@6.5.0 -------------------------
    qtrzwov zlib@1.2.11

==> Do you want to proceed? [y/N] y
==> Successfully uninstalled zlib@1.2.11%gcc@6.5.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64/qtrzwov
```

```bash
$ spack find -lf zlib
==> 5 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
pdfmc5x zlib@1.2.8%clang   5qffmms zlib@1.2.11%clang

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
t4p5jnm zlib@1.2.8%gcc	 h6i53if zlib@1.2.8%gcc  cppflags="-O3"   smoyzzo zlib@1.2.11%gcc
```

ハッシュのみを参照してパッケージをアンインストールすることもできます。

`-f`（force）または `-R`（依存関係の削除）のいずれかを使用して、インストールされている別のパッケージに必要なパッケージを削除することができます。

```bash
$ spack uninstall zlib/pdf
==> Will not uninstall zlib@1.2.8%clang@6.0.0/pdfmc5x
The following packages depend on it:
    -- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
    yqtoq2e tcl@8.6.10

==> Error: There are still dependents.
  use `spack uninstall --dependents` to remove dependents too
```

```bash
$ spack uninstall -R zlib/pdf
==> The following packages will be uninstalled:

    -- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
    yqtoq2e tcl@8.6.10
    pdfmc5x zlib@1.2.8

==> Do you want to proceed? [y/N] y
==> Successfully uninstalled tcl@8.6.10%clang@6.0.0 arch=linux-ubuntu18.04-x86_64/yqtoq2e
==> Successfully uninstalled zlib@1.2.8%clang@6.0.0+optimize+pic+shared arch=linux-ubuntu18.04-x86_64/pdfmc5x
```

Spackは、十分に指定されていないパッケージをアンインストールしません。
`-a`（all）フラグは一度に複数のパッケージをアンインストールするために使用することができます。

```bash
$ spack uninstall trilinos
==> Error: trilinos matches multiple packages:

    -- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
    qmjbxf2 trilinos@13.0.1  xvzpvxd trilinos@13.0.1

==> Error: You can either:
    a) use a more specific spec, or
    b) specify the spec by its hash (e.g. `spack uninstall /hash`), or
    c) use `spack uninstall --all` to uninstall ALL matching specs.
```

```bash
$ spack uninstall -a trilinos
==> The following packages will be uninstalled:

    -- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
    xvzpvxd trilinos@13.0.1  qmjbxf2 trilinos@13.0.1

==> Do you want to proceed? [y/N] y
==> Successfully uninstalled trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64/qmjbxf2
==> Successfully uninstalled trilinos@13.0.1%gcc@7.5.0~adios2~alloptpkgs+amesos+amesos2+anasazi+aztec+belos+boost~cgns~chaco~complex~cuda~debug~dtk+epetra+epetraext+exodus+explicit_template_instantiation~float+fortran+glm+gtest+hdf5~hwloc+hypre+ifpack+ifpack2~intrepid~intrepid2~ipo~isorropia+kokkos+matio~mesquite+metis~minitensor+ml+mpi+muelu+mumps+netcdf~nox~openmp~phalanx~piro~pnetcdf~python~rol~rythmos+sacado~shards+shared~shylu~stk~stratimikos~strumpack+suite-sparse~superlu~superlu-dist~teko~tempus+teuchos+tpetra~wrapper~x11~xsdkflags~zlib+zoltan+zoltan2 build_type=RelWithDebInfo cuda_arch=none cxxstd=11 gotype=long arch=linux-ubuntu18.04-x86_64/xvzpvxd
```

## 高度な `spack find` の使い方

これまでのセクションで説明していない `spack find` コマンドの使い方について解説します。

`spack find` コマンドは、「匿名仕様（anonimous specs）」と呼ばれるクエリも受け入れることができます。
匿名仕様とは、パッケージ名を含まない仕様構文を指します。
たとえば、 `spack find ^mpich` コマンドはMPICHに依存するインストール済みのすべてのパッケージを返し、 `spack find cppflags="-O3"` コマンドはコンパイルオプション `cppflags="-O3"` でビルドされたすべてのパッケージを返します。

```bash
$ spack find ^mpich
==> 8 installed packages
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
hdf5@1.10.7  hypre@2.20.0  matio@1.5.17  mumps@5.3.3  netcdf-c@4.7.4  netlib-scalapack@2.1.0  parmetis@4.0.3  trilinos@13.0.1
```

```bash
$ spack find cppflags=-O3
==> 1 installed package
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
zlib@1.2.8
```

`-x` フラグを使用して、（依存関係としてpullされるのではなく）明示的にインストールされたパッケージを表示することもできます。
`-X` フラグは、暗黙のインストールのみを表示します。
`-p` フラグを使用して、spackパッケージがインストールされたパスを表示することもできます。

```bash
$ spack find -px
==> 11 installed packages
-- linux-ubuntu18.04-x86_64 / clang@6.0.0 -----------------------
zlib@1.2.11  /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/clang-6.0.0/zlib-1.2.11-5qffmms6gwykcikh6aag4h3z4scrfdla

-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
hdf5@1.10.7	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-sldl7zxtufal5qvh2zrggnxtrtava7a7
hdf5@1.10.7	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-vedchc5aoqyu3ydbp346qrbpe6kg46rq
hdf5@1.10.7	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/hdf5-1.10.7-cq4k4cvkdbwzejti2f6za4morbud7ars
patchelf@0.10	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/patchelf-0.10-2nj3vxxz5lwexpp5srseegxt7v6n4whp
tcl@8.6.10	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10-m2hc42k5bmqzqwudra5a3qx3fgyplnjr
tcl@8.6.10	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/tcl-8.6.10-rtz76647rejri5fkpdmnthttncv2vgkx
trilinos@13.0.1  /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/trilinos-13.0.1-xvzpvxdt7x66bifz4kg3t7jvsqewjytw
zlib@1.2.8	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8-t4p5jnme6csjrx4vbnla5za7ghcctg76
zlib@1.2.8	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.8-h6i53ifim7gcwnzo4s6thvcbe7nxybps
zlib@1.2.11	 /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
```

## コンパイラのカスタマイズ

Spackは環境変数 `PATH` 上に存在するシステムで使用可能なコンパイラリストを管理します。
`spack compilers` コマンドは `spack compiler list` コマンドのエイリアスです。

```bash
$ spack compilers
==> Available compilers
-- clang ubuntu18.04-x86_64 -------------------------------------
clang@6.0.0

-- gcc ubuntu18.04-x86_64 ---------------------------------------
gcc@7.5.0  gcc@6.5.0
```

コンパイラはYAML形式のファイルで管理されます。
チュートリアルの後半では、特別な場合に手動でコンパイラを構成する方法を学習します。
Spackにはコンパイラを追加するツールもあり、Spackでビルドされたコンパイラを構成に追加できます。

```bash
$ spack install gcc@8.3.0
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libiconv-1.16-jearpk4xci4zc7dkrza4fufaqfkq7rfl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libsigsegv-2.12-lbrx7lnfz46ukewxbhxnucmx76g23c6q
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/berkeley-db-18.1.40-4ihuiazsglf22f3pntq5hc4kyszqzexn
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/pkgconf-1.7.3-4sh6pymrm2ms4auu3ajbjjr6fiuhz5g7
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/zlib-1.2.11-smoyzzo2qhzpn6mg6rd3l2p7b23enshg
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/diffutils-3.7-otkktenrrvcaf37a7zyeytmjuwsalvtr
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/m4-1.4.18-mkc3u4x2p2wie6jfhuku7g5rkovcrxps
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/ncurses-6.2-crhlefo3dv7lmsv5pf4icsy4gepkdorm
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/libtool-2.4.6-jdxbjftheiotj6solpomva7dowrhlerl
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/readline-8.0-t54jzdy2jj4snltjazlm3br2urcilc6v
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gdbm-1.18.1-4av4gywgpaspkhy3dvbb62nulqogtzbb
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/perl-5.32.0-zfdvt2jjuaees43ffrrtphqs2ky3o22t
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/autoconf-2.69-mm33a3ocsv3jsh2tfxc4mlab4xsurtdd
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/automake-1.16.2-d2krmb5gweivlnztcymhklzsqbrpatt6
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gmp-6.1.2-3ol3tldmm3oqflzrqonh6p2jmhgwsr4t
==> Installing isl-0.18-4lkvfkviv45b2htxuyfxbivafx4wajud
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/isl-0.18/linux-ubuntu18.04-x86_64-gcc-7.5.0-isl-0.18-4lkvfkviv45b2htxuyfxbivafx4wajud.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting isl-0.18-4lkvfkviv45b2htxuyfxbivafx4wajud from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/isl-0.18-4lkvfkviv45b2htxuyfxbivafx4wajud
==> Installing mpfr-3.1.6-nwsubswuytvvsjtu3nuyuxquwyxyfebj
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-3.1.6/linux-ubuntu18.04-x86_64-gcc-7.5.0-mpfr-3.1.6-nwsubswuytvvsjtu3nuyuxquwyxyfebj.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mpfr-3.1.6-nwsubswuytvvsjtu3nuyuxquwyxyfebj from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpfr-3.1.6-nwsubswuytvvsjtu3nuyuxquwyxyfebj
==> Installing mpc-1.1.0-uur2dd6aco22hxagwzxco23rvtyvqd7t
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpc-1.1.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-mpc-1.1.0-uur2dd6aco22hxagwzxco23rvtyvqd7t.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting mpc-1.1.0-uur2dd6aco22hxagwzxco23rvtyvqd7t from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/mpc-1.1.0-uur2dd6aco22hxagwzxco23rvtyvqd7t
==> Installing gcc-8.3.0-7mh4po7nmny3ku6c3jv3ovcssb6wwqlw
==> Fetching file:///mirror/build_cache/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0/linux-ubuntu18.04-x86_64-gcc-7.5.0-gcc-8.3.0-7mh4po7nmny3ku6c3jv3ovcssb6wwqlw.spack
 ###################################################################################################################################################################### 100.0%
==> Extracting gcc-8.3.0-7mh4po7nmny3ku6c3jv3ovcssb6wwqlw from binary cache
[+] /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-7mh4po7nmny3ku6c3jv3ovcssb6wwqlw
```

```bash
$ spack find -p gcc
==> 1 installed package
-- linux-ubuntu18.04-x86_64 / gcc@7.5.0 -------------------------
gcc@8.3.0  /home/spack/spack/opt/spack/linux-ubuntu18.04-x86_64/gcc-7.5.0/gcc-8.3.0-7mh4po7nmny3ku6c3jv3ovcssb6wwqlw
```

`spack compiler add` コマンドを使用して、使用可能なコンパイラとしてGCCをSpackに追加できます。
これにより、将来のパッケージを `gcc@8.3.0` でビルドできるようになります 。GCCのパスをコピーして貼り付ける必要がないように、`spack location -i` でインストール先のプレフィックスを取得できます 。

```bash
$ spack compiler add $(spack location -i gcc@8.3.0)
==> Added 1 new compiler to /home/spack/.spack/linux/compilers.yaml
    gcc@8.3.0
==> Compilers are defined in the following files:
    /home/spack/.spack/linux/compilers.yaml
```

`spack compiler remove <compiler_spec>` を使用して、構成からコンパイラーを削除することもできます。

```bash
$ spack compiler remove gcc@8.3.0
==> Removed compiler gcc@8.3.0
```


