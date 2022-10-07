---
lab:
    title: 'Lab 9 – データベース設計の問題を特定する'
    module: 'Azure SQL でクエリ パフォーマンスを最適化する'
---

# データベース設計の問題を特定する

**見積もり時間：15 分**

受講者は、レッスンで得た情報をもとに、AdventureWorks内のデジタルトランスフォーメーションプロジェクトの成果物のスコープを設定します。Azureポータルやその他のツールを検証し、ネイティブツールを活用してパフォーマンス関連の問題を特定し解決する方法を決定します。最後に、正規化、データ型選択、インデックス設計に問題がないか、データベース設計を評価できるようになります。

あなたは、パフォーマンス関連の問題を特定し、発見された問題を解決するための実行可能なソリューションを提供するデータベース管理者として採用されました。AdventureWorks は、10 年以上にわたり、自転車と自転車部品を消費者と販売店に直販しています。あなたの仕事は、クエリパフォーマンスの問題を特定し、このモジュールで学んだテクニックを使用してそれらを改善することです。

**注意：**これらの演習では、T-SQL コードをコピーして貼り付けるよう求められます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

## データベースの復元

1. **https://github.com/MicrosoftLearning/dp-300-database-administrator/blob/master/Instructions/Templates/AdventureWorks2017.bak** にあるデータベースのバックアップファイルをラボ仮想マシン上の **C:\LabFiles\Monitor and optimize** パスにダウンロードします（存在しない場合はフォルダ構造を作成します）。

    ![Picture 03](../images/dp-300-module-07-lab-03.png)

1. Windowsのスタートボタンを選択し、SSMSと入力します。リストから **Microsoft SQL Server Management Studio 18** を選択します。 

1. SSMSが開くと、**Connect to Server** ダイアログにデフォルトのインスタンス名が事前に入力されていることに注意してください。Connect**を選択します。

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

## クエリを調べて問題を特定する

1. **新規クエリ**を選択します。以下の T-SQL コードをコピーしてクエリウィンドウに貼り付けます。実行**を選択し、このクエリを実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

1. クエリを実行する前に、下図のように **実際の実行計画を含める** アイコンを選択するか、**CTRL+M** を押してください。これにより、クエリの実行時に実行プランが表示されます。このクエリを実行するには、**Execute** を選択します。

    ![Picture 01](../images/dp-300-module-09-lab-01.png)

1. 結果パネルの**実行計画**タブを選択し、実行計画に移動します。実行計画で、`SELECT`演算子にマウスオーバーします。下図のように、黄色い三角形の中に感嘆符が表示された警告メッセージが表示されます。警告メッセージの内容を確認してください。

    ![Picture 02](../images/dp-300-module-09-lab-02.png)

## 警告メッセージを修正する方法を確認する

*[HumanResources].[Employee]* テーブルの構造は、以下のデータ定義言語（DDL）ステートメントに示されています。このDDLに対して、前のSQLクエリで使用されているフィールドを、その型に注意しながら確認してください。

```sql
CREATE TABLE [HumanResources].[Employee](
     [BusinessEntityID] [int] NOT NULL,
     [NationalIDNumber] [nvarchar](15) NOT NULL,
     [LoginID] [nvarchar](256) NOT NULL,
     [OrganizationNode] [hierarchyid] NULL,
     [OrganizationLevel] AS ([OrganizationNode].[GetLevel]()),
     [JobTitle] [nvarchar](50) NOT NULL,
     [BirthDate] [date] NOT NULL,
     [MaritalStatus] [nchar](1) NOT NULL,
     [Gender] [nchar](1) NOT NULL,
     [HireDate] [date] NOT NULL,
     [SalariedFlag] [dbo].[Flag] NOT NULL,
     [VacationHours] [smallint] NOT NULL,
     [SickLeaveHours] [smallint] NOT NULL,
     [CurrentFlag] [dbo].[Flag] NOT NULL,
     [rowguid] [uniqueidentifier] ROWGUIDCOL NOT NULL,
     [ModifiedDate] [datetime] NOT NULL
) ON [PRIMARY]
```


