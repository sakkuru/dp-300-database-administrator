---
lab:
    title: 'ラボ07: 断片化の問題を検出および修正する'
    module: 'Azure SQL の運用リソースの監視と最適化'
---

# ラボ07: 断片化の問題を検出および修正する

**推定時間：15分**

受講者は、レッスンで得た情報をもとに、AdventureWorks内のデジタルトランスフォーメーションプロジェクトの成果物のスコープを作成します。Azureポータルや他のツールを調べ、ネイティブツールをどのように活用してパフォーマンス関連の問題を特定し解決するかを決定します。最後に、データベース内のフラグメンテーションを特定し、適切に解決するためのステップを学びます。

あなたは、データベース管理者として、パフォーマンスに関連する問題を特定し、発見された問題を解決するための実行可能なソリューションを提供するために採用されました。AdventureWorks は、10 年以上にわたり、自転車と自転車部品を消費者と販売店に直販しています。最近、同社は、顧客の要求を処理するために使用される製品の性能低下に気づきました。あなたは、SQL ツールを使用して、パフォーマンスの問題を特定し、それを解決する方法を提案する必要があります。

**注意:** これらの演習では、T-SQL コードをコピーして貼り付けることが要求されます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

## データベースの復元

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** にあるデータベースのバックアップファイルをラボ仮想マシン上の **C:\LabFiles\Monitor and optimize** パスにダウンロードします（存在しない場合はフォルダ構造を作成します）。

    ![Picture 03](../images/dp-300-module-07-lab-03.png)

1. Windowsのスタートボタンを選択し、SSMSと入力します。リストから **Microsoft SQL Server Management Studio 18** を選択します。 

1. SSMSが開くと、**Connect to Server** ダイアログにデフォルトのインスタンス名が事前に入力されていることに注意してください。**Connect**を選択します。

    ![Picture 02](../images/dp-300-module-07-lab-01.png)

1. Databases** フォルダを選択し、**New Query** を選択します。

    ![Picture 03](../images/dp-300-module-07-lab-04.png)

1. 新しいクエリウィンドウで、以下のT-SQLをコピーして貼り付けます。データベースを復元するためにクエリを実行する。

    ```sql
    RESTORE DATABASE AdventureWorks2017
    FROM DISK = 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.bak'
    WITH RECOVERY,
          MOVE 'AdventureWorks2017' 
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017.mdf',
          MOVE 'AdventureWorks2017_log'
            TO 'C:\LabFiles\Monitor and optimize\AdventureWorks2017_log.ldf';
    ```

    **注意：** データベースのバックアップファイルの名前とパスは、ステップ1でダウンロードしたものと一致させる必要があります。

1. リストア完了後、成功のメッセージが表示されるはずです。

    ![Picture 03](../images/dp-300-module-07-lab-05.png)

## インデックスの断片化を調べる

1. **新規クエリ**を選択します。以下のT-SQLコードをコピーしてクエリウィンドウに貼り付けます。実行**を選択して、このクエリーを実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT i.name Index_Name
     , avg_fragmentation_in_percent
     , db_name(database_id)
     , i.object_id
     , i.index_id
     , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
     INNER JOIN sys.indexes i ON ps.object_id = i.object_id 
     AND ps.index_id = i.index_id
    WHERE avg_fragmentation_in_percent > 50 -- find indexes where fragmentation is greater than 50%
    ```

    このクエリーは、断片化が **50%** を超えるインデックスを報告します。このクエリーは結果を返すべきではありません。

1. **新規クエリ** を選択します。以下のT-SQLコードをコピーして、クエリウィンドウに貼り付けます。実行**を選択して、このクエリを実行します。

    ```sql
    USE AdventureWorks2017
    GO
        
    INSERT INTO [Person].[Address]
        ([AddressLine1]
        ,[AddressLine2]
        ,[City]
        ,[StateProvinceID]
        ,[PostalCode]
        ,[SpatialLocation]
        ,[rowguid]
        ,[ModifiedDate])
        
    SELECT AddressLine1,
        AddressLine2, 
        'Amsterdam',
        StateProvinceID, 
        PostalCode, 
        SpatialLocation, 
        newid(), 
        getdate()
    FROM Person.Address;
    
    GO
    ```

    このクエリは、大量の新しいレコードを追加することで、Person.Address テーブルとそのインデックスの断片化レベルを増加させます。

1. 最初のクエリをもう一度実行します。今度は、4つの高度に断片化されたインデックスを確認することができるはずです。

    ![Picture 03](../images/dp-300-module-07-lab-06.png)


1. 以下のT-SQLコードをコピーして、クエリウィンドウに貼り付けます。このクエリを実行するには、**実行** を選択します。

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

    SQL Server Management Studio の結果ペインで **Messages** タブをクリックします。クエリによって実行された論理読み込みのカウントをメモしておきます。

    ![Picture 03](../images/dp-300-module-07-lab-07.png)

## 断片化したインデックスの再構築

1. 以下のT-SQLコードをコピーして、クエリウィンドウに貼り付けます。**実行**を選択して、このクエリを実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    ALTER INDEX [IX_Address_StateProvinceID] ON [Person].[Address] REBUILD PARTITION = ALL 
    WITH (PAD_INDEX = OFF, 
        STATISTICS_NORECOMPUTE = OFF, 
        SORT_IN_TEMPDB = OFF, 
        IGNORE_DUP_KEY = OFF, 
        ONLINE = OFF, 
        ALLOW_ROW_LOCKS = ON, 
        ALLOW_PAGE_LOCKS = ON)
    ```

1. 以下のクエリーを実行し、**IX_Address_StateProvinceID**インデックスの断片化が50％を超えていないことを確認します。

    ```sql
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT i.name Index_Name
        , avg_fragmentation_in_percent
        , db_name(database_id)
        , i.object_id
        , i.index_id
        , index_type_desc
    FROM sys.dm_db_index_physical_stats(db_id('AdventureWorks2017'),object_id('person.address'),NULL,NULL,'DETAILED') ps
        INNER JOIN sys.indexes i ON (ps.object_id = i.object_id AND ps.index_id = i.index_id)
    WHERE i.name = 'IX_Address_StateProvinceID'
    ```

    この結果を比較すると、フラグメンテーションが81%から0に減少していることがわかります。

1. 前のセクションのselect文を再実行します。Management Studio の **Results** ペインの **Messages** タブに、論理的な読み取りを記録します。インデックスを再構築する前に発生した論理読み込みの数から変化がありましたか？

    ```sql
    SET STATISTICS IO,TIME ON
    GO
        
    USE AdventureWorks2017
    GO
        
    SELECT DISTINCT (StateProvinceID)
        ,count(StateProvinceID) AS CustomerCount
    FROM person.Address
    GROUP BY StateProvinceID
    ORDER BY count(StateProvinceID) DESC;
        
    GO
    ```

インデックスは再構築されたので、可能な限り効率的になり、論理的な読み込みは減少するはずです。これで、インデックスのメンテナンスがクエリのパフォーマンスに影響を与えることがわかりました。

この演習では、クエリパフォーマンスを向上させるために、インデックスの再構築と論理読み込みの分析を行う方法を学びました。
