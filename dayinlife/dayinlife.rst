.. _dayinlife:

-----------------
ある情シス部員の日常
-----------------

このラボでは、この道10年のベテランである情シス部員、新谷さんの1日を追います。彼は3層アーキテクチャの仮想環境を管理しておりましたが、この度Nutanixを導入しました。Nutanixクラスタは、本番環境において様々なITワークロード向けに使用されています。また、会社の主要なアプリケーションであるFiestaと呼ばれる在庫管理ソリューションの開発作業をサポートしています。

.. 注意::

   同じNutanixクラスタを複数人で共用している場合、いくつかの手順はすでに完了している可能性があります。ステップが正しく完了していることを確認した後、ラボを続行してください。

ストレージの構成
+++++++++++++++++++

この演習では、数回クリックするだけでPrism(Nutanixプラットフォームの管理コンソール)を介して仮想環境に対してストレージをプロビジョニングし、監視する方法を体験します。従来のSANストレージの計画と管理とは異なり、これらのステップは非常に簡単です。

#. 講師より割り当てられた、 **Prism Element** のクラスタIPを確認します。

#. ブラウザーで **Prism Element** を開き、次のローカルユーザー資格情報を使用してログインします。

   - **Username** - admin
   - **Password** - *HPOC Password*

   .. figure:: images/1.png

#. ドロップダウンメニューから **Storage(ストレージ)** を選択します。

   .. figure:: images/2.png

#. **Storage Summary(ストレージ概要)** ウィジェットと **Capacity Optimization(容量最適化)** ウィジェットの上にマウスを置くと、それぞれが何を表示しているかの説明が表示されます。

   ここで重要なのはNutanixが **Data Reduction** として定義しているのは、データ圧縮、重複排除、イレイジャーコーディングによる節約のみであるということです 一方、 **Overall Efficiency(全体の効率性)** では他ベンダが”データ削減”として定義しているのと同じく、シンプロビジョニングやインテリジェントクローニングやゼロサプレッション等のデータ効率のための機能を含んだものとなっています。

   .. figure:: images/3.png

#. **Table(テーブル)** を選択し、 **+ Storage Container(ストレージコンテナ)** をクリックします。

   ストレージコンテナは、ストレージの論理ポリシーを表し、容量の予約、圧縮、重複排除、イレイジャーコーディングなどのデータ効率化機能の有効化/無効化、リダンダンシーファクターの構成を可能にします。Nutanixクラスタ上のすべてのストレージコンテナは、ストレージプールと呼ばれる、クラスタ内のすべての物理ディスクを引き続き利用します。典型的なNutanixクラスタには少数のストレージコンテナがあり、通常、さまざまなデータ効率化テクノロジーの恩恵を受けるワークロードに対応しています。

   .. figure:: images/4.png

#. **ストレージコンテナ** に一意の **名前** を指定し、 **Advanced Settings** をクリックして追加の構成オプションを確認します。

   .. figure:: images/5.png

   Nutanixは、インテリジェントでワークロード特性に適応するストレージ容量を最適化するさまざまな方法を提供します。Nutanixは、ネイティブデータ回避（シンプロビジョニング、インテリジェントクローニング、ゼロサプレッション）およびデータ削減（圧縮、重複排除、イレイジャーコーディング）技術を使用して、データを効率的に処理します。すべてのデータ削減最適化はコンテナレベルで実行されるため、コンテナごとに異なる設定を使用できます。

   **圧縮**

   Nutanixには、インラインまたはポストプロセスデータ圧縮という2つの選択肢があります。インラインまたはポストプロセスデータ圧縮に関係なく、OpLogに入る書き込みデータは> 4kであり、圧縮されてOpLogに書き込まれます。インライン圧縮（Delay = 0）の場合、データのシーケンシャルストリームまたは大きなサイズのI / O（> 64K）は、エクステントストアに書き込むときに圧縮されます。ポストプロセス圧縮（遅延> 0）の場合、データは、OpLogからエクステントストアに排出された後、圧縮遅延が満たされた後に圧縮されます。

   圧縮により、データベースなどのアプリケーションのディスク上のスペースが節約され、ストレージに書き込まれる書き込みの数が少なくなります。ポストプロセス圧縮は、すべてのコンテナでデフォルトでオンになっています。5.18以降、すべてのコンテナでインライン圧縮がデフォルトでオンになります。ほとんどすべてのユースケースでインライン圧縮をオンにすることをお勧めします。圧縮に理想的でないワークロードは、暗号化されたデータセットまたはすでに圧縮されたデータセットです。

   **イレイジャーコーディング**

   可用性と必要なストレージの量のバランスをとるために、分散ストレージファブリック（DSF）はイレイジャーコード（EC）を使用してデータをエンコードする機能を提供します。パリティが計算されるRAID（レベル4、5、6など）と同様に、ECは異なるノード間でデータブロックのストリップをエンコードし、パリティを計算します。ホストまたはディスク、あるいはその両方に障害が発生した場合、パリティデータを使用して、欠落しているデータブロックが計算されます（デコード）。DSFの場合、データブロックは別のノードにあり、別のvDiskに属している必要があります。ECは後処理操作であり、コールドデータの書き込み（7日以上上書きされていないデータ）に対して実行されます。ストリップ内のデータブロックとパリティブロックの数は、ノードの数と許容される構成済みの障害に基づいてシステムによって選択されます。

   イレイジャーコーディングは書き込みコールドデータで機能し、より使いやすいストレージを提供するため、ミッションクリティカルではないワークロードと大量の書き込みコールドデータがあるワークロードに対してEC-X（イレイジャーコーディング）をオンにします。詳細については、 `アプリケーション固有のベストプラクティスガイド <https://portal.nutanix.com/page/documents/solutions/list/>`_ を参照してください。

   **重複排除**

   有効にすると、分散ストレージファブリック（DSF）はキャパシティ層とパフォーマンス層の重複排除を実行します。データは、メタデータとして保存されているSHA-1ハッシュを使用して、取り込み時にフィンガープリントされます。同じフィンガープリントを持つ複数のコピーに基づいて重複データが検出された場合、バックグラウンドプロセスが重複を削除します。重複排除されたデータが読み取られると、それは統合キャッシュに配置され、同じフィンガープリントのデータに対する後続の要求は、キャッシュから直接満たされます。

   フルクローン、P2V移行、永続デスクトップには重複排除が推奨されます。

   **リダンダンシーファクター（冗長度）**

   リダンダンシーファクターは、データコピーの数を制御します。このクラスタにはリダンダンシーファクターを構成できないことに注意してください。これは、RF3をサポートするために必要なノードの最小数が5であるためです。

   .. 注意::

      Nutanixがデータを保護する方法またはデータ削減を実装する方法の詳細については、下の図をクリックしてNutanixバイブルの関連セクションを確認してください。

      .. figure:: https://nutanixbible.com/imagesv2/data_protection.png
         :target: https://nutanixbible.com/#anchor-book-of-acropolis-data-protection
         :alt: Nutanix Bible - Data Protection

