---
lab:
    title: 'Lab 2 - Azure SQLデータベースのプロビジョニング'
    module: 'データプラットフォーム・リソースの計画・導入'
---

# Azure SQL データベースのプロビジョニング

**見積もり時間: 40 分**

仮想ネットワークエンドポイントを使用して、Azure SQL Database を展開するために必要な基本的なリソースを設定します。SQL データベースへの接続性は、ラボ VM から Azure Data Studio を使用して検証します。

AdventureWorksのデータベース管理者として、仮想ネットワークエンドポイントを含む新しいSQLデータベースをセットアップし、導入のセキュリティを向上させ、簡素化することができます。Azure Data Studio を使用して、データ照会と結果保持のための SQL Notebook の使用について評価します。

## Azure ポータルに移動します。

1. ラボの仮想マシンから、ブラウザセッションを開始し、[https://portal.azure.com](https://portal.azure.com/) に移動します。このラボ仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure ポータルから、上部の検索ボックスで「リソース グループ」を検索し、オプションの一覧から **リソース グループ** を選択します。

    ![Picture 1](../images/dp-300-module-02-lab-45.png)

1. リソースグループ**のページで、リストされたリソースグループ（*contoso-rg*で始まるはず）を確認し、リソースグループに割り当てられた**Location**をメモしてください、次の演習で使用します。

    **注意：** 別の場所が割り当てられている可能性があります。

    ![Picture 1](../images/dp-300-module-02-lab-46.png)

## 仮想ネットワークの作成

1. Azureポータルホームページで、左側のメニューを選択します。 

    ![Picture 2](../images/dp-300-module-02-lab-01_1.png)

1. 左側のナビゲーションペインで、**仮想ネットワーク** をクリックします。 

    ![Picture 3](../images/dp-300-module-02-lab-04.png)

1. **作成**をクリックすると、**仮想ネットワークの作成**ページが表示されます。基本情報(Basics)タブで、以下の情報を入力します。

    - **サブスクリプション:** &lt;Your subscription&gt;
    - **リソースグループ：** *contoso-rg* で始まるリソースグループを作成します。
    - **名前:** lab02-vnet
    - **地域:** 最寄りのリージョンを選択します。

    ![Picture 2](../images/dp-300-module-02-lab-05.png)

1. **次へ**をクリックします。

    ![Picture 3](../images/dp-300-module-02-lab-06.png)

1. Azure SQL データベースエンドポイントの仮想ネットワークの IP 範囲を以下のように設定します。

    - **IP アドレス空間** タブで、IPv4 アドレスはデフォルトのままにします。
    - **default**サブネットリンクをクリックします。表示されるサブネットアドレスの範囲は異なる可能性があることに注意してください。

        ![Picture 4](../images/dp-300-module-02-lab-07.png)

    - 右側の **サブネットの編集** ペインで、 **サービス** ドロップダウンを展開し、 **Microsoft.Sql** を選択します。**保存**を選択します。
    - **確認および作成** ボタンをクリックし、新しい仮想ネットワークの設定を確認してから、**作成** をクリックします。

## Azure SQLデータベースのプロビジョニング

1. Azureポータルから、上部の検索ボックスで「SQLデータベース」を検索し、オプションの一覧から**SQLデータベース**をクリックします。

    ![Picture 5](../images/dp-300-module-02-lab-10.png)

1. **SQLデータベース**のブレードで、**+作成**を選択します。

    ![Picture 6](../images/dp-300-module-02-lab-10_1.png)

1. **SQLデータベースの作成**ページで、**基本**タブで以下のオプションを選択します。

    - **サブスクリプション:** &lt;Your subscription&gt;
    - **リソースグループ：** **contoso-rg** で始まります。
    - **データベース名:** AdventureWorksLT
    - **サーバー：** **新規作成**リンクをクリックします。**SQLデータベースサーバーの作成**ページが開きます。サーバーの詳細を以下のように入力します。
        - **サーバー名:** dp300-lab-&lt;your initials (lower case)&gt; (サーバー名はグローバルで一意でなければなりません)
        - **場所:** &lt;リソースグループと同じリージョン&gt;
        - **認証方法:** SQL 認証を使用する
        - **サーバー管理者ログイン:** dp300admin
        - **パスワード:** dp300P@ssword!
        - **パスワードの確認:** dp300P@ssword!

        **SQLデータベースサーバーの作成** ページは、以下のようなものになります。そして、**OK**をクリックします。

        ![Picture 7](../images/dp-300-module-02-lab-11.png)

    - **SQLデータベースの作成** ページに戻り、**Elastic Poolを使用しますか** が**No** に設定されていることを確認します。
    - **コンピューティングとストレージ** オプションで、**データベースの構成** リンクをクリックします。**構成** ページで、**サービスレベル** ドロップダウンで、**Basic** を選択し、**適用** をクリックします。

    **注：** このサーバー名とログイン情報をメモしておいてください。このサーバー名とログイン情報は、この後のラボで使用します。

1. **バックアップ ストレージの冗長性** オプションは、デフォルト値のままにしておきます。**バックアップストレージの冗長性**は、デフォルトのままにしておきます。

1. 次に、**次: ネットワーク**をクリックします。

1. **ネットワーク** タブの **ネットワーク接続** オプションで、**プライベート エンドポイント** ラジオボタンをクリックします。

    ![Picture 8](../images/dp-300-module-02-lab-14.png)


1. 次に、**プライベート エンドポイント** オプションの下にある **プライベート エンドポイントを追加する** リンクをクリックします。

	![Picture 9](../images/dp-300-module-02-lab-15.png)

1. 右ペインの **プライベート エンドポイントの作成** を以下のように完成させます。

    - **サブスクリプション:** &lt;あなたのサブスクリプション&gt;
    - **リソースグループ:** *contoso-rg* で始まるリソースグループ
    - **場所:** &lt;リソースグループと同じリージョン&gt;
    - **名前:** DP-300-SQL-Endpoint
    - **対象サブリソース:** SqlServer
    - **仮想ネットワーク:** lab02-vnet
    - **Subnet:** lab02-vnet/default (10.x.0.0/24)
    - **プライベート DNS ゾーンとの統合:** Yes
    - **プライベートDNSゾーン:** デフォルト値のまま
    - 設定を確認し、**OK**をクリックします。 

    ![Picture 10](../images/dp-300-module-02-lab-16.png)

1. 新しいエンドポイントが **プライベート エンドポイント** リストに表示されます。

    ![Picture 11](../images/dp-300-module-02-lab-17.png)

1. **次: セキュリティ**、**次: 追加設定**をクリックします。 

1. **追加設定** ページで、**既存のデータを使用します** オプションで **サンプル** を選択します。サンプルデータベースのポップアップメッセージが表示されたら、**OK**を選択します。

    ![Picture 12](../images/dp-300-module-02-lab-18.png)

1. **確認および作成**をクリックします。

1. **作成** をクリックする前に、設定を確認します。

1. デプロイメントが完了したら、**リソースへ移動**をクリックします。

## Azure SQL Databaseへのアクセスを有効にする。

1. **SQL データベース** ページから、**概要** セクションを選択し、上部のセクションでサーバー名のリンクを選択します。

    ![Picture 13](../images/dp-300-module-02-lab-19.png)

1. SQLサーバーのナビゲーションブレードで、**セキュリティ**セクションの下にある**ネットワーク**を選択します。

    ![Picture 14](../images/dp-300-module-02-lab-20.png)

1. **パブリックアクセス** タブで、「選択したネットワーク」をクリックします。

1. **Azure サービスとリソースがこのサーバーにアクセスすることを許可する** プロパティをチェックし、**保存** をクリックします。

    ![Picture 15](../images/dp-300-module-02-lab-21.png)

## Azure Data Studio で Azure SQL Database に接続する。

1. ラボの仮想マシンから Azure Data Studio を起動します。

    - Azure Data Studio の初回起動時に、このポップアップが表示される場合があります。表示された場合は、**Yes (推奨)** をクリックします。 

        ![Picture 16](../images/dp-300-module-02-lab-22.png)

1. Azure Data Studioが開いたら、左上の **Connections** ボタンをクリックし、**Add Connection** をクリックします。

    ![Picture 17](../images/dp-300-module-02-lab-25.png)

1. サイドバーの**Connection**に、先に作成したSQLデータベースに接続するための接続情報を記入します。

    - 接続の種類: *Microsoft SQL Server*
    - サーバー: 以前に作成したSQLサーバーの名前を入力します。例えば、以下のようになります。例： **dp300-lab-xxxxxxxx.database.windows.net** (xxxxxxxxはランダムの番号です)
    - 認証の種類: **SQLログイン**
    - ユーザー名: **dp300admin**
    - パスワード **dp300P@ssword!**
    - **データベース**ドロップダウンを展開し、**AdventureWorksLT** を選択します。
        - **注：**クライアントIPがこのサーバーにアクセスすることを許可するファイアウォールルールを追加するよう求められることがあります。ファイアウォールルールの追加を求められた場合、**アカウントの追加**をクリックし、Azureアカウントにログインします。ファイアウォールルールの**新規作成**画面で、**OK**をクリックします。

        ![Picture 18](../images/dp-300-module-02-lab-26.png)

        または、SQLサーバーに移動し、**ネットワーク**を選択し、**+クライアントIPv4アドレス（あなたのIPアドレス）を追加する**を選択し、Azureポータル上でSQLサーバーのファイアウォールルールを手動で作成することも可能です。

        ![Picture 18](../images/dp-300-module-02-lab-47.png)

    **接続** サイドバーに戻り、接続の詳細を入力し続けます。 

    - サーバーグループは **&lt;default&gt;** のままです。
    - Name（オプション）には、必要であれば、データベースのフレンドリーな名前を入力することができます。
    - 設定を確認し、**Connect**をクリックします。 

    ![Picture 19](../images/dp-300-module-02-lab-27.png)

1. Azure Data Studioがデータベースに接続し、データベースの基本情報とオブジェクトの一部リストが表示されます。

    ![Picture 20](../images/dp-300-module-02-lab-28.png)

## SQL ノートブックで Azure SQL データベースにクエリーを実行する

1. Azure Data Studio で、このラボの AdventureWorksLT データベースに接続し、**New Notebook** ボタンをクリックします。

    ![Picture 21](../images/dp-300-module-02-lab-29.png)

1. ノートブックに新しいテキストボックスを追加するには、**+Text**リンクをクリックします。 

    ![Picture 22](../images/dp-300-module-02-lab-30.png)

**注意:** ノートブック内にプレーンテキストを埋め込んで、クエリや結果セットを説明することができます。

1. **受注小計上位10社** というテキストを入力し、必要であれば太字にします。

    ![携帯電話のスクリーンショット 自動生成された説明文](../images/dp-300-module-02-lab-31.png)

1. **+ Cell** ボタンをクリックし、次に **Code cell** をクリックして、ノートブックの末尾に新しいコードセルを追加してください。 

    ![Picture 23](../images/dp-300-module-02-lab-32.png)

5. 新しいセルに次の SQL 文を貼り付けます。

    ```sql
    SELECT TOP 10 cust.[CustomerID], 
        cust.[CompanyName], 
        SUM(sohead.[SubTotal]) as OverallOrderSubTotal
    FROM [SalesLT].[Customer] cust
        INNER JOIN [SalesLT].[SalesOrderHeader] sohead
            ON sohead.[CustomerID] = cust.[CustomerID]
    GROUP BY cust.[CustomerID], cust.[CompanyName]
    ORDER BY [OverallOrderSubTotal] DESC
    ```

1. 矢印のついた青い丸をクリックして、クエリを実行します。結果がクエリのあるセル内にどのように含まれるかに注意してください。

1. **+ Text** ボタンをクリックすると、新しいテキストセルが追加されます。

1.  **注文の多い製品カテゴリートップ10** というテキストを入力し、必要であれば太字にします。

1. もう一度 **+ Code** ボタンをクリックして新しいセルを追加し、次の SQL 文をセルに貼り付けます。

    ```sql
    SELECT TOP 10 cat.[Name] AS ProductCategory, 
        SUM(detail.[OrderQty]) AS OrderedQuantity
    FROM salesLT.[ProductCategory] cat
    INNER JOIN [SalesLT].[Product] prod
        ON prod.[ProductCategoryID] = cat.[ProductCategoryID]
    INNER JOIN [SalesLT].[SalesOrderDetail] detail
        ON detail.[ProductID] = prod.[ProductID]
    GROUP BY cat.[name]
    ORDER BY [OrderedQuantity] DESC
    ```

1. 青丸の矢印をクリックするとクエリが実行されます。

1. ノートブック内のすべてのセルを実行して結果を表示するには、ツールバーの **Run all** ボタンをクリックします。

	![Picture 17](../images/dp-300-module-02-lab-33.png)

1. Azure Data Studio 内の File メニュー（Save または Save As）からノートブックを **C:\LabfilesDeploy Azure SQL Database** のパスに保存します（存在しない場合はフォルダ構造を作成します）。ファイルの拡張子が **.ipynb** であることを確認します。

1. Azure Data Studio 内から Notebook のタブを閉じます。ファイルメニューから［ファイルを開く］を選択し、保存したノートブックを開く。クエリの結果が、クエリとともにノートブックに保存されていることを確認します。

この演習では、仮想ネットワークエンドポイントを使用して Azure SQL Database をデプロイする方法を確認しました。また、SQL Server Management Studio を使用して、作成した SQL データベースに接続することができました。
