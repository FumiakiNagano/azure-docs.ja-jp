---
title: "Azure Cosmos DB: Python と Azure Cosmos DB MongoDB API による Flask Web アプリの構築 | Microsoft Docs"
description: "Azure Cosmos DB MongoDB API への接続とクエリに使用できる Python Flask コード サンプルについて説明します"
services: cosmos-db
documentationcenter: 
author: hshapiro
manager: scicoria
editor: 
ms.assetid: 
ms.service: cosmos-db
ms.custom: quick start connect, mvc
ms.workload: 
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: quickstart
ms.date: 10/2/2017
ms.author: hshapiro
ms.openlocfilehash: f86c6cce82812e02f373d7307c76ace26ea3e99b
ms.sourcegitcommit: b07d06ea51a20e32fdc61980667e801cb5db7333
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/08/2017
---
# <a name="azure-cosmos-db-build-a-flask-app-with-the-mongodb-api"></a>Azure Cosmos DB: MongoDB API による Flask アプリの構築

Azure Cosmos DB は、Microsoft のグローバルに分散されたマルチモデル データベース サービスです。 Azure Cosmos DB の中核をなすグローバルな分散と水平方向のスケール機能を利用して、ドキュメント、キー/値、およびグラフ データベースをすばやく作成およびクエリできます。

このクイック スタート ガイドでは、次の [Flask 例](https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample)を使用し、MongoDB の代わりに [Azure Cosmos DB Emulator](/local-emulator.md) を使用してシンプルな To-Do Flask アプリを構築する方法を示します。

## <a name="prerequisites"></a>前提条件

- [Azure Cosmos DB Emulator](/local-emulator.md) をダウンロードします。 エミュレーターは現在、Windows でのみサポートされています。 このサンプルでは、Azure のプロダクション キーと共にサンプルを使用する方法を示します。この操作は任意のプラットフォームで実行できます。