#. **Save** をクリックしてストレージを作成し、クラスタ内の使用可能なすべてのホストにマウントします。

   vSphereまたはHyper-V環境では、ストレージコンテナを作成すると、ハイパーバイザーにストレージをマウントするプロセスも自動化されます。

#. 既存のストレージコンテナを選択し、さまざまなデータ削減/回避機能による個々の節約と、全体的な効率に基づいて利用可能なストレージの予測である **Effective Capacity** を確認します。これらの値は、**Storage Container Details** テーブルにあります。

   残念ながら、共有環境でクラスタのデータ復元機能をテストすることはできませんが、以下の短いビデオでは、クラスタ内のノードが予期せず失われた場合のPrismのエクスペリエンスについて説明します。


   .. raw:: html

     <center><iframe width="640" height="360" src="https://www.youtube.com/embed/hA4l1UHZO2w?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

新しいネットワークのプロビジョニング
++++++++++++++++++++++++++

この演習では、新谷さんはPrismを使用して、クラスタの新しいVMネットワークを構成します。

AHVは、すべてのVMネットワーキングにOpen vSwitch（OVS）を活用します。OVSは、Linuxカーネルに実装され、マルチサーバー仮想化環境で動作するように設計されたオープンソースソフトウェアスイッチです。各AHVサーバーはOVSインスタンスを維持し、すべてのOVSインスタンスが結合して単一の論理スイッチを形成します。各ノードは通常、仮想ネットワークとして公開される複数のVLANにトランク/タグ付けされた物理スイッチポートにアップリンクされます。

#. **Prism Element** ドロップダウンメニューから **VM（仮想マシン）** を選択します。

#. **Network Config（ネットワーク設定）** を選択します。

   .. figure:: images/9.png

#. **+ Create Network（ネットワークを作成）** をクリックし、以下の情報を入力します。

   - **Name（名前）** - *Initials* -Network_IPAM
   - **VLAN ID** - パラメータシート **VLAN ID** を参照
   - **Enable IP Address Management（IPアドレス管理を可能にする）** を選択
   - **Network IP Address / Prefix Length（ネットワーク IP アドレス/プリフィックスの長さ）** - 10.0.0.0/24
   - **Gateway（ゲートウェイIPアドレス）** - 10.0.0.1
   - **Configure Domain Settings（ドメイン設定の構成）** を選択しない
   - **+ Create Pool（プールを作成）** を選択
   - **Start Address（開始アドレス）** - 10.0.0.100
   - **End Address（終了アドレス）** - 10.0.0.150
   - **Submit（送信）** をクリック

   .. figure:: images/network_config_03.png

   AHVは統合DHCPサービス（IPAM）を提供できるため、仮想化管理者は構成済みプールからIPをVMに割り当てることができます。また、仮想NICをVMに追加するときに、IPをDHCP予約として簡単に指定できます。

#. **Save（保存）** をクリックします。

   これで、構成された仮想ネットワークがクラ​​スター内のすべてのノードで利用できるようになります。AHVの仮想ネットワークはESXiの分散仮想スイッチのように動作します。つまり、クラスタ内の個々のホストごとに同じ設定を構成する必要はありません。

#. **Network Configuration（ネットワーク設定）** ウィンドウを閉じます。

   これで完了です！

VM作成リクエストへの応答
++++++++++++++++++++++++++++++++++

仮想化管理者は通常、新しいVMの展開を担当します。この演習では、新谷さんがNutanix管理者としてPrismにAHV VMをデプロイする手順を説明します。

#. **Prism Element** のドロップダウンメニューから **VM（仮想マシン）** ページに移動します。

#. **+ Create VM（仮想マシンを作成）** をクリックします。

   .. figure:: images/10.png

