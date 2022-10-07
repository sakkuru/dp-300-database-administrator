---
lab:
    title: 'Lab 5 – SQL とデータの分類のために Microsoft Defender を有効にする'
    module: 'データベースサービスのための安全な環境の実装'
---

# SQL とデータ分類のために Microsoft Defender を有効にする

**推定時間: 20 分**

受講者は、レッスンで得た情報をもとに、AzureポータルおよびAdventureWorksデータベース内のセキュリティを設定し、その後実装します。

あなたは、データベース環境のセキュリティを確保するためのシニアデータベース管理者として採用されました。これらのタスクは、Azure SQL Databaseに焦点を当てます。

## Microsoft Defender for SQL を有効にする

1. ラボの仮想マシンから、ブラウザ セッションを開始し、[https://portal.azure.com](https://portal.azure.com/) に移動します。このラボ仮想マシンの **Resources** タブで提供された Azure **Username** と **Password** を使用して、ポータルに接続します。

1. Azure ポータルから、上部の検索ボックスで「SQL サーバー」を検索し、オプションの一覧から **SQL サーバー** をクリックします。

    ![ソーシャルメディアへの投稿画面 説明が自動生成される](../images/dp-300-module-04-lab-1.png)

1. サーバー名 **dp300-lab-XXXXXXXX** を選択して、詳細ページを表示します（SQLサーバーには別のリソースグループと場所が割り当てられている場合があります）。

    ![ソーシャルメディアへの投稿画面 自動生成された説明文](../images/dp-300-module-04-lab-2.png)

1. Azure SQLサーバーのメインブレードから、**セキュリティ**セクションに移動し、**Microsoft Defender for Cloud**を選択します。

   ![ Microsoft Defender for Cloud オプションを選択した画面](../images/dp-300-module-05-lab-01.png)

    **Microsoft Defender for Cloud**ページで、**Enable Microsoft Defender for SQL**を選択します。

1. Azure Defender for SQL が正常に有効化されると、以下の通知メッセージが表示されます。

    ![構成オプションを選択した画面](../images/dp-300-module-05-lab-02_1.png)

1. **Microsoft Defender for Cloud** ページで、**構成** リンクを選択します (このオプションを表示するには、ページを更新する必要がある場合があります)。

    ![設定オプションを選択した画面](../images/dp-300-module-05-lab-02.png)

1. **サーバー設定** ページで、**MICROSOFT DEFENDER FOR SQL** のトグルスイッチが **ON** に設定され、**ストレージ アカウント** 名が提供されていることに注目してください。**スキャンレポートの送信先**に、Azureポータルにログインする際に使用したAzureアカウントのメールアドレスを入力し、**保存**を選択します。

    ![サーバー設定画面のスクリーンショット](../images/dp-300-module-05-lab-03.png)

## データ分類を有効にする

1. Azure SQLサーバーのメインブレードから、**設定**セクションに移動し、**SQLデータベース**を選択し、データベース名を選択します。

    ![AdobeWOrksLTデータベースを選択した画面](../images/dp-300-module-05-lab-04.png)

1. **AdventureWorksLT**データベースのメインブレードで、**セキュリティ**セクションに移動し、**データの検出と分類**を選択します。

    ![データ検索と分類を表示する画面](../images/dp-300-module-05-lab-05.png)

1. **データの検出と分類** ページで、次のような情報メッセージが表示されます。**現在、SQL 情報保護ポリシーを使用しています。分類が推奨される15個の列が見つかりました**。このリンクを選択します。

    ![分類の推奨を示すスクリーンショット](../images/dp-300-module-05-lab-06.png)

1. 次の **データの検出と分類** 画面で **すべて選択** にチェックを入れ、 **選択した推奨事項を受け入れます** を選択し、 **保存** を選択して分類をデータベースに保存します。

    ![選択されたレコメンデーションを受け入れる画面](../images/dp-300-module-05-lab-07.png)をご覧ください。

1. **データの検出と分類** 画面に戻り、5つのテーブルで15個の列が正常に分類されたことに気がつきます。

    ![この画面は、「Accept selected recommendations」の画面です](../images/dp-300-module-05-lab-08.png)

この演習では、Microsoft Defender for SQLを有効にして、Azure SQL Databaseのセキュリティを強化しました。また、Azure ポータルの推奨事項に基づいて分類されたカラムを作成しました。
