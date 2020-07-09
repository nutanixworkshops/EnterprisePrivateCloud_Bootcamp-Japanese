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

Nutanix Enterprise Private Cloud Bootcampへようこそ！この演習では、Nutanix Enterprise Cloud OSを利用して一般的な管理タスクがいかに簡略化されるかを、実運用を想定したシナリオに沿ってご体験頂きます。

アジェンダ
++++++

- はじめに
- ある情シス部員の日常
- Prism Pro
- プラットフォームのセキュリティ
- Files
- Flow

初期セットアップ
+++++++++++++

- 使用されているパスワードをメモします。
- 仮想デスクトップにログインします（以下の接続情報）

環境の詳細
+++++++++++++++++++

Nutanixワークショップは、Nutanix Hosted POC環境を利用します。演習を完了するために必要なイメージ、ネットワーク、VMは事前に構築されています。

ネットワーク
..........

Hosted POC クラスタは 以下の命名規則に従い命名されています:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.**21**.\ *XYZ*\ .0
- **Cluster IP** - 10.**21**.\ *XYZ*\ .37

例:

- **Cluster Name** - POC055
- **Subnet** - 10.21.55.0
- **Cluster IP** - 10.21.55.37

環境内には、* XYZ *をサブネットの正しいオクテットに置き換える必要があるVMが存在しています。たとえば：

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - 説明
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster 仮想IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local ドメインコントローラー

各クラスタは、VMに使用できる2つのVLANで構成されています。

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

  <クラスタ・パスワード>は各クラスタに固有であり、講師よって提供されます。

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

各クラスタには専用のドメインコントローラーVM、DCが存在し、ntnxlab.localドメインにActive Directoryサービスを提供します。ドメインには、次のユーザーとグループが格納されています。

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

Nutanix Hosted POC 環境には以下の方法で接続できます。:

ユーザ名、パスワード
...........................

PHX クラスタ:
**Username:** PHX-POCxxx-User01 〜 PHX-POCxxx-User20, **Password:** *<講師から提供>*

RTP クラスタ:
**Username:** RTP-POCxxx-User01 〜 RTP-POCxxx-User20, **Password:** *<講師から提供>*

Frame VDI
.........

ログイン: https://frame.nutanix.com/x/labs

上記 **Username**, **Password** を入力

Parallels VDI
.................

PHX クラスタ: https://xld-uswest1.nutanix.com

RTP クラスタ: https://xld-useast1.nutanix.com

上記 **Username**, **Password** を入力

Nutanix Version Info
++++++++++++++++++++

- **AHV Version** - AHV 20170830.337
- **AOS Version** - 5.11.2.3
- **PC Version** - 5.11.2.1
