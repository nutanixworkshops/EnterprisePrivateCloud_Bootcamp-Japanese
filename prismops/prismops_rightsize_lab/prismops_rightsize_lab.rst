-------------------------------
Prism Proを利用した適切なサイジング
-------------------------------

.. figure:: images/operationstriangle.png

このラボでは、IT管理者がPrism Proを使用して、VMのメモリリソースが制限されている場合に、監視、分析、および自動的に対処する方法を学習します。

ラボの準備
+++++++++

※PrismProのキャパシティランウェイ演習と同様の事前準備です。

#. **Prism Central** にて **VMs** ページに移動し、 **PrismProLabUtilityServer** のIPアドレスを控える。※この後の演習にてこのIPアドレスにアクセスします。

   .. figure:: images/init1.png

#. ブラウザを起動し、 http://`<PrismProLabUtilityServer_IP_ADDRESS>`/alerts に接続する。[例 http://10.42.113.52/alerts] 下図のようなログイン画面が表示された場合は **Prism Central IP** と、Prismログインのための **Username**  **Password** を入力し **Login** をクリックする。

   .. figure:: images/init2.png

#. 下図のようなページが表示されたら、タブを開いたままにする。このラボの後の部分で使用します。

   .. figure:: images/init2b.png

#. また別のタブで、 http://`<PrismProLabUtilityServer_IP_ADDRESS>`/ にアクセスします。 [例 http://10.42.113.52/] このURLのPrismUIを利用して演習を進めます。

   .. figure:: images/init3.png

Prism Pro X-FITによる非効率状態の検出
+++++++++++++++++++++++++++++++++++++++++++

Prism Proは、X-FIT機械学習を使用して、管理対象クラスタ内で実行されているVMの動作を検出および監視します。

Prism Proは、機械学習を使用してデータを分析し、非効率であると学習されたVMに分類を適用します。以下は分類の簡単な説明です。:

* **Overprovisioned（オーバープロビジョニング）:** 割り当てられたリソースの最小量を使用していると特定されたVM。（余剰リソースが多い）
* **Inactive（保護無効）:** 一定期間電源がオフになっているVM、またはCPU、メモリ、またはI / Oリソースを消費していないVM。
* **Constrained（制約あり）:** 追加のリソースでパフォーマンスが向上するVM。（リソース不足）
* **Bully:** 多くのリソースを使用し、その結果他のVMに影響を与えると特定されたVM。

#. **Prism Central** にてダッシュボードに移動する。 :fa:`bars` **> Dashboard**

#. ダッシュボードから、仮想マシン効率（VM Efficiency）ウィジェットを確認する。※このウィジェットは、Prism ProのX-FIT機械学習が検出した非効率的なVMの概要を提供します。ウィジェットの下部にある ‘非効率な仮想マシンをすべて表示（View All Inefficeint VMs）’ リンクをクリックして、詳細を確認します。

   .. figure:: images/ppro_58.png

#. VMリストビューで、Prism ProがこれらのVMにフラグを立てた理由の詳細を含む効率性の詳細を表示している。　※効率の詳細(Efficiency Detail)列のテキストにカーソルを合わせると、詳細な説明を表示できます。

   .. figure:: images/ppro_59.png

#. 管理者は、効率リストでVMのリストを確認し、アクションを実行する対象を決定できる。※リソースが多すぎる、または少なすぎるVMでは、個々のVMのサイズを変更する必要があります。これは、以下にリストするいくつかの例を使用して、さまざまな方法で実行できます。:

   * **Manually:** 管理者は、Prism（またはESXiの場合はvCenter）を介してVM構成を編集し、割り当てられたリソースを変更する。
   * **X-Play:** X-Plays自動プレイブックを使用して、トリガーまたは管理者の指示によりVMのサイズを自動的に変更する。この演習の後半で、実習する項目があります。
   * **Automation:** powershellやREST-APIなどの他の自動化方法を使用して、VMのサイズを変更する。


   この機械学習データを使用して、Prism ProはVM、ホスト、およびクラスタメトリックデータのベースライン（予想される範囲）を生成することもできます。X-FITアルゴリズムは、これらのエンティティの通常の動作を学習し、さまざまなチャートのベースライン範囲としてそれを表します。メトリック値がこの予想範囲から逸脱するたびに、Prism Proは異常として検知します。

#. PrismCentralにて ‘bootcamp_good’ を検索し、 ‘bootcamp_good_1’ を確認する。

   .. figure:: images/ppro_61.png

#. 評価指標（Metrics） > CPU使用率に移動する。 ※濃い青色の線と、その周囲の明るい青色の領域に注目してください。濃い青色の線はCPU使用率です。水色の領域は、このVMの予想CPU使用範囲です。このVMは、毎日同じ時間にアップグレードされるアプリケーションを実行しており、X-FITがそのパターンを学習し、それに応じて予想範囲を調整していることを確認します。そして今回は、使用量が予想範囲をはるかに超えているため、このVMで異常が発生していると検知しています。「過去24時間」の時間範囲を縮小して、チャートをより詳しく調べることもできます。

   .. figure:: images/ppro_60.png

