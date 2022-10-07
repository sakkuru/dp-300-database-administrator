---
lab:
    title: 'Lab 15 – URLへのバックアップとURLからのリストア'
    module: '高可用性およびディザスタ リカバリ ソリューションを計画および実装する'
---

# URL へのバックアップ

**見積もり時間: 30分**

AdventureWorksのDBAとして、データベースをAzureのURLにバックアップし、ヒューマンエラーが発生した後にAzure blobストレージからリストアする必要があります。

## データベースの復元

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** にあるデータベースのバックアップファイルを、ラボ仮想マシンの **C:\LabFiles\HADR** パスにダウンロードします（フォルダ構造が存在しない場合は作成します）。

    ![Picture 03](../images/dp-300-module-15-lab-00.png)

1. Windowsのスタートボタンを選択し、SSMSと入力します。リストから **Microsoft SQL Server Management Studio 18** を選択します。 

1. SSMSが開くと、**Connect to Server** ダイアログにデフォルトのインスタンス名が事前に入力されていることに注意してください。**Connect**を選択します。

    ![Picture 02](../images/dp-300-module-07-lab-01.png)

1. Databases** フォルダを選択し、**New Query** を選択します。

    ![Picture 03](../images/dp-300-module-07-lab-04.png)

1. 新しいクエリウィンドウで、以下のT-SQLをコピーして貼り付けます。データベースを復元するためにクエリを実行する。

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\HADR\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\HADR\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\HADR\AdventureWorks2017_log.ldf';
    ```

    **注意：** データベースのバックアップファイルの名前とパスは、ステップ1でダウンロードしたものと一致させないと、コマンドは失敗します。

1. リストア完了後、成功のメッセージが表示されるはずです。

    ![Picture 03](../images/dp-300-module-07-lab-05.png)

## URLへのバックアップの設定

1. ラボ仮想マシンから、ブラウザセッションを開始し、[https://portal.azure.com](https://portal.azure.com/) に移動します。このラボ仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用して、ポータルに接続します。

1. 以下のアイコンを選択し、**クラウドシェル** プロンプトを開きます。

    ![Azureポータル上のクラウドシェルアイコンの画面](../images/dp-300-module-15-lab-01.png)

1. ポータルの下半分に、Azure Cloud Shellを利用することを歓迎するメッセージが表示されることがありますが、まだCloud Shellを利用していない場合は、このメッセージをクリックします。Bash**を選択します。

    ![Azureポータルのクラウドシェルのウェルカムページの画面](../images/dp-300-module-15-lab-02.png)

1. クラウドシェルを使用したことがない場合は、ストレージの設定を行う必要があります。 **詳細設定の表示** を選択します（別のサブスクリプションが割り当てられている場合があります）。

    ![Azureポータルでクラウドシェル用のストレージを作成する画面](../images/dp-300-module-15-lab-03.png)

1. 既存の **リソースグループ** を使用し、以下のダイアログに示すように、**ストレージアカウント** と **ファイル共有** に新しい名前を指定します。リソースグループ**の名前をメモしておいてください。これは *contoso-rg* で始まっている必要があります。次に、**Create storage**を選択します。

    **注意：** ストレージアカウント名は一意でなければならず、すべて小文字で特殊文字は使用しないでください。ユニークな名前を付けてください。

    ![Azureポータルでのストレージアカウントとファイル共有の作成画面](../images/dp-300-module-15-lab-04.png)

1. 完了すると、以下のようなプロンプトが表示されます。Cloud Shellの画面左上に**Bash**と表示されていることを確認します。

    ![AzureポータルのCloud Shellプロンプトの画面](../images/dp-300-module-15-lab-05.png)

1. CLIからCloud Shellで以下のコマンドを実行し、新しいストレージアカウントを作成します。リソースグループ名は、上記でメモした **contoso-rg** で始まるものを使用します。

    > 注意
    > リソースグループ名(**-g**パラメータ)を変更し、一意のストレージアカウント名(**-n**パラメータ)を指定します。

    ```bash
    az storage account create -n "dp300backupstorage1234" -g "contoso-rglod23149951" --kind StorageV2 -l eastus2
    ```

    ![Azureポータルでのストレージアカウント作成画面](../images/dp-300-module-15-lab-16.png)

1. 次に、この後の手順で使用するストレージアカウントのキーを取得します。ストレージアカウントとリソースグループの一意名を使って、Cloud Shellで以下のコードを実行します。

    ```bash
    az storage account keys list -g contoso-rglod23149951 -n dp300backupstorage1234
    ```

    上記のコマンドの結果に、あなたのアカウントキーが表示されます。前のコマンドで使用したのと同じ名前(**-n**の後)とリソースグループ(**-g**の後)を使用していることを確認してください。以下のように、**key1**の戻り値をコピーします（二重引用符は付けないでください）。

    ![Azure ポータル上のストレージ アカウント キーのスクリーンショット](../images/dp-300-mod)

1. SQL Server のデータベースを URL にバックアップするには、ストレージアカウント内のコンテナを使用します。このステップでは、バックアップストレージ専用のコンテナを作成します。そのために、以下のコマンドを実行します。

    ```bash
    az storage container create --name "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --fail-on-exist
    ```

    ここで、**dp300backupstorage1234** はストレージアカウントを作成する際に使用した一意のストレージアカウント名で、**storage_key** は上記で生成されたキーになります。出力は **true** を返すはずです。

    ![コンテナ作成時の出力画面](../images/dp-300-module-15-lab-07.png)

1. コンテナのバックアップが正しく作成されたかどうかを確認するために、以下を実行します。

    ```bash
    az storage container list --account-name "dp300backupstorage1234" --account-key "storage_key"
    ```

    ここで、**dp300backupstorage1234** はストレージアカウントを作成する際に使用した一意のストレージアカウント名で、**storage_key** は生成されたキーです。出力は以下のようなものを返すはずです。

    ![コンテナ一覧のスクリーンショット](../images/dp-300-module-15-lab-08.png)

1. セキュリティのために、コンテナレベルでの共有アクセス署名(SAS)が必要です。これは、Cloud ShellまたはPowerShellで実行することができます。以下を実行します。

    ```bash
    az storage container generate-sas -n "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --permissions "rwdl" --expiry "date_in_the_future" -o tsv
    ```

    ここで、**dp300backupstorage1234** はストレージアカウント作成時に使用した一意のストレージアカウント名、**storage_key** は生成したキー、**date_in_the_future** は今より後の時間です。**date_in_the_future**はUTCである必要があります。例えば、**2021-12-31T00:00Z**は、2020年12月31日午前0時に期限切れとなることを意味します。

    以下のような出力が得られるはずです。共有アクセス署名全体をコピーして、**メモ帳**に貼り付けてください。

    ![共有アクセス署名のスクリーンショット](../images/dp-300-module-15-lab-09.png)

## クレデンシャルの作成

これで機能が設定されたので、Azure Storage Account に blob としてバックアップファイルを生成することができます。

1. **SQL Server Management Studio (SSMS)** を起動します。

1. SQL Server への接続を求めるプロンプトが表示されます。**Windows 認証** が選択されていることを確認し、**接続** を選択します。

1. **New Query**を選択します。

1. 以下のTransact-SQLを使用して、クラウド上のストレージにアクセスするために使用されるクレデンシャルを作成します。適切な値を入力し、**Execute**を選択します。

    ```sql
    IF NOT EXISTS  
    (SELECT * 
        FROM sys.credentials  
        WHERE name = 'https://<storage_account_name>.blob.core.windows.net/backups')  
    BEGIN
        CREATE CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]
        WITH IDENTITY = 'SHARED ACCESS SIGNATURE',
        SECRET = '<key_value>'
    END;
    GO  
    ```

    ここで、**<storage_account_name>** の両方の出現は、作成されたユニークなストレージアカウント名で、**<key_value>** はこのフォーマットで前のタスクの最後に生成された値です。

    `'se=2020-12-31T00%3A00Z&sp=rwdl&sv=2018-11-09&sr=csig=rnoGlveGql7ILhziyKYUPBq5ltGc/pzqOCNX5rrLdRQ%3D'`

1. クレデンシャルが正常に作成されたかどうかは、Object Explore の **Security -> Credentials** に移動することで確認できます。

    ![SSMS上のクレデンシャルのスクリーンショット](../images/dp-300-module-15-lab-17.png)

1. もし、入力ミスがあり、クレデンシャルを再作成する必要がある場合は、以下のコマンドでドロップすることができます。

    ```sql
    -- Only run this command if you need to go back and recreate the credential! 
    DROP CREDENTIAL [https://<storage_account_name>.blob.core.windows.net/backups]  
    ```

## バックアップ先URL

1. データベース **AdventureWorks2017** を Transact-SQL で以下のコマンドを使用して Azure にバックアップします。

    ```sql
    BACKUP DATABASE AdventureWorks2017   
    TO URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak';
    GO 
    ```

    ここで、**<storage_account_name>** は、作成された一意のストレージアカウント名です。出力は以下のようなものが返ってくるはずです。

    ![バックアップエラーのスクリーンショット](../images/dp-300-module-15-lab-18.png)

    設定に誤りがあった場合、以下のようなエラーメッセージが表示されます。

    ![バックアップエラーの画面](../images/dp-300-module-15-lab-10.png)

    エラーが発生した場合は、クレデンシャル作成時に入力ミスがなかったか、すべてが正常に作成されたかを確認します。

## Azure CLI を使ってバックアップを検証する

ファイルが実際にAzureにあることを確認するために、Storage Explorer（プレビュー）またはAzure Cloud Shellを使用することができます。

1. ブラウザセッションを開始し、[https://portal.azure.com](https://portal.azure.com/)に移動する。このラボの仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure Cloud Shell を使用して、次の Azure CLI コマンドを実行します。

    ```bash
    az storage blob list -c "backups" --account-name "dp300backupstorage1234" --account-key "storage_key" --output table
    ```

    ストレージアカウント名（**--account-name**の後）とアカウントキー（**--account-key**の後）は、前のコマンドで使用したものと同じ一意のものを使用していることを確認してください。

    ![コンテナ内のバックアップの画面](../images/dp-300-module-15-lab-19.png)

    バックアップファイルが正常に生成されたことが確認できます。

## ストレージエクスプローラーでバックアップを確認します。

1. ストレージエクスプローラー（プレビュー）を使用するには、Azure ポータルのホーム ページから **ストレージアカウント** を選択します。

    ![ストレージアカウントを選択する画面](../images/dp-300-module-15-lab-11.png)

1. バックアップ用に作成した固有のストレージアカウント名を選択します。

1. 左側のナビゲーションで、**Storage browser (preview)** を選択します。ブロブコンテナ**を展開します。

    ![ストレージアカウントにバックアップされたファイルを表示するスクリーンショット](../images/dp-300-module-15-lab-12.png)

1. バックアップ**を選択します。

    ![ストレージアカウントにバックアップされたファイルが表示されます](../images/dp-300-module-15-lab-13.png)

1. バックアップファイルはコンテナ内に保存されていることに注意してください。

    ![ストレージブラウザ上のバックアップファイルを表示した画面](../images/dp-300-module-15-lab-14.png)

## URLからリストアする

このタスクでは、Azure blob ストレージからデータベースを復元する方法を説明します。

1. **SQL Server Management Studio (SSMS)** から、**New Query** を選択し、以下のクエリーを貼り付けて実行します。

    ```sql
    USE AdventureWorks2017;
    GO
    SELECT * FROM Person.Address WHERE AddressId = 1;
    GO
    ```

    ![更新実行前の顧客名を表示した画面](../images/dp-300-module-15-lab-21.png)

1. その顧客の名前を変更するために、このコマンドを実行します。

    ```sql
    UPDATE Person.Address
    SET AddressLine1 = 'This is a human error'
    WHERE AddressId = 1;
    GO
    ```

1. **ステップ1**を再実行し、住所が変更されたことを確認します。ここで、誰かが何千、何百万行をWHERE句なしで、あるいは間違ったWHERE句で変更した場合を想像してみてください。解決策の1つは、利用可能な最後のバックアップからデータベースをリストアすることです。

    このような場合、データベースを復元する必要があります。

1. データベースを復元して、顧客名が誤って変更される前の状態に戻すには、以下を実行します。

    **注：** **SET SINGLE_USER WITH ROLLBACK IMMEDIATE** 構文は、開いているトランザクションがすべてロールバックされるようにします。これにより、アクティブな接続が原因でリストアが失敗するのを防ぐことができます。

    ```sql
    USE [master]
    GO

    ALTER DATABASE AdventureWorks2017 SET SINGLE_USER WITH ROLLBACK IMMEDIATE
    GO

    RESTORE DATABASE AdventureWorks2017 
    FROM URL = 'https://<storage_account_name>.blob.core.windows.net/backups/AdventureWorks2017.bak'
    GO

    ALTER DATABASE AdventureWorks2017 SET MULTI_USER
    GO
    ```

    ここで、**<storage_account_name>** は、作成した一意のストレージアカウント名です。

    このような出力になるはずです。

    ![URLからデータベースの復元を実行した画面](../images/dp-300-module-15-lab-20.png)

1. **ステップ1**を再実行して、顧客名がリストアされたことを確認します。

    ![正しい値を持つ列を示すスクリーンショット](../images/dp-300-module-15-lab-21.png)

Azure Blob Storage サービスへのバックアップまたはサービスからの復元を行うには、コンポーネントとその相互作用を理解することが重要です。

これで、データベースを Azure の URL にバックアップし、必要に応じてリストアできることがわかりました。








