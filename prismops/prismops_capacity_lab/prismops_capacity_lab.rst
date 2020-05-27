------------------------
Prism Ops によるキャパシティランウェイ
------------------------

.. figure:: images/operationstriangle.png

Prism Opsは、お客様のIT運用にスマートな自動化をもたらします。典型的な運用ワークフローは、必要に応じて監視、分析、およびアクションを実行する連続サイクルです。 Prism Opsは、従来のIT管理者のワークフローを反映して、運用効率を向上させます。Prism Opsを使用すると、IT管理者は、機械学習エンジンX-FITおよびX-Play自動化エンジンを使用して、この典型的なフローを自動化できます。

この演習では、Prism Opsを使用した監視、分析、および自動化の方法を学習します。

ラボの準備
+++++++++

#. **Prism Central** にて **仮想マシン（VMs）** ページに移動し、 **GTSPrismOpsLabUtilityServer** のIPアドレスを控える。※この後の演習にてこのIPアドレスにアクセスします。

   .. figure:: images/init1.png

#. ブラウザを起動し、http://`<GTSPrismOpsLabUtilityServer_IP_ADDRESS>`/alerts に接続する。[例 http://10.42.113.52/alerts] 下図のようなログイン画面が表示された場合は **Prism Central IP** と、Prismログインのための **Username**  **Password** を入力し **Login** をクリックする。

   .. figure:: images/init2.png

#. 下図のようなページが表示されたら、タブを開いたままにする。このラボの後の部分で使用します。

   .. figure:: images/init2b.png

#. また別のタブで、 http://`<GTSPrismOpsLabUtilityServer_IP_ADDRESS>`/ にアクセスします。 [例 http://10.42.113.52/] このURLのPrismUIを利用して演習を進めます。

   .. figure:: images/init3.png

ランウェイ機能を利用したキャパシティプランニング
++++++++++++++++++++++++++++++++++++++

キャパシティランウェイとは、特定のクラスターまたはノード内のリソース残量の尺度です。 全体的なクラスターのランウェイだけでなく、CPU、メモリ、ストレージ容量の個々の測定値があります。キャパシティランウェイは、Prism OpsのマシンインテリジェンスエンジンであるX-FITを使用して計算されます。

#. **Prism Central > オペレーション（Operations） > 計画（Planning） > 容量のランウェイ（Capacity Runway）** と進む。

   - ランウェイのサマリから各クラスターのランウェイを確認する
   - メモリ、CPU、ストレージが枯渇するまでどれくらいとなっていますか？

#. **Prism-Pro-Cluster** をクリックする。

   .. figure:: images/ppro_12.png

#. メモリタブを選択すると、このクラスタでメモリが不足する時期を示す赤いマークが表示されます。このマークにカーソルを合わせると、発生する日を確認できます。

   .. figure:: images/ppro_13.png

#. 画面左側の **リソースの最適化（Optimize Resources）** をクリックする。※PrismOpsは環境内の非効率的なVMを確認し、これらのリソースを可能な限り効率的に最適化する方法を提案します。

   .. figure:: images/ppro_14.png

#. リソース最適化のポップアップを閉じる。

キャパシティランウェイの分析
++++++++++++++++++++++++++++++++++++++

Prism OpsのX-FITエンジンは、将来のワークロードを計画する機能も提供し、新しいワークロードのリソース要件に対応するために追加できるハードウェアを提案します。

#. ページ左側にある **リソースの調整（Adjust Resources）** にて、 **はじめに（Get Started）** ボタンをクリックする。※ここで、新しいワークロードの計画を入力し、今後のリソースランウェイを延長する必要があるかどうかを確認できます。

#. 画面左側の **追加/調整（add/adjust）** をクリックする。

   .. figure:: images/ppro_15.png

#. VDIを選択し、1000ユーザーを選択し、保存する。　※このワークロードをシステムに追加する日付を設定することもできます。

   .. figure:: images/ppro_16.png

   .. figure:: images/ppro_17.png