#. **アラート設定（Alert Setting）** をクリックし、このような状況を検知するためのアラートポリシーを設定する。

#. Window右側で、必要に応じていくつかの設定を変更できることを確認する。※下図の例では、行動異常のしきい値を変更して、10％から70％の間の異常を無視しています。他のすべての異常は、警告アラートを生成します。また、このVMのCPU使用率が95％を超える場合、静的しきい値をアラートクリティカルに調整しました。

   .. figure:: images/ppro_25.png

#. **キャンセル（Cancel）** をクリックし、画面を閉じる。　※キャパシティランウェイの演習と同様の理由により、実際にアラートを生成できる環境ではないため、キャンセルします。

X-Playを利用したメモリの自動追加
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

非効率性の一部を解決するために自動化されたアクションを実行する方法を見てみましょう。このラボでは、このVMのメモリが制限されていると想定し、このVMの適切なサイズ設定を自動的に修正する方法を示します。また、カスタムチケットシステムを使用して、この典型的なワークフローがService Nowなどのチケットシステムとどのように統合できるかを考えます。

#. VMリストから **_Initials_-LinuxToolsVM** を確認する。 以降の画面例では、 **ABC - VM** を利用します。

   .. figure:: images/rs1.png

#. 現在の **メモリー容量（Memory Capacity）** を確認する。　※後でX-Playを使用してメモリ容量を増やします。メモリの値はプロパティウィジェット内を下にスクロールすると見つけられます。

   .. figure:: images/rs2.png

#. 検索バーから **Action Gallery** に移動する。

   .. figure:: images/rs3.png

#. **REST API** を選択し、 **アクション（Action） > クローン（Clone）** をクリックする。

   .. figure:: images/rs4.png

#. 以下を入力し **コピー（Copy）** をクリックする。　※作成しているアクションは、後でPlaybookからチケット発行させるためのものです。※<PrismProLabUtilityServer_IP_ADDRESS>は変数なので、IPアドレスを代入してください。

   - **氏名（Name）:** *Initials* - Service Ticketの作成
   - **Method:** POST
   - **URL:** http://<PrismProLabUtilityServer_IP_ADDRESS>/generate_ticket/
   - **Request Body:** ``{"vm_name":"{{trigger[0].source_entity_info.name}}","vm_id":"{{trigger[0].source_entity_info.uuid}}","alert_name":"{{trigger[0].alert_entity_info.name}}","alert_id":"{{trigger[0].alert_entity_info.uuid}}"}``
   - **Request Header:** Content-Type:application/json;charset=utf-8

   .. figure:: images/rs5.png

#. 検索バーから **Playbooks** に移動する。

   .. figure:: images/rs6.png

#. サービスチケットの生成を自動化するPlaybookを作成する。上部にある **プレイブックの作成（Create Playbook）** をクリックする。

   .. figure:: images/rs7.png

#. トリガーとして **Alert** を選択する。

   .. figure:: images/rs8.png

#. アラートポリシーとして **VM {vm_name} Memory Constrained** を検索して選択する。　※このアラート検知〜対処を自動化します。

   .. figure:: images/rs9.png

#. *VMsを指定（Specify VMs）* のラジオボタンを選択し、 **_Initials_-LinuxToolsVM** を選択する。※このVMで発生したアラートに対して自動対処されることを意味します。

   .. figure:: images/rs10.png

#. 左側の **アクションの追加（Add Action）** をクリックし、作成した *Initials* - **Service Ticket** の作成アクションを選択する。注：ラボでは、独自に作成したチケットシステムを設定しましたが、Service Nowにはすぐに使用できるService Nowアクションのテンプレートもあります。

   .. figure:: images/rs11.png

#. 作成したサービスチケット生成アクションの詳細が自動的に入力されることを確認する。

   .. figure:: images/rs12.png

#. X-Playによってチケットが作成されたことをメールで通知する。 **アクションの追加（Add Action）** をクリックし、Emailを選択し、以下を入力する。　※<PrismProLabUtilityServer_IP_ADDRESS>は変数なので、IPアドレスを代入してください。

   - **Recipient:** - メールアドレスを入力
   - **Subject :** - ``Service Ticket Pending Approval: {{trigger[0].alert_entity_info.name}}``
   - **Message:** - ``The alert {{trigger[0].alert_entity_info.name}} triggered Playbook {{playbook.playbook_name}} and has generated a Service ticket for the VM: {{trigger[0].source_entity_info.name}} which is now pending your approval. A ticket has been generated for you to take action on at http://<PrismProLabUtilityServer_IP_ADDRESS>/ticketsystem``

   .. figure:: images/rs13.png

#. **保存して閉じる（Save & Close）** を選択し、名前を “*Initials* - Generate Service Ticket for Constrained VM” と設定する。 **‘Enabled’ トグルで有効にすることを忘れないでください。**

   .. figure:: images/rs14.png

