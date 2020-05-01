.. _platformmsecurity:

-----------------
Platform Security
-----------------

Nutanixにとって、セキュリティは最も優先されており、いわゆる第一級市民(first-class citizen)です。
製品リリースの基本姿勢として、多層防御(Defense in Depth)と堅牢性の両方のベストプラクティスを提供しています。
増加し続けるサイバーセキュリティの脅威に対処するために、機微で効果的な状態を維持する様に努めています。
Nutanixのセキュリティ開発ライフサイクルは、開発プロセスの各段階でセキュリティを後処理で適用するのではなく、開発プロセスの段階で対処しています。
この備えには開発における自動セキュリティテストや脅威に対する評価を含み、コード変更による顧客リスクを軽減しています。

Nutanixは、次のような追加のプラットフォームセキュリティ機能を提供します。

- セキュリティ技術実装ガイド（STIG）自動化
- KMSを統合したソフトウェアベースの保存データ暗号化
- Prism 監査ログ
- VM セキュアブート

Nutanix STIGSとの連携
++++++++++++++++++++++++++

Nutanixは、管理者にセキュリティ技術実装ガイド(STIG)を提供しています。
これは、自動化された検証、継続的な監視、および自己修正を容易にするために、機械で読める形式でエンコードされたセキュリティツールです。
これにより、製品の進化を遅らせることなく、セキュリティコンプライアンスの検証に必要な時間を数週間から数ヶ月から数日に短縮することができます。
STIGsは、米国国防総省およびPCI-DSSの複数のベースライン要件に適用可能な、米国国立標準技術研究所（NIST）による確立された標準に基づいています。
しかし、NutanixのSTIGは、Acropolisプラットフォームに固有のものであり、より効果的です。


このラボでは、STIGsによるNutanix Enterprise Cloud OSの堅牢化と、どのようにゼロデイ脆弱性を軽減するのに役立つのかを確認するために、実際に動作しているのを確認します。

   .. note::

      Nutanix AOS および AHV STIGs は Nutanix ポータルの `ココ <https://portal.nutanix.com/#/page/static/stigs>`_ からダウンロード出来ます。

STIGレポートの実行
.....................

以下のエクササイズに従って、お使いのNutanixクラスタ上でSTIGレポートを手動で実行してください。
デフォルトでは、これらのチェックはNutanixクラスタ上で24時間に1回実行されますが、一方、従来のレガシーインフラストラクチャ環境では、STIGはデプロイ時に手動で実施され、元の設定が変更されないということを自動検出する手段がありません。

#. コントローラーVM(CVM)にユーザー名：nutanixを使用しSSHで接続します。(Terminal, putty などSSHクライアント機能を有するクライアントを使用してください）

#. CVMのルートディレクトリに移動します。

   .. code-block:: bash

     cd /

#. rootディレクトリ内でrootユーザーが実行できるファイルを一覧表示します。

   .. code-block:: bash

     sudo -u root ls -l root

     You should see a similar output:

   .. code-block:: bash

     nutanix@NTNX-14SX35100046-A-CVM:10.21.71.29:~# sudo -u root ls -l root
     total 248
     -rw-------. 1 root root   3314 Sep 11  2017 anaconda-ks.cfg
     drwxr-x---. 2 root root   4096 Dec 13 23:04 filesystems
     -rw-r-----. 1 root root   1132 May  3  2018 homeaudit.pp
     -rw-r-----. 1 root root   1231 May  3  2018 my-runcon.pp
     -rw-r-----. 1 root root    464 May  3  2018 my-runcon.te
     -rw-------. 1 root root   3222 Sep 11  2017 original-ks.cfg
     -rwxr-x---. 1 root root  10034 May  3  2018 report_open_jre8_stig.sh
     -rwx------. 1 root root 132760 Aug 30 23:50 report_stig.sh
     -rwxr-x---. 1 root root  72376 May  3  2018 report_web_stig.sh
     drwxr-x---. 2 root root   4096 Dec 13 23:17 sretools
     -rw-r-----. 1 root root    840 May  3  2018 sshdlocal.pp

   _stig.sh で終わる3つの.shファイルが必要となります。レポートを出力するために実行します。

#. この例では、一般的なテキスト出力「report_stig.sh」を実行します。

   .. code-block:: bash

     sudo -u root /root/report_stig.sh

     The output will go into the root user log folder.

     .. note::

       The command will take ~1 minute to complete.

#. フォルダ内のファイルをリストアップし、レポート名をメモします。

   .. code-block:: bash

     sudo -u root ls -l /home/log | grep STIG

#. レポートをNutanixユーザーのホームディレクトリにコピーします。アスタリスク部分は実際のファイル名に置き換えてください。

   .. code-block:: bash

     sudo -u root cp /home/log/STIG-report-**-**-****-**-**-** /home/nutanix

