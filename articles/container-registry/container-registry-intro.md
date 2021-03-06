---
title: "Azure のプライベート Docker コンテナー レジストリ | Microsoft Docs"
description: "クラウド ベースの管理されたプライベート Docker レジストリを提供する Azure Container Registry サービスの紹介です。"
services: container-registry
documentationcenter: 
author: stevelas
manager: balans
editor: dlepow
tags: 
keywords: 
ms.assetid: ee2b652b-fb7c-455b-8275-b8d4d08ffeb3
ms.service: container-registry
ms.devlang: na
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/24/2017
ms.author: stevelas
ms.custom: H1Hack27Feb2017
ms.translationtype: HT
ms.sourcegitcommit: 83f19cfdff37ce4bb03eae4d8d69ba3cbcdc42f3
ms.openlocfilehash: fd0356286be46f99fd9ab8eabc53256103038407
ms.contentlocale: ja-jp
ms.lasthandoff: 08/22/2017

---
# <a name="introduction-to-private-docker-container-registries"></a>プライベート Docker コンテナー レジストリの概要


Azure コンテナー レジストリは、オープンソースの Docker Registry 2.0 に基づいた、管理された [Docker レジストリ](https://docs.docker.com/registry/) サービスです。 プライベート [Docker コンテナー](https://www.docker.com/what-docker) イメージを保存および管理する Azure コンテナー レジストリを作成および管理します。 既存のコンテナーの開発とデプロイ パイプラインで Azure のコンテナー レジストリを使用し、Docker コミュニティの専門知識を活用します。

Docker とコンテナーに関する背景情報については、次を参照してください。

* [Docker ユーザー ガイド](https://docs.docker.com/engine/userguide/)




## <a name="use-cases"></a>ユース ケース
Azure コンテナー レジストリからさまざまなデプロイ ターゲットにイメージをプルできます。

* [DC/OS](https://docs.mesosphere.com/)、[Docker Swarm](https://docs.docker.com/swarm/)、[Kubernetes](http://kubernetes.io/docs/) など、ホストのクラスターのコンテナー化されたアプリケーションを管理する**スケーラブルなオーケストレーション システム**。
* [Container Service](../container-service/index.yml)、[App Service](/app-service/index.md)、[Batch](../batch/index.md)、[Service Fabric](/azure/service-fabric/) など、アプリケーションの大規模な構築と実行をサポートする **Azure サービス**。

開発者は、コンテナー開発ワークフローの一環としてコンテナー レジストリにプッシュすることもできます。 たとえば、[Visual Studio Team Services](https://www.visualstudio.com/docs/overview) や [Jenkins](https://jenkins.io/) などの継続的な統合とデプロイ ツールからコンテナー レジストリを対象にすることができます。





## <a name="key-concepts"></a>主要な概念
* **レジストリ** - Azure サブスクリプションに 1 つ以上のコンテナー レジストリを作成できます。 各レジストリは、同じ場所の Standard Azure [ストレージ アカウント](../storage/common/storage-introduction.md)でサポートされます。 デプロイと同じ Azure の場所にレジストリを作成することで、ネットワーク上の近い場所にローカルで保存されたコンテナー イメージを活用します。 レジストリの完全修飾名は `myregistry.azurecr.io` という形式になります。

  コンテナー レジストリへの[アクセスを制御](container-registry-authentication.md)するには、Azure Active Directory でサポートされている[サービス プリンシパル](../active-directory/active-directory-application-objects.md)または提供された管理者アカウントを使用します。 レジストリによる認証を行うには、標準の `docker login` コマンドを実行します。

* **管理されたレジストリ** - 3 つの SKU (Basic、Standard、Premium) のレジストリの機能を拡充した階層です。 これらの SKU のイメージは、Azure Container Registry サービスによって管理されたストレージ アカウントに保存され、信頼性が向上すると共に新しい機能が利用できます。 新しい機能の例として、webhook 統合、Azure Active Directory によるリポジトリ認証、削除機能のサポートが挙げられます。 ユーザーはレジストリを作成する際に、管理されたレジストリを作成するか、または独自のストレージ アカウントを基にレジストリを作成するかを選択できます。

* **リポジトリ** - レジストリには、1 つ以上のリポジトリ (コンテナー イメージのグループ) が含まれています。 Azure Container Registry では、複数レベルのリポジトリ名前空間をサポートしています。 この機能により、特定のアプリに関連するイメージのコレクション (つまり、アプリのコレクション) を特定の開発チームや運用チーム別にグループ化できます。 次に例を示します。

  * `myregistry.azurecr.io/aspnetcore:1.0.1` は、会社全体のイメージを表します
  * `myregistry.azurecr.io/warrantydept/dotnet-build` は、保証部門全体で共有される、.NET アプリを構築するために使用するイメージを表します
  * `myregistry.azrecr.io/warrantydept/customersubmissions/web` は、保証部門が所有する、customer submissions アプリでグループ化された Web イメージを表します

* **イメージ** - リポジトリに格納されます。各イメージは、Docker コンテナーの読み取り専用のスナップショットです。 Azure コンテナー レジストリには、Windows と Linux の両方のイメージを含めることができます。 すべてのコンテナーのデプロイのイメージ名を制御できます。 イメージをリポジトリにプッシュしたり、イメージをリポジトリからプルしたりするには、標準の [Docker コマンド](https://docs.docker.com/engine/reference/commandline/)を使用します。

* **コンテナー** - コンテナーでは、ソフトウェア アプリケーションと、コード、ランタイム、システム ツール、ライブラリを含む完全なファイル システム内にラップされた依存関係を定義します。 コンテナー レジストリからプルした Windows または Linux イメージに基づいて Docker コンテナーを実行します。 1 台のコンピューターで実行されているすべてのコンテナーでは、オペレーティング システムのカーネルを共有します。 Docker コンテナーは、主要なすべての Linux ディストリビューション、Mac、および Windows に完全に移植できます。




## <a name="next-steps"></a>次のステップ
* [Azure Portal を使用したコンテナー レジストリの作成](container-registry-get-started-portal.md)
* [Azure CLI を使用したコンテナー レジストリの作成](container-registry-get-started-azure-cli.md)
* [Docker CLI を使用した最初のイメージのプッシュ](container-registry-get-started-docker-cli.md)
* Visual Studio Team Services、Azure Container Service、Azure Container Registry を使用して継続的な統合とデプロイ ワークフローを構築する場合は、[こちらのチュートリアル](../container-service/dcos-swarm/container-service-docker-swarm-setup-ci-cd.md)を参照してください。
* Azure で (パブリック エンドポイントなしの) 独自の Docker プライベート レジストリを設定したい場合は、「[Azure への独自のプライベート Docker Registry のデプロイ](../virtual-machines/virtual-machines-linux-docker-registry-in-blob-storage.md)」を参照してください。

