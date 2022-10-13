---
lab:
    title: 'ラボ10: SQL Database でパフォーマンスの悪いクエリの問題領域を分離する'
    module: 'Azure SQL でクエリパフォーマンスを最適化する'
---

# ラボ10: SQL Database でパフォーマンスの悪いクエリの問題領域を分離する

**見積もり時間: 30 分**です。

あなたは、ユーザーが *AdventureWorks2017* データベースにクエリを実行したときに現在発生しているパフォーマンスの問題を解決するために、シニアデータベース管理者として雇われました。あなたの仕事は、クエリパフォーマンスの問題を特定し、このモジュールで学んだテクニックを使用してそれらを改善することです。

最適でないパフォーマンスのクエリを実行し、クエリプランを検証し、データベース内で改善を試みます。

**注意:** これらの演習では、T-SQL コードをコピーして貼り付けるよう求められます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

## データベースの復元

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** にあるデータベースのバックアップファイルをラボ仮想マシン上の **C:\LabFiles\Monitor and optimize** パスにダウンロードします（存在しない場合はフォルダ構造を作成します）。

    ![Picture 03](../images/dp-300-module-07-lab-03.png)

1. Windowsのスタートボタンを選択し、SSMSと入力します。リストから **Microsoft SQL Server Management Studio 18** を選択します。 

1. SSMSが開くと、**Connect to Server** ダイアログにデフォルトのインスタンス名が事前に入力されていることに注意してください。**Connect**を選択します。

    ![Picture 02](../images/dp-300-module-07-lab-01.png)

1. **Databases** フォルダを選択し、**New Query** を選択します。

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

## 実際の実行計画を作成する

SQL Server Management Studioで実行計画を生成するには、いくつかの方法があります。

1. **New Query**を選択します。以下のT-SQLコードをコピーしてクエリウィンドウに貼り付けます。実行**を選択し、このクエリを実行します。

    **注:** **SHOWPLAN_ALL** を使用すると、クエリの実行計画を別のタブでグラフィカルに表示するのではなく、結果ペインでテキストバージョンとして表示できます。

    ```sql
    USE AdventureWorks2017;
    GO

    SET SHOWPLAN_ALL ON;
    GO

    SELECT BusinessEntityID
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    GO

    SET SHOWPLAN_ALL OFF;
    GO
    ```

    実際の**SELECT**文のクエリ結果ではなく、実行計画のテキスト版が表示されます。

    ![クエリ実行計画のテキスト版を表示するスクリーンショット](../images/dp-300-module-10-lab-01.png)

1. **StmtText** カラムの 2 行目のテキストを確認してください。

    ```console
    |--Index Seek(OBJECT:([AdventureWorks2017].[HumanResources].[Employee].[AK_Employee_NationalIDNumber]), SEEK:([AdventureWorks2017].[HumanResources].[Employee].[NationalIDNumber]=CONVERT_IMPLICIT(nvarchar(4000),[@1],0)) ORDERED FORWARD)
    ```

    上記の文章は、実行計画が**AK_Employee_NationalIDNumber**キーに対して**Index Seek**を使用することを説明しています。また、実行計画が**CONVERT_IMPLICIT**ステップを実行する必要があることも示しています。

    クエリオプティマイザは、必要なレコードをフェッチするために適切なインデックスを見つけることができました。

## 最適でないクエリプランを解決する

1. 以下のコードをコピーして、新しいクエリウィンドウに貼り付けてください。

    クエリを実行する前に、以下に示すように **Include Actual Execution Plan** アイコンを選択するか、<kbd>CTRL+M</kbd> を押します。**Execute** を選択するか、<kbd>F5</kbd> を押してクエリを実行します。[メッセージ] タブで、実行計画と論理読み取りをメモします。

    ```sql
    SET STATISTICS IO, TIME ON;

    SELECT [SalesOrderID] ,[CarrierTrackingNumber] ,[OrderQty] ,[ProductID], [UnitPrice] ,[ModifiedDate]
    FROM [AdventureWorks2017].[Sales].[SalesOrderDetail]
    WHERE [ModifiedDate] > '2012/01/01' AND [ProductID] = 772;
    ```


    ![クエリの実行計画を表示した画面](../images/dp-300-module-10-lab-02.png)

    実行計画を確認すると、**Key Lookup**という項目があることがわかります。アイコンの上にマウスを置くと、プロパティがクエリで取得された各行に対して実行されることを示しているのがわかります。実行計画が **Key Lookup** 操作を実行していることがわかります。

    ![このように、列の出力リストを示すスクリーンショット](../images/dp-300-module-10-lab-03.png)が表示されます。

    **出力リスト**にある列をメモしておいてください。このクエリをどのように改善しますか？

    キーのルックアップを削除するためにどのインデックスを変更する必要があるかを特定するために、その上のインデックスシークを検査する必要があります。インデックスシーク演算子の上にマウスを置くと、その演算子のプロパティが表示されます。

    ![NonClusteredインデックスを示すスクリーンショット](../images/dp-300-module-10-lab-04.png)

