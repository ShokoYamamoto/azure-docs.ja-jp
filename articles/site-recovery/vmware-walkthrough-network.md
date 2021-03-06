---
title: "VMware から Azure へのレプリケーションのネットワークを計画する | Microsoft Docs"
description: "この記事では、VMware VM を Azure にレプリケートする場合に必要なネットワーク計画について説明します"
services: site-recovery
documentationcenter: 
author: rayne-wiselman
manager: carmonm
editor: 
ms.assetid: 5578a0b4-0352-41c3-9cce-56beb17a4d97
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 06/27/2017
ms.author: raynew
ms.translationtype: Human Translation
ms.sourcegitcommit: 138f04f8e9f0a9a4f71e43e73593b03386e7e5a9
ms.openlocfilehash: f164ac68ba6ec650bb3996b4aa870e1b98533a23
ms.contentlocale: ja-jp
ms.lasthandoff: 06/29/2017


---

# <a name="step-4-plan-networking-for-vmware-to-azure-replication"></a>手順 4: VMware から Azure へのレプリケーションのネットワークを計画する

この記事では、[Azure Site Recovery](site-recovery-overview.md) サービスを使用してオンプレミスの VMware VM を Azure にレプリケートする場合のネットワーク計画の考慮事項について説明します。

この記事の末尾にあるコメント欄でご意見をお送りください。または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr) に質問を投稿してください。


## <a name="connect-to-replica-vms"></a>レプリカ VM に接続する

レプリケーションとフェールオーバー戦略を計画する際の重要な問題の 1 つとして、フェールオーバー後の Azure VM への接続方法があります。 レプリカの Azure VM のネットワーク戦略を設計する場合は、いくつかの選択肢があります。

- **異なる IP アドレスを使用する**: レプリケートされた Azure VM ネットワークに対して異なる IP アドレス範囲を使用することを選択できます。 このシナリオでは、VM はフェールオーバー後に新しい IP アドレスを取得するため、DNS の更新が必要になります。
- **同じ IP アドレスを保持する**: フェールオーバー後、Azure ネットワークに対して、プライマリのオンプレミス サイトと同じ IP アドレス範囲を使用することができます。 同じ IP アドレスを維持することで、フェールオーバー後のネットワーク関連の問題が少なくなるため、復旧が簡単になります。 ただし、Azure にレプリケートする場合は、フェールオーバー後に IP アドレスの新しい場所でルートを更新する必要があります。 


## <a name="retain-ip-addresses"></a>IP アドレスを維持する

Site Recovery には、サブネットのフェールオーバーを使用して、Azure へのフェールオーバー時に固定 IP アドレスを保持する機能があります。

サブネットのフェールオーバーでは、特定のサブネットはサイト 1 とサイト 2 のどちらかにのみ存在し、両方のサイトに同時に存在することはありません。 フェールオーバーの発生時に IP アドレス空間を維持するには、サイトからサイトへサブネットを移動するよう、ルーター インフラストラクチャをプログラムで制御します。 フェールオーバー時に、サブネットは、関連付けられた保護対象 VM と共に移動します。 主な欠点は、障害が発生した場合にサブネット全体を移動する必要があることです。


### <a name="failover-example"></a>フェールオーバーの例

Azure へのフェールオーバーの例を見てみましょう。

- 架空の会社 Woodgrove Bank は、自社のビジネス アプリをホストするオンプレミス インフラストラクチャを所有しています。 この会社のモバイル アプリケーションは Azure でホストされています。
- Woodgrove Bank が Azure 内でホストしている VM とオンプレミス サーバーとの接続は、オンプレミス エッジ ネットワークと Azure 仮想ネットワーク間のサイト間 (VPN) 接続によって提供されます。
- この VPN は、Azure 内の会社の仮想ネットワークがオンプレミス ネットワークの拡張機能として表示されることを意味します。
- Woodgrove では、Site Recovery を使用して、オンプレミスのワークロードを Azure にレプリケートしたいと考えています。
 - Woodgrove が取り組むべき課題は、ハードコーディングされた IP アドレスに依存するアプリケーションと構成です。結果として、Azure へのフェールオーバー後もアプリケーションの IP アドレスを維持することが必要となります。
 - Woodgrove では、範囲 (172.16.1.0/24、172.16.2.0/24) からの IP アドレスを、Azure で実行されるリソースに割り当てています。