#. 次のフィールドに入力して、ユーザーVMリクエストを完了します。

   - **Name** - *Initials*\ -WinToolsVM
   - **Description** -
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - **+ Add New Disk** を選択
      - **Type** - DISK
      - **Operation** - イメージサービスからクローン
      - **Image** - WinToolsVM.qcow2
      - Select **Add**

   - **Add New NIC** を選択
      - **VLAN Name** - Secondary
      - **Add** をクリック

パブリッククラウドプロバイダーと同様に、Nutanix AHVはイメージサービス機能を提供し、インポートしたファイルのストアを構築して、VMの作成時にISOイメージまたはオペレーティングシステムからCD-ROMデバイスをマウントしたり、ディスクイメージからオペレーティングシステムをマウントしたりできます。Image Serviceは、raw、vhd、vhdx、vmdk、vdi、iso、およびqcow2ディスク形式をサポートしています。

VM作成ウィザードには、Windows Sysprep自動化用のUnattend.xmlファイル、またはLinux OS構成用のCloud-Initファイルを指定する機能もあります。

#. **Save** をクリックしてVMを作成します。

   .. 注意::

      VMの作成を含む多くのVM操作は、AHV CLI、``acli``を使用してスクリプト化できます。現在、セキュアブートやvNUMAなどの特定の機能は、コマンドラインを介してVMに対してのみ有効にできます。ACLIリファレンスガイドは `こちら <https://portal.nutanix.com/#/page/docs/details?targetId=Command-Ref-AOS-v5_16:acl-acli-vm-auto-r.html>`_ です。

      Nutanix CVMのいずれかにSSH接続し、acliを使用して追加のVMの作成を試みることができます。

#. テーブルの上部にある検索フィールドを使用して、リクエストされたVMをフィルタリングします。VMを選択し、表の下のアクションのリストから **Power On** をクリックします。

   .. figure:: images/12.png

#. VMの起動が完了したら、 **IP Address** をメモします。

   .. figure:: images/11.png

以前のインフラストラクチャでは、新谷さんは新しく作成されたVMネットワークが期待どおりに機能しないという問題があり、問題の原因を特定するためにネットワーク管理者と長いトラブルシューティングセッションに従事する必要がありました。AHVを使用すると、新谷さんはプロビジョニングした仮想マシンの完全なネットワークパスを簡単に視覚化できます。

#. **Prism Element** の左上プルダウンから **Network（ネットワーク）** ページを選択し、VLANまたはVM名でフィルタリングして、試してみてください。

   .. figure:: images/13.png

ユーザーセルフサービスを有効にする
++++++++++++++++++++++++++

PrismやacliはVMを作成するための簡単なワークフローを提供しますが、新谷さんは定期的にこれらのリクエストが殺到し残業が定常化しており、老朽化したインフラストラクチャの近代化と息子のサッカーの試合観戦にもっと時間を費やしたいと思っています。

次の演習では、 **Prism Central** のネイティブ機能を利用してIaaSセルフサービスをユーザーに提供することで自身の負荷を軽減します。

#. **Prism Element** の **Home（ホーム）** ページに移動します。

#. PrismCentral OKとなっている項目の、 **Launch（起動）** ボタンをクリックし、**Prism Central** に次の資格情報でログインします。

   - **User Name** - admin
   - **Password** - *HPOC Password*

   .. figure:: images/6.png

カテゴリの探索
====================

 **Category** はキーと値のペアです。カテゴリは、いくつかの基準（ロケーション、製品レベル、アプリ名など）に基づいてエンティティ（VM、ネットワーク、イメージなど）に割り当てられます。

たとえば、開発、財務、人事などの値を含む部門カテゴリがあるとします。この場合、開発と人事に適用される1つのバックアップポリシーと、財務のみに適用される別の（より厳格な）バックアップポリシーを作成できます。カテゴリを使用すると、エンティティグループ全体にさまざまなポリシーを実装でき、Prism Centralを使用すると、確立された関係をすばやく表示できます。

この演習では、新谷さんのカスタムカテゴリを作成して、Fiestaアプリチームの適切なリソースへのアクセスを調整します。

#. **Prism Central** にて :fa:`bars` **> Virtual Infrastructure（仮想インフラ） > Categories（カテゴリ）** を選択します。

   .. figure:: images/14.png

#. **New Category** をクリックし、次のフィールドに入力します。

   - **Name** - *Initials*\ -Team
   - **Purpose** - アプリケーションチームへのアクセス許可
   - **Values** - Fiesta

#. **Save（保存）** をクリックします。

#. 既存の **Environment** カテゴリをクリックして、次のフィールドに入力します。 **Environment** は **SYSTEM** カテゴリーであり、追加の値を追加することはできますが、カテゴリーまたはそのままの値を変更または削除することはできません。

   .. figure:: images/16.png

#. :fa:`bars` **> Virtual Infrastructure（仮想インフラ） > VMs（仮想マシン）** を選択します。

#. **AutoAD** と **NTNX-BootcampFS-1** のVMsのチェックボックスにチェックした状態で **Actions（アクション） > Manage Categories（カテゴリの管理）** をクリックします。

   .. figure:: images/17.png

   .. 注意::

      参加者の数によっては、選択する必要があるVMの一部が別のページにある場合があります。対象のVMを検索するか、クリックして追加のページを表示してVMを選択するか、追加の行を表示することを選択します。これらの手法はいずれも、インターフェースの右上部分で実行できます。

