---
lab:
    title: 'Lab 3 – Azure Active Directory を使用して Azure SQL データベースへのアクセスを認証する'
    module: 'データベース サービスに安全な環境を実装する'
---

# データベースの認証と認可を構成する

**推定時間：20分**

受講者は、レッスンで得た情報をもとに、Azure ポータルおよび *AdventureWorks* データベース内にセキュリティを設定し、その後実装します。

あなたは、データベース環境のセキュリティを確保するために、シニアデータベース管理者として採用されました。

**注意：** これらの演習では、T-SQL コードをコピーして貼り付けるように要求されます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

## Azure Active Directory を使用して、Azure SQL Database へのアクセスを許可する。

1. ラボの仮想マシンから、ブラウザー セッションを開始し、[https://portal.azure.com](https://portal.azure.com/) にナビゲートします。このラボ仮想マシンの **Resources** タブで提供される Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure ポータルのホーム ページで、**すべてのリソース** を選択します。

    ![Azure ポータルのホーム ページで、すべてのリソースを選択した画面](../images/dp-300-module-03-lab-01.png)

1. Azure SQL Database サーバー **dp300-lab-xxxxxx**（**xxxxxx** はランダムな文字列）を選択し、**Active Directory 管理者** の下にある **構成されていません** を選択します。

    ![P未設定を選択した画面](../images/dp-300-module-03-lab-02.png)

1. 次の画面で、**管理者の設定**を選択します。

    ![P管理者設定を選択した画面](../images/dp-300-module-03-lab-03.png)

1. サイドバーの **Azure Active Directory** で、Azure ポータルにログインした Azure ユーザー名を検索し、**選択** をクリックします。

1. **保存** を選択し、処理を完了します。これで、以下のようにユーザー名がサーバーのAzure Active Directory管理者になります。

    ![Active Directory 管理者ページのスクリーンショット](../images/dp-300-module-03-lab-04.png)

1. 左側の**概要**を選択し、**サーバー名**をコピーします。

    ![サーバー名をコピーする画面](../images/dp-300-module-03-lab-05.png)

1. SQL Server Management Studioを開き、**Connect** > **Database Engine**を選択します。**サーバー名**に、サーバー名を貼り付けます。認証の種類を **Azure Active Directory Universal with MFA** に変更します。

    ![サーバーへの接続ダイアログのスクリーンショット](../images/dp-300-module-03-lab-06.png)

    **ユーザー名**は、**Resources** タブから Azure の **Username** を選択します。

1. **Connect** を選択します。

> [!注意]。
> Azure SQL データベースに初めてサインインしようとするとき、クライアント IP アドレスをファイアウォールに追加する必要があります。SQL Server Management Studio を使用すると、この作業を行うことができます。Azure Portal の **Resources** タブから **password** を使用し、**Sign in** を選択して Azure の資格情報を選択し、**OK** を選択します。

> ![クライアントIPアドレスの追加画面](../images/dp-300-module-03-lab-07.png)

## データベースオブジェクトへのアクセスを管理する

このタスクでは、データベースとそのオブジェクトへのアクセスを管理します。最初に行うことは、*AdventureWorksLT* データベースに2人のユーザーを作成することです。

1. **オブジェクトエクスプローラ**を使用し、**Databases**を展開します。
1. **AdventureWorksLT**を右クリックし、**New Query**を選択します。

    ![メニューオプションのスクリーンショット](../images/dp-300-module-03-lab-08.png)

1. 新しいクエリウィンドウで、以下のT-SQLをコピーして貼り付けます。2人のユーザーを作成するためにクエリを実行します。

    ```sql
    CREATE USER [DP300User1] WITH PASSWORD = 'Azur3Pa$$';
    GO

    CREATE USER [DP300User2] WITH PASSWORD = 'Azur3Pa$$';
    GO
    ```

    **注：** これらのユーザーは、AdventureWorksLTデータベースのスコープで作成されます。次に、カスタムロールを作成し、ユーザーを追加します。

1. 同じクエリウィンドウで、以下のT-SQLを実行します。

    ```sql
    CREATE ROLE [SalesReader];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User1];
    GO

    ALTER ROLE [SalesReader] ADD MEMBER [DP300User2];
    GO
    ```

    次に、**SalesLT** スキーマに新しいストアドプロシージャを作成します。

1. クエリーウィンドウで以下の T-SQL を実行します。

    ```sql
    CREATE OR ALTER PROCEDURE SalesLT.DemoProc
    AS
    SELECT P.Name, Sum(SOD.LineTotal) as TotalSales ,SOH.OrderDate
    FROM SalesLT.Product P
    INNER JOIN SalesLT.SalesOrderDetail SOD on SOD.ProductID = P.ProductID
    INNER JOIN SalesLT.SalesOrderHeader SOH on SOH.SalesOrderID = SOD.SalesOrderID
    GROUP BY P.Name, SOH.OrderDate
    ORDER BY TotalSales DESC
    GO
    ```

    次に、`EXECUTE AS USER`構文を使って、セキュリティをテストしてみましょう。これにより、データベースエンジンはユーザーのコンテキストでクエリを実行することができます。

1. 以下のT-SQLを実行します。

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    これはメッセージとともに失敗します。

    ![エラーメッセージのスクリーンショット、The EXECUTE permission was denied on the object DemoProc](../images/dp-300-module-03-lab-09.png)


1. 次に、ロールにストアプロシージャの実行を許可する権限を与えます。以下のT-SQLを実行します。

    ```sql
    REVERT;
    GRANT EXECUTE ON SCHEMA::SalesLT TO [SalesReader];
    GO
    ```

    最初のコマンドは、実行コンテキストをデータベース所有者に戻します。

1. 前の T-SQL を再実行します。

    ```sql
    EXECUTE AS USER = 'DP300User1'
    EXECUTE SalesLT.DemoProc
    ```

    ![ストアドプロシージャから返されたデータ行を表示するスクリーンショット](../images/dp-300-module-03-lab-10.png)

この演習では、Azure Active Directory を使用して、Azure でホストされている SQL Server への Azure 認証情報のアクセス権を付与する方法について説明しました。また、T-SQLステートメントを使用して、新しいデータベースユーザーを作成し、ストアドプロシージャーの実行権限を付与しました。