- Visual Studio Code をまだインストールしていない場合は、お使いのプラットフォーム (Windows、Mac、Linux) 用の [VS Code](https://code.visualstudio.com/Download) をすばやくインストールできます。

- 一般的な Python 拡張機能のいずれかをインストールして Python 言語サポートを必ず追加してください。
    1. 拡張機能を選びます。
    2. 「`ext install`」と入力して、拡張機能をコマンド パレット `Ctrl+Shift+P` にインストールします。

    このドキュメントの例では、Don Jayamanne の人気のある機能豊富な [Python 拡張機能](https://marketplace.visualstudio.com/items?itemName=donjayamanne.python)を使用します。

## <a name="clone-the-sample-application"></a>サンプル アプリケーションの複製

github から Flask-MongoDB API アプリを複製し、接続文字列を設定して実行します。 プログラムでデータを処理することが非常に簡単であることがわかります。

1. git ターミナル ウィンドウ (git bash など) を開き、`cd` を実行して作業ディレクトリに移動します。
2. 次のコマンドを実行して、サンプル レポジトリを複製します。

    ```bash
    git clone https://github.com/Azure-Samples/CosmosDB-Flask-Mongo-Sample.git
    ```
3. 次のコマンドを実行して python モジュールをインストールします。

    ```bash 
    pip install -r .\requirements.txt
    ```
4. Visual Studio Code でフォルダーを開きます。

## <a name="review-the-code"></a>コードの確認

アプリで何が行われているかを簡単に確認してみましょう。 ルート ディレクトリの下にある **app.py** ファイルを開くと、これらのコード行によって Azure Cosmos DB 接続が作成されることがわかります。 次のコードは、ローカルの Azure Cosmos DB Emulator 用の接続文字列を使用します。 パスワードを次のように分割して、スラッシュに対応する必要があります。そうしないと、スラッシュを解析できません。

* MongoDB クライアントを初期化し、データベースを取得して認証します。

    ```python
    client = MongoClient("mongodb://127.0.0.1:10250/?ssl=true") #host uri
    db = client.test    #Select the database
    db.authenticate(name="localhost",password='C2y6yDjf5' + r'/R' + '+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw' + r'/Jw==')
    ```

* コレクションを取得するか、まだ存在しない場合は作成します。

    ```python
    todos = db.todo #Select the collection
    ```

* アプリケーションの作成

    ```Python
    app = Flask(__name__)
    title = "TODO with Flask"
    heading = "ToDo Reminder"
    ```
    
## <a name="run-the-web-app"></a>Web アプリの実行

1. Azure Cosmos DB Emulator が実行されていることを確認します。

2. ターミナル ウィンドウを開き、`cd` を実行してアプリが保存されているディレクトリに移動します。

3. Mac を使用している場合は、Flask アプリの環境変数を `set FLASK_APP=app.py` または `export FLASK_APP=app.py` に設定します。

4. `flask run` を使用してアプリを実行し、[http://127.0.0.1:5000/](http://127.0.0.1:5000/) を参照します。

5. タスクを追加および削除し、それらがコレクションで追加および変更されたことを確認します。

## <a name="create-a-database-account"></a>How to create a DocumentDB account (DocumentDB アカウントの作成方法)

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount-mongodb.md)]

## <a name="update-your-connection-string"></a>接続文字列の更新

ライブ Azure Cosmos DB アカウントに対してコードをテストする場合は、Azure Portal に移動してアカウントを作成し、接続文字列情報を取得します。 これをアプリにコピーします。

1. [Azure Portal](http://portal.azure.com/) で、Azure Cosmos DB アカウントの左のナビゲーションから、**[接続文字列]** をクリックし、**[読み取り/書き込みキー]** をクリックします。 次の手順では、画面右側のコピー ボタンを使用して、ユーザー名、パスワード、ホストを Dal.cs ファイルにコピーします。

2. ルート ディレクトリ内の **app.py** ファイルを開きます。

3. (コピー ボタンを使用して) ポータルから**ユーザー名**の値をコピーし、**app.py** ファイル内の **name** の値に設定します。

4. 次に、ポータルから**接続文字列**の値をコピーし、それを **app.py** ファイル内の MongoClient の値に設定します。

5. 最後に、ポータルから**パスワード**の値をコピーし、**app.py** ファイル内の **password** の値に設定します。

これで、Azure Cosmos DB と通信するために必要なすべての情報でアプリを更新しました。 これまでと同じ方法で実行できます。

## <a name="deploy-to-azure"></a>Azure へのデプロイ

このアプリをデプロイするには、Azure で新しい Web アプリを作成し、この github リポジトリのフォークで継続的配置を有効にします。 この[チュートリアル](https://docs.microsoft.com/azure/app-service-web/app-service-continuous-deployment)に従って、Azure で Github による継続的配置を設定します。

Azure にデプロイする場合は、アプリケーション キーを削除し、以下のセクションがコメント アウトされていないことを確認する必要があります。

```python
    client = MongoClient(os.getenv("MONGOURL"))
    db = client.test    #Select the database
    db.authenticate(name=os.getenv("MONGO_USERNAME"),password=os.getenv("MONGO_PASSWORD"))
```

アプリケーション設定に MONGOURL、MONGO_PASSWORD、MONGO_USERNAME を追加する必要があります。 この[チュートリアル](https://docs.microsoft.com/azure/app-service-web/web-sites-configure#application-settings)に従って、Azure Web Apps のアプリケーション設定の詳細を確認できます。

このリポジトリのフォークを作成しない場合は、下の [Azure へのデプロイ] をクリックすることもできます。 Azure に移動し、Cosmos DB アカウント情報でアプリケーション設定を設定する必要があります。

<a href="https://deploy.azure.com/?repository=https://github.com/heatherbshapiro/To-Do-List---Flask-MongoDB-Example" target="_blank">
<img src="http://azuredeploy.net/deploybutton.png"/>
</a>

> [!NOTE]
> Github またはその他のソース管理オプションでコードを格納することを計画している場合は、必ずコードから接続文字列を削除してください。 代わりに、Web アプリのアプリケーション設定で設定することができます。

## <a name="review-slas-in-the-azure-portal"></a>Azure Portal での SLA の確認

[!INCLUDE [cosmosdb-tutorial-review-slas](../../includes/cosmos-db-tutorial-review-slas.md)]

## <a name="clean-up-resources"></a>リソースのクリーンアップ

このアプリの使用を続けない場合は、以下の手順に従い、Azure Portal でこのクイック スタートで作成したすべてのリソースを削除してください。

1. Azure Portal の左側のメニューで、**[リソース グループ]** をクリックし、作成したリソースの名前をクリックします。
2. リソース グループのページで **[削除]** をクリックし、削除するリソースの名前をテキスト ボックスに入力してから **[削除]** をクリックします。

## <a name="next-steps"></a>次のステップ

このクイックスタートでは、Azure Cosmos DB アカウントを作成し、MongoDB 用 API を使用して Flask アプリを実行する方法を学習しました。これで、Cosmos DB アカウントに追加のデータをインポートできます。

> [!div class="nextstepaction"]
> [MongoDB API 用に Azure Cosmos DB にデータをインポートする](mongodb-migrate.md)
