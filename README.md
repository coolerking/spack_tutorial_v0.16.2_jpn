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
- `compilers.yaml`
- `packages.yaml`

## ライセンス

[MITライセンス](./LICENSE) 準拠とする。