1. 実行計画書に示された警告メッセージによると、どのような変更を推奨しますか。

    1. どのフィールドが暗黙の変換を引き起こしているのか、またその理由を特定してください。
    1. 以下のクエリを確認します。

        ```sql
        SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
        FROM HumanResources.Employee
        WHERE NationalIDNumber = 14417807;
        ```

        **14417807** は引用符で囲まれた文字列ではないので、`WHERE`句の *NationalIDNumber* カラムと比較される値は数値として比較されることに注意してください。

        テーブルの構造を調べてみると、*NationalIDNumber* カラムは `INT` データ型ではなく `NVARCHAR` データ型を使っていることがわかります。この不整合により、データベースオプティマイザは暗黙のうちに数値を `NVARCHAR` 値に変換し、最適でない計画を作成してクエリパフォーマンスにさらなるオーバーヘッドを発生させることになります。

暗黙の変換警告を修正するために、2つのアプローチを実装することができます。次のステップで、それぞれを調査します。

### コードを変更する

1. 暗黙の変換を解決するために、コードをどのように変更しますか？コードを変更し、クエリを再実行します。

    まだオンになっていない場合は、**実際の実行計画を含める** (**CTRL+M**)をオンにすることを忘れないでください。

    このシナリオでは、値の両側にシングルクォートを追加するだけで、数値から文字フォーマットに変更されます。このクエリのクエリウィンドウを開いたままにしておきます。

    更新されたSQLクエリを実行します。

    ```sql
    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = '14417807';
    ```

    ![Picture 03](../images/dp-300-module-09-lab-03.png)

    **注意:** 警告メッセージはなくなり、クエリプランは改善されました。`WHERE` 節を変更して、*NationalIDNumber* 列と比較される値がテーブル内の列のデータ型と一致するようにしたところ、オプティマイザは暗黙の変換を取り除くことができました。

### Change the data type

1. テーブルの構造を変更することで、暗黙の変換警告を修正することができます。

    インデックスの修正を試みるには、以下のクエリを新しいクエリウィンドウにコピー＆ペーストして、カラムのデータ型を変更します。**実行**を選択するか、<kbd>F5</kbd>キーを押して、クエリを実行してみてください。

    ```sql
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    ```

    *NationalIDNumber* 列のデータ型を INT に変更すれば、変換の問題は解決されます。しかし、この変更により、データベース管理者として解決しなければならない別の問題が発生します。

    ![Picture 04](../images/dp-300-module-09-lab-04.png)

    *NationalIDNumber* カラムは、すでに存在する非クラスタ化インデックスの一部であり、データ型を変更するためにインデックスの再構築/再作成が必要です。**これは、設計において正しいデータ型を選択することの重要性を強調するものです**。

1. この問題を解決するには、以下のコードをクエリウィンドウにコピー＆ペーストし、**実行**を選択して実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    --Dropping the index first
    DROP INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]
    GO

    --Changing the column data type to resolve the implicit conversion warning
    ALTER TABLE [HumanResources].[Employee] ALTER COLUMN [NationalIDNumber] INT NOT NULL;
    GO

    --Recreating the index
    CREATE UNIQUE NONCLUSTERED INDEX [AK_Employee_NationalIDNumber] ON [HumanResources].[Employee]( [NationalIDNumber] ASC );
    GO
    ```

1. また、以下のクエリを実行することで、データ型が正常に変更されたことを確認することができます。

    ```sql
    SELECT c.name, t.name
    FROM sys.all_columns c INNER JOIN sys.types t
    	ON (c.system_type_id = t.user_type_id)
    WHERE OBJECT_ID('[HumanResources].[Employee]') = c.object_id
        AND c.name = 'NationalIDNumber'
    ```
    
    ![Picture 05](../images/dp-300-module-09-lab-05.png)
    
1. では、実行計画を確認してみましょう。元のクエリを引用符を付けずに再実行します。

    ```sql
    USE AdventureWorks2017
    GO

    SELECT BusinessEntityID, NationalIDNumber, LoginID, HireDate, JobTitle
    FROM HumanResources.Employee
    WHERE NationalIDNumber = 14417807;
    ```

    ![Picture 06](../images/dp-300-module-09-lab-06.png)

    クエリプランを調べて、暗黙の変換警告を出さずに、*NationalIDNumber*でフィルタリングするために整数を使用できることに注意してください。SQLクエリオプティマイザは、最適な計画を生成し実行できるようになりました。

この演習では、暗黙のデータ型変換が原因で発生するクエリの問題を特定する方法と、それを修正してクエリ・プランを改善する方法を学習しました。