1. **キー検索**は、クエリで返される、または検索されるすべてのフィールドを含むカバーインデックスを追加することで、削除することができます。この例では、インデックスは **ProductID** 列のみを使用します。**キールックアップ**を修正し、クエリを再実行すると、新しいプランが表示されます。

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID] ON [Sales].[SalesOrderDetail]
    ([ProductID] ASC)
    ```

    **出力リスト**フィールドを含まれる列としてインデックスに追加すると、**キールックアップ**は削除されます。インデックスはすでに存在しているので、列を追加するためには、インデックスをDROPして再作成するか、**DROP_EXISTING=ON**を設定する必要があります。ProductID**列はすでにインデックスの一部であり、含まれる列として追加する必要はないことに注意してください。ModifiedDate**を追加することで、もう1つインデックスを改善することができます。

    ```sql
    CREATE NONCLUSTERED INDEX [IX_SalesOrderDetail_ProductID]
    ON [Sales].[SalesOrderDetail] ([ProductID],[ModifiedDate])
    INCLUDE ([CarrierTrackingNumber],[OrderQty],[UnitPrice])
    WITH (DROP_EXISTING = on);
    GO
    ```

1. ステップ 1 のクエリを再実行します。論理読み込みの変更と実行計画の変更を記録してください。この計画では、作成した非クラスタ化インデックスを使用するだけでよくなりました。

    ![改善された実行計画を示すスクリーンショット](../images/dp-300-module-10-lab-05.png)

## Query Store を使ってリグレッションを検出し処理する

次に、クエリストア用のクエリ統計情報を生成するワークロードを実行し、**Top Resource Consuming Queries** レポートを調べてパフォーマンスの低下を特定し、より良い実行計画を強制する方法を見ます。

1. **New Query** を選択します。以下のT-SQLコードをコピーしてクエリウィンドウに貼り付けます。実行**を選択して、このクエリを実行します。

    このスクリプトは、AdventureWorks2017データベースのクエリストア機能を有効にして、データベースを互換性レベル100に設定します。

    ```sql
    USE [master];
    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE = ON;
    GO

    ALTER DATABASE [AdventureWorks2017] SET QUERY_STORE (OPERATION_MODE = READ_WRITE);
    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 100;
    GO
    ```

    互換性レベルを変更することは、データベースを過去に戻すようなものです。SQL Server が使用できる機能を、SQL Server 2008 で使用できた機能に制限します。

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/CreateRandomWorkloadGenerator.sql** にある T-SQL スクリプトを **C:\LabFiles\Monitor and optimize** にダウンロードします。

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/ExecuteRandomWorkload.sql** にある T-SQL スクリプトを **C:\LabFile\Monitor and optimizes** にダウンロードします。

1. SQL Server Management Studio で **ファイル** > **開く** > **ファイル** メニューを選択します。

1. **C:\LabFiles\Monitor and optimize\CreateRandomWorkloadGenerator.sql** ファイルに移動します。

1. SQL Server Management Studio で開いたら、[**実行**] を選択するか、<kbd>F5</kbd> を押してクエリを実行します。

1. 新しいクエリ エディターで、ファイル **C:\LabFiles\Monitor and optimize\ExecuteRandomWorkload.sql** を開き、**実行** を選択するか、<kbd>F5</kbd> を押してクエリを実行します。

1. 実行が完了したら、スクリプトをもう一度実行して、サーバーに追加の負荷を作成します。このクエリの [クエリ] タブを開いたままにします。

1. 以下のコードをコピーして新しいクエリウィンドウに貼り付け、**実行**を選択するか、<kbd>F5</kbd>を押して実行します。

    このスクリプトは、データベースの互換性モードを SQL Server 2019 (**150**) に変更します。SQL Server 2008 以降のすべての機能と改善点が、データベースで利用できるようになります。

    ```sql
    USE [master];
    GO

    ALTER DATABASE [AdventureWorks2017] SET COMPATIBILITY_LEVEL = 150;
    GO
    ```

1. **ExecuteRandomWorkload.sql** ファイルからクエリタブに戻り、再実行します。

## リソース消費量の多いクエリレポートを確認する

1. Query Storeノードを表示するには、SQL Server Management StudioでAdventureWorks2017データベースをリフレッシュする必要があります。データベース名を右クリックし、**リフレッシュ**を選択します。すると、データベースの下にクエリストアノードが表示されます。

    ![クエリストアを展開する](../images/dp-300-module-10-lab-06.png)

1. **Query Store** ノードを展開し、利用可能なすべてのレポートを表示します。Top Resource Consuming Queries** レポートを選択します。

    ![クエリストアからのトップリソース消費クエリレポート](../images/dp-300-module-10-lab-07.png)

1. 下図のようなレポートが表示されます。右側のメニューのドロップダウンを選択し、**Configure**を選択します。

    ![トップリソースコンシューミングクエリレポートの設定オプションの選択](../images/dp-300-module-10-lab-08.png)

1. 設定画面で、クエリプランの最小数のフィルタを2に変更し、**OK**を選択します。

    ![クエリプランの最小数の設定](../images/dp-300-module-10-lab-09.png)

1. レポートの左上にある棒グラフの一番左の棒を選択して、最長時間のクエリを選択します。

    ![最も時間が長いクエリ](../images/dp-300-module-10-lab-10.png)

    これは、クエリストア内の最長時間クエリのクエリとプランのサマリーを表示します。

## より良い実行プランを強制的に作成する

1. 以下のように、レポートの計画概要部分に移動してください。大きく異なる期間の2つの実行計画があることに気がつくでしょう。

    ![プランの要約](../images/dp-300-module-10-lab-11.png)

1. レポートの右上のウィンドウで、持続時間が最も短いプラン ID (これはチャートの Y 軸の低い位置で示されます) を選択します。上の図では、それは *PlanID 43* です。Plan Summary チャートの隣にあるプラン ID を選択します (上のスクリーンショットのようにハイライトされているはずです)。

1. サマリーチャートの下にある **Force Plan** を選択します。確認ウィンドウがポップアップするので、**Yes**を選択します。

    ![確認のスクリーンショット](../images/dp-300-module-10-lab-12.png)

    計画が強制されると、**Forced Plan**がグレーアウトし、計画概要ウィンドウの計画が強制されたことを示すチェックマークが付くことがわかります。

    クエリオプティマイザは、どの実行プランを使用するかについて、適切な選択をしないことがあります。これが発生した場合、パフォーマンスが向上していることがわかっている場合は、必要なプランを SQL サーバーに強制的に使用させることができます。


## パフォーマンスに影響を与えるクエリヒントの使用

次に、ワークロードを実行し、パラメータを使用するようにクエリを変更し、クエリヒントを適用し、再実行します。

この演習を続ける前に、**ウィンドウ**メニューを選択し、**すべてのドキュメントを閉じる**を選択して、現在のすべてのクエリウィンドウを閉じます。ポップアップで**No**を選択します。

1. **New Query** を選択し、クエリを実行する前に **Include Actual Execution Plan** アイコンを選択するか、<kbd>CTRL+M</kbd> を使用してください。

    ![Include Actual Execution Plan](../images/dp-300-module-10-lab-13.png)

1. 以下のクエリを実行します。実行計画では、インデックスシーク演算子が表示されていることに注意してください。

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=288;
    ```

    ![実行計画が更新されたスクリーンショット](../images/dp-300-module-10-lab-14.png)

