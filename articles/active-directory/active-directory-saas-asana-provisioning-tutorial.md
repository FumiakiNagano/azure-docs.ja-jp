---
title: "チュートリアル: Asana を構成し、Azure Active Directory を使用した自動ユーザー プロビジョニングに対応させる | Microsoft Docs"
description: "Azure Active Directory を構成して、ユーザー アカウントを Asana に自動的にプロビジョニング/プロビジョニング解除する方法を説明します。"
services: active-directory
documentationcenter: 
author: asmalser-msft
writer: asmalser-msft
manager: sakula
ms.assetid: 0b38ee73-168b-42cb-bd8b-9c5e5126d648
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/26/2018
ms.author: asmalser
ms.reviewer: asmalser
ms.openlocfilehash: d72b5aa352a9c6724e801c1741969525387f6b90
ms.sourcegitcommit: e19742f674fcce0fd1b732e70679e444c7dfa729
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2018
---
# <a name="tutorial-configure-asana-for-automatic-user-provisioning"></a>チュートリアル: Asana を構成し、自動ユーザー プロビジョニングに対応させる

このチュートリアルでは、Azure Active Directory (Azure AD) から Asana にユーザー アカウントを自動的にプロビジョニング/プロビジョニング解除するうえで Asana と Azure AD で実行する必要がある手順について説明します。

## <a name="prerequisites"></a>前提条件

このチュートリアルで説明するシナリオでは、次の項目があることを前提としています。

*   Azure AD テナント
*   [Enterprise](https://www.asana.com/pricing) プラン以上の有効な Asana テナント 
*   Admin アクセス許可がある Asana のユーザー アカウント 

> [!NOTE] 
> Azure AD プロビジョニング統合では、Asana で使用できる [Asana API](https://app.asana.com/api/1.0/scim/Users) が必要です。

## <a name="assign-users-to-asana"></a>ユーザーを Asana に割り当てる

Azure AD では、選択されたアプリへのアクセスが付与されるユーザーを決定する際に "割り当て" という概念が使用されます。 自動ユーザー アカウント プロビジョニングのコンテキストでは、Azure AD 内のアプリケーションに割り当て済みのユーザーのみが同期されます。 

プロビジョニング サービスを構成して有効にする前に、Asana アプリにアクセスする必要がある Azure AD 内のユーザーを決定しておく必要があります。 その後、次の手順でこれらのユーザーを Asana アプリに割り当てることができます。

[エンタープライズ アプリケーションにユーザーを割り当てる](active-directory-coreapps-assign-user-azure-portal.md)

### <a name="important-tips-for-assigning-users-to-asana"></a>ユーザーを Asana に割り当てる際の重要なヒント

Asana には、Azure AD ユーザーを 1 人だけ割り当てて、プロビジョニングの構成をテストすることをお勧めします。 その他のユーザーは後で割り当てることができます。

## <a name="configure-user-provisioning-to-asana"></a>Asana へのユーザー プロビジョニングの構成 

このセクションでは、Asana のユーザー アカウント プロビジョニング API に Azure AD を接続する手順を説明します。 Azure AD でのユーザーの割り当てに基づいて、Asana での割り当て済みユーザー アカウントの作成、更新、および無効化を行うプロビジョニング サービスを構成することもできます。

> [!TIP]
> Asana で SAML ベースのシングル サインオンを有効にするには、[Azure Portal](https://portal.azure.com) で説明されている手順に従ってください。 シングル サインオンは自動プロビジョニングとは別に構成できますが、これらの 2 つの機能は相補的な関係にあります。

### <a name="to-configure-automatic-user-account-provisioning-to-asana-in-azure-ad"></a>Azure AD で Asana への自動ユーザー アカウント プロビジョニングを構成するには

1. [Azure Portal](https://portal.azure.com) で、**[Azure Active Directory]** > **[エンタープライズ アプリ]** > **[すべてのアプリケーション]** セクションの順に移動します。

2. シングル サインオンのために Asana を既に構成している場合は、検索フィールドで Asana のインスタンスを検索します。 構成していない場合は、**[追加]** を選択してアプリケーション ギャラリーで **Asana** を検索します。 検索結果から **Asana** を選択してアプリケーションの一覧に追加します。

3. Asana のインスタンスを選択してから、**[プロビジョニング]** タブを選択します。

4. **[プロビジョニング モード]** を **[自動]** に設定します。

    ![Asana のプロビジョニング](./media/active-directory-saas-asana-provisioning-tutorial/asanaazureprovisioning.png)

5. **[管理者資格情報]** セクションで以下の手順に従ってトークンを生成し、**[シークレット トークン]** に入力します。

    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが RightScale アプリケーションへのサインオンに使用する URL を入力します。 管理者アカウントを使用して [Asana](https://app.asana.com) にサインインします。

    b. 上部のバーのプロフィール写真を選択し、現在の組織名の設定を選択します。

    c. **[Service Accounts]\(サービス アカウント\)** タブに移動します。

    d. **[Add Service Account]\(サービス アカウントの追加\)** を選択します。

    e. **[Name]\(名前\)**、**[About]\(詳細\)**、プロフィール写真を必要に応じて更新します。 **[Token]\(トークン\)** からトークンをコピーし、**[Save Changes]\(変更の保存\)** でそれを選択します。

6. Azure Portal で、**[テスト接続]** を選択して Azure AD が Asana アプリに接続できることを確認します。 接続が失敗した場合、使用中の Asana アカウントに Admin アクセス許可があることを確認して、**[テスト接続]** の手順をもう一度試してください。

7. プロビジョニングのエラー通知を受け取るユーザーまたはグループのメール アドレスを **[通知用メール]** に入力し、 その下のチェック ボックスをオンにします。

8. **[保存]** を選択します。 

9. **[マッピング]** セクションの **[Synchronize Azure Active Directory Users to Asana]\(Azure Active Directory ユーザーを Asana に同期する\)** を選択します。

10. **[属性マッピング]** セクションで、Azure AD から Asana に同期されるユーザー属性を確認します。 **[Matching]\(照合\)** プロパティとして選択されている属性は、更新処理で Asana のユーザー アカウントとの照合に使用されます。 すべての変更をコミットするには、**[保存]** を選択します。 詳細については、[ユーザー プロビジョニング属性マッピングのカスタマイズ](active-directory-saas-customizing-attribute-mappings.md)に関するページを参照してください。

11. Asana に対して Azure AD プロビジョニング サービスを有効にするには、**[設定]** セクションで **[プロビジョニング状態]** を **[オン]** に変更します。

12. **[保存]** を選択します。 

**[ユーザー]** セクションで Asana に割り当てたユーザーの初期同期が開始されます。 初期同期は後続の同期よりも実行に時間がかかります。後続の同期は、サービスが実行されている限り約 20 分ごとに実行されます。 **[同期の詳細]** セクションを使用して、進行状況を監視できるほか、リンクをクリックしてプロビジョニング アクティビティ レポートを取得できます。 これらのレポートには、Asana アプリでプロビジョニング サービスによって実行されたすべてのアクションが記載されています。

Azure AD プロビジョニング ログの読み取りの詳細については、「[自動ユーザー アカウント プロビジョニングについてのレポート](active-directory-saas-provisioning-reporting.md)」をご覧ください。

## <a name="additional-resources"></a>その他のリソース

* [エンタープライズ アプリのユーザー アカウント プロビジョニングの管理](active-directory-enterprise-apps-manage-provisioning.md)
* [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](active-directory-appssoaccess-whatis.md)
* [シングル サインオンの構成](active-directory-saas-asana-tutorial.md)