#. 検索バーで **Environment** と入力し、 **Production** の値を選択してから、プラス記号をクリックします。※既にProductionカテゴリが割り当てられている場合は、キャンセルをクリックし、手順8をスキップする。

   .. figure:: images/18.png

   .. 注意::

      セキュリティ、保護、またはリカバリポリシーに関連付けられているカテゴリの場合、関連するポリシーがこのウィンドウに表示され、カテゴリをエンティティに適用した場合の影響が示されます。

#. **Save** をクリックします。

#. 前の演習で新谷さんによってプロビジョニングされた ** *Initials*\ -WinToolsVM** を選択し、 **Actions（アクション） > Manage Categories（カテゴリの管理）** をクリックします。 ** *Initials*\ -Team: Fiesta** カテゴリを割り当て、 **Save（保存）** をクリックします。

ロールの探索
===============

デフォルトでは、Prism Centralには、一般的なユーザーペルソナにマップするいくつかのロールが付属しています。ロールは、ユーザーが実行できるアクションを定義し、カテゴリまたは他のエンティティにマップされます。

新谷さんは、Fiestaチームで作業する2種類のユーザー、テスト環境用にVMをプロビジョニングする必要があるDeveloper、および組織内の複数の環境を監視するが、各環境を変更する機能が非常に制限されているOperatorをサポートする必要があります。

#. **Prism Central** で :fa:`bars` **> Administration（管理） > Roles（ロール）** を選択する。

   組み込みの開発者ロールにより、ユーザーはVMの作成と変更、Calmブループリントの作成、プロビジョニング、管理などを行うことができます。

#. 組み込みの **Developer** ロールをクリックし、必要に応じてロールの承認されたアクションを確認します。 画面右上の**Manage Assignment** をクリックします。

   .. figure:: images/19.png

#. **Users and Groups（ユーザーとグループ）** で、ntnxlab.localドメインから自動的に検出される **SSP Developers** のユーザーグループを指定します。

#. **Entities（エンティティ）** で、新エンティティをクリックし、ドロップダウンメニューを使用して次のリソースを指定します。※既に割り当てられている場合は、キャンセルをクリックし手順5をスキップする。

   - **AHV Cluster** - *割り当てられたCluster（xxx-POCxx）*
   - **AHV Subnet** - Secondary
   - **Category** - Environment:Testing, Environment:Staging, Environment:Dev, *Initials*\ -Team:Fiesta

   .. figure:: images/20.png

#. **Save（保存）** をクリックし、右上のXをクリックしてこの画面を閉じます。

   デフォルトのOperatorロールには、ブループリントからデプロイされたVMとアプリケーションを削除する機能が含まれていますが、これは私たちの環境では望ましくありません。新しいロールを最初から構築するのではなく、既存のロールにクローンを作成し、ニーズに合わせて変更できます。必要なOperatorのロールは、VMメトリックを表示し、電源操作を実行し、vCPUやメモリなどのVM構成を更新して、アプリケーションのパフォーマンスの問題に対処できる必要があります。

#. **Operator** ロールをクリックし、 **Duplicate** をクリックします。

#. 次のフィールドに入力し、 **Save** をクリックしてカスタムのロールを作成します。

   - **Role Name（ロール名）** - *Initials*\ -SmoothOperator
   - **Description（説明）** - Limited operator accounts
   - **App（アプリ）** - No Access
   - **VM（仮想マシン）** - Edit Access
   - **Allow VM（仮想マシン） Creation** は選択 **しない** 。

   .. figure:: images/21.png

#. **Prism** を更新し、前手順で作成した **SmoothOperator** ロールをクリックします。 **Manage Assignment** をクリックします。

#. 次の割り当てを作成します。

   - **Users and Groups（ユーザーとグループ）** - operator01
   - **Entity Categories（エンティティ）** - Environment:Production, Environment:Testing, Environment:Staging, Environment:Dev

   Operator01は、Environment カテゴリのいずれかでタグ付けされたすべてのVMにアクセスできるユーザーですが、特定のクラスタへの一般的なアクセス権はありません。

   **New Users（新ユーザー）** をクリックして、同じロールに割り当てを追加します。

   - **Users and Groups（ユーザーとグループ）** - operator02
   - **Entity Categories（エンティティ）** - Environment:Dev, *Initials*\ -Team:Fiesta

   Operator02は、DevまたはFiestaカテゴリー値のいずれかでタグ付けされたすべてのVMを表示するユーザーです。

   .. figure:: images/22.png

   **Save（保存）** をクリックします。

#. 新谷さんなどのインフラストラクチャ管理者は、次を選択して、ADユーザーを **Prism Admin** 、または **Super Admin** ロールにマップ出来ます。 :fa:`bars` **> Prism Central Settings（PrismCentral設定） > Role Mapping（役割管理）** に移動し、 **Cluster Admin** と **User Admin** のロールをADアカウントである **adminuser01** に追加します。※既に追加されている場合は本手順をスキップする。

   .. figure:: images/28.png

プロジェクトの探索
==================

前の演習は、新谷さんのユーザーに基本的なVM作成のセルフサービスを提供するのに十分ですが、彼らの作業の多くは、複数のVMで構成されるアプリケーションにより構成されています。開発、テスト、またはステージング環境で複数のVMを手動で展開すると時間がかかり、不整合や人為的ミスが発生しやすくなります。ユーザーに優れたエクスペリエンスを提供するために、新谷さんはNutanix Calmを導入します。

Nutanix Calmを使用すると、プライベート（AHV、ESXi）とパブリッククラウド（AWS、Azure、GCP）の両方のインフラストラクチャでアプリケーションを構築、プロビジョニング、管理できます。

