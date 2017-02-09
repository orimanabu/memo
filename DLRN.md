# DLRN

RDOにはDLRNという仕組みがあって、upstreamの各コミットごとに (実際にはRDOのCIが通ったもののみですが)、RDOディストリビューション全体のパッケージセットが用意されています。なので、最新の開発版を追いかけるのもrpm/yum/dnfの仕組みで超楽ちん！のはず。

...なのですが、この文書はその背景を説明するものです。

# 前置き (もしくは用語集)

## OpenStack

- オープンソースのクラウド基盤ソフトウェア

## RDO

- OpenStackのディストリビューション

- 250以上のrpmパッケージ

## RPMパッケージ

- ソース

  - upstreamのソースコード

- スペック

  - パッケージのビルド手順
  - インストール・アンインストール手順
  - 依存関係
  - パッチ

## dist-git

dist-git<sup>[1] (#footnote2)</sup>とは、RPMパッケージ開発用のgitベースのリポジトリです。主に3つのコンポーネントから構成されています。

- Gitolite: アクセス権の管理
- Lookaside Cache: upstreamソースのtarballを保持する場所
- gitリポジトリ: specファイル、パッチ等のgitリポジトリ

![dist-git](https://github.com/release-engineering/dist-git/raw/master/images/storage.png "dist-git")
(図は https://github.com/release-engineering/dist-git より拝借)

実際はdist-git専用のツールはなく(要出典)、その他のバックエンドのシステム/ツールと連携するフロントエンドのツールを使ってパッケージ開発を行います。
例えばFedoraの場合はfedpkg<sup>[2] (#footnote2)</sup>というフロントエンドツールが相当します。fedpkgは、dist-git(を通したsource rpmの取得、ローカルビルド等)、リモートビルドシステム(Koji)、ディストリビューションのリリース管理システム(Bodhi)を統一的に操作できるCLIツールです。fedpkgの使い方はこの資料<sup>[3] (#footnote3)</sup>がまとまっています。

RDOの場合、rdopkg<sup>[4] (#footnote4)</sup>がdist-gitのフロントエンドツールとなります。

<a name="footnote1">1</a>: https://github.com/release-engineering/dist-git<br/>
<a name="footnote2">2</a>: https://fedoraproject.org/wiki/Package_maintenance_guide<br/>
<a name="footnote3">3</a>: https://fedoraproject.org/w/uploads/1/1c/Fedpkg-presentation.pdf<br/>
<a name="footnote4">4</a>: https://www.rdoproject.org/documentation/rdo-packaging/<br/>

## Koji

RPMパッケージをビルドしてリポジトリに配置するツール。コミュニティで使えるKojiは https://cbs.centos.org/koji にあります。

「麹」から取ってきた名前のようです。

# なぜDLRNが必要か

### OpenStackの開発規模

- 6ヶ月ごとのメジャーバージョンアップ

- 例えばNewton(2017/10 GA)の場合...

  - 2700人以上のコントリビューター

  - 600以上のプロジェクト

  - 43000以上のコミット

    - 平均すると一日あたり240コミット

  - Novaだけでも17000以上のコミット、一日平均10コミット

  - GA前1ヶ月は一日平均280コミット

### 難しい点

- upsteramの更新に合わせて、全パッケージの整合性が取れるようにspecも更新し続ける必要がある

- パッケージ間に強い依存関係がある

  - 特にコアパッケージが更新された場合、全てのパッケージを含めて互換性のテストが必要になる

- 数多くのテストを並列にさばくテスト環境が必要

# そこでDLRN

DLRNは、RDOのパッケージをビルドし、テストするための仕組みです。upstream masterのコミットごとにRDOの全パッケージをビルドし、テストし、テストが通ったらそのコミットに対応するリポジトリを作成します。

前述のrdopkgというRDO用dist-gitのフロントエンドツールは、KojiやSoftware Factory[1]、DLRNと連携します。

ワークフローは下図(https://www.rdoproject.org/documentation/rdo-packaging/ より拝借)のようになります。

![DLRN workflow](https://www.rdoproject.org/images/documentation/rdo-full-workflow-high-level-no-buildlogs.png?1464794623 "DLRN workflow")

specに対する変更もGerritでレビューしてgitで管理。リベースも楽ちん。

ところで、DLRNは元々はDeloreanという名前でした[2]。が商標問題とかがあって、最終的にDLRNという名前になりました。

<a name="footnote1">1</a>: OpenStackのCI/CDプラットフォーム。Gerrit、Jenkins、Zuul等を使用します。https://blogs.rdoproject.org/7276/software-factory-enters-beta-and-is-released-as-an-open-source-project<br/>
<a name="footnote1">1</a>: https://www.redhat.com/archives/rdo-list/2016-March/msg00008.html<br/>

# DLRNのがんばり

Newtonリリース時のDLRNの頑張り:

- 800コミット (RDOパッケージのspecに対するコミット)
- 70人のコントリビューター
- DLRNで発見したビルドの不具合: 230
- upstreamでNewtonがリリースされた後、10時間後にRDO Newtonをリリース！

# 参考文献

- Delorean: OpenStack packages from the future: [xxx] (https://blogs.rdoproject.org/7834/delorean-openstack-packages-from-the-future)
- Fosdem'17: RDO's continuous packaging platform - How to continuously package OpenStack (or other things) for CentOS [yyy] (https://fosdem.org/2017/schedule/event/rdo_continuous_packaging_platform/)
