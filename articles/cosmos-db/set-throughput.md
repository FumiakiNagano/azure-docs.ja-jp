---
title: "Azure Cosmos DB のスループットのプロビジョニング | Microsoft Docs"
description: "Azure Cosmos DB コンテナー、コレクション、グラフ、およびテーブルのプロビジョニング済みスループットを設定する方法について説明します。"
services: cosmos-db
author: mimig1
manager: jhubbard
editor: 
documentationcenter: 
ms.assetid: f98def7f-f012-4592-be03-f6fa185e1b1e
ms.service: cosmos-db
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/02/2018
ms.author: mimig
ms.openlocfilehash: 8797910651c54baa3529b015d4195cf2a5c06ece
ms.sourcegitcommit: 9890483687a2b28860ec179f5fd0a292cdf11d22
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/24/2018
---
# <a name="set-throughput-for-azure-cosmos-db-containers"></a>Azure Cosmos DB コンテナーのスループットの設定

Azure Portal またはクライアントの SDK を使用して、Azure Cosmos DB コンテナーのスループットを設定できます。 

次の表に、コンテナーで使用できるスループットを示します。

<table border="0" cellspacing="0" cellpadding="0">
    <tbody>
        <tr>
            <td valign="top"><p></p></td>
            <td valign="top"><p><strong>単一パーティション コンテナー</strong></p></td>
            <td valign="top"><p><strong>パーティション分割コンテナー</strong></p></td>
        </tr>
        <tr>
            <td valign="top"><p>最小スループット</p></td>
            <td valign="top"><p>400 要求ユニット/秒</p></td>
            <td valign="top"><p>1,000 要求ユニット/秒</p></td>
        </tr>
        <tr>
            <td valign="top"><p>最大スループット</p></td>
            <td valign="top"><p>10,000 要求ユニット/秒</p></td>
            <td valign="top"><p>無制限</p></td>
        </tr>
    </tbody>
</table>

## <a name="to-set-the-throughput-by-using-the-azure-portal"></a>Azure Portal を使用してスループットを設定するには

1. 新しいウィンドウで、[Azure Portal](https://portal.azure.com) を開きます。
2. 左側のバーで **[Azure Cosmos DB]** をクリックします。または、下部の **[その他のサービス]** をクリックし、**[データベース]** までスクロールして、**[Azure Cosmos DB]** をクリックします。
3. Azure Cosmos DB アカウントを選択します。
4. 新しいウィンドウで、ナビゲーション メニューの **[データ エクスプローラー]** をクリックします。
5. 新しいウィンドウで、データベースとコンテナーを展開し、**[Scale & Settings]\(スケールと設定\)** をクリックします。
6. 新しいウィンドウで、**[スループット]** ボックスに新しいスループット値を入力し、**[保存]** をクリックします。

<a id="set-throughput-sdk"></a>

## <a name="to-set-the-throughput-by-using-the-sql-api-for-net"></a>SQL API for .NET を使用してスループットを設定するには

```csharp
// Fetch the offer of the collection whose throughput needs to be updated
Offer offer = client.CreateOfferQuery()
    .Where(r => r.ResourceLink == collection.SelfLink)    
    .AsEnumerable()
    .SingleOrDefault();

// Set the throughput to the new value, for example 12,000 request units per second
offer = new OfferV2(offer, 12000);

// Now persist these changes to the collection by replacing the original offer resource
await client.ReplaceOfferAsync(offer);
```

<a id="set-throughput-java"></a>

## <a name="to-set-the-throughput-by-using-the-sql-api-for-java"></a>SQL API for Java を使用してスループットを設定するには

このスニペットは、[azure-documentdb-java](https://github.com/Azure/azure-documentdb-java/blob/master/documentdb-examples/src/test/java/com/microsoft/azure/documentdb/examples/OfferCrudSamples.java) リポジトリの OfferCrudSamples.java ファイルに含まれています。 

```Java
// find offer associated with this collection
Iterator < Offer > it = client.queryOffers(
    String.format("SELECT * FROM r where r.offerResourceId = '%s'", collectionResourceId), null).getQueryIterator();
assertThat(it.hasNext(), equalTo(true));

Offer offer = it.next();
assertThat(offer.getString("offerResourceId"), equalTo(collectionResourceId));
assertThat(offer.getContent().getInt("offerThroughput"), equalTo(throughput));

// update the offer
int newThroughput = 10300;
offer.getContent().put("offerThroughput", newThroughput);
client.replaceOffer(offer);
```

## <a name="throughput-faq"></a>スループットについてよく寄せられる質問

**スループットを 400 RU/秒未満に設定することはできますか?**

Cosmos DB のシングル パーティション コンテナーで使用できる最小スループットは 400 RU/秒です (パーティション分割コンテナーの最小スループットは 1,000 RU/秒)。 要求ユニットは 100 RU/秒の間隔で設定されますが、スループットを 100 RU/秒または 400 RU/秒未満の値に設定することはできません。 コストを抑えて Cosmos DB の開発とテストを行いたい場合は、無料の [Azure Cosmos DB Emulator](local-emulator.md) を利用して無償でローカルにデプロイすることができます。 

**MongoDB API を使用してスループットを設定する方法**

スループットを設定するための MongoDB API 拡張機能はありません。 「[SQL API for .NET を使用してスループットを設定するには](#set-throughput-sdk)」で示すように、SQL API を使用することをお勧めします。

## <a name="next-steps"></a>次の手順

Cosmos DB を使用したプロビジョニングと地球規模での使用の詳細については、[Cosmos DB でのパーティション分割とスケーリング](partition-data.md)に関するページをご覧ください。