インフラストラクチャ以外の管理者がCalmにアクセスしてアプリケーションを作成または管理できるようにするには、まずユーザーまたはグループをプロジェクトに割り当てる必要があります。プロジェクトは、ユーザーのロール、インフラストラクチャリソース、およびリソースクォータを定義する論理単位として機能します。プロジェクトは、一連の共通の要件または共通の構造と機能を持つユーザーを定義します。たとえば、Fiestaプロジェクトで協力するエンジニアのチームなどです。

#. **Prism Central** において、 :fa:`bars` ** > Services > Calm** を選択します。

#. 左手のメニューで **Projects** を選択し、 **+ Create Project** をクリックします。

   .. figure:: images/23.png

#. 次のフィールドに入力します。

   .. 注意::

      インフラストラクチャを追加する前にユーザー/グループマッピングを追加すると、インフラストラクチャの追加が失敗する可能性があります。これを回避するには、ユーザー/グループマッピングの前にインフラストラクチャを追加します。

   - **Project Name** - *Initials*\ -FiestaProject

   - **Infrastructure**, で **Select Provider > Nutanix** と選択

   - **Select Clusters & Subnets** をクリック

   - *割り当てられたCluster* を選択

   - **Subnets**, にて **Primary**, **Secondary**, を選択し、 **Confirm** をクリック

   - *Primary* の :fa:`star` をクリックしデフォルトネットワークに設定

   - **Users, Groups, and Roles**, にて **+ User** を選択

      - **Name** - SSP Developers
      - **Role** - Developer
      - **Action** - Save

   - **+ User** を選択

      - **Name** - Operator02
      - **Role** - *Initials*\ -SmoothOperator
      - **Action** - Save

   - **Quotas**, にて以下を指定

      - **vCPUs** - 100
      - **Storage** - 空白
      - **Memory** - 100

   .. figure:: images/24.png

#. **Save & Configure Environment** をクリックします。

``Environmentページに遷移しますが、ここでは何も設定する必要はありません。次のステップに移動して下さい。``

すべてのオペレーターアカウントではなく、 **Operator02** のみが **Calm** プロジェクトへのアクセス権を与えられたことに注意してください。

ブループリントの構築
==================

Nutanix Calmのブループリントは、アプリケーションをモデル化するためのフレームワークです。ブループリントは、作成されるサービスおよびアプリケーションでタスクをプロビジョニング、構成、および実行するために必要なすべてのステップを記述するテンプレートです。ブループリントは、アプリケーションとその基盤となるインフラストラクチャのライフサイクルも定義します。これは、アプリケーションの作成から、アプリケーションで実行されるアクション（ソフトウェアの更新、スケールアウトなど）、そしてアプリケーションの終了までです。

ブループリントを使用して、さまざまなアプリケーションをモデル化できます。単一の仮想マシンのプロビジョニングから、複数の仮想マシン、複数レイヤからなるWebアプリケーションのプロビジョニングと管理までのライフサイクル管理が可能です。

開発者ユーザーは独自のブループリントを作成および公開することができますが、新谷さんはチームが使用する共通のFiestaブループリントを提供したいと考えています。

#. `こちら <https://raw.githubusercontent.com/nutanixworkshops/EnterprisePrivateCloud_Bootcamp-Japanese/master/dayinlife/Fiesta-Multi.json>`_.　を右クリックして、Fiestaマルチブループリントをダウンロードします。

#. **Prism Central > Calm** に移動し、左手のメニューから **Blueprints** を選択、 **Upload Blueprint** をクリックします。

   .. figure:: images/25.png

#. **Fiesta-Multi.json** を選択します。

#. **Blueprint Name** にイニシャルが入るように名前を変更します。異なるプロジェクト間であっても、ブループリント名は一意でなければなりません。

#. ご自身のプロジェクトを選択し、 **Upload** をクリックします。

   .. figure:: images/26.png

#. ブループリントをローンチするには、最初にネットワークをVMに割り当てる必要があります。 **NodeReact** サービスを選択し、 **NIC 1** ネットワークとして **Primary** を選択します。

#. Categoryメニューにおいて *Initials*\ **-Team: Fiesta** と **Environment: Dev** を選択します。

   .. figure:: images/27.png

#. **NIC 1** と **Category** の割当を **MySQL** サービスに対しても行います。

#. **Credentials** をクリックし、ブループリントによってプロビジョニングされるCentOS VMへの認証に使用される秘密鍵を定義します。

   .. figure:: images/27b.png

