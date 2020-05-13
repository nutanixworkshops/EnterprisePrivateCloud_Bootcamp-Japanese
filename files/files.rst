.. title:: Files

--------------------------------
ファイルとストレージの統合
--------------------------------

*このラボの所要想定時間は45分です。*

従来のファイルストレージはサイロ化した伝統的構造をしています。 例えばSANストレージのように技術的な進歩を欠いたり、拡張性に難があったりと、我々を悩ませてきました。
Nutanixではエンタープライズクラウドにこのようなサイロの居場所はないと考えています。
仮想基盤として実績のあるHCIコア上のソフトウェアで実行可能なアプリケーションの様にファイルストレージを実装することで、
Nutanix Filesは高性能で、スケーラビリティのある迅速なイノベーションをワンクリックで管理可能にします。

**このラボでは、Filesを使ってSMB共有とNFSサーバの管理を体験していただき、File Analyticsを用いたFilesの新たなる機能性を探りましょう。**

今回は時間短縮のためのため、共有リソースの各クラスタ上にFilesクラスタが構築済みです。
**BootcampFS** はシングルノードで展開されていますが、通常**Flies**の展開は3台のFile Server VMで開始し、パフォーマンスの必要に応じてスケールアップやスケールアウトすることが出来ます。

**BootcampFS** は、プライマリネットワークを使用してバックエンドストレージと通信し、CVM からボリュームグループへの iSCSI 接続を行い、セカンダリネットワークを使用してクライアント、Active Directory、アンチウィルスサービスなどと通信するように設定されています。


.. figure:: images/1.png

.. note::

  本番環境では、クライアントとストレージ トラフィックのために専用の仮想ネットワークを使用して Files を配置することが一般的に望ましいとされています。2つのネットワークを使用する場合、Filesは設計上、クライアントトラフィックがストレージネットワークにアクセスできないようにします。

FilesはデータストレージにNutanixボリュームグループを利用しているため、圧縮、消去コーディング、スナップショット、レプリケーションなど、同じ基本的なストレージの利点を利用することができます。

In **Prism Element > File Server > File Server**, select **BootcampFS** and click **Protect**.

   .. figure:: images/10.png

   デフォルトのSelf Service Restoreスケジュールに着目すると、注目すべきはこれがWindowsの前バージョンのスナップショットスケジュールを機能的に制御するという点です。
   Windowsの以前のバージョン機能をサポートすることで、エンドユーザーはストレージ管理者やバックアップ管理者を介さずにファイルへの変更をロールバックすることができます。
   これらのローカルスナップショットは、ファイルサーバクラスタをローカルの障害から保護するものではなく、ファイルサーバクラスタ全体のレプリケーションをリモートのNutanixクラスタに実行することができることに注意してください。

SMB共有の管理
+++++++++++++++++++

このエクササイズではSMB共有を試してもらいます。SMB共有は、非構造化ファイルデータを共有する混成チームなどによるFiestaアプリケーションの開発をサポートするために使用されます。

共有の作成
..................

#. In **Prism Element > File Server**, click **+ Share/Export**.

#. Fill out the following fields:

   - **名前** - *Initials*\ **-FiestaShare**
   - **説明 (オプション)** - Fiesta app team share, used by PM, ENG, and MKT
   - **ファイルサーバー** - **BootcampFS**
   - **共有パス (オプション)** - 空白のままにします。このフィールドでは、ネストされた共有を作成する既存のパスを指定できます。
   - **最大サイズ (オプション)** - 200GiB
   - **プロトコルの選択** - SMB

   .. figure:: images/2.png

   これはシングルノードAOS、つまりシングルFSVMなので、標準共有で全てまかないます。標準共有とは、すべてのルートディレクトリとファイルがシングルFSVM経由で提供される状態です

   これが3ノードのFilesクラスタ以上であれば、分散共有を作成するオプションがあります。
   分散共有は、ホームディレクトリやユーザープロファイル、アプリケーションフォルダを共有するのに適しています。
   このタイプの共有では、ルートディレクトリ及びファイルへの要求をすべてのFSVMから行うことが可能で、接続に対してロードバランシングが可能です。