Woodgrove が IP アドレスを維持したまま VM を Azure にレプリケートできるようにするには、次のことを行う必要があります。

1. Azure 仮想ネットワークを作成します。 これは、アプリケーションがシームレスにフェールオーバーできるように、オンプレミス ネットワークの拡張機能となります。
2. Azure に作成された仮想ネットワークには、ポイント対サイト接続に加えて、サイト間 VPN 接続を追加することができます。
3. サイト間接続をセットアップする際、Azure ネットワークでは、IP アドレス範囲がオンプレミスの IP アドレス範囲と異なる場合のみ、トラフィックをオンプレミスの場所 (ローカル ネットワーク) にルーティングできます。
    - これは、Azure で拡張サブネットがサポートされていないためです。 そのため、オンプレミス上にサブネット 192.168.1.0/24 がある場合、Azure ネットワークにローカル ネットワーク 192.168.1.0/24 を追加することはできません。
    - これは予想される状況です。その理由は、サブネットにアクティブな VM がないこととサブネットがディザスター リカバリーの目的でのみ作成されることを Azure が認識しないことにあります。
    - Azure ネットワークからネットワーク トラフィックを正しくルーティングできるようにするには、ネットワーク内のサブネットとローカル ネットワーク内のサブネットが競合してはなりません。

![サブネットのフェールオーバー前](./media/site-recovery-network-design/network-design7.png)

### <a name="before-failover"></a>フェールオーバー前

1. 追加のネットワーク (たとえば、復旧ネットワーク) を作成します。 これは、フェールオーバーされた VM が作成されるネットワークです。
2. フェールオーバー後も VM の IP アドレスが確実に維持されるようにするには、VM のプロパティの **[構成]** で、オンプレミス側の VM と同じ IP アドレスを指定し、**[保存]** をクリックします。
3. VM がフェールオーバーされると、Azure Site Recovery によって、指定した IP アドレスが VM に割り当てられます。

    ![Network properties](./media/site-recovery-network-design/network-design8.png)

4. フェールオーバーがトリガーされ、必要な IP アドレスが割り当てられた VM が Azure に作成されると、[VNet 間接続](../vpn-gateway/virtual-networks-configure-vnet-to-vnet-connection.md)を使用して目的のネットワークに接続することができます。 このアクションはスクリプト化することができます。
5. 192.168.1.0/24 が Azure に移動したことが反映されるように、ルートを適切に変更する必要があります。

    ![サブネットのフェールオーバー後](./media/site-recovery-network-design/network-design9.png)

### <a name="after-failover"></a>フェールオーバー後

上記のような Azure ネットワークがない場合は、フェールオーバー後、プライマリ サイトと Azure の間にサイト間 VPN 接続を作成できます。

## <a name="change-ip-addresses"></a>IP アドレスを変更する

フェールオーバー後に IP アドレスを保持する必要がない場合の Azure ネットワーク インフラストラクチャの設定方法については、こちらの[ブログ記事](http://azure.microsoft.com/blog/2014/09/04/networking-infrastructure-setup-for-microsoft-azure-as-a-disaster-recovery-site/)を参照してください。 アプリケーションの説明から始まり、オンプレミスと Azure でのネットワークの設定方法を説明し、最後にフェールオーバーの実行に関する情報が示されています。  

## <a name="next-steps"></a>次のステップ

[手順 5: Azure を準備する](vmware-walkthrough-prepare-azure.md)方法に関するページに進む

