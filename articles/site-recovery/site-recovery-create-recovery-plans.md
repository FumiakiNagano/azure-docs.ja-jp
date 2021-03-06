---
title: "Azure Site Recovery でフェールオーバーと復旧用の復旧計画を作成してカスタマイズする | Microsoft Docs"
description: "Azure Site Recovery で復旧計画を作成してカスタマイズし、VM と物理サーバーをフェールオーバーおよび復旧する方法について説明します"
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: 72408c62-fcb6-4ee2-8ff5-cab1218773f2
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 01/26/2018
ms.author: raynew
ms.openlocfilehash: 9839a989246b28c1a194b8d1f0e99c1bd80ac2e5
ms.sourcegitcommit: ded74961ef7d1df2ef8ffbcd13eeea0f4aaa3219
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/29/2018
---
# <a name="create-recovery-plans"></a>復旧計画の作成


この記事では、[Azure Site Recovery](site-recovery-overview.md) で復旧計画を作成およびカスタマイズするための情報を提供します。

コメントや質問はこの記事の末尾、または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)で投稿してください。

 復旧計画を作成して、次のことを行います。

* 一緒にフェールオーバーし、一緒に起動するマシンのグループを定義します。
* マシンを復旧計画グループにまとめることにより、マシン間の依存関係をモデル化します。 たとえば、特定のアプリケーションをフェールオーバーおよび起動するには、そのアプリケーションのすべての VM を同じ復旧計画グループにまとめます。
* フェールオーバーを実行します。 復旧計画で、テスト フェールオーバー、計画されたフェールオーバー、または計画されていないフェールオーバーを実行できます。

## <a name="why-use-recovery-plans"></a>復旧計画を使用する理由

復旧計画は、管理可能な小規模の独立ユニットを作成して体系的な復旧プロセスを実現するうえで役立ちます。 こうしたユニットは、通常、環境内のアプリケーションを表します。 復旧計画を使用すると、仮想マシンを開始する順番を定義できるだけでなく、復旧時の一般的なタスクの自動化にも役立ちます。


**基本的に、クラウド移行またはディザスター リカバリーの準備ができているかどうかをチェックするには、1 つの方法として、必ずご自身のすべてのアプリケーションを復旧計画に含め、Microsoft Azure への復旧について、それぞれの復旧計画を確実にテストします。このように準備することで、データセンター全体を、Microsoft Azure に自信をもって移行またはフェールオーバーできます。**
 
復旧計画に関する 3 つの重要な価値提案を次に示します。

### <a name="model-an-application-to-capture-dependencies"></a>依存関係をキャプチャするようにアプリケーションをモデル化する

復旧計画では、通常、アプリケーションを構成する仮想マシンのグループがまとめてフェールオーバーされます。 復旧計画コンストラクトを使用すると、このグループを強化して、アプリケーション固有のプロパティをキャプチャできます。
 
次を含む一般的な 3 層アプリケーションの例を見てみましょう

* 1 つの SQL バックエンド
* 1 つのミドルウェア
* 1 つの Web フロント エンド

復旧計画は、フェールオーバー後に仮想マシンが正しい順序で起動するようカスタマイズできます。 最初に起動するのは SQL バックエンド、次はミドルウェア、最後は Web フロントエンドです。 この順序により、最後の仮想マシンが起動するまでに、アプリケーションが確実に動作します。 たとえば、ミドルウェアが起動するとき、SQL 層に接続しようとするので、復旧計画によって SQL 層が既に実行されていることが保証されます。 フロントエンド サーバーを最後に起動することで、すべてのコンポーネントが稼働し、アプリケーションで要求を受け入れる準備ができるまで、エンド ユーザーがアプリケーション URL に誤って接続することがなくなります。 こうした依存関係を構築するために、復旧計画をカスタマイズして、グループを追加できます。 その後、仮想マシンを選択し、そのグループを変更して、グループ間で移動できるようにします。

![復旧計画の例](./media/site-recovery-create-recovery-plans/rp.png)

カスタマイズが完了すると、正確な復旧手順を視覚化できます。 復旧計画のフェールオーバーで実行される手順を次に示します。

* 最初は、オンプレミスの仮想マシンをオフにするシャットダウン手順です (プライマリ サイトを実行し続ける必要があるテスト フェールオーバーを除く)
* 次に、復旧計画に含まれる仮想マシンすべてのフェールオーバーが同時にトリガーされます。 フェールオーバーの手順では、レプリケートされたデータから仮想マシンのディスクが準備されます。
* 最後に、スタートアップ グループが順番に実行され、各グループの仮想マシンが起動します。最初に起動するのは、グループ 1、次にグループ 2、最後がグループ 3 です。 グループに複数の仮想マシンが含まれる場合 (負荷分散された Web フロントエンドなど)、すべての仮想マシンが同時に起動されます。

