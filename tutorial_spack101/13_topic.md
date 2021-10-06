# パッケージングの高度なトピック

Spackは、依存関係からの情報を使いパッケージを自動的に構成します。
これにより、依存関係（つまり `depends_on` ディレクティブの使用）とビルドシステム（たとえば、`CmakePackage`を継承）を一覧表示するだけで済みます。

ただし、多くの特殊なケースがあります。
多くの場合、依存関係に関する詳細を取得して、パッケージ固有の構成オプションを設定したり、パッケージのビルドシステムで使用されるパッケージ固有の環境変数を定義したりする必要があります。
このチュートリアルでは、依存関係からビルド情報を取得する方法と、パッケージ内の依存関係に重要な情報を自動的に提供する方法について説明します。

## チュートリアルのセットアップ

> **注意**
> このセクションの設定をそのまま本番の Spack インスタンスとして使用することは推奨しません。

このチュートリアルでは、チュートリアル中に入力されるセクションが欠落しているカスタムパッケージ定義を使用します。
これらのパッケージ定義は、次の方法で有効にできる別のパッケージリポジトリに保存されます。

```bash
$ spack repo add --scope=site var/spack/repos/tutorial
```

このセクションでは、新しいバージョンの `gcc` も必要になる場合があります。
`gcc@7.2.0` をインストールして構成に追加していない場合は、次の方法で追加できます：

```bash
$ spack install gcc@7.2.0 %gcc@5.4.0
$ spack compiler add --scope=site `spack location -i gcc@7.2.0 %gcc@5.4.0`
```

チュートリアルのDockerイメージを使用している場合は、すべての依存関係パッケージがインストールされています。
それ以外の場合、これらのパッケージをインストールするには、次のコマンドを使います：

```bash
$ spack install openblas
$ spack install netlib-lapack
$ spack install mpich
```

あとは好きなエディタを `EDITOR` 環境変数に設定すれば、準備完了です。

> **注意**
> これらのパッケージのいくつかは、MPIの実装に依存しています。
> OpenMPI を最初からインストールする場合はすぐにチュートリアルにはいれますが、インストールする場合は時間がかかります（10分以上）。
> MPICHのバイナリキャッシュが提供される場合があります。
> その場合、パッケージにそれを使用させてすばやくインストールすることができます。
> MPICHに依存するパッケージを使用したすべてのチュートリアル例には、MPICHを使用してビルドするための仕様構文が含まれています。

## パッケージのビルド環境の変更

Spackは、パッケージの構築を支援するためにデフォルトの `PATH` のようないくつかの環境変数を設定しますが、多くのパッケージは、依存関係に関する特定の情報を伝達する環境変数を利用します（例：`MPICC`）。
このセクションでは、パッケージ固有の環境変数がビルド時に定義されるように Spackパッケージを更新する方法について説明します。

### ビルド時に依存パッケージの環境変数を設定する

依存関係は、依存関係の構築時に必要な環境変数を設定できます。
たとえば、パッケージが `py-numpy` のようなPython拡張機能に依存している場合、Spack の `python` パッケージはそれを `PYTHONPATH` に追加して 、ビルド時に利用できるようにします。
これが必要なのは、Spackのデフォルト設定では、Pythonがモジュールをインポートするのに十分ではないためです。

依存関係の環境設定を提供するために、パッケージは `setup_dependent_build_environment` かつ/または `setup_dependent_run_environment` 関数を実装できます。
これらの関数は、環境を更新するための便利なメソッドを含むオブジェクトをパラメータ `EnvironmentModifications` として受け取ります。
たとえば、MPI 実装は、依存パッケージのビルド時の `MPICC` 使用を設定できます：

```python
def setup_dependent_build_environment(self, env, dependent_spec):
    env.set('MPICC', join_path(self.prefix.bin, 'mpicc'))
```

この場合、`mpi` に依存するパッケージがビルド時に使用する対象が、環境変数 `MPICC` で定義されています。
このセクションでは、`env` で表されるビルド時環境の変更に焦点を当てていますが、`setup_dependent_run_environment` 関数の `env` パラメータを介して行われるランタイム環境への変更は、Spackの自動生成モジュールファイルに含まれていることに注意してください。

`mpich` パッケージを編集して 、依存パッケージのビルド時に環境変数 `MPICC` を設定して学習しましょう。

```bash
root@advanced-packaging-tutorial:/# spack edit mpich
```

終了段階のメソッドは次のようになります：

