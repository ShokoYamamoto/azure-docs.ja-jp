---
title: "Azure Service Fabric CLI (sfctl) の概要"
description: "Azure Service Fabric CLI の使用方法について説明します。 クラスターに接続して、アプリケーションを管理する方法についても説明します。"
services: service-fabric
author: samedder
manager: timlt
ms.service: service-fabric
ms.topic: get-started-article
ms.date: 08/22/2017
ms.author: edwardsa
ms.translationtype: HT
ms.sourcegitcommit: 25e4506cc2331ee016b8b365c2e1677424cf4992
ms.openlocfilehash: 5ce9adf6c82e3a5521883c5de1e0689d5bf0d94e
ms.contentlocale: ja-jp
ms.lasthandoff: 08/24/2017

---
# <a name="azure-service-fabric-command-line"></a>Azure Service Fabric コマンド ライン

Azure Service Fabric CLI (sfctl) は、Azure Service Fabric のエンティティを対話的に操作したり管理したりするためのコマンド ライン ユーティリティです。 sfctl は、Windows クラスターと Linux クラスターのどちらでも使用できます。 sfctl は、python をサポートするあらゆるプラットフォームで動作します。

## <a name="prerequisites"></a>前提条件

インストール前に、ご使用の環境に python と pip の両方がインストールされていることを確認してください。 詳細については、[pip のクイック スタート ドキュメント](https://pip.pypa.io/en/latest/quickstart/)と公式の [python インストール ドキュメント](https://wiki.python.org/moin/BeginnersGuide/Download)を参照してください。

python 2.7 と python 3.6 の両方がサポートされますが、python 3.6 の使用をお勧めします。

## <a name="install"></a>インストール

Azure Service Fabric CLI (sfctl) は、python パッケージになっています。 最新バージョンをインストールするには、次のコマンドを実行します。

```bash
pip install sfctl
```

インストール後、`sfctl -h` を実行すると、使用できるコマンドについての情報が得られます。

## <a name="cli-syntax"></a>CLI 構文

コマンドの先頭には必ず、`sfctl` が付きます。 使用できるすべてのコマンドについて調べたい場合は、`sfctl -h` を使用すると、コマンド全般についての情報が表示されます。 `sfctl <command> -h` を使用すると、特定のコマンドについてのヘルプ情報が表示されます。

コマンドの構造は、動詞またはアクションの前にコマンドの対象が来るという一定のパターンに従います。

```azurecli
sfctl <object> <action>
```

この例では、`<object>` が `<action>` の対象です。

## <a name="select-a-cluster"></a>クラスターの選択

何らかの操作を実行するには、まず、接続先のクラスターを選択する必要があります。 たとえば、`testcluster.com` という名前のクラスターを選択してそのクラスターに接続するには、次のコマンドを実行します。

> [!WARNING]
> セキュリティ保護されていない Service Fabric クラスターを運用環境で使用することは避けてください。

```azurecli
sfctl cluster select --endpoint http://testcluster.com:19080
```

クラスターのエンドポイントには、プレフィックスとして `http` または `https` を付けたうえで、 HTTP ゲートウェイのポートを含める必要があります。 このポートとアドレスは、Service Fabric Explorer の URL と同じです。

証明書を使って保護されているクラスターの場合、PEM でエンコードされた証明書を指定できます。 証明書は、単一のファイルとしてまたは証明書とキーのペアとして指定できます。

```azurecli
sfctl cluster select --endpoint https://testsecurecluster.com:19080 --pem ./client.pem
```

詳細については、[セキュリティ保護された Azure Service Fabric クラスターへの接続](service-fabric-connect-to-secure-cluster.md)に関するページを参照してください。

## <a name="basic-operations"></a>基本操作

クラスター接続情報は sfctl セッションをまたいで維持されます。 Service Fabric クラスターを選択したら、任意の Service Fabric コマンドをクラスターに対して実行できます。

たとえば Service Fabric クラスターの正常性の状態を取得するには、次のコマンドを使用します。

```azurecli
sfctl cluster health
```

コマンドからは、次の出力が生成されます。

```json
{
  "aggregatedHealthState": "Ok",
  "applicationHealthStates": [
    {
      "aggregatedHealthState": "Ok",
      "name": "fabric:/System"
    }
  ],
  "healthEvents": [],
  "nodeHealthStates": [
    {
      "aggregatedHealthState": "Ok",
      "id": {
        "id": "66aa824a642124089ee474b398d06a57"
      },
      "name": "_Test_0"
    }
  ],
  "unhealthyEvaluations": []
}
```

## <a name="tips-and-troubleshooting"></a>ヒントとトラブルシューティング

一般的な問題を解決するうえでの推奨事項とヒントをいくつか紹介します。

### <a name="convert-a-certificate-from-pfx-to-pem-format"></a>PFX 形式から PEM 形式への証明書の変換

Service Fabric CLI は、クライアント側の証明書を PEM (拡張子 .pem) ファイルとしてサポートします。 Windows の PFX ファイルを使用している場合は、これらの証明書を PEM 形式に変換する必要があります。 PFX ファイルを PEM ファイルに変換するには、次のコマンドを使用します。

```bash
openssl pkcs12 -in certificate.pfx -out mycert.pem -nodes
```

詳細については、[ のドキュメント](https://www.openssl.org/docs/)を参照してください。

### <a name="connection-issues"></a>接続に関する問題

一部の操作において、次のメッセージが表示される場合があります。

`Failed to establish a new connection: [Errno 8] nodename nor servname provided, or not known`

この場合は、指定したクラスター エンドポイントが利用可能で、リッスン状態になっていることを確認してください。 また、そのホストとポートで Service Fabric Explorer の UI を利用できることも確認してください。 エンドポイントを更新するには `sfctl cluster select` を使用します。

### <a name="detailed-logs"></a>詳細ログ

問題のデバッグやレポート作成の多くで、詳細ログが役立ちます。 ログ ファイルの詳細レベルを上げるためのグローバルな `--debug` フラグが用意されています。

### <a name="command-help-and-syntax"></a>コマンドのヘルプと構文

特定のコマンドや、コマンドのグループについてのヘルプ情報を表示するには、次の例のように、`-h` フラグを使用します。

```azurecli
sfctl application -h
```

別の例:

```azurecli
sfctl application create -h
```

## <a name="next-steps"></a>次のステップ

* [Azure Service Fabric CLI でアプリケーションをデプロイする](service-fabric-application-lifecycle-sfctl.md)
* [Linux 上の Service Fabric の概要](service-fabric-get-started-linux.md)