#. **CENTOS** の認証情報を展開して、以下の値を **SSH Private Key** に入力します。

   ::

      -----BEGIN RSA PRIVATE KEY-----
      MIIEowIBAAKCAQEAii7qFDhVadLx5lULAG/ooCUTA/ATSmXbArs+GdHxbUWd/bNG
      ZCXnaQ2L1mSVVGDxfTbSaTJ3En3tVlMtD2RjZPdhqWESCaoj2kXLYSiNDS9qz3SK
      6h822je/f9O9CzCTrw2XGhnDVwmNraUvO5wmQObCDthTXc72PcBOd6oa4ENsnuY9
      HtiETg29TZXgCYPFXipLBHSZYkBmGgccAeY9dq5ywiywBJLuoSovXkkRJk3cd7Gy
      hCRIwYzqfdgSmiAMYgJLrz/UuLxatPqXts2D8v1xqR9EPNZNzgd4QHK4of1lqsNR
      uz2SxkwqLcXSw0mGcAL8mIwVpzhPzwmENC5OrwIBJQKCAQB++q2WCkCmbtByyrAp
      6ktiukjTL6MGGGhjX/PgYA5IvINX1SvtU0NZnb7FAntiSz7GFrODQyFPQ0jL3bq0
      MrwzRDA6x+cPzMb/7RvBEIGdadfFjbAVaMqfAsul5SpBokKFLxU6lDb2CMdhS67c
      1K2Hv0qKLpHL0vAdEZQ2nFAMWETvVMzl0o1dQmyGzA0GTY8VYdCRsUbwNgvFMvBj
      8T/svzjpASDifa7IXlGaLrXfCH584zt7y+qjJ05O1G0NFslQ9n2wi7F93N8rHxgl
      JDE4OhfyaDyLL1UdBlBpjYPSUbX7D5NExLggWEVFEwx4JRaK6+aDdFDKbSBIidHf
      h45NAoGBANjANRKLBtcxmW4foK5ILTuFkOaowqj+2AIgT1ezCVpErHDFg0bkuvDk
      QVdsAJRX5//luSO30dI0OWWGjgmIUXD7iej0sjAPJjRAv8ai+MYyaLfkdqv1Oj5c
      oDC3KjmSdXTuWSYNvarsW+Uf2v7zlZlWesTnpV6gkZH3tX86iuiZAoGBAKM0mKX0
      EjFkJH65Ym7gIED2CUyuFqq4WsCUD2RakpYZyIBKZGr8MRni3I4z6Hqm+rxVW6Dj
      uFGQe5GhgPvO23UG1Y6nm0VkYgZq81TraZc/oMzignSC95w7OsLaLn6qp32Fje1M
      Ez2Yn0T3dDcu1twY8OoDuvWx5LFMJ3NoRJaHAoGBAJ4rZP+xj17DVElxBo0EPK7k
      7TKygDYhwDjnJSRSN0HfFg0agmQqXucjGuzEbyAkeN1Um9vLU+xrTHqEyIN/Jqxk
      hztKxzfTtBhK7M84p7M5iq+0jfMau8ykdOVHZAB/odHeXLrnbrr/gVQsAKw1NdDC
      kPCNXP/c9JrzB+c4juEVAoGBAJGPxmp/vTL4c5OebIxnCAKWP6VBUnyWliFhdYME
      rECvNkjoZ2ZWjKhijVw8Il+OAjlFNgwJXzP9Z0qJIAMuHa2QeUfhmFKlo4ku9LOF
      2rdUbNJpKD5m+IRsLX1az4W6zLwPVRHp56WjzFJEfGiRjzMBfOxkMSBSjbLjDm3Z
      iUf7AoGBALjvtjapDwlEa5/CFvzOVGFq4L/OJTBEBGx/SA4HUc3TFTtlY2hvTDPZ
      dQr/JBzLBUjCOBVuUuH3uW7hGhW+DnlzrfbfJATaRR8Ht6VU651T+Gbrr8EqNpCP
      gmznERCNf9Kaxl/hlyV5dZBe/2LIK+/jLGNu9EJLoraaCBFshJKF
      -----END RSA PRIVATE KEY-----

#. **Save** をクリックし、終了後 **Back** をクリックします。

   数分以内に、新谷さんは仮想インフラストラクチャとアプリケーションのセルフサービスをエンドユーザーに直接提供するための基礎を築きました。

開発者のワークフロー
++++++++++++++++++

楠田さんについて紹介しましょう。楠田さんはFiesta開発チームのメンバーです。彼は、テストを実行するために必要な仮想インフラストラクチャを展開するITへの要求が数日遅れているため、新機能のテストが遅延しています。

楠田さんは、お気に入りのパブリッククラウドサービスで企業ネットワークの外部にVMを展開し、セキュリティの監視を行わず、会社のIPアドレスを危険にさらしていました。

ここで新谷さんが助け舟を出します。新谷さんは楠田さんがPrismを通してFiestaプロジェクト内のリソースを簡単に展開できるようにするために以下の演習に従うことを勧めます。

#. **admin** アカウントからログアウトし、以下の楠田さんのアカウントで **Prism Central** にログインします。

   - **User Name** - devuser01@ntnxlab.local
   - **Password** - nutanix/4u

   .. 注意::

      ログインに時間がかかる場合は、シークレット/プライベートブラウジングセッションを使用してログインしてみてください。

#. :fa:`bars` メニューにアクセスして、環境へのアクセスが大幅に制限されていることを確認して下さい。

#. **VMs（仮想マシン）** ページに *Initials*\ **-WinToolsVM** が楠田さんが管理可能なVMとして表示されます。

#. VMをクリックして、楠田さんが彼のVMに関連付けられた基本的なメトリックを取得し、VMの構成、電源操作を制御し、さらにはVMを削除できることに注意してください。

   .. figure:: images/29.png

   VMのセルフサービスによる作成には、2つのワークフローがあります。従来のVM作成ウィザードとCalmです。楠田さんの要件の1つは、彼の開発ワークフローの一部として必要な複数のツールを実行するLinux仮想マシンです。

#. **Create VM（仮想マシンを作成）** をクリックし、次のフィールドに入力して、ラボの前半で新谷さんが実行した手動のVM導入プロセスと同様に、従来の仮想マシンをプロビジョニングします。

   - **Create VM from** - Disk Images
   - **Select Disk Images** - Linux_ToolsVM.qcow2
   - **Name** - *Initials* -LinuxToolsVM
   - **Target Project** - *Initials* -FiestaProject
   - **Network** - Secondary
   - **Categories** - Envrionment:Dev
   - **Manually configure CPU and Memory for this VM** を選択
   - **CPU** - 2
   - **Cores Per CPU** - 1
   - **Memory** - 4 GiB