#. 同様の操作を実施し、適当なワークロードを追加する。

#. **推奨（Recommend）** ボタンをクリックする。

   .. figure:: images/ppro_18.png

#. 推奨事項が利用できるようになったら、リストビューとグラフビューを切り替えて、シナリオの概要を確認する。

   .. figure:: images/ppro_19.png

#. 画面右上にある **PDFを作成（Generate PDF）** ボタンをクリックする。※これにより作成したシナリオの報告書を自動生成してくれます。

   .. figure:: images/ppro_19b.png

#. PDFレポートを確認する。

   .. figure:: images/ppro_20.png

X-Playによる容量予測レポートの自動生成
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

次に、キャパシティランウェイが残り少なくなった際にこのレポートを生成するアクションを自動的に実行する方法を体験します。この演習ではPrism Opsのシンプルな自動化エンジンであるX-Playを使用します。

#. PrismCentralで検索バーを利用して **Playbooks** を検索する。

   .. figure:: images/cap1.png

#. **プレイブックの作成（Create Playbook）** をクリックする。

   .. figure:: images/cap2.png

#. トリガーとして **Alert** を選択する。

   .. figure:: images/cap3.png

#. アラートポリシーとして **Cluster running out of Memory Capacity (low runway)** を検索して選択する。 ※この演習ではメモリ不足をシミュレートした自動対応を検証します。

   .. figure:: images/cap4.png

#. 左側のアクションにて **アクションの追加（Add Action）** を選択し、 **Generate Forecast Report** を選択する。　※これは前項で設定したアラートを検知した後に、まずレポートを生成するということを意味します。

   .. figure:: images/cap5.png

#. Cluster項目には **Alert Source Entity** が設定される。　※必要に応じて、ランウェイの期間を変更することもできます。

   .. figure:: images/cap6.png

#. 次に、X-Playによってチケットが生成されたことを管理者に通知するタスクを追加する。 **Add Action** を選択し、 **Email** を選択する。

   .. figure:: images/cap7.png

#. 以下を入力する。

   - **Recipient:** - メールアドレスを入力
   - **Subject :** - ``Playbook {{playbook.playbook_name}} が実行されました``
   - **Message:** - `アラート {{trigger[0].alert_entity_info.name}}が発生し、プレイブック {{playbook.playbook_name}}が実行されました。レポートが添付されます。``

   .. note::

      独自の件名メッセージを作成してください。上記のような「パラメータ」を使用してメッセージを充実させることができます。

   .. figure:: images/cap8.png

#. **保存して閉じる（Save & Close）** をクリックし、 “*Initials* - Automatically Generate Forecast Report” という名前で保存する。 ** ‘Enabled’ のトグルで有効にしてください。**

   .. figure:: images/cap9.png

#. **演習用に用意されたメタデータだけの環境であるため、この環境では実際にこのPlaybookをシミュレートすることはできません。** 代わりに、アラートが正常に生成された場合の外観を示します。 “*Initials* - Automatically Generate Forecast Report” Playbookをクリックして開きます。

   .. figure:: images/cap11.png

#. **プレイ（Plays）** タブに切り替える。もし実際にアラートが発生したら、下図の様な画面でPlaybookの実行を確認できます。

   .. figure:: images/cap12.png

#. クリックすると、下図の様なビューが表示されます。このビューのセクションを展開して、各アイテムの詳細を表示できます。エラーがある場合は、このビューでもエラーが表示されます。

   .. figure:: images/cap13.png

#. また、下図のようなメールが届きます。

   .. figure:: images/cap14.png

お持ち帰り
.........

- Prism Opsは、IT OPSをよりスマートかつ自動化するためのソリューションです。インテリジェントな検出から自動修復まで、IT OPSプロセスを対象としています。

- X-FITは、容量予測などのスマートIT OPSをサポートする機械学習エンジンです。

- 企業向けIFTTTであるX-Playは、日々の運用タスクの自動化を可能にするためのエンジンであり、すべての管理者が自動化を簡単に構築できるようにします。