**グループ間でのシーケンス処理により、さまざまなアプリケーション層の間の依存関係が受け入れられ、適切な場合は並列処理が行われてアプリケーション復旧の RTO が短縮されます。**

   > [!NOTE]
   > 1 つのグループに含まれるマシンは同時にフェールオーバーされます。 異なるグループに含まれるマシンは、グループの順番に従ってフェールオーバーされます。 グループ 1 のすべてのマシンのフェールオーバーと起動が完了しないと、グループ 2 のマシンのフェールオーバーは開始されません。

### <a name="automate-most-recovery-tasks-to-reduce-rto"></a>大部分の復旧タスクを自動化して RTO を短縮する

大規模なアプリケーションの復旧は複雑になる可能性があります。 また、フェールオーバー後や移行後の正確なカスタマイズ手順を覚えるのも大変です。 アプリケーションの複雑さをわかっていない他の人が、フェールオーバーをトリガーしなければならないこともあります。 混乱中に煩雑な手動の手順を覚えるのは大変です。エラーも発生しやすくなります。 復旧計画では、すべての手順で実行する必要があるアクションを自動化する方法を、Microsoft Azure Automation Runbook を使用して提供します。 Runbook を使用すると、次の例のような一般的な復旧タスクを自動化できます。 復旧計画には、自動化できないタスク向けに手動アクションを挿入する機能も用意されています。

* フェールオーバー後の Azure 仮想マシンでのタスク: 仮想マシンに接続するために通常必要なタスクです。次に例を示します。
    * フェールオーバー後の仮想マシンでのパブリック IP の作成
    * フェールオーバーされた仮想マシンの NIC への NSG の割り当て
    * 可用性セットへのロード バランサーの追加
* フェールオーバー後の仮想マシン内のタスク: アプリケーションを再構成して、新しい環境で適切に動作し続けるようにします。次に例を示します。
    * 仮想マシン内のデータベース接続文字列の変更
    * サーバーの構成/ルールの変更

**Automation Runbook を使用して復旧後のタスクを自動化する完全な復旧計画により、ワンクリック フェールオーバーを実現し、RTO を最適化できます。**

### <a name="test-failover-to-be-ready-for-a-disaster"></a>テスト フェールオーバーで災害に備える

復旧計画を使用して、フェールオーバーまたはテスト フェールオーバーの両方をトリガーできます。 フェールオーバーを実行する前に、必ずアプリケーションでテスト フェールオーバーを完了する必要があります。 テスト フェールオーバーは、アプリケーションが復旧サイトで利用可能になるかどうかを確認するうえで役立ちます。  何かが足りなかった場合は、簡単にクリーンアップをトリガーして、テスト フェールオーバーを再実行できます。 テスト フェールオーバーは、アプリケーションがスムーズに復旧することが確信できるまで複数回実行します。

![復旧計画のテスト](./media/site-recovery-create-recovery-plans/rptest.png)

**アプリケーションはそれぞれ異なるため、それぞれに合わせてカスタマイズされた復旧計画を作成する必要があります。また、現在の動的なデータ センター環境では、アプリケーションとその依存関係は常に変化しています。四半期に 1 回はアプリケーションのテスト フェールオーバーを行い、復旧計画が最新であることを確認してください。**

## <a name="how-to-create-a-recovery-plan"></a>復旧計画を作成する方法

1. **[復旧計画]** > **[復旧計画の作成]** の順にクリックします。
   復旧計画の名前、およびソースとターゲットを指定します。 ソースの場所には、フェールオーバーと復旧が有効になった仮想マシンが必要になります。 復旧計画に含める仮想マシンに基づいてソースとターゲットを選択します。 

   |シナリオ                   |ソース               |ターゲット           |
   |---------------------------|---------------------|-----------------|
   |Azure から Azure             |Azure リージョン         |Azure リージョン     |
   |VMware から Azure            |構成サーバー |Azure            |
   |VMM から Azure               |VMM フレンドリ名    |Azure            |
   |Hyper-V サイトから Azure      |Hyper-V サイト名    |Azure            |
   |物理マシンから Azure |構成サーバー |Azure            |
   |VMM から VMM                 |VMM フレンドリ名    |VMM フレンドリ名|

   > [!NOTE]
   > 復旧計画には、ソースとターゲットが同一の仮想マシンを含めることができます。 VMware と VMM の仮想マシンを、同じ復旧計画に含めることはできません。 ただし、VMware 仮想マシンと物理マシンは同じ計画に追加できます。これは、その両方のソースが構成サーバーであるためです。