#. **次へ**をクリックします。

#.  **Enable Access Based Enumeration** 、 **Self Service Restore**にチェックを入れ. **Blocked File Types** に .flv,.mov を入力します。

   .. figure:: images/3.png

   .. note::

      **Access Based Enumeration (ABE)** 特定のユーザーが読み取りアクセス権を持つファイルとフォルダーのみがそのユーザーに表示されるようにします。 これは通常、Windowsファイル共有で有効です。

      **Self Service Restore** Windowsの以前のバージョン機能をから、Nutanixスナップショットに基づいて個々のファイルを以前のリビジョンに簡単に復元できます。

      **Blocked File Types** 特定のタイプのファイル（大容量の個人用メディアファイルなど）を企業の共有に書き込まないように制限することができます。また、これはサーバ毎、共有グループ毎に設定でき、サーバ全体のルールよりも優先して適応されます。

#. **次へ**をクリックします。

#. **サマリー** を確認し **作成**をクリックします。

   .. figure:: images/4.png

   多くの人が利用する共有では、リソースの公平な使用を確保するためにクォータを活用するのが一般的です。
   Filesは、Active Directory内の個々のユーザー、または特定のActive Directoryセキュリティグループのいずれかに対して
   共有ごとにソフトクォータまたはハードクォータを設定する機能を提供します。

#. In **Prism Element > File Server > Share/Export**, select your share and click **+ Add Quota Policy**.

#. 以下のフィールドに入力し、**保存**をクリックします。:

   - Select **グループ**
   - **ユーザーもしくはグループ** - SSP Developers
   - **クォータ** - 10 GiB
   - **タイプ** - Hard Limit

   .. figure:: images/9.png

#. **保存**をクリックします。

Testing the Share
.................

#.  *Initials*\ **-WinTools** のコンソールから  **NTNXLABのadministratorアカウント以外**でログインします:

   .. note::

      これらのアカウントを使用してはRDP経由で接続することはできません。

   - user01 - user25
   - devuser01 - devuser25
   - operator01 - operator25
   - **Password** nutanix/4u

   .. figure:: images/16.png

   .. note::

     Windows Tools VMは既に** NTNXLAB.local **ドメインに参加しています。 ドメインに参加しているVMを使用して、次の手順を実行します。

#. **エクスプローラー**で ``\\BootcampFS.ntnxlab.local\`` を開きます.

#. *Initials*\ **-WinTools** のブラウザーで以下にアクセスサンプルファイルをダウンロードし、共有に置きます。。:

   - **If using a PHX cluster** - http://10.42.194.11/workshop_staging/peer/SampleData_Small.zip
   - **If using a RTP cluster** - http://10.55.251.38/workshop_staging/peer/SampleData_Small.zip

#. zipファイルを展開します。

   .. figure:: images/5.png

   - **NTNXLAB\\Administrator**ユーザーは、ファイルクラスターの展開中にファイル管理者として指定され、デフォルトですべての共有への読み取り/書き込みアクセス権を付与しました。
   - 他のユーザーのアクセス管理は、他のSMB共有と同じです。

..   #.  ``\\BootcampFS.ntnxlab.local\``, の *Initials*\ **-FiestaShare を右クリックし、プロパティを開きます **.

   #. **セキュリティ** タブの **詳細**を選択します.

      .. figure:: images/6.png

   #. **Users (BootcampFS\\Users)** を選択し、**Remove** をクリックします。.

   #. **Add**をクリックします。

   #. **プリンシパルを選択** を選択し、**オブジェクト名** のフィールドに **Everyone** を入力し、**OK** をクリックします。

      .. figure:: images/7.png

   #. 下記フィールドを入力し **OK** をクリックします。:

      - **Type** - Allow
      - **Applies to** - This folder only
      - Select **Read & execute**
      - Select **List folder contents**
      - Select **Read**
      - Select **Write**

      .. figure:: images/8.png

   #. **OK > OK > OK** とクリックし、変更を保存します。

   これで、すべてのユーザーが *Initials*\ **-FiestaShare** 共有内にフォルダーとファイルを作成できるようになります。

