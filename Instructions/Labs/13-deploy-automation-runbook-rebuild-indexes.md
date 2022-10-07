---
lab:
    title: 'Lab 13 – インデックスを自動的に再構築するための自動化Runbookを導入する'
    module: 'Azure SQL のデータベース タスクを自動化する'
---

# インデックスを自動的に再構築するための自動化Runbookの導入

**見積もり時間: 30 分**

あなたは、データベース管理の日常業務を自動化するために、シニアデータベース管理者として採用されました。この自動化は、AdventureWorksのデータベースが最高のパフォーマンスで動作し続けることを保証し、特定の基準に基づいてアラートを出す方法を提供するためのものです。AdventureWorksは、Infrastructure as a Service (IaaS) とPlatform as a Service (PaaS) の両方でSQL Serverを利用しています。

## 自動化アカウントの作成

1. ラボ仮想マシンから、ブラウザセッションを開始し、[https://portal.azure.com](https://portal.azure.com/) に移動します。このラボ仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用してポータルサイトに接続します。

1. Azure ポータルの検索バーに *automation* と入力し、検索結果から **Automation アカウント** を選択し、**+作成** を選択します。

    ![オートメーションアカウントを選択する画面](../images/dp-300-module-13-lab-01.png)

1. **Automationアカウントの作成** ページで、以下の情報を入力し、「**確認および作成**」を選択します。

    - **リソースグループ：** *contoso-rg*ではじまるリソースグループ
    - **名前:** autoAccount
    - **地域:** デフォルトを使用します。

    ![オートメーションアカウントの追加画面](../images/dp-300-module-13-lab-02.png)

1. レビューページで、**作成**を選択します。

## 既存の Azure SQL データベースに接続する

1. Azureポータルで、**sql databases** を検索して、データベースに移動します。

    ![既存の SQL データベースを検索する画面](../images/dp-300-module-13-lab-03.png)

1. SQLデータベース **AdventureWorksLT** を選択します。

1. SQLデータベースページのメインセクションで、**クエリエディタ（プレビュー）** を選択します。

    ![クエリエディタ(プレビュー)を選択した画面](../images/dp-300-module-13-lab-05.png)

1. データベースにサインインするためのクレデンシャルの入力を求められます。このクレデンシャルを使用します。

    - **ログイン:** sqladmin
    - **パスワード:** P@ssw0rd01

    次のようなエラーメッセージが表示された場合は、エラーメッセージの最後に表示される **Allowlist IP ...** のリンクを選択します。これにより、クライアントのIPがSQLデータベースのファイアウォールルールエントリとして自動的に追加されます。

    ![サインインエラーのスクリーンショット](../images/dp-300-module-13-lab-06.png)

    クエリエディタに戻り、**OK**を選択してデータベースにサインインします。

1. ブラウザーで新しいタブを開き、GitHub ページに移動して [**AdaptativeIndexDefragmentation**](https://github.com/microsoft/tigertoolbox/blob/master/AdaptiveIndexDefrag/usp_AdaptiveIndexDefrag.sql) スクリプトにアクセスします。次に、**Raw** を選択します。

    ![GitHubでRawを選択したスクリーンショット](../images/dp-300-module-13-lab-08.png)

    これにより、コピーできる形式でコードが提供されます。すべてのテキストを選択し ( <kbd>CTRL</kbd> + <kbd>A</kbd> )、クリップボードにコピーします ( <kbd>CTRL</kbd> + <kbd>C</kbd> )。

    >[!NOTE]
    > このスクリプトの目的は、1 つまたは複数のデータベースに対して、1 つまたは複数のインデックスに対してインテリジェントな最適化を実行し、必要な統計情報を更新することです。

1. GitHub ブラウザー タブを閉じて、Azure portal に戻ります。

1. コピーしたテキストを **クエリ 1** ペインに貼り付けます。

    ![新しいクエリ ウィンドウにコードを貼り付けたスクリーンショット](../images/dp-300-module-13-lab-09.png)

1. クエリの 5 行目と 6 行目 (スクリーンショットで強調表示されている部分) の `USE msdb` と `GO` を削除し、[実行] を選択します。

1. **ストアド プロシージャ** フォルダーを展開して、作成された内容を確認します。

    ![新しいストアドプロシージャのスクリーンショット](../images/dp-300-module-13-lab-10.png)

## オートメーションアカウントリソースの設定

次のステップでは、Runbookの作成準備として必要なリソースを設定します。次に、**Automation アカウント** を選択します。

1. Azure ポータル上、上部の検索ボックスに **automation** と入力します。

    ![オートメーションアカウントを選択する画面](../images/dp-300-module-13-lab-11.png)

1. 作成したオートメーションアカウントを選択します。

    ![オートメーションアカウントを選択する画面](../images/dp-300-module-13-lab-12.png)

1. オートメーションブレードの **共有リソース** セクションから **モジュール** を選択します。次に、**ギャラリーを参照**を選択します。

    ![モジュールメニューを選択した画面](../images/dp-300-module-13-lab-13.png)

1. Galleryの中から**sqlserver**を検索します。

    ![SqlServerモジュールを選択した画面](../images/dp-300-module-13-lab-14.png)

1. **SqlServer**を選択すると、次の画面になりますので、**選択**を選択します。

    ![Selectを選択した画面](../images/dp-300-module-13-lab-15.png)

1. **モジュールの追加** ページで、利用可能な最新のランタイムバージョンを選択し、**インポート** を選択します。これにより、PowerShellモジュールがAutomationアカウントにインポートされます。

    ![インポートを選択した画面](../images/dp-300-module-13-lab-16.png)

1. データベースに安全にサインインするために、クレデンシャルを作成する必要があります。オートメーションアカウントのブレードから**共有リソース**セクションに移動し、**資格情報**を選択します。

    ![認証情報オプションを選択した画面](../images/dp-300-module-13-lab-17.png)

1. **資格情報の追加** を選択し、以下の情報を入力し、**作成** を選択します。

    - 名前 **SQLUser**
    - ユーザー名 **sqladmin**
    - パスワード **P@ssw0rd01**
    - パスワードの確認 **P@ssw0rd01**

     ![アカウントの認証情報を追加する画面](../images/dp-300-module-13-lab-18.png)

## PowerShell Runbookの作成

1. Azureポータルで、**sql databases** を検索して、データベースに移動します。

    ![既存の SQL データベースを検索している画面](../images/dp-300-module-13-lab-03.png)

1. SQLデータベース **AdventureWorksLT** を選択します。

    SQLデータベース**AdventureWorksLT**を選択します。

1. **概要** ページで、以下のように Azure SQL Database の **サーバー名** をコピーします（サーバー名は *dp300-lab* で始まっている必要があります）。これは、後の手順で貼り付けます。

    ![サーバー名をコピーした画面](../images/dp-300-module-13-lab-19.png)

1. Azureポータルで、上部の検索ボックスに **automation** と入力します。

    ![オートメーションアカウントを選択する画面](../images/dp-300-module-13-lab-11.png)

1. 作成したオートメーションアカウントを選択します。

    ![オートメーションアカウントを選択する画面](../images/dp-300-module-13-lab-12.png)

1. オートメーションアカウントブレードの**プロセスオートメーション**セクションまでスクロールし、**Runbook**を選択し、**+Runbookの作成**をクリックします。

    ![Runbookの作成を選択し、Runbookページのスクリーンショット](../images/dp-300-module-13-lab-20.png)。

    >[!NOTE]です。
    > 学習したように、既存のRunbookが2つ作成されていることに注意してください。これらは、自動化アカウントのデプロイ時に自動的に作成されました。

1. Runbook名を **IndexMaintenance** とし、Runbookのタイプを **PowerShell** と入力します。利用可能な最新のランタイムバージョンを選択し、**作成**を選択します。

    ![Runbookの作成画面](../images/dp-300-module-13-lab-21.png)

1. Runbookの作成が完了したら、以下のPowershellコードスニペットをコピーし、Runbookエディターに貼り付けます。スクリプトの最初の行に、上記の手順でコピーしたサーバー名を貼り付けます。**保存**を選択し、次に**公開**を選択します。

    **注：** Runbookを保存する前に、コードが正しくコピーされていることを確認してください。

    ```powershell
    $AzureSQLServerName = ''
    $DatabaseName = 'AdventureWorksLT'
    
    $Cred = Get-AutomationPSCredential -Name "SQLUser"
    $SQLOutput = $(Invoke-Sqlcmd -ServerInstance $AzureSQLServerName -UserName $Cred.UserName -Password $Cred.GetNetworkCredential().Password -Database $DatabaseName -Query "EXEC dbo.usp_AdaptiveIndexDefrag" -Verbose) 4>&1

    Write-Output $SQLOutput
    ```

    ![貼り付けたコードのスクリーンショット](../images/dp-300-module-13-lab-22.png)

1. うまくいくと、成功のメッセージが表示されます。

    ![Runbookの作成に成功したときの画面](../images/dp-300-module-13-lab-23.png)

## Runbookのスケジュールを作成する

次に、Runbookを定期的に実行するためのスケジュールを作成します。

1. **IndexMaintenance**Runbookの左側のナビゲーションにある**リソース**の下で、**スケジュール**を選択します。次に、**+ スケジュールの追加**を選択します。

    ![スケジュールの追加を選択した画面](../images/dp-300-module-13-lab-24.png)

1. **スケジュールをRunbookにリンクする**を選択します。

    ![スケジュールをRunbookにリンクするを選択する画面](../images/dp-300-module-13-lab-25.png)

1. **スケジュールの追加**を選択します。

    ![スケジュール作成リンクの画面](../images/dp-300-module-13-lab-26.png)

1. スケジュール名と説明を入力します。（例: IndexMaintenance）

1. 開始時刻を翌日の**午前4時**分とし、**1**日ごとに発生するように設定します。有効期限は設定しないでください。

    新しいスケジュールがポップアップ表示され、情報が表示されます。

1. **作成** を選択します。


1. これでスケジュールが作成され、Runbookにリンクされました。**OK**を選択します。

   ![作成されたスケジュールのスクリーンショット](../images/dp-300-module-13-lab-28.png)

Azure Automation は、Azure 環境と非 Azure 環境の間で一貫した管理をサポートする、クラウドベースの自動化および設定サービスを提供します。

この演習を完了することで、SQL サーバー データベースのインデックスのデフラグを毎日午前 4 時に実行するように自動化することができました。