#. もう一つPlaybookを作成します。これはサービスチケットを解決するときに呼び出すものであり、影響を受けるVMにメモリを追加して電子メールを送信します。テーブルビューの上部にある **プレイブックの作成（Create Playbook）** をクリックします。

   .. figure:: images/rs15.png

#. トリガーとして **Manual** を選択する。 Note: このラボ用に構築したチケットシステムは、手動トリガーによって提供されるトリガーAPIを呼び出しますが、このAPIは現バージョンでは公開されていません。Version 5.17では、これと同じ動作を実現するパブリックAPIを公開する「Webhookトリガー」を導入しています。Service Nowなどのツールは、このWebhookを使用してPrism Centralにコールバックし、プレイブックをトリガーできます。

   .. figure:: images/rs16.png

#. ドロップダウンで **VM** を選択する。

   .. figure:: images/rs17.png

#. 左側の **アクションの追加（Add Action）** をクリックし、右側で **VM Add Memory** を選択する。

   .. figure:: images/rs18.png

#. 以下の画面に従って空のフィールドを設定する。また次に、自動化されたアクションが行われたことを誰かに通知する。 **アクションの追加（Add Action）** をクリックして、Emailを追加する。

   .. figure:: images/rs19.png

#. 以下を入力する。

   - **Recipient:** - メールアドレスを入力
   - **Subject :** - ``Playbook {{playbook.playbook_name}} was executed.``
   - **Message:** ``{{playbook.playbook_name}} has run and has added 1GiB of Memory to the VM {{trigger[0].source_entity_info.name}}.``

   .. note::

      独自のメッセージを作成してください。上記は例です。「パラメータ」を使用してメッセージを充実させることができます。

   .. figure:: images/rs20.png

#. 最後に、チケットサービスにコールバックして、チケットサービスのチケットを解決する。 **アクションの追加（Add Action）** をクリックして、 **REST API** アクションを追加する。※<PrismProLabUtilityServer_IP_ADDRESS>は変数なので、IPアドレスを代入してください。

   - **Method:** PUT
   - **URL:** http://<PrismProLabUtilityServer_IP_ADDRESS>/resolve_ticket
   - **Request Body:** ``{"vm_id":"{{trigger[0].source_entity_info.uuid}}"}``
   - **Request Header:** Content-Type:application/json;charset=utf-8

   .. figure:: images/rs21.png

#. **保存して閉じる（Save & Close）** をクリックし、名前は “*Initials* - Resolve Service Ticket” とする。 ** ‘Enabled’ トグルで有効化することを忘れないでください。** 

   .. figure:: images/rs22.png

#. ワークフローをトリガーする。演習のはじめに開いておいた **/alerts** URL [例 10.42.113.52/alerts] に移動する。 **VM Memory Constrained** のラジオボタンを選択し、 **_Initials_-LinuxToolsVM** を指定する。 **Simulate Alert** ボタンをクリックし、メモリ制約のアラートをシミュレートする。

   .. figure:: images/rs23.png

#. 指定したメールアドレスにメールが届くことを確認する。※5分ほどかかる場合があります。

   .. figure:: images/rs24.png

#. メール内のリンクをクリックして、チケットシステムにアクセスする。または、ブラウザの新しいタブから http://`<PrismProLabUtilityServer_IP_ADDRESS>`/ticketsystem にアクセスする。

   .. figure:: images/rs25.png

#. VM用に作成されたチケットを特定し、縦のドットアイコンをクリックして[アクション]メニューを表示し、 **Run Playbook** をクリックする。

   .. figure:: images/rs26.png

#. 作成した二つ目のplaybook **`Initials` - Resolve Service Ticket** を選択し、このチケットで実行する。

   .. figure:: images/rs27.png

#. Prism Centralコンソールを開いた状態で前のタブに戻る。 **`Initials` - Resolve Service Ticket** の詳細を開き **プレイ（Plays）** タブを表示することで、このプレイブックで実行された内容を確認できる。

   .. figure:: images/rs29.png

#. このビューのセクションを展開して、各アイテムの詳細を表示できる。エラーがある場合は、このビューでもエラーが表示される。

   .. figure:: images/rs30.png

#. VMの情報を確認し、メモリが1GB増えていることを確認する。

   .. figure:: images/rs31.png

#. また、プレイブックが実行されたことを通知するメールを確認する。

   .. figure:: images/rs32.png

まとめ
.........

- Prism Proは、IT Proをよりスマートかつ自動化するためのソリューションです。インテリジェントな検出から自動修復まで、IT Proプロセスを対象としています。

- X-FITは、異常検出や非効率検出を含むスマートIT Proをサポートする機械学習エンジンです。

- 企業向けのIFTTTであるX-Playは、日々の運用タスクの自動化を可能にするエンジンです。

- X-Playを使用すると、管理者は毎日のタスクを数分で自信を持って自動化できます。

- X-Playは豊富で、Playbookの一部として顧客の既存のAPIとスクリプトを使用でき、顧客の既存のチケットワークフローとうまく統合できます。
