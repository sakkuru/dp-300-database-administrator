---
lab:
    title: 'Lab 8 – ブロッキングの問題を特定し解決する'
    module: 'Azure SQL でクエリ パフォーマンスを最適化する'
---


# ブロッキングの問題を特定し解決する

**推定時間：15 分**

受講者は、レッスンで得た情報をもとに、AdventureWorks内のデジタルトランスフォーメーションプロジェクトの成果物のスコープを設定します。Azureポータルや他のツールを調べ、ネイティブツールをどのように活用してパフォーマンス関連の問題を特定し解決するかを決定します。最後に、ブロックの問題を適切に特定し、解決できるようになります。

あなたは、パフォーマンス関連の問題を特定し、発見された問題を解決するための実行可能なソリューションを提供するデータベース管理者として採用されました。あなたは、パフォーマンスの問題を調査し、それを解決するための方法を提案する必要があります。

**注意：** これらの演習では、T-SQL コードをコピーして貼り付けることが求められます。コードを実行する前に、コードが正しくコピーされたことを確認してください。

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

## ブロックされたクエリレポートの実行

1. **新規クエリ**を選択します。以下の T-SQL コードをコピーしてクエリーウィンドウに貼り付けます。**実行**を選択し、このクエリを実行します。

    ```sql
    USE MASTER

    GO

    CREATE EVENT SESSION [Blocking] ON SERVER 
    ADD EVENT sqlserver.blocked_process_report(
    ACTION(sqlserver.client_app_name,sqlserver.client_hostname,sqlserver.database_id,sqlserver.database_name,sqlserver.nt_username,sqlserver.session_id,sqlserver.sql_text,sqlserver.username))
    ADD TARGET package0.ring_buffer
    WITH (MAX_MEMORY=4096 KB, EVENT_RETENTION_MODE=ALLOW_SINGLE_EVENT_LOSS, MAX_DISPATCH_LATENCY=30 SECONDS, MAX_EVENT_SIZE=0 KB,MEMORY_PARTITION_MODE=NONE, TRACK_CAUSALITY=OFF,STARTUP_STATE=ON)
    GO

    -- Start the event session 
    ALTER EVENT SESSION [Blocking] ON SERVER 
    STATE = start; 
    GO
    ```

    上記のT-SQLコードは、ブロッキング・イベントを捕捉する拡張イベント・セッションを作成します。データには以下の要素が含まれます。

    - クライアントアプリケーション名
    - クライアントホスト名
    - データベースID
    - データベース名
    - NTユーザー名
    - セッションID
    - T-SQL テキスト
    - ユーザー名

1. **新規クエリ**を選択します。以下のT-SQLコードをコピーして、クエリウィンドウに貼り付けます。**実行**を選択して、このクエリを実行します。

    ```sql
    EXEC sys.sp_configure N'show advanced options', 1
    RECONFIGURE WITH OVERRIDE;
    GO
    EXEC sp_configure 'blocked process threshold (s)', 60
    RECONFIGURE WITH OVERRIDE;
    GO
    ```

    **注意:** 上記のコマンドは、ブロックされたプロセスのレポートが生成される閾値を秒単位で指定します。その結果、このレッスンでは、*blocked_process_report* が発生するのを長く待つ必要はありません。

1. **新規クエリ** を選択します。以下のT-SQLコードをコピーしてクエリウィンドウに貼り付けます。**実行** を選択して、このクエリを実行します。

    ```sql
    USE AdventureWorks2017
    GO

    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;

    GO
    ```

1. **新しいクエリ**ボタンを選択して、別のクエリウィンドウを開きます。次の T-SQL コードをコピーして、新しいクエリウィンドウに貼り付けます。**実行** を選択し、このクエリを実行します。

    ```sql
    USE AdventureWorks2017
    GO

    SELECT TOP (1000) [LastName]
      ,[FirstName]
      ,[Title]
    FROM Person.Person
    WHERE FirstName = 'David'
    ```

    **注意：** このクエリは結果を返さず、無限に実行されるように見えます。

1. **オブジェクト・エクスプローラー**で、**管理→拡張イベント→セッション**を展開します。

    先ほど作成した*Blocking*という名前の拡張イベントがリストに入っていることに注意してください。

    ![Picture 01](../images/dp-300-module-08-lab-01.png)