```pyton
def setup_dependent_environment(self, spack_env, run_env, dependent_spec):
    spack_env.set('MPICC',  join_path(self.prefix.bin, 'mpicc'))
    spack_env.set('MPICXX', join_path(self.prefix.bin, 'mpic++'))
    spack_env.set('MPIF77', join_path(self.prefix.bin, 'mpif77'))
    spack_env.set('MPIF90', join_path(self.prefix.bin, 'mpif90'))

    spack_env.set('MPICH_CC', spack_cc)
    spack_env.set('MPICH_CXX', spack_cxx)
    spack_env.set('MPICH_F77', spack_f77)
    spack_env.set('MPICH_F90', spack_fc)
    spack_env.set('MPICH_FC', spack_fc)
```

この時点で、たとえば、次のコマンドで `mpich` を使った `netlib-scalapack` パッケージをインストールできます：

```bash
root@advanced-packaging-tutorial:/# spack install netlib-scalapack ^mpich
...
==> Created stage in /usr/local/var/spack/stage/netlib-scalapack-2.0.2-km7tsbgoyyywonyejkjoojskhc5knz3z
==> No patches needed for netlib-scalapack
==> Building netlib-scalapack [CMakePackage]
==> Executing phase: 'cmake'
==> Executing phase: 'build'
==> Executing phase: 'install'
==> Successfully installed netlib-scalapack
  Fetch: 0.01s.  Build: 3m 59.86s.  Total: 3m 59.87s.
[+] /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/netlib-scalapack-2.0.2-km7tsbgoyyywonyejkjoojskhc5knz3z
```

環境ログを再確認して、すべての変数が正しい値に設定されていることを確認します。

## 独自のパッケージに環境変数を設定する

パッケージは、`setup_build_environment <spack.package.PackageBase.setup_build_environment()` 関数を実装することで独自のビルド時環境を変更し、`setup_run_environment <spack.package.PackageBase.setup_run_environment()` 関数を実装することでランタイム 環境を変更できます。
`qt` パッケージの場合、次のようになります：

```python
def setup_build_environment(self, env):
    env.set('MAKEFLAGS', '-j{0}'.format(make_jobs))
    if self.spec.satisfies('@5.11:'):
        # QDoc uses LLVM as of 5.11; remove the LLVM_INSTALL_DIR to
        # disable
        try:
            llvm_path = self.spec['llvm'].prefix
        except KeyError:
            # Prevent possibly incompatible system LLVM from being found
            llvm_path = "/spack-disable-llvm"
        env.set('LLVM_INSTALL_DIR', llvm_path)

def setup_run_environment(self, env):
    env.set('QTDIR', self.prefix)
    env.set('QTINC', self.prefix.inc)
    env.set('QTLIB', self.prefix.lib)
    env.prepend_path('QT_PLUGIN_PATH', self.prefix.plugins)
```

`qt` パッケージビルド時に、`MAKEFLAGS` は環境内で定義されます（5.11以降のバージョンであることと、`LLVM_INSTALL_DIR` も定義されます）。

`qt` の `setup_dependent_build_environment` 関数と比較のために：

```python
def setup_dependent_build_environment(self, env, dependent_spec):
    env.set('QTDIR', self.prefix)
    env.set('QTINC', self.prefix.inc)
    env.set('QTLIB', self.prefix.lib)
    env.prepend_path('QT_PLUGIN_PATH', self.prefix.plugins)
```

`qt` のための `setup_dependent_run_environment` メソッドを実装する必要がないため、提供されていません。

`elpa` パッケージを完成させて、どのように機能するかを見てみましょう：

```bash
root@advanced-packaging-tutorial:/# spack edit elpa
```

最終的に次のようなメソッドになります：

```python
def setup_environment(self, spack_env, run_env):
    spec = self.spec

    spack_env.set('CC', spec['mpi'].mpicc)
    spack_env.set('FC', spec['mpi'].mpifc)
    spack_env.set('CXX', spec['mpi'].mpicxx)
    spack_env.set('SCALAPACK_LDFLAGS', spec['scalapack'].libs.joined())

    spack_env.append_flags('LDFLAGS', spec['lapack'].libs.search_flags)
    spack_env.append_flags('LIBS', spec['lapack'].libs.link_flags)
```

この時点で、`elpa ^mpich` のインストールを続行することが可能です。

## ライブラリ情報の取得

