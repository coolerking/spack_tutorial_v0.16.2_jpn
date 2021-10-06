# チュートリアル：Spack 101

講義とライブデモで構成されたSpackについて講義とライブデモを交え1日で紹介します。このチュートリアルは、2021年に開催されたバーチャルイベント Advanced Research Computing カンファレンス(PEARC'21)の最後に発表されました。

これらの資料を使用して、自分のサイトでSpackのコースを教えることができます。
もしくはスキップして、ライブデモスクリプトを読んで Spack が実際にどのように使用されているかを確認することもできます。

## スライド

- [Spackを使って複雑なHPCソフトウェア環境を管理しよう](../slides/spack_pearc21_tutorial_slides_jpn.pptx) PPTX形式

**引用元：** 2021年7月19日開催のバーチャルイベントAdvanced Research Computing (PEARC’21)における Practice & Experience「Managing HPC Software Complexity with Spack」（Gregory Becker、Robert Blake、Massimiliano Culpo、Tammy Dahlgren, Adam Stewart、そして Todd Gamblin） より。

## ライブデモ

基本的なSpackタスクを段階的に実行するスクリプトを提供します。
これらは上のスライドのセクションに対応しています。
次のいずれかの方法を使用して、スクリプトを実行できます。

1. ローカルマシンでチュートリアルを実行するために使用できる DockerHub 上の [spack/tutorialコンテナイメージ](https://hub.docker.com/r/spack/tutorial) を提供。 `docker run -it spack/tutorial` を実行してコンテナ利用することができる。
2. Dockerに慣れていないユーザがチュートリアルをホストしたい場合、 [AWS](https://aws.amazon.com/) でVMインスタンスもプロビジョニングし、VPMにログインするだけでデモ演習が可能。

これで、デモスクリプトを実行する準備が整いました：

1. [基本的なインストールチュートリアル](01_basic.md)
2. [環境チュートリアル](02_environment.md)
3. [構成チュートリアル](03_config.md)
4. [パッケージ作成チュートリアル](04_package_creation.md)
5. [開発者ワークフローチュートリアル](05_workflow.md)
6. [ミラーチュートリアル](06_mirror.md)
7. [スタックチュートリアル](07_stack.md)
8. [Spackを使ったスクリプト](08_script.md)

過去のチュートリアルの他のセクションも利用できます（最新の状態に保たれない場合があります）。

1. [モジュールファイルチュートリアル](11_module.md)
2. [Spackパッケージビルドシステム](12_build.md)
3. [パッケージングの高度なトピック](13_topic.md)

目次：

## リンク

- [メインSpackドキュメント](https://spack.readthedocs.io/)

## 全チュートリアル目次

- [基本的なインストールチュートリアル](01_basic.md)
  - [Spack インストール](01_basic.md#Spackインストール)
  - [Spack とは？](01_basic.md#Spack とは？)
  - [パッケージのインストール](01_basic.md#パッケージのインストール)
  - [パッケージのアンインストール](01_basic.md#パッケージのアンインストール)
  - [高度な `spack find` の使い方](01_basic.md#高度なspackfindの使い方)
  - [コンパイラのカスタマイズ](01_basic.md#コンパイラのカスタマイズ)
- [環境チュートリアル](02_environment.md)
  - [環境の基本](02_environment.md#環境の基本)
  - [環境の作成および有効化](02_environment.md#環境の作成および有効化)
    - [パッケージのインストール](02_environment.md#パッケージのインストール)
    - [パッケージの使用](02_environment.md#パッケージの使用)
    - [パッケージのアンインストール](02_environment.md#パッケージのアンインストール)
  - [同時に複数の仕様に対応](02_environment.md#同時に複数の仕様に対応)
    - [仕様の追加](02_environment.md#仕様の追加)
    - [仕様の構成](02_environment.md#仕様の構成)
    - [環境の再コンクリート化](02_environment.md#環境の再コンクリート化)
  - [環境内でのビルド](02_environment.md#環境内でのビルド)
  - [ビルドの再現](02_environment.md#ビルドの再現)
    - [環境ファイル](02_environment.md#環境ファイル)
    - [Spackにより管理された環境と独立した環境](02_environment.md#Spackにより管理された環境と独立した環境)
    - [Spackにより管理された環境のレビュー](02_environment.md#Spackにより管理された環境のレビュー)
    - [独立した環境の作成](02_environment.md#独立した環境の作成)
    - [インストールされた環境の更新](02_environment.md#インストールされた環境の更新)
    - [`spack.lock` のレビュー](02_environment.md#spack.lockのレビュー)
    - [環境の再現](02_environment.md#環境の再現)
    - [`spack.yaml` の使用](02_environment.md#spack.yamlの使用)
    - [`spack.lock` の使用](02_environment.md#spack.lockの使用)
  - [さらなる詳細](02_environment.md#さらなる詳細)
    - [環境のセットアップと構築](02_environment.md#環境のセットアップと構築)
    - [環境の使用](02_environment.md#環境の使用)
    - [環境のサンプル](02_environment.md#環境のサンプル)
- [構成チュートリアル](03_config.md)
  - [構成スコープ](03_config.md#構成スコープ)
    - [プラットフォーム固有のスコープ](03_config.md#プラットフォーム固有のスコープ)
  - [YAML形式](03_config.md#YAML形式)
  - [コンパイラー構成](03_config.md#コンパイラ構成)
    - [コンパイラフラグ](03_config.md#コンパイラフラグ)
  - [高度なコンパイラ構成](03_config.md#高度なコンパイラ構成)
  - [パッケージ設定の構成](03_config.md#パッケージ設定の構成)
    - [バリアント設定](03_config.md#バリアント設定)
    - [外部パッケージ](03_config.md#外部パッケージ)
    - [インストール権限](03_config.md#インストール権限)
  - [高度な設定](03_config.md#高度な設定)
  - [まとめ](03_config.md#まとめ)
- [パッケージ作成チュートリアル](04_package_creation.md)
  - [Spackパッケージとは何？](04_package_creation.md#Spackパッケージとは何？)
  - [注意事項](04_package_creation.md#注意事項)
  - [入門](04_package_creation.md#入門)
  - [パッケージファイルの作成](04_package_creation.md#パッケージファイルの作成)
  - [パッケージドキュメントの追加](04_package_creation.md#パッケージドキュメントの追加)
  - [依存関係の追加](04_package_creation.md#依存関係の追加)
  - [パッケージビルドのデバッグ](04_package_creation.md#パッケージビルドのデバッグ)
    - [ビルドログの確認](04_package_creation.md#ビルドログの確認)
    - [パッケージの手動ビルド](04_package_creation.md#パッケージの手動ビルド)
  - [構成引数の指定](04_package_creation.md#構成引数の指定)
  - [バリアントの追加](04_package_creation.md#バリアントの追加)
  - [Specオブジェクトの参照](04_package_creation.md#Specオブジェクトの参照)
    - [Spec バージョンの参照](04_package_creation.md#Specバージョンの参照)
    - [Spec 名の参照](04_package_creation.md#Spec名の参照)
    - [バリアントの参照](04_package_creation.md#バリアントの参照)
  - [クリーンアップ](04_package_creation.md#クリーンアップ)
- [開発者ワークフローチュートリアル](05_workflow.md)
  - [ローカルソースからのインストール](05_workflow.md#ローカルソースからのインストール)
  - [開発反復サイクル](05_workflow.md#開発反復サイクル)
  - [ワークフローまとめ](05_workflow.md#ワークフローまとめ)
- [ミラーチュートリアル](06_mirror.md)
  - [ソースキャッシュミラーの設定](06_mirror.md#ソースキャッシュミラーの設定)
  - [バイナリキャッシュミラーの設定](06_mirror.m#バイナリキャッシュミラーの設定)
  - [キャッシュのまとめ](06_mirror.md#キャッシュのまとめ)
- [スタックチュートリアル](07_stack.md)
  - [スペックマトリックス](07_stack.md#スペックマトリクス)
  - [spack環境での名前付きリスト](07_stack.md#Spack 環境での名前付きリスト)
  - [条件付き定義](07_stack.md#条件付き定義)
  - [view 記述子](07_stack.md#view 記述子)
- [Spackを使ったスクリプト](08_script.md)
  - [チュートリアルのセットアップ](08_script.md#チュートリアルのセットアップ)
  - [`spack find` によるスクリプティング](08_script.md#spackfindによるスクリプティング)
    - [`spack find --format`](08_script.md#spackfind--format)
    - [`spack find --json`](08_script.md#spackfind--json)
  - [`spack python` コマンドの紹介](08_script.md#spackpythonコマンドの紹介)
    - [`Spec` オブジェクトへのアクセス](08_script.md#Specオブジェクトへのアクセス)
  - [Spackデータベース照会](08_script.md#Spackデータベース照会)
  - [スクリプトの使用](08_script.md#スクリプトの使用)
  - [実行可能ファイル `spack-python` の使用](08_script.md#実行可能ファイルspack-pythonの使用)
- [モジュールファイルチュートリアル](11_module.md)
  - [チュートリアルのためのセットアップ](11_module.md#チュートリアルのためのセットアップ)
    - [モジュールツールの作成](11_module.mdモジュールツールの作成)
    - [新しいコンパイラの追加](11_module.md#新しいコンパイラの追加)
    - [チュートリアルで使用するソフトウェアをビルド](11_module.md#チュートリアルで使用するソフトウェアをビルド)
  - [モジュールファイルとは何か？](11_module.md#モジュールファイルとは何か？)
    - [モジュールシステム](11_module.md#モジュールシステム)
      - [環境モジュール](11_module.md#環境モジュール)
      - [Lmod](11_module.md#Lmod)
  - [Spack はどのようにしてモジュールファイルを生成するのか？](11_module.md#Spackはどのようにしてモジュールファイルを生成するのか？)
    - [モジュール vs `spack load`](11_module.md#モジュールvsspackload)
  - [非階層モジュールファイル](11_module.md#非階層モジュールファイル)
    - [環境への不要な変更をフィルタリングする](11_module.md#環境への不要な変更をフィルタリングする)
    - [一部のモジュールファイル作成を抑制する](11_module.md#一部のモジュールファイル作成を抑制する)
    - [モジュールファイル名を変更する](11_module.md#モジュールファイル名を変更する)
    - [カスタム環境の変更を追加するる](11_module.md#カスタム環境の変更を追加する)
    - [依存関係を自動ロードする](11_module.md#依存関係を自動ロードする)
  - [階層モジュールファイル](11_module.md#階層モジュールファイル)
    - [Core/Compiler/MPI](11_module.md#Core/Compiler/MPI)
    - [LAPACKを階層に追加する](11_module.md#LAPACKを階層に追加する)
  - [テンプレートの操作](11_module.md#テンプレートの操作)
    - [モジュールファイルテンプレート](11_module.md#モジュールファイルテンプレート)
    - [デフォルトのテンプレートを拡張する](11_module.md#デフォルトのテンプレートを拡張する)
  - [次のセクションのためのリストア設定](11_module.md次のセクションのためのリストア設定#)
- [Spackパッケージビルドシステム](12_build.md)
  - [Packageクラス階層](12_build.md#Packageクラス階層)
  - [Package](12_build.md#Package)
  - [Autotools](12_build.md#Autotools)
  - [Makefile](12_build.md#Makefile)
  - [CMake](12_build.md#CMake)
  - [PythonPackage](12_build.md#PythonPackage)
  - [その他のビルドシステム](12_build.md#その他のビルドシステム)
- [パッケージングの高度なトピック](13_topic.md)
  - [チュートリアルのセットアップ](13_topic.md#チュートリアルのセットアップ)
  - [パッケージのビルド環境の変更](13_topic.md#パッケージのビルド環境の変更)
    - [ビルド時に依存パッケージの環境変数を設定する](13_topic.md#ビルド時に依存パッケージの環境変数を設定する)
    - [独自のパッケージに環境変数を設定する](13_topic.md#独自のパッケージに環境変数を設定する)
  - [ライブラリ情報の取得](13_topic.md#ライブラリ情報の取得)
    - [依存関係ライブラリへのアクセス](13_topic.md#依存関係ライブラリへのアクセス)
  - [非依存関係へのライブラリ提供](13_topic.md#非依存関係へのライブラリ提供)
  - [その他のパッケージングトピック](13_topic.md#その他のパッケージングトピック)
    - [他のパッケージに属性を添付する](13_topic.md#他のパッケージに属性を添付する)
    - [追加の照会パラメータ](13_topic.md#追加の照会パラメータ)