#. /home/nutanix フォルダーのファイルをリストで表示させます。

   .. code-block:: bash

     ls -l ~

#. レポートファイルのオーナーをNutanixユーザーに変更します。アスタリスク部分は実際のファイル名に置き換えてください。

   .. code-block:: bash

     sudo -u root chown nutanix:nutanix /home/nutanix/STIG-report-**-**-****-**-**-**

#. セキュアコピーツール (SCP, WINSCP, PSCP, etc)を使ってCVMからあなたの端末へレポート結果ファイルをコピーします。
あるいは(vi, more, cat, etc)などを使用して、SSHセッションでテキストファイルを開いて表示することもできます。

   .. note::

     **nutanix**ユーザーを使用してCVMにログインしそのホームディレクトリを参照して上記で作成したファイルを見つけてください。

STIGレポートの分析
.........................

STIGレポートは、セキュリティコンプライアンスの検証および認定要件に使用できます。

レポート内の各結果の形式は次のとおりです。:

- **Line 1** - 名前確認
- **Line 2** - チェックの説明
- **Line 3** - 凡例、またはチェックの予期される結果
- **Line 4** - 結果確認
- **Line 5** - チェックの完了ステータス

以下は、STIGレポートの非検出の例です。これは、チェックで望まない構成が検出されなかったことを意味します。:

::

   CAT II RHEL-07-021030 SRG-OS-000480-GPOS-00227 CCI-000366 CM-5 (1)
   All world-writable directories must be group-owned by root, sys, bin, or an application group.
   The result of the check should be yes.  If no, then it's a finding
   yes
   Completed.

チェックの結果望ましくない構成であることが検出された例です。:
::

   CAT I RHEL-07-021710 SRG-OS-000095-GPOS-00049 CCI-000381 CM-7 a, CM-7 b
   The telnet-server package must not be installed.
   The result of the check should be yes.  If no, then it's a finding
   no
   Completed.

環境の侵害
............................

この最後の演習では、規定外の変更を行いクラスタをセキュリティ的に危険に晒した場合にどうなるか見てみましょう。
そして、それがSTIGsのためではなかった場合は、あまりにも厄介で逃げ出しているだろう。

（* 観客の中にスクービードゥーファンはいますか？いない？OK。じゃあ私だけで良いです...*）
※スクービードゥーは何かしらからドタバタ逃げ回る描写の多いコメディ作品)

例 1
=========

#. 次のテキストは、AOS STIG のセキュリティチェックの1つから検出されました：:

   - **Rule Version (STIG-ID)**: NTNX-51-000034
   - **Rule Title**: The /etc/shadow file must be group-owned by root.
   - **Fix Text**: salt-call state.sls security/CVM/fdpermsownerCVM

  Linux OSでは、セキュアユーザーデータ、特に暗号化されたパスワードが /etc/shadow ファイルに保存されるため、
  root以外のユーザーにこの機密ファイルへのアクセスを提供することは推奨されません。

#. CVMのルートディレクトリに移動します。:

   .. code-block:: bash

     cd /

#. 現在のオーナーを確認します。:

   .. code-block:: bash

     sudo -u root ls -l etc/shadow
     ----------. 1 root root 943 Dec 18 15:37 /etc/shadow

#. グループのオーナーを**nutanix**に変更します。:

   .. code-block:: bash

     sudo -u root chown root:nutanix /etc/shadow
     ls -l /etc/shadow
     ----------. 1 root nutanix 943 Dec 18 15:37 /etc/shadow

#. 脆弱性を修正する為に、salt callを実行します。:

   .. code-block:: bash

     sudo -u root salt-call state.sls security/CVM/fdpermsownerCVM

   .. note::

      識別された問題を修正するためにこのラボで手動で行われますが、すべてのSTIG関連の処理はデフォルトで24時間ごとに1回行われます。

#. ファイルの所有者が**nutanix**ではなく**root**グループによって再び所有されていることを確認します。:

   .. code-block:: bash

     sudo -u root ls -l etc/shadow

例 2
=========

この例では、以前に作成されたレポートからの次のチェックに焦点を当てていきます:

::

   All world-writable directories must be group-owned by root, sys, bin, or an application group.
   The result of the check should be yes.  If no, then it's a finding
   yes
   Completed.

**/tmp**などの誰でも書き込み可能なディレクトリが悪意のある人物に乗っ取られた場合、システムの運用に影響を与えセキュリティを危険にさらす可能性があります。

#. CVMのルートディレクトリに移動します。:

   .. code-block:: bash

     cd /