Spackは、パッケージが依存関係ライブラリを(例えば、`PKG_CONFIG_PATH` や `CMAKE_PREFIX_PATH` を設定することで)自動的に見つけるのを支援しようとしますが、パッケージには、ライブラリを見つけるために必要な固有の構成オプションがある場合があります。
パッケージが依存関係ライブラリに関する情報を必要とする場合、Spackの一般的なアプローチは、依存関係にライブラリの場所を照会し、それに応じて構成オプションを設定することです。
デフォルトでは、ほとんどのSpackパッケージは、ライブラリを自動的に見つける方法を知っています。
このセクションでは、依存関係からライブラリ情報を取得する方法と、デフォルトロジックが機能しない場合にライブラリを見つける方法について説明します。

### 依存関係ライブラリへのアクセス

依存関係のあるライブラリにアクセスする必要がある場合は、仕様の `libs` プロパティを介してアクセスできます。たとえば、`arpack-ng` パッケージでは：

```python
def install(self, spec, prefix):
    lapack_libs = spec['lapack'].libs.joined(';')
    blas_libs = spec['blas'].libs.joined(';')

    cmake(*[
        '-DLAPACK_LIBRARIES={0}'.format(lapack_libs),
        '-DBLAS_LIBRARIES={0}'.format(blas_libs)
    ], '.')
```

`arpack-ng` は、Spackが自動的にインストール済みの実装（例えば `blas` として`openblas`を）仮想依存関係を照会していることに注意してください。

`armadillo` のためのパッケージの作業を開始しました。`spack edit` で開き、`# TUTORIAL:` で始まるコメントを読み、``cmake_args` セクションを完了する必要があります：

```bash
root@advanced-packaging-tutorial:/# spack edit armadillo
```

パッケージの指示に従った場合、終了時の `cmake_args` メソッドは次のようになります：

```python
def cmake_args(self):
      spec = self.spec

      return [
          # ARPACK support
          '-DARPACK_LIBRARY={0}'.format(spec['arpack-ng'].libs.joined(";")),
          # BLAS support
          '-DBLAS_LIBRARY={0}'.format(spec['blas'].libs.joined(";")),
          # LAPACK support
          '-DLAPACK_LIBRARY={0}'.format(spec['lapack'].libs.joined(";")),
          # SuperLU support
          '-DSuperLU_INCLUDE_DIR={0}'.format(spec['superlu'].prefix.include),
          '-DSuperLU_LIBRARY={0}'.format(spec['superlu'].libs.joined(";")),
          # HDF5 support
          '-DDETECT_HDF5={0}'.format('ON' if '+hdf5' in spec else 'OFF')
      ]
```

ご覧のとおり、依存関係が提供するライブラリのリストを取得するのは、それらの `libs` 属性にアクセスするのと同じくらい簡単です。
さらに、通常の依存関係と仮想の依存関係のどちらをクエリする場合でも、インターフェイスは同じままです。

この時点で 、LAPACKプロバイダとして `openblas` を使用した `armadillo` インストール（`armadillo ^openblas ^mpich`）を完了することができます：

```bash
root@advanced-packaging-tutorial:/# spack install armadillo ^openblas ^mpich
==> pkg-config is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/pkg-config-0.29.2-ae2hwm7q57byfbxtymts55xppqwk7ecj
...
==> superlu is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/superlu-5.2.1-q2mbtw2wo4kpzis2e2n227ip2fquxrno
==> Installing armadillo
==> Using cached archive: /usr/local/var/spack/cache/armadillo/armadillo-8.100.1.tar.xz
==> Staging archive: /usr/local/var/spack/stage/armadillo-8.100.1-n2eojtazxbku6g4l5izucwwgnpwz77r4/armadillo-8.100.1.tar.xz
==> Created stage in /usr/local/var/spack/stage/armadillo-8.100.1-n2eojtazxbku6g4l5izucwwgnpwz77r4
==> Applied patch undef_linux.patch
==> Building armadillo [CMakePackage]
==> Executing phase: 'cmake'
==> Executing phase: 'build'
==> Executing phase: 'install'
==> Successfully installed armadillo
  Fetch: 0.01s.  Build: 3.96s.  Total: 3.98s.
[+] /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/armadillo-8.100.1-n2eojtazxbku6g4l5izucwwgnpwz77r4
```

インストールがうまくいき、追加したコードがセミコロンで区切られたライブラリの正しいリストに拡張されました（`armadillo` のビルドログを開いて再確認することをお勧めします）。

### 非依存関係へのライブラリ提供

Spackは `libs` のためのデフォルト実装を提供し、大抵の場合は箱から出してすぐに機能します。
上記のセクションに示すように、ユーザは `libs` プロパティを実装せずにパッケージ定義を記述でき、非依存関係パッケージはそのライブラリを取得できます。
ただし、デフォルト実装では、ライブラリが命名スキーム `lib<package name>.so` （もしくは、例えば静的ライブラリ`lib<package name>.a`）に従っていることが前提です。
この命名スキームに従わないパッケージは、この関数を自分で実装する必要があります。例えば`opencv`では：

```python
@property
def libs(self):
    shared = "+shared" in self.spec
    return find_libraries(
        "libopencv_*", root=self.prefix, shared=shared, recursive=True
    )