#. **PowerShell** を開き、以下のコマンドを使ってブロックされたファイルタイプのファイルを作成を試みます。:

   .. code-block:: PowerShell

      New-Item \\BootcampFS\INITIALS-FiestaShare\MyFile.flv

   新しいファイルの作成が拒否されたことを確認します。

#. **Prism Element > File Server > Share/Export** に戻り、共有を選択します。 使用状況やパフォーマンスタブを見て共有毎の詳細情報を確認します(ファイル数や接続数、ストレージ使用率、レイテンシ、スループット、IOPSなど)。

   .. figure:: images/11.png

  次の演習では、ファイルを使用して各ファイルサーバーと共有の使用状況をさらに詳しく分析する方法を説明します。

File Analytics
++++++++++++++

この演習では新機能“統合File Analytics”を見てみましょう、これは既存の共有をスキャンし、異常アラートを作成します。また、スキャン結果の詳細も確認できます。
File Analyticsは、Prism Elementの自動化されたワンクリック操作により、スタンドアロンVMとして数分でデプロイされます。
このVMは、あなたの環境に既にデプロイされ、有効化されています。

#. **Prism Element > File Server > File Server** , **BootcampFS** と選択し、 **File Analytics**をクリックします。

   .. figure:: images/12.png

   .. note::

      File Analyticsはすでに有効になっているはずですが、プロンプトが表示された場合はすべての共有をスキャンするためにFiles管理者権限が必要となります。

      - **Username**: NTNXLAB\\administrator
      - **Password**: nutanix/4u

      .. figure:: images/old13.png

#. これは共有環境であるため、ダッシュボードには他のユーザーが作成した共有のデータがすでに表示されている可能性があります。 新しく作成した共有をスキャンするには、:fa:`gear` **> Scan File System** をクリックします。
   作成した共有を選択し、[スキャン]をクリックします

   .. figure:: images/14.png

   .. note::

      共有が表示されない場合は、入力されるまでしばらくお待ちください...

#. **Scan File System** ウィンドウを閉じて、のブラウザーを更新します。

#. **Data Age**, **File Distribution by Size** と **File Distribution by Type**のダッシュボードパネルが更新されます。

   .. figure:: images/15.png

   Under....

#. *Initials*\ **-WinTools** VMから**サンプルデータ**の下にあるいくつかのファイルを開いて、監査証跡アクティビティを作成します。

   .. note::
　ファイルを開く際に、OpenOfficeのウィザードが表示された場合は、次へを押して完了させます。

#. **Dashboard**ページを更新し、**Top 5 Active Users**, **Top 5 Accessed Files** そして **File Operations** パネルを確認します。

   .. figure:: images/17.png

#. ユーザーアカウントの監査証跡にアクセスするには、**Top 5 Active Users** でユーザーをクリックします。

#. または、ツールバーから **Audit Trails** を選択して、ユーザーまたは特定のファイルを検索することもできます。

   .. figure:: images/17b.png

   .. figure:: images/18.png

   .. note::

      ワイルドカードを使った検索も可能です。例えば、**.doc**
..

