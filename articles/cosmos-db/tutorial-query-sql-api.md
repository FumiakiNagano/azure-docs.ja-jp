---
title: "Azure Cosmos DB で SQL を使用してクエリを実行する方法 | Microsoft Docs"
description: "Azure Cosmos DB で SQL を使用してクエリを実行する方法について説明します"
services: cosmos-db
documentationcenter: 
author: rafats
manager: jhubbard
editor: 
tags: 
ms.assetid: 
ms.service: cosmos-db
ms.custom: tutorial-develop, mvc
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 05/10/2017
ms.author: rafats
ms.openlocfilehash: ffef6ec2120a80d907449470efb7b4ab6dca8037
ms.sourcegitcommit: 0e4491b7fdd9ca4408d5f2d41be42a09164db775
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/14/2017
---
# <a name="azure-cosmos-db-how-to-query-using-sql"></a>Azure Cosmos DB: SQL を使用してクエリを実行する方法

[!INCLUDE [cosmos-db-sql-api](../../includes/cosmos-db-sql-api.md)]

Azure Cosmos DB [SQL API](documentdb-introduction.md) では SQL を使用したドキュメントのクエリがサポートされます。 この記事には、1 つのサンプル ドキュメントと 2 つのサンプル SQL クエリおよび結果が含まれます。

この記事に含まれるタスクは次のとおりです。 

> [!div class="checklist"]
> * SQL を使用してデータのクエリを実行する

## <a name="sample-document"></a>サンプル ドキュメント

この記事の SQL クエリは、次のサンプル ドキュメントを使用します。

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam", 
        "givenName": "Jesse", 
        "gender": "female", "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller", 
         "givenName": "Lisa", 
         "gender": "female", 
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```
## <a name="where-can-i-run-sql-queries"></a>SQL クエリを実行できるところ

クエリを実行するには、Azure Portal のデータ エクスプローラーを使用するか、[REST API および SDK](sql-api-sdk-dotnet.md) で行います。既存のサンプル データ セットに対してクエリを実行する [Query Playground](https://www.documentdb.com/sql/demo) を使用することもできます。

SQL クエリについて詳しくは、次を参照してください。
* [SQL クエリと SQL 構文](sql-api-sql-query.md)

## <a name="prerequisites"></a>前提条件

このチュートリアルでは、Azure Cosmos DB アカウントとコレクションがあると仮定しています。 どちらもない場合には、 [5 分でできるクイックスタート](create-mongodb-nodejs.md)か[開発者向けチュートリアル](tutorial-develop-mongodb.md)を実行して、アカウントとコレクションを作成します。

## <a name="example-query-1"></a>サンプル クエリ 1

上記の家族に関するサンプル ドキュメントに対して、次の SQL クエリは ID フィールドが `WakefieldFamily` と一致するドキュメントを返します。 `SELECT *` ステートメントであるため、クエリの出力は完全な JSON ドキュメントになります。

**クエリ**

    SELECT * 
    FROM Families f 
    WHERE f.id = "WakefieldFamily"

**結果**

```json
{
  "id": "WakefieldFamily",
  "parents": [
      { "familyName": "Wakefield", "givenName": "Robin" },
      { "familyName": "Miller", "givenName": "Ben" }
  ],
  "children": [
      {
        "familyName": "Merriam", 
        "givenName": "Jesse", 
        "gender": "female", "grade": 1,
        "pets": [
            { "givenName": "Goofy" },
            { "givenName": "Shadow" }
        ]
      },
      { 
        "familyName": "Miller", 
         "givenName": "Lisa", 
         "gender": "female", 
         "grade": 8 }
  ],
  "address": { "state": "NY", "county": "Manhattan", "city": "NY" },
  "creationDate": 1431620462,
  "isRegistered": false
}
```

## <a name="example-query-2"></a>サンプル クエリ 2

次のクエリでは、ID が `WakefieldFamily` と一致する家族の子供の名前がすべて学年順に返されます。

**クエリ**

    SELECT c.givenName 
    FROM Families f 
    JOIN c IN f.children 
    WHERE f.id = 'WakefieldFamily'
    ORDER BY f.children.grade ASC

**結果**

    [
      { "givenName": "Jesse" }, 
      { "givenName": "Lisa"}
    ]


## <a name="next-steps"></a>次の手順

このチュートリアルでは、次の手順を行いました。

> [!div class="checklist"]
> * SQL を使用してクエリを実行する方法を学習しました。  

次のチュートリアルに進んで、データをグローバルに分散する方法について学習できます。

> [!div class="nextstepaction"]
> [データをグローバルに分散する](tutorial-global-distribution-sql-api.md)

