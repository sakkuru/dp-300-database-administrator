---
lab:
    title: 'Lab 6 – 監視によるパフォーマンス問題の特定'
    module: 'Azure SQL の運用リソースの監視と最適化'
---

# モニタリングでパフォーマンスの問題を特定する

**推定時間：30分**

受講者は、レッスンで得た情報をもとに、AdventureWorks社内のデジタル変革プロジェクトにおける成果物のスコープを設定します。Azureポータルやその他のツールを検証し、パフォーマンス関連の問題を特定し解決するためのツールの活用方法を決定します。

あなたは、データベース管理者として、パフォーマンス関連の問題を特定し、発見された問題を解決するための実行可能なソリューションを提供するために採用されました。あなたは、Azureポータルを使用して、パフォーマンスの問題を特定し、それらを解決するための方法を提案する必要があります。

**注意：** これらの演習では、T-SQL コードをコピーして貼り付けるよう求められます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

## AzureポータルのCPU使用率を確認する

1. ラボの仮想マシンから、ブラウザ セッションを開始し、[https://portal.azure.com](https://portal.azure.com/) にナビゲートします。このラボ仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure ポータルから、上部の検索ボックスで「SQL サーバー」を検索し、オプションの一覧から **SQL サーバー** をクリックします。

    ![ソーシャルメディアへの投稿画面 説明が自動生成される](../images/dp-300-module-04-lab-1.png)

1. サーバー名 **dp300-lab-XXXXXXXX** を選択して、詳細ページを表示します（SQLサーバーには別のリソースグループと場所が割り当てられている場合があります）。

    ![ソーシャルメディアへの投稿画面 自動生成された説明文](../images/dp-300-module-04-lab-2.png)

1. Azure SQLサーバーのメインブレードから、**設定**セクションに移動し、**SQLデータベース**を選択し、データベース名を選択します。

    ![AdventureWOrksLTデータベースを選択する画面](../images/dp-300-module-05-lab-04.png)

1. データベースのメインページで、**サーバーファイアウォールの設定**を選択します。

1. **ネットワーク**ページで、**+ クライアントIPv4アドレスの追加**を選択し、**保存**を選択します。

    ![クライアントIPの追加を選択した画面](../images/dp-300-module-06-lab-02.png)

1. 右上のアイコンから、**ネットワーク**ページを閉じます。

1. 左側のナビゲーションで、 **クエリエディタ（プレビュー）** を選択します。

    ![クエリエディタ(プレビュー)のリンクを選択した画面](../images/dp-300-module-06-lab-04.png)

    **注意：** この機能はプレビュー版です。

1. **Password** に **P@ssw0rd01** と入力し、**OK** を選択します。

    クエリエディタの接続プロパティを表示するスクリーンショット](../images/dp-300-module-06-lab-05.png)

1. **クエリ1**に以下のクエリを入力し、**実行**を選択します。

    ```sql
    DECLARE @Counter INT 
    SET @Counter=1
    WHILE ( @Counter <= 10000)
    BEGIN
        SELECT 
             RTRIM(a.Firstname) + ' ' + RTRIM(a.LastName)
            , b.AddressLine1
            , b.AddressLine2
            , RTRIM(b.City) + ', ' + RTRIM(b.StateProvince) + '  ' + RTRIM(b.PostalCode)
            , CountryRegion
            FROM SalesLT.Customer a
            INNER JOIN SalesLT.CustomerAddress c 
                ON a.CustomerID = c.CustomerID
            RIGHT OUTER JOIN SalesLT.Address b
                ON b.AddressID = c.AddressID
        ORDER BY a.LastName ASC
        SET @Counter  = @Counter  + 1
    END
    ```

    ![クエリを表示するスクリーンショット](../images/dp-300-module-06-lab-06.png)

1. クエリが完了するのを待ちます。

1. **AdventureWorksLT** データベースのブレードで、**監視** セクションの **メトリック** アイコンを選択します。

    ![メトリクスアイコンを選択した画面](../images/dp-300-module-06-lab-07.png)

1. メニューの**メトリック**を**CPU Percentage**に変更し、**平均**を選択します。これにより、指定された時間枠の平均CPUパーセンテージが表示されます。

    ![CPUパーセンテージを表示するスクリーンショット](../images/dp-300-module-06-lab-08.png)

1. CPUの平均値を観察してください。若干異なる結果になるかもしれません。また、このクエリを複数回実行することで、より実質的な結果を得ることができます。

    ![平均の集計を示すスクリーンショット](../images/dp-300-module-06-lab-09.png)

## 高いCPUのクエリを特定する

1. **AdventureWorksLT** データベースのブレードの **インテリジェント パフォーマンス** セクションにある **Query Performance Insight** アイコンを探します。

    ![クエリパフォーマンスインサイトアイコンを示すスクリーンショット](../images/dp-300-module-06-lab-10.png)

1. **設定のリセット**を選択します。

1. グラフの下のグリッドにあるクエリをクリックします。クエリが表示されない場合は、2分ほど待って**更新**を選択します。

    **注意:** 期間とクエリIDが異なる場合があります。複数のクエリが表示された場合は、それぞれのクエリをクリックして結果を観察してください。

    ![平均集計の画面](../images/dp-300-module-06-lab-12.png)

このクエリでは、合計時間が1分以上、実行回数が約10,000回であることが確認できます。

この演習では、Azure SQL Database のサーバー リソースを調査し、Query Performance Insight を使用して潜在的なクエリ パフォーマンスの問題を特定する方法を学びました。