#. 次に、ファイルサーバーで異常な動作を検知するルールを作成します。ツールバーから:fa:`gear` **> Define Anomaly Rules**をリックします。

      .. figure:: images/19.png

      .. note::

         異常ルールはファイルサーバーごとに定義されるため、別のユーザーによって既に作成されている可能性があります。

   #. **Define Anomaly Rules** をクリックして、次の設定でルールを作成します。

      - **Events:** Delete
      - **Minimum Operation %:** 1
      - **Minimum Operation Count:** 10
      - **User:** All Users
      - **Type:** Hourly
      - **Interval:** 1

   #. **Actions** 以下の **Save**をクリックします。

   #. **+ Configure new anomaly** を選択し、次の設定でルールを作成します。

      - **Events**: Create
      - **Minimum Operation %**: 1
      - **Minimum Operation Count**: 10
      - **User**: All Users
      - **Type**: Hourly
      - **Interval**: 1

   #. **Actions**の**Save**をクリックします。

      .. figure:: images/20.png

   #. Click **Save** to exit the **Define Anomaly Rules** window.
   #. **Save**をクリックして**Define Anomaly Rules**ウィンドウを閉じます。


   #. 異常アラートをテストします。*Initials*\ **-WinTools** VMに戻り、*Initials*\ **-FiestaShare** 共有内に、サンプルデータをコピーアンドペーストします。

   #. Delete the original sample data folders.

      .. figure:: images/21.png

      While waiting for the Anomaly Alerts to populate, next we’ll create a permission denial.

      .. note:: The Anomaly engine runs every 30 minutes.  While this setting is configurable from the File Analytics VM, modifying this variable is outside the scope of this lab.

   #. Create a new directory called *Initials*\ **-MyFolder** in the *Initials*\ **-FiestaShare** share.

   #. Create a text file in the *Initials*\ **-MyFolder** directory and take out your deep seeded worldly frustrations on your for a few moments to populate the file. Save the file as *Initials*\ **-file.txt**.

      .. figure:: images/22.png

   #. Right-click *Initials*\ **-MyFolder > Properties**. Select the **Security** tab and click **Advanced**. Observe that **Users (BootcampFS\\Users)** lack the **Full Control** permission, meaning that they would be unable to delete files owned by other users.

      .. figure:: images/23.png

   #. Open a PowerShell window as another non-Administrator user account by holding **Shift** and right-clicking the **PowerShell** icon in the taskbar and selecting **Run as different user**.

      .. figure:: images/24.png

   #. Change Directories to *Initials*\ **-MyFolder** in the *Initials*\ **-FiestaShare** share.

        .. code-block:: bash

           cd \\BootcampFS.ntnxlab.local\XYZ-FiestaShare\XYZ-MyFolder

   #. Execute the following commands:

        .. code-block:: bash

           cat .\XYZ-file.txt
           rm .\XYZ-file.txt

      .. figure:: images/25.png

   #. Return to **Analytics > Dashboard** and note the **Permission Denials** and **Anomaly Alerts** widgets have updated.

      .. figure:: images/26.png

   #. Under **Permission Denials**, select your user account to view the full **Audit Trail** and observe that the specific file you tried to removed is recorded, along with IP address and timestamp.

      .. figure:: images/27.png

   #. Select **Anomalies** from the toolbar for an overview of detected anomalies.

      .. figure:: images/28.png

File Analytics puts simple, yet powerful information in the hands of storage administrators, allowing them to understand and audit both utilization and access within a Nutanix Files environment.

Using NFS Exports
+++++++++++++++++

In this exercise you will create and test a NFSv4 export, used to support clustered applications, store application data such as logging, or storing other unstructured file data commonly accessed by Linux clients.

Enabling NFS Protocol
.....................

.. note::

   Enabling NFS protocol only needs to be performed once per Files server, and may have already been completed in your environment. If NFS is already enabled, proceed to `Configure User Mappings`_.

#. In **Prism Element > File Server**, select your file server and click **Protocol Management > Directory Services**.

   .. figure:: images/29.png

#. Select **Use NFS Protocol** with **Unmanaged** User Management and Authentication, and click **Update**.

   .. figure:: images/30.png

Creating the Export
...................

#. In **Prism > File Server**, click **+ Share/Export**.