#. **Save** をクリックし、作成後すぐにVMの電源がオンになることに注意してください。

   楠田さんは、ツールVMに加えて、Fiestaアプリケーションの新しいビルドのテストに使用できるインフラストラクチャを展開したいと考えています。エンドユーザーが単一のVMプロビジョニングと手動による構築作業によって複雑なアプリケーションを展開するのは遅く、一貫性がなく、ユーザー満足度は高くありません。幸運なことに、新谷さんによってプロジェクトに公開された、事前に作成されたFiestaアプリケーションのブループリントを活用できます。

#. :fa:`bars` **> Services（サービス） > Calm** を選択します。

#. 左のメニューから **Blueprints** を選択し、 **Fiesta-Multi** ブループリントを開きます。

   .. figure:: images/30.png

   .. 注意::

      ブループリントに慣れていない場合は、時間をかけてFiesta-Multiブループリントの以下の主要コンポーネントを調べてみてください。

      - **NodeReact** または **MySQL** サービスを選択し、画面の右側の構成ペインでVM構成を確認します。

         .. figure:: images/31.png

      - **Package** タブに移動し **Configure Install** をクリックして、選択したサービスのインストールタスクを表示します。これらは、各サービスまたはVMの構成に関連付けられたスクリプトとアクションです。

         .. figure:: images/32.png

      - **Application Profile** の下で **AHV** を選択し、ブループリントのために定義された変数を表示します。変数はランタイムでのカスタマイズが可能であり、アプリケーションプロファイルごとに使用して、AHV、ESXi、AWS、GCP、Azureなどの複数の環境に同一アプリケーションをプロビジョニングできる単一のブループリントを構築することもできます。

         .. figure:: images/33.png

      - **Application Profile** 配下の **Create** をクリックし、サービス間の依存関係を視覚化します。依存関係は明示的に定義できますが、変数の割り当てに応じて、Calmは暗黙的な依存関係も識別します。このブループリントでは、MySQLデータベースが実行されるまでWeb層のインストールプロセスが開始されないことがわかります。

         .. figure:: images/34.png

      - ブループリントエディタの上部にあるツールバーの **Credentials** をクリックし、 **CENTOS** の認証情報を展開します。ブループリントには複数の資格情報を含めることができ、これらを使用してVMを認証し、スクリプトを実行したり、資格情報を安全に直接スクリプトに渡したりできます。

         .. figure:: images/35.png

      - **Back** をクリックします。

#. **Launch** をクリックして、ブループリントのインスタンスをプロビジョニングします。

   .. figure:: images/36.png

#. 次のフィールドに入力して、 **Create** をクリックします。

   - **Name of of the Application** - *Initials* -FiestaMySQL
   - **db_password** - nutanix/4u

   .. figure:: images/37.png

#. **Audit** タブを選択して、Fiesta開発環境のデプロイメントを監視します。アプリの完全なプロビジョニングには約5分かかります。

   .. figure:: images/38.png

#. アプリケーションのプロビジョニング中、 :fa:`bars` **> Administration > Projects** を選択し、プロジェクトをクリックします。

#. **Summary** 、 **Usage** 、 **VMs** 、 **Users** のタブを確認します。これらの情報により、プロジェクト、VM、またはユーザーレベルで、どれだけリソースが消費されているかを簡単に把握できます。

   .. figure:: images/39.png

#. **Calm > Applications >** *Initials*\ **-FiestaMySQL** に戻り、アプリケーションが **Provisioning** 状態から **Running** 状態になるのを待ちます。 **Services** タブから **NodeReact** サービスを選択して、Web層のIPを取得します。

   .. figure:: images/40.png

#. 新しいブラウザータブで\http://< *NodeReact-VM-IP* >を開き、アプリが実行されていることを確認します。

   .. figure:: images/41.png

   チケットを提出してひたすら待つ代わりに、楠田さんは昼食前にテスト環境を稼働させることができました。今日は早く帰れそうです。

オペレーターのワークフロー
++++++++++++++++++

倉さんと宇土（うど）さんをご紹介します。 倉さんは、ITヘルプデスクでレベル3の運用エンジニアとして働いており、宇土さんはFiestaチームの品質保証としてインターンとして働いています。以下の簡単な演習では、新谷さんによって定義されたロールと割り当てられたカテゴリに基づいて、それらのアクセスレベルを調べて比較します。

#. **devuser01** アカウントからログアウトし、倉さんの認証情報を使って **Prism Central** にログインします。

   - **User Name** - operator01@ntnxlab.local
   - **Password** - nutanix/4u

#. 予想どおり、 **Environment** カテゴリ値が割り当てられたすべてのVMが使用可能です。VM を **Create** または **Delete** する機能は利用できませんが、VM構成を電源管理および変更する機能は利用できます。

   このユーザーは他に何にアクセスできますか？Calmにはアクセスできますか？

   .. figure:: images/42.png

#. **operator01** アカウントからログアウトし、宇土さんの資格情報を使用して **Prism Central** にログインします。

   - **User Name** - operator02@ntnxlab.local
   - **Password** - nutanix/4u

#. *Initials*\ **-Team: Fiesta** カテゴリでタグ付けされたリソースのみを管理できます。

   .. figure:: images/43.png

