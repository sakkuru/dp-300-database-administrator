---
lab:
    title: 'Lab 1 - Azure仮想マシンにSQL Serverをプロビジョニングする'
    module:  'データ プラットフォーム リソースを計画および実装する'
---

# Azure 仮想マシン上で SQL サーバーをプロビジョニングする

**見積もり時間: 30 分**

受講者は、Azureポータルを探索し、それを使用してSQL Server 2019がインストールされたAzure VMを作成します。その後、リモートデスクトッププロトコルを使用して仮想マシンに接続します。

あなたはAdventureWorksのデータベース管理者です。あなたは概念実証で使用するためのテスト環境を作成する必要があります。概念実証では、Azure仮想マシン上のSQL Serverと、AdventureWorksDWデータベースのバックアップを使用する予定です。仮想マシンをセットアップし、データベースを復元し、利用可能であることを確認するためにクエリを実行する必要があります。

## Azure 仮想マシンに SQL Server を導入する。

1. ラボ仮想マシンから、ブラウザセッションを開始し、[https://portal.azure.com](https://portal.azure.com/) に移動し、Azure サブスクリプションに関連付けられた Microsoft アカウントを使用してサインインします。


1. ページの上部にある検索バーを探します。**Azure SQL** を検索します。サービス]の下に表示される[Azure SQL]の検索結果を選択します。

    ![Picture 9](../images/dp-300-module-01-lab-09.png)

1. **Azure SQL**ブレードで、**作成**を選択します。

    ![Picture 10](../images/dp-300-module-01-lab-10.png)

1. **SQL展開オプションの選択**ブレードで、**SQL仮想マシン** の下にあるドロップダウン・ボックスをクリックします。Free SQL Server Licenseと書かれたオプションを選択します。**SQL 2019 Developer on Windows Server 2022**と書かれたオプションを選択します。その後、**作成**を選択します。

    ![Picture 11](../images/dp-300-module-01-lab-11.png)

1. **仮想マシンの作成**ページで、以下の情報を入力します。

    - サブスクリプション: **&lt;Your subscription&gt;**
    - リソースグループ:**&lt;Your resource group&gt;**
    - 仮想マシン名:**azureSQLServerVM**
    - **Region:** &lt;リソースグループと同じリージョン&gt;
    - **可用性オプション:** **インフラストラクチャ冗長必要ありません**
    - **イメージ:** Free SQL Server License: SQL Server 2019 Developer on Windows Server 2022 - Gen1
    - **Spot割引:** チェックしない
    - **Size:** Standard *D2s_v3* (2 vCPU, 8 GiB memory)です。このオプションを表示するには、"See all sizes" リンクを選択する必要がある場合があります)
    - **管理者アカウントのユーザー名:** **sqladmin**
    - **管理者アカウントのパスワード:** **pwd!DP300lab01** (または条件を満たす独自のパスワード)
    - **受信ポート:** RDP (3389)を選択します。
    - **既存の Windows Server ライセンスを使用しますか?:** チェックしない

    ユーザー名とパスワードは、後で使用するためにメモしておいてください。

    ![Picture 12](../images/dp-300-module-01-lab-12.png)

1. **ディスク** タブに移動し、設定を確認します。


1. **ネットワーク** タブを開き、設定を確認します。


1. **管理**タブを開き、設定を確認します。

    **自動シャットダウンを有効にする**のチェックが外れていることを確認します。

1. **詳細** タブに移動し、設定を確認します。

1. **SQL Server の設定** タブを開き、設定を確認します。

    ![Picture 17](../images/dp-300-module-01-lab-17.png)

    **注意 -** この画面では、SQL Server VM のストレージを設定することもできます。デフォルトでは、SQL Server Azure VM テンプレートは、データ用にリードキャッシュ付きのプレミアムディスクを 1 つ、トランザクションログ用にキャッシュなしのプレミアムディスクを 1 つ作成し、 tempdb 用にローカル SSD (Windows では D:\) を使用します。

1. **確認および作成**ボタンを選択します。次に**作成**を選択します。

    ![Picture 18](../images/dp-300-module-01-lab-18.png)

1. デプロイメントブレード上で、デプロイが完了するまで待ちます。VMのデプロイにはおよそ5～10分かかります。デプロイメントが完了したら、**Go to resource**を選択します。

    **注意:** デプロイが完了するまでに数分かかる場合があります。

    ![Picture 19](../images/dp-300-module-01-lab-19.png)

1. 仮想マシンの**概要**ページで、このリソースのメニューオプションを確認し、利用可能なものを確認します。

    ![Picture 20](../images/dp-300-module-01-lab-20.png)

## Azure 仮想マシンの SQL Server に接続する

1. 仮想マシンの **概要** ページで、**接続** ボタンを選択し、RDP を選択します。

    ![Picture 21](../images/dp-300-module-01-lab-21.png)

1. RDPタブで、**Download RDP File**ボタンを選択します。

    ![Picture 22](../images/dp-300-module-01-lab-22.png)

    **注意:** Port prerequisite not met**というエラーが表示された場合。必ずリンクを選択して、*ポート番号*フィールドに記載された宛先ポートを持つ受信ネットワークセキュリティグループルールを追加してください。

    ![Picture 22_1](../images/dp-300-module-01-lab-22_1.png)


1. 先ほどダウンロードしたRDPファイルを開きます。接続するかどうかのダイアログが表示されたら、**接続**を選択します。

    ![Picture 23](../images/dp-300-module-01-lab-23.png)

1. 仮想マシンのプロビジョニングプロセスで選択したユーザー名とパスワードを入力します。その後、**OK**を選択します。

    ![Picture 24](../images/dp-300-module-01-lab-24.png)

1. **リモートデスクトップ接続**のダイアログが表示されたら、**Yes**を選択します。

    ![Picture 26](../images/dp-300-module-01-lab-26.png)

1. Windowsのスタートボタンを選択し、SSMSと入力します。リストから **Microsoft SQL Server Management Studio 18** を選択します。 

    ![Picture 34](../images/dp-300-module-01-lab-34.png)

1. SSMSが開くと、**Connect to Server** ダイアログにデフォルトのインスタンス名が事前に入力されていることに注意してください。**Connect**を選択します。

    ![Picture 35](../images/dp-300-module-01-lab-35.png)

Azureポータルには、仮想マシンでホストされているSQL Serverを管理するための強力なツールが用意されています。これらのツールには、自動パッチ適用、自動バックアップの制御、高可用性の簡単なセットアップ方法などが含まれます。
