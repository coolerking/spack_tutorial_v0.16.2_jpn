# spack_tutorial_v0.16.2_jpn

Spack 公式ドキュメント v0.16.2 をもとにチュートリアルを翻訳したものです。

- [Tutorial: Spack 101](tutorial_spack101/00_tutorial.md)

## 訳者メモ

管理者権限をもたない一般ユーザで富岳上にてOSSを活用するには、Spack というOSSツールでホームディレクトリ内でビルドしたものを活用することになる。

### Spack環境

- Spack環境には2種類存在する
  - Spackにより管理された環境(managed environment)
  - `spack env create <名前>` で環境を作成
  - **環境名がある**
  - 自分が今使っている環境はこれにあたる
- 独立した環境(independent environment)
- `spack env cretae -d <ディレクトリ>`
- もしくは2つの環境ファイルをディレクトリに置くことでも可能
- 別のマシンで構築した環境ファイルを移動して環境を構築できる→ポータビリティ（移植性）が高い
- `spack.yaml`：抽象的に記述された仕様ファイル
  - バージョンやパラメータが曖昧、spack install実行時の設定に従い決定する
- `spack.lock` ：具体的に記述された仕様ファイル
  - バージョンやパラメータが明示的に記述されており、別のマシンでも全く同じ設定を再構築できる
  - spack コマンドでコンクリート化(concretize)することで作成される
  - **環境名がない**
  - 富岳管理者が一般ユーザ向けに公開している `/opt/apps/spack` 上の環境がこれにあたる

### 構成

- 計算ノードで `spack install` するためにはコンパイラ、パッケージビルド時の設定を変更する必要がある
- Spackではコンパイラをラップして使用する
  - `compilers.yaml` 上に記述
    - 独立環境/管理された環境でパスが異なる
    - 親設定を環境側がオーバライドする
  - 独自コンパイラパスの変更(gcc/gfortran,Intel,fujitsu,..)、コンパイラ固有の最適化オプション指定
  - `-I` 、 `-L` に指定するデフォルトSO、パス群を明示的指定しなくて良い
  - C/C++/Fortranコンパイラ別に個別オプションを管理
- Spack パッケージのspack install 設定
  - `packages.yaml` 上に記述
    - 独立環境/管理された環境でパスが異なる
    - 親設定を環境側がオーバライドする
  - パッケージ全体、パッケージ個別どちらも指定可能
  - `packages.yaml` 内の全体の設定を個別がオーバライドする
  - コンパイラおよびそのバージョン
  - パッケージおよびそのバージョン
  - 依存関係パッケージの個別指定
  - OS側のsoを使いたい場合→ `externals` で指定
  - バリアント
    - Spackにより固有設定を汎化したオプション
      - `spack info <パッケージ名>` で参照可能
    - 富岳計算ノードだと `-cuda` や `-nccl` を指定させたい
- コンクリート（具現・具体）化
  - `packages.yaml` / `compilers.yaml` は実行時にインストール仕様詳細が確定する
  - 別のファシリティ上で動かす場合、細かなバージョンや依存関係にあるパッケージが合わない場合が出てくる
  - 依存関係パッケージ含めた詳細なインストール仕様がDAC化される
  - 手動インストールした独自パッケージやOSパッケージの代用などはDACを書き換える

### パッケージ作成

- ソースコード(tar.gz形式でおとしてきたやつ)ファイルをもとにSpack レシピを作れる

```bash
spack create https://github.com/LLNL/mpileaks/releases/download/v1.0/mpileaks-1.0.tar.gz
```

- `$EDITOR` 指定エディタが開いて `packages.py` が編集可能に
- Autotools ベースならAutotoolsベースのクラステンプレートになっている
- Pythonコメント、URL指定、メンテナ、バージョン、依存関係、`config_args` メソッドを埋める
- `spack install` や `spack info` しながらデバッグ
- 場合により `spack cd` で依存関係先のパッケージを直接いじる
  - `spack build-env ＜パッケージ名＞ bash` 上で `configure` をかけながらデバッグ
  - おわったら `exit`

### 開発者ワークフロー

- 既存のSpack 管理化パッケージのコードにパッチを当てspack installできるようにする

```bash
spack develop ＜Spackパッケージ名＞@＜バージョン番号＞
spack concretize -f
$EDITOR ＜Spackパッケージ内ソースコード＞　←パッチを適用
spack install
spack build-env ＜Spackパッケージ名＞@＜バージョン番号＞ -- bash ←spack install がエラーの場合は原因究明
spack undevelop ＜Spackパッケージ名＞
```

## ライセンス

[MITライセンス](./LICENSE) 準拠とする。