```

この問題は、インターフェイスを実装するパッケージ（つまり、Spackの仮想パッケージプロバイダ）に共通しています。
（）に`netlib-lapack` に関連付けられた `armadillo` の別バージョン（`armadillo ^netlib-lapack ^mpich`）をビルドしようとすると、今回はインストールが完了しないことがわかります：

```bash
root@advanced-packaging-tutorial:/# spack install  armadillo ^netlib-lapack ^mpich
==> pkg-config is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/pkg-config-0.29.2-ae2hwm7q57byfbxtymts55xppqwk7ecj
...
==> openmpi is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/openmpi-3.0.0-yo5qkfvumpmgmvlbalqcadu46j5bd52f
==> Installing arpack-ng
==> Using cached archive: /usr/local/var/spack/cache/arpack-ng/arpack-ng-3.5.0.tar.gz
==> Already staged arpack-ng-3.5.0-bloz7cqirpdxj33pg7uj32zs5likz2un in /usr/local/var/spack/stage/arpack-ng-3.5.0-bloz7cqirpdxj33pg7uj32zs5likz2un
==> No patches needed for arpack-ng
==> Building arpack-ng [Package]
==> Executing phase: 'install'
==> Error: RuntimeError: Unable to recursively locate netlib-lapack libraries in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/netlib-lapack-3.6.1-jjfe23wgt7nkjnp2adeklhseg3ftpx6z
RuntimeError: RuntimeError: Unable to recursively locate netlib-lapack libraries in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/netlib-lapack-3.6.1-jjfe23wgt7nkjnp2adeklhseg3ftpx6z