#. Fill out the following fields:

   - **Name** - logs
   - **Description (Optional)** - File share for system logs
   - **File Server** - *Initials*\ **-Files**
   - **Share Path (Optional)** - Leave blank
   - **Max Size (Optional)** - Leave blank
   - **Select Protocol** - NFS

   .. figure:: images/24.png

#. Click **Next**.

#. Fill out the following fields:

   - Select **Enable Self Service Restore**
      - These snapshots appear as a .snapshot directory for NFS clients.
   - **Authentication** - System
   - **Default Access (For All Clients)** - No Access
   - Select **+ Add exceptions**
   - **Clients with Read-Write Access** - *The first 3 octets of your cluster network*\ .* (e.g. 10.38.1.\*)

   .. figure:: images/25.png

   By default an NFS export will allow read/write access to any host that mounts the export, but this can be restricted to specific IPs or IP ranges.

#. Click **Next**.

#. Review the **Summary** and click **Create**.

Testing the Export
..................

You will first provision a CentOS VM to use as a client for your Files export.

.. note:: If you have already deployed the :ref:`linux_tools_vm` as part of another lab, you may use this VM as your NFS client instead.

#. In **Prism > VM > Table**, click **+ Create VM**.

#. Fill out the following fields:

   - **Name** - *Initials*\ -NFS-Client
   - **Description** - CentOS VM for testing Files NFS export
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 2 GiB
   - Select **+ Add New Disk**
      - **Operation** - Clone from Image Service
      - **Image** - CentOS
      - Select **Add**
   - Select **Add New NIC**
      - **VLAN Name** - Secondary
      - Select **Add**

#. Click **Save**.

#. Select the *Initials*\ **-NFS-Client** VM and click **Power on**.

#. Note the IP address of the VM in Prism, and connect via SSH using the following credentials:

   - **Username** - root
   - **Password** - nutanix/4u

#. Execute the following:

     .. code-block:: bash

       [root@CentOS ~]# yum install -y nfs-utils #This installs the NFSv4 client
       [root@CentOS ~]# mkdir /filesmnt
       [root@CentOS ~]# mount.nfs4 <Intials>-Files.ntnxlab.local:/ /filesmnt/
       [root@CentOS ~]# df -kh
       Filesystem                      Size  Used Avail Use% Mounted on
       /dev/mapper/centos_centos-root  8.5G  1.7G  6.8G  20% /
       devtmpfs                        1.9G     0  1.9G   0% /dev
       tmpfs                           1.9G     0  1.9G   0% /dev/shm
       tmpfs                           1.9G   17M  1.9G   1% /run
       tmpfs                           1.9G     0  1.9G   0% /sys/fs/cgroup
       /dev/sda1                       494M  141M  353M  29% /boot
       tmpfs                           377M     0  377M   0% /run/user/0
       *intials*-Files.ntnxlab.local:/             1.0T  7.0M  1.0T   1% /afsmnt
       [root@CentOS ~]# ls -l /filesmnt/
       total 1
       drwxrwxrwx. 2 root root 2 Mar  9 18:53 logs

#. Observe that the **logs** directory is mounted in ``/filesmnt/logs``.

#. Reboot the VM and observe the export is no longer mounted. To persist the mount, add it to ``/etc/fstab`` by executing the following:

     .. code-block:: bash

       echo 'Intials-Files.ntnxlab.local:/ /filesmnt nfs4' >> /etc/fstab

#. The following command will add 100 2MB files filled with random data to ``/filesmnt/logs``:

     .. code-block:: bash

       mkdir /filesmnt/logs/host1
       for i in {1..100}; do dd if=/dev/urandom bs=8k count=256 of=/filesmnt/logs/host1/file$i; done

#. Return to **Prism > File Server > Share > logs** to monitor performance and usage.

   Note that the utilization data is updated every 10 minutes.

Multi-Protocol Shares
+++++++++++++++++++++

Files provides the ability to provision both SMB shares and NFS exports separately - but also now supports the ability to provide multi-protocol access to the same share. In the exercise below, you will configure your existing *Initials*\ **-FiestaShare** to allow NFS access, allowing developer users to re-direct application logs to this location.

