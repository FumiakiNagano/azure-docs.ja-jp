---
title: "Windows VM のクラシック デプロイメントのトラブルシューティング | Microsoft Docs"
description: "Azure での新しい Windows 仮想マシンの作成に関するクラシック デプロイメントの問題のトラブルシューティング"
services: virtual-machines-windows
documentationcenter: 
author: JiangChen79
manager: felixwu
editor: 
tags: top-support-issue
ms.assetid: 9f01d237-ba39-4c32-b72d-18f5f505d43a
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: troubleshooting
ms.date: 11/03/2016
ms.author: cjiang
ms.openlocfilehash: 5a1ad33bd8c287910458d26a96858e85c689fb16
ms.sourcegitcommit: adf6a4c89364394931c1d29e4057a50799c90fc0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/09/2017
---
# <a name="troubleshoot-classic-deployment-issues-with-creating-a-new-windows-virtual-machine-in-azure"></a>Azure での新しい Windows 仮想マシンの作成に関するクラシック デプロイメントの問題のトラブルシューティング
[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-selectors](../../../../includes/virtual-machines-windows-troubleshoot-deployment-new-vm-selectors-include.md)]

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-opening](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-opening-include.md)]

> [!IMPORTANT] 
> Azure には、リソースの作成と操作に関して、 [Resource Manager とクラシック](../../../resource-manager-deployment-model.md)の 2 種類のデプロイメント モデルがあります。 この記事では、クラシック デプロイ モデルの使用方法について説明します。 最新のデプロイでは、リソース マネージャー モデルを使用することをお勧めします。 この記事の Resource Manager バージョンについては、[こちら](../../virtual-machines-windows-troubleshoot-deployment-new-vm.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)をご覧ください。
> [!INCLUDE [virtual-machines-common-classic-createportal](../../../../includes/virtual-machines-classic-portal.md)]

[!INCLUDE [support-disclaimer](../../../../includes/support-disclaimer.md)]

## <a name="collect-audit-logs"></a>監査ログの収集
トラブルシューティングを開始するには、監査ログを収集して問題に関連するエラーを特定します。

Azure ポータルで、**[参照]** > **[仮想マシン]** > *対象の Windows 仮想マシン* > **[設定]** > **[監査ログ]** をクリックします。

[!INCLUDE [virtual-machines-troubleshoot-deployment-new-vm-issue1](../../../../includes/virtual-machines-troubleshoot-deployment-new-vm-issue1-include.md)]

[!INCLUDE [virtual-machines-windows-troubleshoot-deployment-new-vm-table](../../../../includes/virtual-machines-windows-troubleshoot-deployment-new-vm-table.md)]

**Y:** OS が一般化された Windows であり、一般化された設定でアップロード/キャプチャされた場合、エラーは発生しません。 同様に、OS が特殊化された Windows であり、特殊化された設定でアップロード/キャプチャされた場合、エラーは発生しません。

**アップロード エラー:**

**N<sup>1</sup>:** OS が一般化された Windows であり、特殊化された Windows としてアップロードされた場合、OOBE 画面の VM スタックでプロビジョニング タイムアウト エラーが発生します。

**N<sup>2</sup>:** OS が特殊化された Windows であり、一般化された Windows としてアップロードされた場合、新しい VM は元のコンピューター名、ユーザー名、パスワードを使用して実行されるため、OOBE 画面の VM スタックでプロビジョニング失敗エラーが発生します。

**解決策:**

これらのエラーを解決するには、OS と同じ設定 (一般化/特殊化) を使用して、オンプレミスで使用可能な元の VHD をアップロードします。 一般化された OS としてアップロードするには、まず sysprep を必ず実行してください。 詳細については、「 [Windows Server VHD の作成と Azure へのアップロード](createupload-vhd.md) 」を参照してください。

**キャプチャ エラー:**

**N<sup>3</sup>:** OS が一般化された Windows であり、特殊化された Windows としてキャプチャされた場合、一般化としてマークされた元の VM を使用できないため、プロビジョニング タイムアウト エラーが発生します。

**N<sup>4</sup>:** OS が特殊化された Windows であり、一般化された Windows としてキャプチャされた場合、新しい VM は元のコンピューター名、ユーザー名、パスワードを使用して実行されるため、プロビジョニング失敗エラーが発生します。 また、元の VM は特殊化としてマークされているので使用できません。

