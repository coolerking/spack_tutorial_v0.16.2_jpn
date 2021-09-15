# [ビルドのカスタマイズ](https://spack.readthedocs.io/en/latest/build_settings.html#build-settings)

Spackは `packages.yaml` ファイルを介してソフトウェアを構築する方法でカスタマイズを実施できます。
`packages.yaml` を使って、Spackに仮想依存関係の特定の実装（MPIやBLAS/LAPACKなど）を優先させたり、特定のコンパイラでビルドすることを優先させることができます。
また、システムに既に存在する外部ソフトウェアインストールを使用してビルドするようにSpackに指示することもできます。

大まかに説明すると、`packages.yaml` ファイルは次のようなセクションで構成されています。

```yaml
packages:
  package1:
    # package1 の設定
  package2:
    # package2 の設定
  # ...
  all:
    # すべてのパッケージに適用される設定
```

したがって、1つのパッケージ専用にビルド設定を設定することも、特定の設定をすべてのパッケージに適用するように指定することもできます。
。カスタマイズ可能な設定の種類については後述します。

Spackのビルドのデフォルトはデフォルトファイル `etc/spack/defaults/packages.yaml` にあります。
`~/.spack/packages.yaml` もしくは `etc/spack/packages.yaml` で設定をオーバーライドできます。
これがどのように機能するかの詳細は、「[設定のスコープ](https://spack.readthedocs.io/ja/latest/build_settings.html)」を参照してください。


## 外部パッケージ

Spackは、外部にインストールされたパッケージを使用するように構成できます。
これは、Spackが独自の MPI を自身で構築する代わりに外部パッケージを使用する必要がある（カスタマイズされたMPIなどのシステムパッケージがマシンに付属している場合など）場合に対応するためです。

`etc/spack/` またはユーザーの `~/.spack/` ディレクトリにあるファイル `packages.yaml` を使って、Spackインストールを外部パッケージを使うように構成できます。
外部構成の例を次に示します。

```yaml
packages:
  openmpi:
    externals:
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.4.3
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64+debug"
      prefix: /opt/openmpi-1.4.3-debug
    - spec: "openmpi@1.6.5%intel@10.1 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.6.5-intel
```

この例では、OpenMPIの3つのインストールを示しています。
最初の1つ目はgccで構築され、2つ目はデバッグ情報つきでgccで構築され、3つ目はIntelコンパイラで構築されています。
SpackがこれらのMPIの1つを依存関係として使用するパッケージをビルドするように求められた場合、Spackは指定されたディレクトリにプリインストールされたOpenMPIを使用します。
パス指定が `bin` サブディレクトリではなく、最上位のインストール先のプレフィックスであることに注意してください。

`packages.yaml` インストールプレフィックスの代わりに、ロードするモジュールを指定するために使用することもできます。
次の例は、モジュール `CMake/3.7.2` がcmakeバージョン3.7.2を提供することを示しています。

```yaml
cmake:
  externals:
  - spec: cmake@3.7.2
    modules:
    - CMake/3.7.2
```

`packages.yaml` は `packages:` 属性で始まり、パッケージ名のリストが続きます。
外部インストール済みパッケージを指定するには、属性 `externals:` を追加してパッケージリストを記述します。
`externals:` の `spec:` には、合理的に可能な限り明確に定義した文字列を指定する必要があります。
ここにコンパイラ指定やパッケージのバージョンがないなど、パッケージ特定に必要なスペックコンポーネントがない場合、Spackは最も人気のあるパッケージに基づいて不足しているコンポーネントを推測するので、誤推測する可能性があります。

外部にリストされている各パッケージバージョンとコンパイラには、パッケージとコンパイラがビルドされていない場合でも、Spackパッケージとコンパイラ構成にエントリが含まれている必要があります。

パッケージ構成は、特定のパッケージバージョンに外部の場所を使用するようにSpackに指示できますが、Spackが外部パッケージを使用するように制限することはありません。
上記の例では、OpenMPIの新しいバージョンが利用可能であるため、SpackはプレインストールされたOpenMPIバージョンを引き続き使用するのではなく、最新バージョンのビルドとリンクを開始することを選択します。

これを防ぐために、この `packages.yaml` 構成では、パッケージにビルド不可のフラグを付けることもできます。
前の例は、次のように変更できます：

```yaml
packages:
  openmpi:
    externals:
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.4.3
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64+debug"
      prefix: /opt/openmpi-1.4.3-debug
    - spec: "openmpi@1.6.5%intel@10.1 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.6.5-intel
    buildable: False
```

`buildable` 属性の追加は、Spackに独自のバージョンのOpenMPIをビルドを禁止し、代わりにビルド済みの OpenMPI に依存関連付けることを通知します。
`paths` 、 `buildable` と同様にパッケージ名の下の属性として指定されます。

外部モジュールがビルド不可として指定されている場合、Spackはリンクに使用できるビルド環境に外部モジュールをロードします。

`buildable` 属性は外部パッケージとペアにする必要はありません。
バグがあったり、望ましい動作をしない可能性のあるパッケージを禁止するために単独で使用することもできます。

Spack仮想パッケージは、ビルド不可として指定することもでき、外部実装を提供することもできます。
上記の例では、OpenMPIはビルド不可として構成されていますが、Spackは外部で利用可能なOpenMPIよりも他のMPI実装を優先することがよくあります。
Spackは MPI プロバイダごとに構成でき、個別ビルドせず便利です。


```yaml
packages:
  mpi:
    buildable: False
  openmpi:
    externals:
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.4.3
    - spec: "openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64+debug"
      prefix: /opt/openmpi-1.4.3-debug
    - spec: "openmpi@1.6.5%intel@10.1 arch=linux-debian7-x86_64"
      prefix: /opt/openmpi-1.6.5-intel
```

提供される仮想の実装の下にリストすることもできます。

```yaml
packages:
  mpi:
    buildable: False
      openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64: /opt/openmpi-1.4.3
      openmpi@1.4.3%gcc@4.4.7 arch=linux-debian7-x86_64+debug: /opt/openmpi-1.4.3-debug
      openmpi@1.6.5%intel@10.1 arch=linux-debian7-x86_64: /opt/openmpi-1.6.5-intel
      mpich@3.3 %clang@9.0.0 arch=linux-debian7-x86_64: /opt/mpich-3.3-intel
```

SpackはリストされているMPIの外部実装のいずれかを使用して依存関係を満たすことができ、コンパイラとアーキテクチャに応じて選択します。

## 外部パッケージを自動的に検索

`spack external find` コマンドを実行して、システム提供パッケージを検索し、`packages.yaml` に追加できます。
コマンドを実行すると、`packages.yaml` に新しいエントリが追加されます。

```yaml
packages:
  cmake:
    externals:
    - spec: cmake@3.17.2
      prefix: /usr
```

このコマンドは一般利用されるパッケージのスモールセットを検出するのに役立ちます。
今のところ、これは通常、ビルド時のみの依存関係検索に限定されています。
具体的な制限は次のとおりです：

パッケージはデフォルトでは検出不可：パッケージを `spack external find` コマンドで検出可能にするには、特別なロジックを追加する必要があります。
詳細は [こちら](https://spack.readthedocs.io/en/latest/packaging_guide.html#make-package-findable) を参照してください。

現在の実装は実行可能ファイルを収集して検査するだけなので、通常はビルド/実行の依存関係にのみ役立ちます（場合によっては、ライブラリパッケージが実行可能ファイルも提供する場合、実行可能ファイルを実行することで意味のある仕様を抽出できる可能性があります 例：MPI 実装のコンパイララッパ）。

ロジックはモジュールファイルを検索せず、環境変数 `PATH` 内に定義された実行可能ファイルを含むパッケージのみを検出できます。
`spack external find` コマンドを実行する前にSpackに認識させたいパッケージに関連するモジュールをロードすることにより、Spackがモジュールファイルを使用する外部を見つけるのを助けることができます。

Spackはパッケージ構成内の既存のエントリを上書きしません。
いずれかの構成スコープで仕様に対して外部が定義されている場合、Spackは新しい外部エントリを追加しません（ `spack config blame packages` コマンドは、すべての外部エントリを見つけるのに役立ちます）。

## コンクリート（具現）化の優先

Spackはコンクリート化中に特定のコンパイラ、パッケージバージョン、依存関係、およびバリアントを優先するように構成できます。
優先構成は、ユーザ構成ファイル `~/.spack/packages.yaml` または `etc/spack/packages.yaml` のサイト構成を介して制御します。

優先パッケージを設定するファイル `packages.yaml` の例を次に示します：

```yaml
packages:
  opencv:
    compiler: [gcc@4.9]
    variants: +debug
  gperftools:
    version: [2.2, 2.4, 2.3]
  all:
    compiler: [gcc@4.4.7, 'gcc@4.6:', intel, clang, pgi]
    target: [sandybridge]
    providers:
      mpi: [mvapich2, mpich, openmpi]
```

大まかに言うと、この例では、パッケージをコンクリート化する方法を指定しています。
opencv パッケージは、 GCC 4.9 の使用を優先し、デバッグオプションを使用してビルドする必要があります。
gperftools パッケージは、バージョン2.4よりも2.2の方を優先する必要があります。
システム上のすべてのパッケージは、MPI および GCC 4.4.7 に mvapich2 を優先する必要があります（ GCC 4.9を優先することで優先順位をオーバーライドするopencvを除く）。
これらのオプションは、暗黙のデフォルトを入力するために使用されます。
明示的に要求された場合、コマンドラインでそれらのいずれかを上書きできます。

`packages.yaml` ファイルは文字列 `packages:` で始まり、その次のレベルでパッケージ名を指定します。
特殊な文字列である `all` は、すべてのパッケージに設定を適用します。
パッケージ名の下に1つ以上のコンポーネント（ `compiler` 、 `variants` 、 `version` 、 `providers` 、 `target` ）を指定します。
それぞれのコンポーネントには仕様 `constraints` の順序付きリストがあり、リスト内の前のエントリが後のエントリよりも優先されます。

パッケージのインストールには、最初のコンクリート化ルールを禁止する制約がある場合があります。
その場合、Spackは最初のルールをコンクリート化に使用します。
例に戻ると、ユーザがバージョン2.3 以降の gperftools を要求した場合、バージョン 2.3 よりもバージョン 2.4バージョンの gperftools を優先するため、Spackはバージョン2.4をインストールします。

優先セクションの明示的な具体化ルールは、リストにない具体化よりも常に優先されます。
上記の例では、 xlc はコンパイラリストにリストされていません。
したがって、 gcc から pgi までのリストされているすべてのコンパイラは、xlc コンパイラよりも優先されます。

この `provider` セクションの構文は、他の具体化ルールとは少し異なります。
プロバイダは、パッケージが持つ可能性のある値 `depend_on`（MPIなど）と、その依存関係を満たすためのルールのリストを列挙します。


## パッケージパーミッション

Spackは、パッケージごとにインストールされたファイルにアクセス許可を指定する構成が可能です。

`packages.yaml` ファイルの `permissions` の下に、属性 `read` 、`write` および `group` を指定してパッケージの権限を制御します。
これらの属性は、パッケージごとに設定することも、`all` の下に記述してすべてのパッケージに設定することもできます。
`all` と特定のパッケージ両方にパッケージに権限が設定されている場合は、パッケージ固有の設定が優先されます。

`read` と `write` には、 `user` 、 `group` 、 `world` のいずれかを指定します。

```yaml
packages:
  all:
    permissions:
      write: group
      group: spack
  my_app:
    permissions:
      read: group
      group: my_team
```

パーミッション設定は、指定されたパッケージのインストールへの最も広いレベルのアクセスを記述します。
ファイルの実行権限は、実行可能なファイルの読み取り権限と同じレベルに設定されます。
`read` のデフォルト設定は `world`、 `write` のデフォルト設定は `user` です。
上記の例では、 `my_app` のインストレーションは、はユーザとグループのアクセス許可でインストールされますが、ワールドのアクセス許可はなく、グループ `my_team` によって所有されます。
他のすべてのパッケージは、ユーザとグループの書き込み権限、およびワールド読み取り権限でインストールされます。
これらのパッケージはグループ `spack` が所有します。

この `group` 属性は、Unixスタイルのグループをパッケージに割り当てます。
パッケージによってインストールされるすべてのファイルは、割り当てられたグループによって所有され、スティッキーグループビットは、インストールプレフィックスとインストールプレフィックス内のすべてのディレクトリに設定されます。
これにより、インストールプレフィックス内に手動で配置されたファイルでも、割り当てられたグループが所有するようになります。
グループが割り当てられていない場合、SpackはOSのデフォルトの動作を期待どおりに実行できるようにします。