/usr/local/var/spack/repos/builtin/packages/arpack-ng/package.py:105, in install:
     5             options.append('-DCMAKE_INSTALL_NAME_DIR:PATH=%s/lib' % prefix)
     6
     7             # Make sure we use Spack's blas/lapack:
  >> 8             lapack_libs = spec['lapack'].libs.joined(';')
     9             blas_libs = spec['blas'].libs.joined(';')
     10
     11            options.extend([

See build log for details:
  /usr/local/var/spack/stage/arpack-ng-3.5.0-bloz7cqirpdxj33pg7uj32zs5likz2un/arpack-ng-3.5.0/spack-build-out.txt
```

`libopenblas.so` と命名されたライブラリとして提供された `openblas` と違い、`netlib-lapack` は `liblapack.so` という名前で提供します。
このためカスタマイズされたライブラリの検索ロジックを実装する必要があります。
では、

```bash
root@advanced-packaging-tutorial:/# spack edit netlib-lapack
```

前のように `# TUTORIAL:` コメント内の従って編集しましょう。
実装が必要な箇所は次のとおりです：

```python
@property
def lapack_libs(self):
    shared = True if '+shared' in self.spec else False
    return find_libraries(
        'liblapack', root=self.prefix, shared=shared, recursive=True
    )
```

つまり、LAPACKインターフェイスのライブラリの正しいリストを返すプロパティです。

`netlib-lapack` は `blas` も提供可能なので、`libs`よりむしろ`lapack_libs` の名前を使い、その際に別のライブラリファイルとして提供されます。
この名前を使用すると、非依存関係側が `lapack` ライブラリを要求したときに、`netlib-lapack` は `lapack` インターフェイスに関連付けられているライブラリのみを取得するようになります。
これでようやく `armadillo ^netlib-lapack ^mpich` をインストールすることができます：

```bash
root@advanced-packaging-tutorial:/# spack install  armadillo ^netlib-lapack ^mpich
...

==> Building armadillo [CMakePackage]
==> Executing phase: 'cmake'
==> Executing phase: 'build'
==> Executing phase: 'install'
==> Successfully installed armadillo
  Fetch: 0.01s.  Build: 3.75s.  Total: 3.76s.
[+] /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/armadillo-8.100.1-sxmpu5an4dshnhickh6ykchyfda7jpyn
```

仮想パッケージの各実装は、それが提供するインターフェイスに関連付けられたライブラリを見つける責任があるため、依存関係側は、さまざまな実装の特殊なケースのロジックを含める必要はなく、たとえば `spec['blas'].libs` を要求するだけで済みます。

## その他のパッケージングトピック

### 他のパッケージに属性を添付する

ビルドツールは通常、別のパッケージがインストールされているときに使用できる実行可能ファイルのセットも提供します。
Spackを使用すると、依存モジュールにモンキーパッチを適用して属性をアタッチすることができます。
これにより、同じパッケージを手動でインストールした場合と可能な限り類似したパッケージャーエクスペリエンスを実現できます。

ここでの例は、`setup_dependent_package` をオーバーライドする `automake` パッケージ です：

```python
def setup_dependent_package(self, module, dependent_spec):
    # Automake is very likely to be a build dependency,
    # so we add the tools it provides to the dependent module
    executables = ['aclocal', 'automake']
    for name in executables:
        setattr(module, name, self._make_executable(name))
```

依存する他のすべてのパッケージは、`Executable` の通常の関数呼び出し構文で `aclocal` や `automake` を直接使用が可能となります：

```python
aclocal('--force')
```

### 追加の照会パラメータ

Specのビルドインターフェイスプロトコルの高度な機能は、添え字キーの後の追加パラメータのサポートです。
実際、照会で使用されるキーの後には、追加のパラメータのコンマ区切りのリストを続けることができます。
これらのリストは、応答を微調整するための要求を受信するパッケージによって検査できます。

例を確認して、`netcdf ^mpich` をインストールしてみましょう：

```bash
root@advanced-packaging-tutorial:/# spack install netcdf ^mpich
==> libsigsegv is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/libsigsegv-2.11-fypapcprssrj3nstp6njprskeyynsgaz
==> m4 is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/m4-1.4.18-r5envx3kqctwwflhd4qax4ahqtt6x43a
...
==> Error: AttributeError: 'list' object has no attribute 'search_flags'
AttributeError: AttributeError: 'list' object has no attribute 'search_flags'

/usr/local/var/spack/repos/builtin/packages/netcdf/package.py:207, in configure_args:
     50            # used instead.
     51            hdf5_hl = self.spec['hdf5:hl']
     52            CPPFLAGS.append(hdf5_hl.headers.cpp_flags)
  >> 53            LDFLAGS.append(hdf5_hl.libs.search_flags)
     54
     55            if '+parallel-netcdf' in self.spec:
     56                config_args.append('--enable-pnetcdf')

See build log for details:
  /usr/local/var/spack/stage/netcdf-4.4.1.1-gk2xxhbqijnrdwicawawcll4t3c7dvoj/netcdf-4.4.1.1/spack-build-out.txt
```

`netcdf` が `hdf5` の高レベルインターフェイスのリンク方法を知る必要があり、このため取得のための要求後追加パラメータ`hl`が渡されている旨のエラーメッセージが確認できます。
明らかに `hdf5` パッケージの実装が完全ではないため、修正する必要があります：

```bash
root@advanced-packaging-tutorial:/# spack edit hdf5
```

指示に正しく従った場合、`lib` プロパティに追加されるコードは次のようになります：

```python
query_parameters = self.spec.last_query.extra_parameters
key = tuple(sorted(query_parameters))
libraries = query2libraries[key]
shared = '+shared' in self.spec
return find_libraries(
    libraries, root=self.prefix, shared=shared, recurse=True
)
```

1行目で追加のパラメータを取得しています。
これで、`netcdf ^mpich` のインストールを正常に完了することができます：

```bash
root@advanced-packaging-tutorial:/# spack install netcdf ^mpich
==> libsigsegv is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/libsigsegv-2.11-fypapcprssrj3nstp6njprskeyynsgaz
==> m4 is already installed in /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/m4-1.4.18-r5envx3kqctwwflhd4qax4ahqtt6x43a
...
==> Installing netcdf
==> Using cached archive: /usr/local/var/spack/cache/netcdf/netcdf-4.4.1.1.tar.gz
==> Already staged netcdf-4.4.1.1-gk2xxhbqijnrdwicawawcll4t3c7dvoj in /usr/local/var/spack/stage/netcdf-4.4.1.1-gk2xxhbqijnrdwicawawcll4t3c7dvoj
==> Already patched netcdf
==> Building netcdf [AutotoolsPackage]
==> Executing phase: 'autoreconf'
==> Executing phase: 'configure'
==> Executing phase: 'build'
==> Executing phase: 'install'
==> Successfully installed netcdf
  Fetch: 0.01s.  Build: 24.61s.  Total: 24.62s.
[+] /usr/local/opt/spack/linux-ubuntu16.04-x86_64/gcc-5.4.0/netcdf-4.4.1.1-gk2xxhbqijnrdwicawawcll4t3c7dvoj
```
