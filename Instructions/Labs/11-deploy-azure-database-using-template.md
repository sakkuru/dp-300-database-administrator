---
lab:
    title: 'Lab 11 – Azure Resource Manager テンプレートを使用した Azure SQL Database のデプロイ'
    module: 'Azure SQL のデータベース タスクを自動化する'
---

# テンプレートから Azure SQL Database をデプロイする

**見積もり時間: 15 分**

あなたは、データベース管理の日常業務を自動化するために、シニアデータエンジニアとして採用されました。この自動化は、AdventureWorksのデータベースが最高のパフォーマンスで動作し続けることを保証し、特定の基準に基づいて警告するための方法を提供するためのものです。AdventureWorksは、Infrastructure as a Service (IaaS) とPlatform as a Service (PaaS) の両方のオファリングでSQL Serverを使用しています。

## Azure Resource Manager テンプレートの探索

1. Microsoft Edge で新しいタブを開き、GitHub リポジトリの次のパスに移動します。このパスには、SQL Database リソースをデプロイするための ARM テンプレートが含まれています。

    ```
    https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.sql/sql-database
    ```

1. **azuredeploy.json** を右クリックし、**Open link in new tab** を選択すると、以下のような ARM テンプレートが表示されるはずです。


    ```JSON
    {
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "serverName": {
        "type": "string",
        "defaultValue": "[uniqueString('sql', resourceGroup().id)]",
        "metadata": {
            "description": "The name of the SQL logical server."
        }
        },
        "sqlDBName": {
        "type": "string",
        "defaultValue": "SampleDB",
        "metadata": {
            "description": "The name of the SQL Database."
        }
        },
        "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
            "description": "Location for all resources."
        }
        },
        "administratorLogin": {
        "type": "string",
        "metadata": {
            "description": "The administrator username of the SQL logical server."
        }
        },
        "administratorLoginPassword": {
        "type": "securestring",
        "metadata": {
            "description": "The administrator password of the SQL logical server."
        }
        }
    },
    "variables": {},
    "resources": [
        {
        "type": "Microsoft.Sql/servers",
        "apiVersion": "2020-02-02-preview",
        "name": "[parameters('serverName')]",
        "location": "[parameters('location')]",
        "properties": {
            "administratorLogin": "[parameters('administratorLogin')]",
            "administratorLoginPassword": "[parameters('administratorLoginPassword')]"
        },
        "resources": [
            {
            "type": "databases",
            "apiVersion": "2020-08-01-preview",
            "name": "[parameters('sqlDBName')]",
            "location": "[parameters('location')]",
            "sku": {
                "name": "Standard",
                "tier": "Standard"
            },
            "dependsOn": [
                "[resourceId('Microsoft.Sql/servers', concat(parameters('serverName')))]"
            ]
            }
        ]
        }
    ]
    }
    ```

1. JSONのプロパティを確認し、観察する。

1. **azuredeploy.json** タブを閉じ、**sql-database** GitHub フォルダを含むタブに戻ります。スクロールダウンして、**Deploy to Azure**を選択します。

    ![Azureにデプロイボタン](../images/dp-300-module-11-lab-01.png)

1. Azureポータルに**Create a SQL Server and Database** quickstart templateページが開き、リソースの詳細がARMテンプレートから部分的に入力されます。空欄に以下の情報を入力してください。

    - **リソースグループ：** *contoso-rg*で始まる。
    - **Sql 管理者ログイン:** labadmin
    - **SQL管理者ログインパスワード:** &lt;enter a strong password&gt;

1. **確認と作成** を選択し、次に **作成** を選択します。5分ほどでデプロイが完了します。

    ![Picture 2](../images/dp-300-module-11-lab-02.png)

1. デプロイが完了したら、**リソースグループへ移動** を選択します。Azure Resource Group に移動し、デプロイで作成されたランダムな名前の **SQL Server** リソースが含まれています。

    ![Picture 3](../images/dp-300-module-11-lab-03.png)

Azure Resource Manager のテンプレートリンクを 1 回クリックするだけで、Azure SQL サーバーとデータベースの両方を簡単に作成できることがお分かりいただけたかと思います。
