.. title:: Nutanix Enterprise Private Cloud Bootcamp

.. toctree::
   :maxdepth: 2
   :caption: Enterprise Private Cloud
   :name: _enterprise_privatecloud
   :hidden:

   dayinlife/dayinlife
   prismops/prismops_capacity_lab/prismops_capacity_lab
   prismops/prismops_rightsize_lab/prismops_rightsize_lab
   security/security
   files/files
   flow_secure_fiesta/flow_secure_fiesta
..   beam_cost_governance/beam_cost_governance

.. toctree::
  :maxdepth: 2
  :caption: Optional Labs
  :name: _optional_labs
  :hidden:

  image_create/image_create
.. lab_image_configuration/lab_image_configuration

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  appendix/glossary

.. _getting_started:

---------------
はじめに
---------------

Nutanix Enterprise Private Cloud Bootcampへようこそ！このワークブックには、Nutanix Coreテクノロジーと多くの一般的な管理タスクを紹介する、インストラクター主導のセッションが含まれています。

Prism Centralを探索し、その機能とナビゲーションに慣れます。Prismを使用して、ストレージやネットワークなどの基本的なクラスター管理タスクを実行します。また、PrismおよびAHVを使用した基本的なVMの導入および管理タスクについても説明します。最後に、スナップショットやレプリケーションを含むVMデータ保護について説明します。インストラクターが演習を説明し、あなたが持つかもしれない追加の質問に答えます。

ブートキャンプの最後に、参加者は、Nutanix Enterprise Cloudスタックを構成するコアの概念とテクノロジーを理解し、ホスト型またはオンサイトの概念実証（POC）契約に備える準備を整える必要があります。

アジェンダ
++++++

- はじめに
- ある情シス部員の日常
- Prism Ops
- プラットフォームのセキュリティ
- Files
- Flow

はじめに
+++++++++++++

- お名前
- Nutanixの知識

初期セットアップ
+++++++++++++

- 使用されているパスワードをメモします。
- 仮想デスクトップにログインします（以下の接続情報）

環境の詳細
+++++++++++++++++++

Nutanixワークショップは、Nutanix Hosted POC環境で実行することを目的としています。演習を完了するために必要なすべての必要なイメージ、ネットワーク、VMがクラスターにプロビジョニングされます。

ネットワーキング
..........

Hosted POCクラスターは、標準の命名規則に従います。

- **クラスター名** - POC\ *XYZ*
- **サブネット** - 10.**21**.\ *XYZ*\ .0
- **クラスタIP** - 10.**21**.\ *XYZ*\ .37

例えば:

- **クラスター名** - POC055
- **サブネット** - 10.21.55.0
- **クラスターIP** - 10.21.55.37

ワークショップ全体を通して、XYZをサブネットの正しいオクテットに置き換える必要がある複数のインスタンスがあります。次に例を示します。

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - Description
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

各クラスターは、VMに使用できる2つのVLANで構成されています。

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.21.\ *XYZ*\ .1/25
    - 0
    - 10.21.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.21.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.21.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

認証情報
...........

.. note::

  <クラスタ・パスワード>は各クラスタに固有であり、ワークショップのリーダーによって提供されます。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - ノード
     - ユーザ名
     - パスワード
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

各クラスターには専用のドメインコントローラーVM、DCが存在し、ntnxlab.localドメインにActive Directoryサービスを提供します。ドメインには、次のユーザーとグループが格納されています。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1
   
   * - グループ
     - ユーザ名
     - パスワード
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

アクセス手順
+++++++++++++++++++

Nutanix Hosted POC環境には、さまざまな方法でアクセスできます。

ラボアクセスのユーザー認証情報
...........................

PHX Based Clusters:
**ユーザ名:** PHX-POCxxx-User01 (up to PHX-POCxxx-User20), **パスワード:** *<Provided by Instructor>*

RTP Based Clusters:
**ユーザ名:** RTP-POCxxx-User01 (up to RTP-POCxxx-User20), **パスワード:** *<Provided by Instructor>*

Frame VDI
.........

ログイン: https://frame.nutanix.com/x/labs

**Nutanix社員** - **NUTANIXDC**の認証情報を使用
**非Nutanix社員** - **Lab Access User**の認証情報を使用

Parallels VDI
.................

PHX Based Clusters ログイン: https://xld-uswest1.nutanix.com

RTP Based Clusters ログイン: https://xld-useast1.nutanix.com

**Nutanix Employees** - **NUTANIXDC**の認証情報を使用
**Non-Employees** - **Lab Access User**の認証情報を使用

Pulse Secure VPN
..........................

クライアントをダウンロードします:

PHX Based Clusters ログイン: https://xld-uswest1.nutanix.com

RTP Based Clusters ログイン: https://xld-useast1.nutanix.com

**Nutanix Employees** - **NUTANIXDC**の認証情報を使用
**Non-Employees** - **Lab Access User**の認証情報を使用

クライアントをインストールします。

Pulse Secure Clientで、接続を**追加**します。

For PHX:

- **Type** - ポリシー保護（UAC）または接続サーバー
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

For RTP:

- **Type** - ポリシー保護（UAC）または接続サーバー
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com


Nutanixバージョン情報
++++++++++++++++++++

- **AHVバージョン** - AHV 20170830.337
- **AOSバージョン** - 5.11.2.3
- **PCバージョン** - 5.11.2.1
