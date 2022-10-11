---
lab:
    title: 'Lab 4 – Azure SQL データベース ファイアウォール ルールを構成する'
    module: 'データベース サービスのセキュアな環境を実装する'
---

# セキュアな環境の実装

**推定時間：30分**

受講者は、レッスンで得た情報をもとに、AzureポータルおよびAdventureWorksデータベース内のセキュリティを設定し、その後、実装します。

あなたは、データベース環境のセキュリティを確保するために、シニアデータベース管理者として採用されました。これらのタスクは、Azure SQL Databaseに焦点を当てます。

**注：** これらの演習では、T-SQL コードをコピーして貼り付けるよう求められます。コードを実行する前に、コードが正しくコピーされていることを確認してください。

## Azure SQL Database のファイアウォールルールを設定する。

1. ラボ仮想マシンから、ブラウザー セッションを開始し、[https://portal.azure.com](https://portal.azure.com/) にナビゲートします。このラボ仮想マシンの **Resources** タブで提供される Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure ポータルから、上部の検索ボックスで「SQL Server」を検索し、オプションの一覧から **SQL Server** をクリックします。

    ![ソーシャルメディアへの投稿画面 説明が自動生成される](../images/dp-300-module-04-lab-1.png)

1. サーバー名 **dp300-lab-XXXXXXXX** を選択して、詳細ページを表示します（SQLサーバーには別のリソースグループと場所が割り当てられている場合があります）。

    ![ソーシャルメディアへの投稿画面 自動生成された説明文](../images/dp-300-module-04-lab-2.png)

1. SQLサーバーの詳細画面で、以下のようにサーバー名をクリップボードにコピーします。

    ![写真2](../images/dp-300-module-04-lab-3.png)

1. **ネットワーク設定を表示する**を選択します。

    ![写真2](../images/dp-300-module-04-lab-4.png)

1. **ネットワーク**ページで、**+ クライアントIPv4アドレスの追加**をクリックし、**保存**をクリックします。

   ![ 写真3](../images/dp-300-module-04-lab-5.png)

    **注意：** クライアントのIPアドレスは自動的に入力されました。クライアントIPアドレスを追加することで、SQL Server Management Studioやその他のクライアントツールを使用してAzure SQL Databaseに接続することができるようになります。**クライアント IP アドレス** は、後で使用するのでメモしておいてください。

1. SQL Server Management Studio を起動します。サーバーへの接続ダイアログボックスで、Azure SQL Databaseのサーバー名を貼り付けて、以下の認証情報でログインします。

    - **サーバー名:** &lt;_paste your Azure SQL Database server name here_&gt;
    - **認証:** SQL Server 認証
    - **サーバー管理者ログイン:** sqladmin
    - **パスワード:** P@ssw0rd01

    ![携帯電話の画面説明の自動生成](../images/dp-300-module-04-lab-6.png)

1. **Connect** をクリックします。

1. オブジェクトエクスプローラでサーバーノードを展開し、**Databases**を右クリックします。**データ層アプリケーションのインポート** をクリックします。

    ![ソーシャルメディア投稿のスクリーンショット 自動生成された説明](../images/dp-300-module-04-lab-7.png)をクリックします。

1. **データ層アプリケーションのインポート**ダイアログで、最初の画面で **Next** をクリックします。

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorksLT.bacpac** にある .bacpac ファイルを、ラボ VM 上の **C:\LabFiles\Secure Environment** パスにダウンロードします（存在しない場合はフォルダー構造を作成します）。

1. **Import Settings** 画面で **Browse** をクリックして **C:\LabFilesSecure Environment** フォルダに移動し、 **AdventureWorksLT.bacpac** ファイルをクリックし、**Open** をクリックしてください。**Import Data-tier Application** の画面に戻り、**Next**をクリックします。

    ![ソーシャルメディアへの投稿の説明文が自動生成されます](../images/dp-300-module-04-lab-8.png)

    ![ソーシャルメディアへの投稿画面 説明文は自動生成されます](../images/dp-300-module-04-lab-9.png)

1. **データベース設定** 画面にて、以下のように変更します。

    - **データベース名:** AdventureWorksFromBacpac
    - **Microsoft Azure SQL Databaseのエディション:** Basic

    ![携帯電話の画面説明の自動生成](../images/dp-300-module-04-lab-10.png)

1. **次へ**をクリックします。

1. **Summary** 画面で、**Finish**をクリックします。インポートが完了すると、以下のような結果が表示されます。その後、**Close**をクリックします。

    ![携帯電話の画面説明の自動生成](../images/dp-300-module-04-lab-11.png)

1. SQL Server Management Studioに戻り、**Object Explorer**で**Databases**フォルダを展開します。次に、**AdventureWorksFromBacpac**データベースを右クリックし、**New Query**を選択します。

    ![携帯電話の説明文のスクリーンショットが自動的に生成されます](../images/dp-300-module-04-lab-12.png)


1. 以下の T-SQL クエリをクエリウィンドウに貼り付けて実行します。
    1. **重要：** **000.000.000.00** をクライアントのIPアドレスに置き換えてください。 **実行** をクリックするか、**F5** を押します。

    ```sql
    EXECUTE sp_set_database_firewall_rule 
            @name = N'AWFirewallRule',
            @start_ip_address = '000.000.000.00', 
            @end_ip_address = '000.000.000.00'
    ```

1. 次に、**AdventureWorksFromBacpac** データベースに含まれるユーザーを作成します。**New Query**をクリックし、以下のT-SQLを実行します。

    ```sql
    USE [AdventureWorksFromBacpac]
    GO
    CREATE USER ContainedDemo WITH PASSWORD = 'P@ssw0rd01'
    ```

    ![携帯電話のスクリーンショット 自動生成された説明文](../images/dp-300-module-04-lab-13.png)

    **注：** このコマンドは、**AdventureWorksFromBacpac**データベース内に含まれるユーザーを作成します。次のステップでは、このクレデンシャルをテストします。

1. **Object Explorer** に移動します。**Connect** をクリックし、次に **Database Engine** をクリックします。

    ![Picture 1960831949](../images/dp-300-module-04-lab-14.png)

1. 前のステップで作成した認証情報を使って接続を試みます。以下の情報を使用する必要があります。

    - **ログイン:** ContainedDemo
    - **パスワード:** P@ssw0rd01

     **Connect** をクリックします。

     次のエラーが表示されます。

    ![携帯電話の説明のスクリーンショットが自動生成されました](../images/dp-300-module-04-lab-15.png)

    **注：** このエラーは、接続が、ユーザーが作成された**AdventureWorksFromBacpac**ではなく、**master*データベースにログインしようとしたために発生します。OK**をクリックしてエラーメッセージを終了し、次に示すように **Connect to Server** ダイアログボックスで **Options >>** をクリックして、接続コンテキストを変更します。

    ![Picture 9](../images/dp-300-module-04-lab-16.png)

1. **接続のプロパティ** タブで、データベース名 **AdventureWorksFromBacpac** を入力し、 **接続** をクリックします。

    ソーシャルメディアへの投稿のスクリーンショット 自動生成された説明](../images/dp-300-module-04-lab-17.png)

1. **ContainedDemo** というユーザーを使って認証に成功したことに注意してください。今回は、新しく作成したユーザーがアクセスできる唯一のデータベースである **AdventureWorksFromBacpac** に直接ログインしています。

    ![Picture 10](../images/dp-300-module-04-lab-18.png)

この演習では、Azure SQL Database でホストされているデータベースにアクセスするために、サーバーとデータベースのファイアウォール ルールを設定しました。また、T-SQLステートメントを使用して、含まれるユーザーを作成し、SQL Server Management Studioを使用して、アクセスを確認しました。