1. **package0.ring_buffer**を右クリックし、**View Target Data**を選択します。

    ![Picture 02](../images/dp-300-module-08-lab-02.png)

1. ハイパーリンクを選択します。

    ![Picture 03](../images/dp-300-module-08-lab-03.png)

1. どのプロセスがブロックされているのか、どのプロセスがブロックを起こしているのか、XMLで表示されます。システム情報だけでなく、このプロセスで実行されたクエリも確認することができます。

    ![Picture 04](../images/dp-300-module-08-lab-04.png)

1. また、以下のクエリを実行することで、他のセッションをブロックしているセッションを特定することができます（*session_id*ごとにブロックされているセッションIDのリストも含まれます）。

    ```sql
    WITH cteBL (session_id, blocking_these) AS 
    (SELECT s.session_id, blocking_these = x.blocking_these FROM sys.dm_exec_sessions s 
    CROSS APPLY    (SELECT isnull(convert(varchar(6), er.session_id),'') + ', '  
                    FROM sys.dm_exec_requests as er
                    WHERE er.blocking_session_id = isnull(s.session_id ,0)
                    AND er.blocking_session_id <> 0
                    FOR XML PATH('') ) AS x (blocking_these)
    )
    SELECT s.session_id, blocked_by = r.blocking_session_id, bl.blocking_these
    , batch_text = t.text, input_buffer = ib.event_info, * 
    FROM sys.dm_exec_sessions s 
    LEFT OUTER JOIN sys.dm_exec_requests r on r.session_id = s.session_id
    INNER JOIN cteBL as bl on s.session_id = bl.session_id
    OUTER APPLY sys.dm_exec_sql_text (r.sql_handle) t
    OUTER APPLY sys.dm_exec_input_buffer(s.session_id, NULL) AS ib
    WHERE blocking_these is not null or r.blocking_session_id > 0
    ORDER BY len(bl.blocking_these) desc, r.blocking_session_id desc, r.session_id;
    ```

    ![Picture 05](../images/dp-300-module-08-lab-05.png)

1. 拡張イベント「**Blocking**」を右クリックし、「*Stop Session*」を選択します。

    ![Picture 06](../images/dp-300-module-08-lab-06.png)

1. ブロックの原因となっているクエリセッションに戻り、クエリの下の行に `ROLLBACK TRANSACTION` と入力します。ROLLBACK TRANSACTION`をハイライト表示し、**Execute**を選択します。

    ![Picture 07](../images/dp-300-module-08-lab-07.png)

1. ブロックされていたクエリセッションに戻ります。クエリーが完了したことがわかります。

    ![Picture 08](../images/dp-300-module-08-lab-08.png)

## リード・コミット・スナップショットの分離レベルを有効にする

1. SQL Server Management Studioから**New Query**を選択します。以下のT-SQLコードをコピーしてクエリウィンドウに貼り付けます。**実行**ボタンを選択し、このクエリを実行します。

    ```sql
    USE master
    GO
    
    ALTER DATABASE AdventureWorks2017 SET READ_COMMITTED_SNAPSHOT ON WITH ROLLBACK IMMEDIATE;
    GO
    ```

1. 新しいクエリエディタで、ブロックの原因となったクエリを再実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    BEGIN TRANSACTION
        UPDATE Person.Person 
        SET LastName = LastName;
    GO
    ```

1. 新しいクエリエディタでブロックされているクエリを再実行します。

    ```sql
    USE AdventureWorks2017
    GO
    
    SELECT TOP (1000) [LastName]
     ,[FirstName]
     ,[Title]
    FROM Person.Person
    WHERE firstname = 'David'
    ```

    ![Picture 09](../images/dp-300-module-08-lab-09.png)

    前のタスクではupdate文でブロックされたのに、なぜ同じクエリが完了するのでしょうか？

    リード・コミット・スナップショット分離レベルはトランザクション分離の楽観的な形態であり、最後のクエリはブロックされるのではなく、データの最新コミット・バージョンを表示することになります。

この演習では、ブロックされているセッションを特定する方法と、それらのシナリオを軽減する方法を学びました。