2. **[仮想マシンの選択]** で、復旧計画の既定のグループ (グループ 1) に追加する必要のある仮想マシン (またはレプリケーション グループ) を選択します。 (復旧計画で選択されているように) ソースで保護されていた仮想マシンで、(復旧計画で選択されているように) ターゲットに対して保護されているものだけを選択できます。

## <a name="how-to-customize-and-extend-recovery-plans"></a>復旧計画をカスタマイズおよび拡張する方法

復旧計画をカスタマイズおよび拡張するには、Site Recovery 復旧計画リソース ブレードに移動し、[カスタマイズ] タブをクリックします。

復旧計画は、次のようにカスタマイズおよび拡張することができます。

- **新しいグループを追加する** — 復旧計画グループ (最大 7 つ) を既定のグループに追加し、これらの復旧計画グループにマシンまたはレプリケーション グループをさらに追加します。 グループには、追加順を示す番号が割り当てられます。 仮想マシンまたはレプリケーション グループは、1 つの復旧計画グループのみに含めることができます。
- **手動アクションを追加する**— 復旧計画グループの前後に実行する手動アクションを追加できます。 復旧計画を実行すると、手動アクションを挿入した位置で停止します。 ダイアログ ボックスで、手動アクションが完了したことを指定するように求められます。
- **スクリプトを追加する** — 復旧計画グループの前後に実行されるスクリプトを追加できます。 スクリプトを追加すると、グループに対して新しい一連のアクションが追加されます。 たとえば、グループ 1 の前処理ステップ セットが "グループ 1: 前処理ステップ" という名前で作成されます。 すべての前処理ステップが、このセット内に一覧表示されます。 VMM サーバーがデプロイされている場合は、プライマリ サイトのみにスクリプトを追加できます。 [詳細情報](site-recovery-how-to-add-vmmscript.md)。
- **Azure Runbook を追加する** — Azure Runbook で復旧計画を拡張することができます。 たとえば、タスクを自動化したり、シングル ステップ復旧を作成したりします。 [詳細情報](site-recovery-runbook-automation.md)。


## <a name="how-to-add-a-script-runbook-or-manual-action-to-a-plan"></a>スクリプト、Runbook、または手動アクションを計画に追加する方法

VM またはレプリケーション グループを既定の復旧計画グループに追加して、計画を作成した後、スクリプトまたは手動アクションを追加できます。

1. 復旧計画を開きます。
2. **[ステップ]** リストの項目をクリックし、**[スクリプト]** または **[手動アクション]** をクリックします。
3. 選択した項目の前または後のどちらにスクリプトまたはアクションを追加するかを指定します。 スクリプトの位置を上下に移動するには、**[上へ移動]** と **[下へ移動]** を使用します。
4. VMM スクリプトを追加する場合は、**[Failover to VMM script (VMM へのフェールオーバー スクリプト)]** を選択します。 **[スクリプト パス]** で、共有への相対パスを入力します。 次の VMM の例では、**\RPScripts\RPScript.PS1** というパスを指定します。
5. Azure Automation Runbook を追加する場合は、Runbook が配置されている Azure Automation アカウントを指定し、適切な Azure Runbook スクリプトを選択します。
6. スクリプトが想定どおりに動作することを確認するには、復旧計画のフェールオーバーを実行します。

スクリプトか Runbook かのオプションは、フェールオーバーまたはフェールバックを行うときに、次のシナリオにおいてのみ使用できます。 手動アクションは、フェールオーバーとフェールバックの両方で使用できます。


|シナリオ               |フェールオーバー |フェールバック |
|-----------------------|---------|---------|
|Azure から Azure         |Runbooks |Runbook  |
|VMware から Azure        |Runbooks |該当なし       | 
|VMM から Azure           |Runbooks |スクリプト   |
|Hyper-V サイトから Azure  |Runbooks |該当なし       |
|VMM から VMM             |スクリプト   |スクリプト   |


## <a name="next-steps"></a>次の手順

フェールオーバーの実行については、[こちら](site-recovery-failover.md)を参照してください。

このビデオで実行中の復旧計画をご確認ください。

> [!VIDEO https://channel9.msdn.com/Series/Azure-Site-Recovery/One-click-failover-of-a-2-tier-WordPress-application-using-Azure-Site-Recovery/player]
