---
title: 'クイックスタート: 認可コードを使用して JavaScript Angular シングルページ アプリ (SPA) でユーザーをサインインさせて Microsoft Graph を呼び出す | Azure'
titleSuffix: Microsoft identity platform
description: このクイックスタートでは、JavaScript Angular シングルページ アプリケーション (SPA) で認可コード フローを使用して、個人アカウント、職場アカウント、学校アカウントのユーザーをサインインさせ、Microsoft Graph を呼び出す方法について説明します。
services: active-directory
author: j-mantu
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: portal
ms.workload: identity
ms.date: 11/12/2021
ROBOTS: NOINDEX
ms.author: jamesmantu
ms.custom: aaddev, "scenarios:getting-started", "languages:JavaScript", devx-track-js, mode-other
ms.openlocfilehash: 784e3cdd85c3cbc0db089a3f2617975cc8077b5f
ms.sourcegitcommit: 8d56da997b0b3ab07f91f6bef0c5661847758c4a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/05/2022
ms.locfileid: "138036335"
---
# <a name="quickstart-sign-in-and-get-an-access-token-in-an-angular-spa-using-the-auth-code-flow"></a>クイックスタート: 認可コード フローを使用して Angular SPA 内でユーザーをサインインさせ、アクセス トークンを取得する


> [!div renderon="docs"]
> ようこそ。 ご要望のページを表示できません。 問題の修正に取り組んでいますが、次のリンクから目的の記事にアクセスできるかお試しください。
>
> > [クイックスタート: ユーザーのサインイン処理がある Angular シングルページ アプリ](single-page-app-quickstart.md?pivots=devlang-angular)
> 
> ご不便をおかけして申し訳ありませんが、問題が解決するまで今しばらくお待ちください。
 
> [!div renderon="portal" class="sxs-lookup"]
> このクイックスタートでは、認可コード フローを使用して、JavaScript Angular シングルページ アプリケーション (SPA) でユーザーをサインインさせ、Microsoft Graph を呼び出す方法を示すコード サンプルをダウンロードして実行します。 このコード サンプルでは、Microsoft Graph API または任意の Web API を呼び出すためのアクセス トークンを取得する方法を示します。
> 
> 図については、「[このサンプルのしくみ](#how-the-sample-works)」を参照してください。
> 
> このクイックスタートでは、認可コード フローで MSAL Angular v2 を使用します。
> 
> ## <a name="prerequisites"></a>前提条件
> 
> * Azure サブスクリプション - [Azure サブスクリプションを無料で作成する](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
> * [Node.js](https://nodejs.org/en/download/)
> * [Visual Studio Code](https://code.visualstudio.com/download) または別のコード エディター
> 
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>手順 1:Azure portal でのアプリケーションの構成
> このクイックスタートのコード サンプルを動作させるには、**リダイレクト URI** (`http://localhost:4200/`) を追加します。
> 
> >[!div class="nextstepaction"]
> >[これらの変更を行います]()
> 
> > [!div class="alert alert-info"]
> > ![構成済み](media/quickstart-v2-javascript/green-check.png) アプリケーションはこれらの属性で構成されています。
> 
>  #### <a name="step-2-download-the-project"></a>手順 2:プロジェクトのダウンロード
> 
> Node.js を使用して Web サーバーでプロジェクトを実行する
> 
> >[!div class="nextstepaction"]
> >[コード サンプルをダウンロードします](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa/archive/main.zip)
> 
> > [!div class="sxs-lookup"]
> > > [!NOTE]
> > > `Enter_the_Supported_Account_Info_Here`
> 
> 
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>手順 3:アプリが構成され、実行準備ができる
> 
> アプリのプロパティの値を使用してプロジェクトを構成しました。
> 
> #### <a name="step-4-run-the-project"></a>手順 4:プロジェクトを実行する
> 
> Node.js を使用して Web サーバーでプロジェクトを実行します。
> 
> 1. サーバーを起動するために、プロジェクト ディレクトリ内から次のコマンドを実行します。
>     ```console
>     npm install
>     npm start
>     ```
> 1. `http://localhost:4200/` を参照します。
> 
> 1. **[ログイン]** を選択してサインイン プロセスを開始してから、Microsoft Graph API を呼び出します。
> 
>     初回サインイン時に、自分のプロファイルにアプリケーションがアクセスして自分をサインインさせることについて同意を求められます。 正常にサインインした後、 **[プロファイル]** ボタンをクリックして、自分のユーザー情報をページに表示します。
> 
> ## <a name="more-information"></a>詳細情報
> 
> ### <a name="how-the-sample-works"></a>このサンプルのしくみ
> 
> ![シングルページ アプリケーションの認証コード フローを示す図。](media/quickstart-v2-javascript-auth-code/diagram-01-auth-code-flow.png)
> 
> ### <a name="msaljs"></a>msal.js
> 
> MSAL.js ライブラリは、ユーザーをサインインさせ、Microsoft ID プラットフォームによって保護された API へのアクセスに使用されるトークンを要求します。
> 
> Node.js がインストール済みの場合は、次のように Node.js Package Manager (npm) を使用して最新バージョンをダウンロードできます。
> 
> ```console
> npm install @azure/msal-browser @azure/msal-angular@2
> ```
> 
> ## <a name="next-steps"></a>次のステップ
> 
> vanilla JavaScript を使用して認可コード フロー アプリケーションを作成する詳細なステップ バイ ステップ ガイドについては、次のチュートリアルを参照してください。
> 
> > [!div class="nextstepaction"]
> > [サインインして MS Graph を呼び出すチュートリアル](./tutorial-v2-javascript-auth-code.md)