Configure User Mappings
.......................

A Nutanix Files share has the concept of a native and non-native protocol.  All permissions are applied using the native protocol. Any access requests using the non-native protocol requires a user or group mapping to the permission applied from the native side. There are several ways to apply user and group mappings including rule based, explicit and default mappings.  You will first configure a default mapping.

#. In **Prism Element > File Server**, select your file server and click **Protocol Management > User Mapping**.

#. Click **Next** twice to advance to **Default Mapping**.

#. From the **Default Mapping** page choose both **Deny access to NFS export** and **Deny access to SMB share** as the defaults for when no mapping is found.

   .. figure:: images/31.png

#. Click **Next > Save** to complete the default mapping.

#. In **Prism Element > File Server**, select your *Initials*\ **-FiestaShare** and click **Update**.

#. Under **Basics**, select **Enable multiprotocol access for NFS** and click **Next**.

   .. figure:: images/32.png

#. Under **Settings > Multiprotocol Access** select **Simultaneous access to the same files from both protocols**.

   .. figure:: images/33.png

#. Click **Next > Save** to complete updating the share settings.

Testing the Export
.......................

#. To test the NFS export, connect via SSH to your *Initials*\ **-LinuxToolsVM** VM:

   - **User Name** - root
   - **Password** - nutanix/4u

#. Execute the following commands:

     .. code-block:: bash

       [root@CentOS ~]# yum install -y nfs-utils #This installs the NFSv4 client
       [root@CentOS ~]# mkdir /filesmulti
       [root@CentOS ~]# mount.nfs4 bootcampfs.ntnxlab.local:/<Initials>-FiestaShare /filesmulti
       [root@CentOS ~]# dir /filesmulti
       dir: cannot open directory /filesmulti: Permission denied
       [root@CentOS ~]#

   .. note:: The mount operation is case sensitive.

Because the default mapping is to deny access the Permission denied error is expected. You will now add an explicit mapping to allow access to the non-native NFS protocol user. We will need to get the user ID (UID) to create the explicit mapping.

#. Execute the following command and take note of the UID:

     .. code-block:: bash

       [root@CentOS ~]# id
       uid=0(root) gid=0(root) groups=0(root)
       [root@CentOS ~]#

#. In **Prism Element > File Server**, select your file server and click **Protocol Management > User Mapping**.

#. Click **Next** to advance to **Explicit Mapping**.

#. Under **One-to-onemapping list**, click **Add manually**.

#. Fill out the following fields:

   - **SMB Name** - NTNXLAB\\devuser01
   - **NFS ID** - UID from previous step (0 if root)
   - **User/Group** - User

   .. figure:: images/34.png

#. Under **Actions**, click **Save**.

#. Click **Next > Next > Save** to complete updating your mappings.

#. Return to your *Initials*\ **-LinuxToolsVM** SSH session and try to access the share again:

     .. code-block:: bash

       [root@CentOS ~]# dir /filesmulti
       Documents\ -\ Copy  Graphics\ -\ Copy  Pictures\ -\ Copy  Presentations\ -\ Copy  Recordings\ -\ Copy  Technical\ PDFs\ -\ Copy  XYZ-MyFolder
       [root@CentOS ~]#

#. From your SSH session, create a text file and then validate you can access the file from your Windows client.

Takeaways
+++++++++

What are the key things you should know about **Nutanix Files**?

- Files can be rapidly deployed on top of existing Nutanix clusters, providing SMB and NFS storage for user shares, home directories, departmental shares, applications, and any other general purpose file storage needs.
- Files is not a point solution. VM, File, Block, and Object storage can all be delivered by the same platform using the same management tools, reducing complexity and management silos.
- Files can scale up and scale out with One Click performance optimization.
- File Analytics helps you better understand how data is utilized by your organizations to help you meet your data auditing, data access minimization and compliance requirements.