#. 宇土さんは **nodereact** VMのメモリ使用率が高いというアラートを受け取ります。構成を更新して、メモリを増やし、VMの電源を再投入します。

エンティティブラウザ、検索、分析の使用
++++++++++++++++++++++++++++++++++++++++++

新谷さんは、レガシーインフラストラクチャを近代的なアーキテクチャへと刷新するにあたり、大規模で多様な環境をすべてPrism Centralで管理および監視することを検討しています。以下の演習では、Nutanix環境で複数のクラスタにまたがるエンティティを操作するための一般的なワークフローを探ります。

#. **operator02** アカウントからログアウトし、管理者用のAD認証情報を使用してPrism Central に再度ログインします:

   - **User Name** - adminuser01@ntnxlab.local
   - **Password** - nutanix/4u

#. :fa:`bars` **> Virtual Infrastructure（仮想インフラ） > VMs（仮想マシン）** を開きます。Prism Centralの **Entity Browser** は、VM、イメージ、クラスタ、ホスト、アラートなどのエンティティをソート、検索、表示するための堅牢なUIを提供します。

#. **Filters（フィルタ）** を選択して、使用可能なオプションを確認します。次のサンプルフィルターを指定し、対応するボックスがオンになっていることを確認します。

   - **Name（名前）** - *Initials*
   - **Categories** - *Initials*\ -Team: Fiesta
   - **Hypervisor** - AHV
   - **Power State** - On

   VM効率、メモリ使用量、ストレージレイテンシなど、利用可能な他の有用なフィルターに注意してください。

#. フィルターされたVMをすべて選択し、**Label（ラベル）** アイコンをクリックして、フィルターされたVMのグループにカスタムラベルを適用します。 (例: *Initials* AHV Fiesta VMs).

   .. figure:: images/44.png

#. すべてのフィルターをクリアし、新しく作成したラベルを選択して、以前にフィルターしたVMにすばやくアクセスできることを確認します。ラベルは、エンティティをカテゴリのように特定のポリシーに関連付けることなく、エンティティの分類に追加の手段を提供します。

   .. figure:: images/45.png

#. **Focus（フォーカス）** ドロップダウンを選択して、ボックス外のさまざまなビューにアクセスします。例えばVMがDR計画の一部として含まれているかどうかを理解するには、どのビューを使用する必要がありますか？

#. **Focus（フォーカス） > + Add Custom（カスタムを追加）** をクリックして、 **CPU Usage** 、 **CPU Ready Time** 、 **IO Latency** 、 **Working Set Size Read** 、 **Working Set Size Write** を表示するVMビュー（XYZ-VM-Viewなど）を作成します。このようなビューは、VMパフォーマンスの問題を特定するのに役立ちます。

   .. figure:: images/46.png

#. Prism Centralのエンティティの検索、並べ替え、および分析の機能を十分に理解するには、次の短いビデオをご覧ください。

   .. raw:: html

     <center><iframe width="640" height="360" src="https://www.youtube.com/embed/HXWCExTlXm4?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

改善されたライフサイクル管理
++++++++++++++++++++++++++++++

新谷さんは日常の活動ではありませんが、以前は時間の40％をレガシーインフラストラクチャのソフトウェアとファームウェアの更新の計画と実行に費やしており、イノベーションに費やす時間はほとんどありませんでした。彼女のNutanix環境では、新谷さんはライフサイクルマネージャー（LCM）のルールエンジンと豊富な自動化を活用して、インフラストラクチャソフトウェアの更新を計画および適用する手間を省いています。

残念ながら、共有クラスタ環境では、LCMを直接テストすることはできません。LCMの機能と使いやすさについて理解を深めるには、下記のインタラクティブなデモをクリックしてください。

5.11 Prism Element LCM Interactive Demo
=======================================

.. figure:: https://demo-captures.s3-us-west-1.amazonaws.com/pe-5.11-lcm/story_content/thumbnail.jpg
   :target: https://demo-captures.s3-us-west-1.amazonaws.com/pe-5.11-lcm/story.html
   :alt: Prism Element 5.11 LCM Interactive Demo

5.11 Prism Central LCM Interactive Demo
=======================================

.. figure:: https://demo-captures.s3-us-west-1.amazonaws.com/pc-5.11-lcm/story_content/thumbnail.jpg
   :target: https://demo-captures.s3-us-west-1.amazonaws.com/pc-5.11-lcm/story.html
   :alt: Prism Central 5.11 LCM Interactive Demo

次のステップ
++++++++++

2時間以内の時間で、ストレージ、ネットワーク、ワークロードの導入、環境の監視、ソフトウェアの更新に関して、Prismが仮想インフラストラクチャ管理者にスムーズな体験を提供する方法を示しました。ネイティブのPrism Central機能をActive Directoryと組み合わせて使用​​して、アクセスを制御し、管理者以外のユーザのセルフサービスを有効にする方法を見てきました。さらに、Nutanix Calmを介してプライベートクラウドの豊富なアプリケーション自動化機能を有効にしました。

ただし、プライベートクラウドは、IaaS、セルフサービス、およびアプリケーションの自動化だけで構築されているわけではありません。今後のラボでは、Nutanixがその基盤をどのように構築して、 **Prism Pro** 機能による高度な監視および運用機能を提供するか、ストレージテクノロジーを **Files** に統合するか、ネイティブマイクロセグメンテーションを **Flow** に統合する方法を確認します。