#. コンソールからこの特定のレポートを検索します。アスタリスクは実際のファイル名に置き換えます。:

   .. code-block:: bash

     sudo -u root grep -A 4 -B 1 "All world-writable directories " /home/log/STIG-report-**-**-****-**-**-**

#. 出力が例の先頭と一致することを確認します。 このチェックで「いいえ」と表示されるようにシステムを危険にさらしてから、手動で問題を修正してください。

   .. note::

      If there is currently a finding for this check, another user may be in the middle of this exercise. You may still proceed.

#. 現在のオーナーを確認します。:

   .. code-block:: bash

     sudo -u root ls -l / | grep  tmp
     drwxrwxrwt.  14 root root  1024 Dec 21 02:59 tmp

#. グループのオーナーを変更します。:

   .. code-block:: bash

     sudo -u root chown root:nutanix /tmp

#. オーナーの変更を確認します。:

   .. code-block:: bash

     sudo -u root ls -l / | grep  tmp
     drwxrwxrwt.  14 root nutanix  1024 Dec 21 03:16 tmp

#. レポートを再度実行して、この変更が検出されたかどうかを確認します

   .. code-block:: bash

     sudo -u root /root/report_stig.sh
     sudo -u root grep -A 4 -B 1 "All world-writable directories " /home/log/STIG-report-**-**-****-**-**-**

#. チェックの結果が**no**であることを確認します。:


#. 脆弱性を修正する為に、salt-callを実行します。:

   .. code-block:: bash

     sudo -u root salt-call state.sls security/CVM/fdpermsownerCVM

#. ディレクトリを再度リストし、‘compromise’が元に戻されたことを確認します。
オプション: レポートを再実行して、チェックの結果がなくなったことを確認できます。

   .. code-block:: bash

     sudo -u root ls -l / | grep  tmp
     drwxrwxrwt.  14 root root  1024 Dec 21 03:42 tmp


ソフトウェアベースの暗号化
+++++++++++++++++++++++++

保存データの暗号化は、プラットフォームセキュリティの重要機能です。:


- 故障したディスクドライブを介してユーザーデータがデータセンターから流出を抑制します、
- ドライブの盗難から保存データを保護します。
- 多くの連邦、ヘルスケア、金融、および法的環境でのコンプライアンスに必要です。


Nutanixは、保存データ暗号化するためのさまざまなオプションを提供します。

.. figure:: images/1.png

Nutanixの統合鍵管理サービス（KMS）によるソフトウェアベースの暗号化は、パフォーマンスに影響を与えることなくスムーズに暗号化を有効にすることができます。

.. figure:: images/2.png

ソフトウェアベースの暗号化を有効にすることは、クラスタレベルで一度だけの操作のため、共有のラボ環境では実行できません。この機能を有効にするために必要ないくつかのステップを、以下のナレーション付きの簡単なビデオで説明します。

.. raw:: html

  <center><iframe width="640" height="360" src="https://www.youtube.com/embed/-6fIL3FJjN8?rel=0&amp;showinfo=0&amp;t=53" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

監査ログ
++++++++++

システム監査はセキュリティ・コンプライアンスにおいて必須事項でもあります。
システム（ファイル、ディレクトリ、システムリソース、システムコール）に対して行われた変更やアクセスの履歴を読みやすいフォーマットで出力することは顧客の望むところであり、Nutanixクラスタ構築後1週間以内に要求される可能性が高いです。

Nutanixは、数分で詳細なシステムイベントをsyslogサーバに転送することができます。このナレーション付きビデオでは、どのような監査ログが利用可能か、どこでsyslogサーバを設定するか、一般的な問題をトラブルシューティングするためにどのようなアクションを取ることができるかを学びます。

.. raw:: html

  <center><iframe width="640" height="360" src="https://www.youtube.com/embed/YuhC5nWd5Is?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

VM セキュアブート
++++++++++++++

AOS 5.16 の新機能である Secure Boot for user VMs は、ゲスト OS ブートローダが UEFI ファームウェアに含まれるデータベースによって認証された暗号鍵で署名されていることを AHV がチェックし、OS ブートローダの整合性を検証して信頼するセキュリティ機能です。

.. figure:: images/3.png

.. raw:: html

  <center><iframe width="640" height="360" src="https://www.youtube.com/embed/dRs5QpFke2U?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

まとめ
+++++++++

- Nutanixは、次のようなセキュアなプラットフォームを提供します。

   - STIGの自動化されたアプリケーションとリメディエーション
   - Data at Rest暗号化を提供するための複数のオプション（ソフトウェアベースも含む）
   - システムログとフローログを外部のsyslogサーバに送信する機能を含む監査ログ
   - AHV上で動作するゲストVMのための信頼性の高いブート技術