1. 新しいクエリウィンドウで、次のクエリを実行します。両方の実行計画を比較してください。

    ```sql
    USE AdventureWorks2017;
    GO

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID=277;
    ```

    今回の変更点は、SalesPersonIDの値が277に設定されていることだけです。実行プランのクラスタ化インデックススキャンに注目してください。

    ![SQL文を表示したスクリーンショット](../images/dp-300-module-10-lab-15.png)

見ての通り、インデックスの統計情報から、クエリオプティマイザは`WHERE`句の値が異なるため、異なる実行計画を選択しました。

*SalesPersonID*の値を変更しただけなのに、なぜ異なるプランになるのでしょうか？

このクエリは `WHERE` 節で定数を使用しているため、オプティマイザはこれらのクエリをそれぞれユニークなものとして捉え、毎回異なる実行計画を生成しています。

## 変数を使用するようにクエリを変更し、クエリヒントを使用する

1. SalesPersonID に変数値を使用するようにクエリを変更します。

1. T-SQL **DECLARE** ステートメントを使用して @SalesPersonID を宣言し、 **WHERE** 句で値をハードコードする代わりに、値を渡すことができるようにします。変数のデータ型がターゲット・テーブルのカラムのデータ型と一致することを確認し、暗黙の変換を回避する必要があります。

    ```sql
    USE AdventureWorks2017;
    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID;
    ```

    実行計画を見ると、結果を得るためにインデックススキャンを使用していることがわかります。クエリオプティマイザは実行時までローカル変数の値を知ることができないため、良い最適化を行うことができなかったのです。

1. クエリヒントを提供することで、クエリオプティマイザがより良い選択をするのを助けることができます。OPTION (RECOMPILE)` を指定して、上記のクエリを再実行します。

    ```sql
    USE AdventureWorks2017
    GO

    SET STATISTICS IO, TIME ON;

    DECLARE @SalesPersonID INT;

    SELECT @SalesPersonID = 288;

    SELECT SalesOrderId, OrderDate
    FROM Sales.SalesOrderHeader
    WHERE SalesPersonID= @SalesPersonID
    OPTION (RECOMPILE);
    ```

    クエリオプティマイザがより効率的な実行計画を選択できるようになったことに注目してください。RECOMPILE` オプションは、クエリコンパイラが変数をその値に置き換えるようにします。

    統計情報を比較すると、メッセージタブで、論理読み込みの差はクエリヒントなしのクエリの方が **68%** 多い（689対409）ことがわかります。

この演習では、クエリの問題を特定する方法と、クエリプランを改善するためにそれを修正する方法を学びました。
