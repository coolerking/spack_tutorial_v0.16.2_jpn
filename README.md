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

### ミラー

- ソースコードキャッシュミラーとバイナリキャッシュミラーの２つが存在
- ソースコードキャッシュミラー
  - ソースコードのダウンロードをミラーで代替
  - インターネット不通環境で Spack を運用できる
    - 後付で追加インストールしたパッケージをミラーに加えるにはインストール後再度 `spack mirror create` を実行する

```bash
spack mirror create -d ~/mirror -a
umask 027
chmod -R g+rs ~/mirror
chgrp -R <your_group_name> ~/mirror
```

- バイナリキャッシュミラー
  - Spackソースコードのコンパイルに時間がかかる問題の解決策の１つ
    - もうひとつがSpack チェーン
      - 親となる環境を指定することで親側で構築したバイナリをキャッシュのようにつかう
  - Spack におけるバイナリパッケージは `*.spack` 拡張子がつく
  - まず環境を作り、必要なパッケージを `spack add` だけ実行
  - `spack install` 前にSpack 構成ファイルを `spack config add "config:install_tree:padded_length:128"` を実行して編集
    - 128文字にインストールツリー（パス）を128文字にパディングする
  - キャッシュを使わずに `spack install --no-cache`
    - 初回のインストールのための時間は必要
    - Cコード内のパス指定も128文字以内のものに置き換わってコンパイルされる
  - パッケージ署名をし直す
    - `spack gpg create "My Name" "<my.email@my.domain.com>"`
  - gpgキーはバックアップしておく
  - すべてのSpack管理化非外部パッケージに対して `spack buildcache create --only=package <パッケージ名>`
    - スクリプトでforループを書いて実行するサンプルがチュートリアルに載っている
  - umask、chmod、chgrp
  - ユーザがバイナリキャッシュを使う前にキャッシュ上のすべてのパッケージを信頼することを確認
    - `spack buildcache keys --install --trust --force`

### スタック

- 多くのパッケージを多くのプラットフォームやコンパイラを提供する設備などで提供する際に利用する機能
  - チュートリアルではコンパイルすると時間がかかるので`spack concretize`しての確認にとどめる
- `specs`内の`matrix`記述子でコンパイラリストとパッケージリストを指定すると、両方のリストを行列演算したパッケージがコンパイルされる
　- 依存関係やバリアントもリスト化して記述できる→h行列の掛け算分だけパッケージがコンパイルされる
  - `matrix` 内に `exclude` 記述子をかいてコンパイル除外ペアを指定可能
- `definitions`を使うとリスト記述を名前付きリストにして定義でき、`matrix`内などで`$リストの名前`といった記述で参照できる
  - 名前付きリストに対して`^`を使うこともできる`$`とリスト名のあいだに`^`を入れる
- 制約を`when`を使って記述可能
  - `env`や`hostname`など`when` 句内で使用可能なエンティティ名がある
- `view` 記述子をつかうとビューに登場するパッケージを制限できる
  - 見えないパッケージはビュー上だけ見えていないだけで使用可能

### スクリプティング

- Spackに関する情報照会を行うには`spack find`を使う方法と`spack python`を使う方法がある
- `spack find`コマンドの出力をコントロールするフォーマットオプション `--format`
- `spack find`コマンドの出力をJSONにするするオプション `--json`
- `spack python` でPythonスクリプトを対話形式で照会
- 先頭行`#!` ではじまるPythonスクリプトにする方法で`spack-python`というラッパを使うことでどの環境でも実行可能なPythonスクリプトファイルにする方法

### モジュールファイル

TBD

## ライセンス

[MITライセンス](./LICENSE) 準拠とする。
