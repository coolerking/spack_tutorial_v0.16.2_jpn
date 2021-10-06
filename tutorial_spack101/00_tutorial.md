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
  - [構成スコープ](03_config.md#)
  - [プラットフォーム固有のスコープ](03_config.md#)
  - [YAML形式](03_config.md#)
  - [コンパイラー構成](03_config.md#)
  - [コンパイラフラグ](03_config.md#)
  - [高度なコンパイラ構成](03_config.md#)
  - [パッケージ設定の構成](03_config.md#)
  - [バリアント設定](03_config.md#)
  - [外部パッケージ](03_config.md#)
  - [インストール権限](03_config.md#)
  - [高レベルの構成](03_config.md#)
  - [結論](03_config.md#)
- [パッケージ作成チュートリアル](04_package_creation.md)
  - [Spackパッケージとは何ですか？](04_package_creation.md#)
  - [警告](04_package_creation.md#)
  - [入門](04_package_creation.md#)
  - [パッケージファイルの作成](04_package_creation.md#)
  - [パッケージドキュメントの追加](04_package_creation.md#)
  - [依存関係の追加](04_package_creation.md#)
  - [パッケージビルドのデバッグ](04_package_creation.md#)
  - [ビルドログの確認](04_package_creation.md#)
  - [手動で構築する](04_package_creation.md#)
  - [構成引数の指定](04_package_creation.md#)
  - [バリアントの追加](04_package_creation.md#)
  - [スペックオブジェクトのクエリ](04_package_creation.md#)
  - [スペックバージョンのクエリ](04_package_creation.md#)
  - [スペック名のクエリ](04_package_creation.md#)
  - [バリアントのクエリ](04_package_creation.md#)
  - [清掃](04_package_creation.md#)
- [開発者ワークフローチュートリアル](05_workflow.md)
  - [ローカルソースからのインストール](05_workflow.md#ローカルソースからのインストール)
  - [開発反復サイクル](05_workflow.md#開発反復サイクル)
  - [ワークフローの概要](05_workflow.md#ワークフローの概要)
- [ミラーチュートリアル](06_mirror.md)
  - [ソースキャッシュミラーの設定](06_mirror.md#ソースキャッシュミラーの設定)
  - [バイナリキャッシュミラーの設定](06_mirror.m#バイナリキャッシュミラーの設定)
  - [キャッシュの概要](06_mirror.md#キャッシュの概要)
- [スタックチュートリアル](07_stack.md)
  - [スペックマトリックス](07_stack.md#)
  - [spack環境での名前付きリスト](07_stack.md#)
  - [条件付き定義](07_stack.md#)
  - [記述子を表示](07_stack.md#)
- [Spackを使ったスクリプト](08_script.md)
  - [チュートリアルの設定](08_script.md#)
  - [とのスクリプティング spack find](08_script.md#)
  - [spack find --format](08_script.md#)
  - [spack find --json](08_script.md#)
  - [コマンドの紹介spack python](08_script.md#)
  - [Specオブジェクトへのアクセス](08_script.md#)
  - [Spackデータベースのクエリ](08_script.md#)
  - [スクリプトの使用](08_script.md#)
  - [spack-python実行可能ファイルの使用](08_script.md#)
- [モジュールファイルチュートリアル](11_module.md)
  - [チュートリアルのセットアップ](11_module.md#)
  - [モジュールツールを作成する](11_module.md#)
  - [新しいコンパイラを追加する](11_module.md#)
  - [チュートリアルで使用するソフトウェアをビルドします](11_module.md#)
  - [モジュールファイルとは何ですか？](11_module.md#)
  - [モジュールシステム](11_module.md#)
  - [Spackはどのようにモジュールファイルを生成しますか？](11_module.md#)
  - [モジュールvs spack load](11_module.md#)
  - [非階層モジュールファイル](11_module.md#)
  - [環境への不要な変更をフィルタリングする](11_module.md#)
  - [一部のモジュールファイルが生成されないようにする](11_module.md#)
  - [モジュールファイルの命名を変更する](11_module.md#)
  - [カスタム環境の変更を追加する](11_module.md#)
  - [依存関係を自動ロード](11_module.md#)
  - [階層モジュールファイル](11_module.md#)
  - [コア/コンパイラ/ MPI](11_module.md#)
  - [LAPACKを階層に追加します](11_module.md#)
  - [テンプレートの操作](11_module.md#)
  - [モジュールファイルテンプレート](11_module.md#)
  - [デフォルトのテンプレートを拡張する](11_module.md#)
  - [今後のセクションの設定を復元する](11_module.md#)
- [Spackパッケージビルドシステム](12_build.md)
  - [Packageクラス階層](12_build.md#Packageクラス階層)
  - [Package](12_build.md#Package)
  - [Autotools](12_build.md#Autotools)
  - [Makefile](12_build.md#Makefile)
  - [CMake](12_build.md#CMake)
  - [PythonPackage](12_build.md#PythonPackage)
  - [その他のビルドシステム](12_build.md#その他のビルドシステム)
- [パッケージングの高度なトピック](13_topic.md)
  - [チュートリアルのセットアップ](13_topic.md#)
  - [パッケージのビルド環境の変更](13_topic.md#)
  - [ビルド時に依存パッケージに環境変数を設定する](13_topic.md#)
  - [独自のパッケージに環境変数を設定する](13_topic.md#)
  - [図書館情報の取得](13_topic.md#)
  - [依存関係ライブラリへのアクセス](13_topic.md#)
  - [扶養家族への図書館の提供](13_topic.md#)
  - [その他のパッケージングトピック](13_topic.md#)
  - [他のパッケージに属性を添付する](13_topic.md#)
  - [追加のクエリパラメータ](13_topic.md#)