**解決策:**

これらのエラーを解決するには、ポータルから現在のイメージを削除し、OS と同じ設定 (一般化/特殊化) で [現在の VHD からイメージをキャプチャし直します](capture-image.md) 。

## <a name="issue-custom-gallery-marketplace-image-allocation-failure"></a>問題: カスタム/ギャラリー/Marketplace イメージ - 割り当てエラー
このエラーは、新しい VM 要求が、要求に対応するのに十分な空き領域がないか、要求されている VM サイズをサポートできないクラスターに新しい VM 要求が送信された場合に発生します。 同じクラウド サービスに別シリーズの VM を混在させることはできません。 したがって、クラウド サービスでサポートできるのとは異なるサイズの新しい VM を作成したい場合は、コンピューティング要求は失敗します。

新しい VM の作成に使用したクラウド サービスの制約により、次の 2 つのいずれかの状況が原因のエラーが発生します。

**原因 1:** クラウド サービスが特定のクラスターに固定されているか、アフィニティ グループにリンクされているので、仕様により特定のクラスターに固定されています。 したがって、そのアフィニティ グループでの新しいコンピューティング リソース要求は、既存のリソースがホストされているのと同じクラスターで試行されます。 ただし、同じクラスターが要求された VM のサイズをサポートしないか、十分な空き領域がない場合があり、割り当てエラーが発生します。 新しいリソースが新しいクラウド サービスを通じて作成された場合でも、既存のクラウド サービスを通じて作成された場合でも、そのようになります。

**解決策 1:**

* 新しいクラウド サービスを作成し、リージョンまたはリージョン ベースの仮想ネットワークに関連付けます。
* 新しいクラウド サービスに新しい VM を作成します。
  新しいクラウド サービスを作成しようとしたときにエラーが発生した場合は、後で再試行するか、クラウド サービスのリージョンを変更します。

> [!IMPORTANT]
> 既存のクラウド サービスに新しい VM を作成しようとしたができず、新しい VM 用に新しいクラウド サービスを作成する必要がある場合、同じクラウド サービスにすべての VM を統合するよう選択することもできます。 これを実行するには、既存のクラウド サービス内の VM を削除し、新しいクラウド サービスのディスクからそれらを再キャプチャします。 ただし、新しいクラウド サービスには新しい名前と VIP が割り当てられるので、既存のクラウド サービスでこれらの情報を現在使用しているすべての依存関係について、情報を更新する必要があります。
> 
> 

**原因 2:** クラウド サービスは、アフィニティ グループにリンクされている仮想ネットワークに関連付けられているため、仕様により、特定のクラスターにピン設定されています。 したがって、そのアフィニティ グループでのすべての新しいコンピューティング リソース要求は、既存のリソースがホストされているのと同じクラスターで試行されます。 ただし、同じクラスターが要求された VM のサイズをサポートしないか、十分な空き領域がない場合があり、割り当てエラーが発生します。 新しいリソースが新しいクラウド サービスを通じて作成された場合でも、既存のクラウド サービスを通じて作成された場合でも、そのようになります。

**解決策 2:**

* 新しいリージョン仮想ネットワークを作成します。
* 新しい仮想ネットワークに、新しい VM を作成します。
* [、既存の仮想ネットワークを接続](https://azure.microsoft.com/blog/vnet-to-vnet-connecting-virtual-networks-in-azure-across-different-regions/) します。 詳細については、「 [リージョン仮想ネットワーク](https://azure.microsoft.com/blog/2014/05/14/regional-virtual-networks/)」を参照してください。 または、 [アフィニティ グループ ベースの仮想ネットワークをリージョン仮想ネットワークに移行](https://azure.microsoft.com/blog/2014/11/26/migrating-existing-services-to-regional-scope/)し、その後に新しい VM を作成します。

## <a name="next-steps"></a>次のステップ
Azure での停止していた Windows VM の再起動または既存の Windows VM のサイズ変更に問題が発生する場合は、 [Azure での既存の Windows 仮想マシンの再起動またはサイズ変更に関するクラシック デプロイメントの問題のトラブルシューティング](virtual-machines-windows-classic-restart-resize-error-troubleshooting.md)を参照してください。

