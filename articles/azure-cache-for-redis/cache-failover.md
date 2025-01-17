---
title: フェールオーバーと修正プログラムの適用- Azure Cache for Redis
description: Azure Cache for Redis のフェールオーバー、修正プログラムの適用、および更新プロセスについて学習します。
author: curib
ms.author: cauribeg
ms.service: cache
ms.topic: conceptual
ms.date: 11/3/2021
ms.openlocfilehash: 67b3ad49033a2fb4708b93c65e0d9f7110726cee
ms.sourcegitcommit: 8946cfadd89ce8830ebfe358145fd37c0dc4d10e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/05/2021
ms.locfileid: "131850966"
---
# <a name="failover-and-patching-for-azure-cache-for-redis"></a>Azure Cache for Redis のフェールオーバーと修正プログラムの適用

回復性の高い優れたクライアント アプリケーションを構築するには、Azure Cache for Redis サービスでのフェールオーバーを理解することが重要です。 フェールオーバーは、計画された管理操作の一環であることも、計画外のハードウェアやネットワーク障害で発生することもあります。 一般的にキャッシュ フェールオーバーは、管理サービスが Azure Cache for Redis バイナリに修正プログラムを適用するときに使用します。

この記事では、次の情報を確認します。  

- フェールオーバーとは
- 修正プログラムの適用中にフェールオーバーがどのように発生するか。
- 回復性があるクライアント アプリケーションを構築する方法。

## <a name="what-is-a-failover"></a>フェールオーバーとは

まず、Azure Cache for Redis のフェールオーバーの概要について説明します。

### <a name="a-quick-summary-of-cache-architecture"></a>キャッシュ アーキテクチャの簡単な概要

キャッシュは、個別のプライベート IP アドレスを持つ複数の仮想マシンで構成されます。 ノードとも呼ばれる各仮想マシンは、1 つの仮想 IP で共有のロード バランサーに接続されています。 各ノードでは Redis サーバー プロセスを実行し、ホスト名と Redis ポートを使用してアクセスできます。 各ノードは、プライマリまたはレプリカ ノードのいずれかと見なされています。 クライアント アプリケーションがキャッシュに接続されると、そのトラフィックはこのロード バランサーを通過し、自動的にプライマリ ノードにルーティングされます。

Basic キャッシュでは、1 つのノードが常にプライマリとなります。 Standard または Premium キャッシュには、プライマリとして選択されているノードと、レプリカとして選択されているノードの 2 つのノードがあります。 ノードが複数ある Standard および Premium キャッシュでは、1 つのノードが要求の処理を続けているとき、1 つのノードは利用できない場合があります。 クラスター化されたキャッシュは多くのシャードで構成され、それぞれに個別のプライマリ ノードとレプリカ ノードがあります。 他のシャードが使用可能な状態であるとき、1 つのシャードがダウンしている場合があります。

> [!NOTE]
> Basic のキャッシュにはノードは複数はなく、可用性に関する SLA は提供されていません。 Basic のキャッシュは、開発およびテストの目的でのみに推奨されます。 マルチノード デプロイには、可用性を高めるために Standard または Premium キャッシュを使用してください。

### <a name="explanation-of-a-failover"></a>フェールオーバーの説明

フェールオーバーは、1 つのレプリカ ノードが自身をプライマリ ノードに昇格し、古いプライマリ ノードが既存の接続を切断したときに発生します。 復帰後、そのプライマリ ノードによってロールの変更が認識されると、自身をレプリカに降格させます。 その後、新しいプライマリに接続し、データを同期します。 フェールオーバーは、計画されている場合とされなていない場合があります。

"*計画されたフェールオーバー*" は、次の 2 つの異なるときに実行されます。

- Redis の修正プログラムの適用や OS のアップグレードなどのシステムの更新時。  
- スケーリングや再起動などの管理操作時。

ノードには更新が事前通知されているため、ロールを協調的に交換し、ロード バランサーをすばやく変更で更新できます。 計画フェールオーバーは通常 1 秒未満で終了します。

*計画外のフェールオーバー* は、ハードウェア障害、ネットワーク障害、またはその他のプライマリ ノードに対する予期しない停止が原因で発生します。 レプリカ ノードは自身をプライマリに昇格しますが、そのプロセスには時間がかかります。 レプリカ ノードでは、フェールオーバー プロセスを開始する前に、そのプライマリ ノードが使用できないことを最初に検出する必要があります。 また、不要なフェールオーバーがなされないよう、この計画外の障害が一時的なものでもローカルのものでもないことをレプリカ ノードによって確認される必要もあります。 検出が遅れるということは、計画外のフェールオーバーは通常 10 秒から 15 秒以内に完了することを意味します。

## <a name="how-does-patching-occur"></a>修正プログラムの適用はどのように行われますか?

Azure Cache for Redis サービスでは、お使いのキャッシュを最新のプラットフォーム機能と修正プログラムで定期的に更新します。 このサービスは、キャッシュへの修正プログラムの適用を次の手順で実行します。

1. 管理サービスでは、修正プログラムを適用する 1 つのノードを選択します。
1. 選択したノードがプライマリ ノードである場合、そのレプリカ ノードは自身を協調的に昇格させます。 この昇格は、計画されたフェールオーバーと見なされます。
1. 選択されたノードは再起動されて新しい変更が取得され、レプリカ ノードとして復帰します。
1. レプリカ ノードがプライマリ ノードに接続し、データを同期します。
1. データの同期が完了すると、残りのノードに対する修正プログラムの適用プロセスが繰り返されます。

修正プログラムの適用は計画されたフェールオーバーであるため、レプリカ ノードではプライマリになるために迅速に自身を昇格させ、 次に、ノードは要求の処理と新しい接続を開始します。 Basic のキャッシュにはレプリカ ノードはなく、更新が完了するまで使用することはできません。 クラスター化されたキャッシュの各シャードには個別に修正プログラムが適用され、別のシャードへの接続は閉じられません。

> [!IMPORTANT]
> データの損失を防ぐために、ノードには一度に 1 つずつ修正プログラムが適用されます。 Basic キャッシュではデータが失われます。 クラスター化されたキャッシュでは、一度に 1 つのシャードに修正プログラムが適用されます。

同じリソー スグループとリージョン内の複数のキャッシュにも、一度に 1 つずつ修正プログラムが適用されます。  異なるリソース グループまたは異なるリージョンにあるキャッシュには、同時に修正プログラムが適用できる場合があります。

データの完全な同期はプロセスが繰り返される前に行われるため、Standard または Premium キャッシュを使用している場合にデータが失われる可能性は低いです。 データを[エクスポート](cache-how-to-import-export-data.md#export)して、[永続化](cache-how-to-premium-persistence.md)を有効にすると、データの損失をさらに防御できます。

## <a name="additional-cache-load"></a>追加のキャッシュの負荷

フェールオーバーが発生するたびに、Standard および Premium キャッシュでは、ノード間でデータをレプリケートする必要があります。 このレプリケーションによって、サーバーのメモリと CPU の両方で負荷が増加します。 キャッシュ インスタンスに既に大きな負荷がかかっている場合は、クライアント アプリケーションの待機時間が長くなることがあります。 極端な場合、クライアント アプリケーションがタイムアウト例外を受け取ることがあります。 このさらなる負荷の影響を軽減するには、キャッシュの `maxmemory-reserved` 設定を[構成](cache-configure.md#memory-policies)します。

## <a name="how-does-a-failover-affect-my-client-application"></a>フェールオーバーはクライアント アプリケーションにどのような影響を与えますか?

クライアント アプリケーションは、Azure Cache for Redis からいくつかのエラーを受信することがあります。 クライアント アプリケーションで確認されるエラーの数は、フェールオーバー時にその接続で保留になっている操作の数によって異なります。 接続が閉じられているノード経由でルーティングされている接続では、エラーが発生します。

接続が中断されると、多くのクライアント ライブラリでは、次のようなさまざまな種類のエラーがスローされます。

- タイムアウト例外
- 接続例外
- ソケット例外

例外の数と種類は、キャッシュで接続が閉じられたときに、その要求がコード パス内のどこにあるかによって異なります。 たとえば、要求を送信した操作が、フェールオーバーの発生で応答を受信しない場合、タイムアウト例外を取得する可能性があります。 接続が閉じられたオブジェクトは、再接続が正常に行われるまで、新しい要求で接続例外を受け取ります。

ほとんどのクライアント ライブラリは、構成されている場合、キャッシュへの再接続を試みます。 ただし、予期しないバグによってライブラリ オブジェクトが回復不能な状態になることがあります。 事前に構成された時間を超えてエラーが続く場合は、接続オブジェクトを再作成する必要があります。 Microsoft .NET およびその他のオブジェクト指向言語では、[ForceReconnect パターン](cache-best-practices-connection.md#using-forcereconnect-with-stackexchangeredis)を使用することで、アプリケーションを再起動せずに接続を再作成できます。

### <a name="can-i-be-notified-in-advance-of-planned-maintenance"></a>計画メンテナンスの通知を前もって受け取ることはできますか?

Azure Cache for Redis では、[AzureRedisEvents](https://github.com/Azure/AzureCacheForRedis/blob/main/AzureRedisEvents.md) という名前の発行とサブスクライブ (pub/sub) チャネルで、計画されている更新の約 30 秒前に通知を発行します。 通知はランタイム通知です。

この通知は、サーキット ブレーカーを使用してキャッシュをバイパスするアプリケーションや、コマンドをバッファリングするアプリケーションのためのものです。 たとえば、計画されているすべての更新時にキャッシュをバイパスできます。

`AzureRedisEvents` チャネルは、数日や数時間も前にユーザーに通知できるメカニズムではありません。 このチャネルは、サーバーの使用可能性に影響するおそれがある、計画されたサーバー メンテナンス イベントの予定をクライアントに通知することができます。

一般的な Redis クライアント ライブラリの多くで、pub/sub チャネルのサブスクライブがサポートされています。 通常、クライアント アプリケーションに `AzureRedisEvents` チャネルからの通知の受信を追加することは簡単です。

アプリケーションは、`AzureRedisEvents` にサブスクライブされると、ノードがメンテナンス イベントの影響を受ける 30 秒前に通知を受け取ります。 この通知には発生する予定のイベントの詳細が含まれ、プライマリまたはレプリカ ノードに影響するかどうかが示されます。

数分後にメンテナンス操作が完了した時点で、別の通知が送信されます。

アプリケーションは、この通知の内容を使用して、メンテナンスの実行中にキャッシュの使用を回避するアクションを実行できます。 キャッシュには、メンテナンス操作中にトラフィックのルートがそのキャッシュから切り離されるサーキット ブレーカー パターンが実装されている場合があります。 代わりに、トラフィックは永続ストアに直接送信されます。 この通知は、ユーザーがアラートを受け取り、手動でアクションを実行するための時間を確保するためのものではありません。

ほとんどの場合、アプリケーションで `AzureRedisEvents` にサブスクライブしたり、通知に応答したりする必要はありません。 代わりに、[回復性の組み込み](#build-in-resiliency)を実装することをお勧めします。

アプリケーションが十分な回復性を備えていれば、ノードのメンテナンス時に発生するような、短時間だけ接続が失われたりキャッシュを使用できなくなったりする問題に適切に対処できます。 また、ネットワーク エラーや他のイベントのために、`AzureRedisEvents` からの警告がないまま、アプリケーションのキャッシュへの接続が予期せずに失われる場合もあります。

次のようないくつかの注目すべきケースでのみ、`AzureRedisEvents` にサブスクライブすることをお勧めします。

- 極度のパフォーマンス要件が求められ、わずかな遅延も回避する必要のあるアプリケーション。 このようなシナリオでは、現在のキャッシュでメンテナンスが開始される前に、トラフィックをバックアップ キャッシュにシームレスに再ルーティングできます。
- プライマリ ノードではなく、レプリカからデータを明示的に読み取るアプリケーション。 レプリカ ノードのメンテナンス中、プライマリ ノードからデータを読み取ることができるよう、アプリケーションで一時的な切り替えを行うことができます。
- 書き込み操作が通知なしに失敗したり、確認なしに成功したりするリスクを回避する必要のあるアプリケーション。このような現象は、メンテナンスのために接続が閉じられているときに発生する場合があります。 これらのケースが危険なデータ損失につながる場合は、メンテナンスの開始がスケジュールされている時間よりも前に、アプリケーションで書き込みコマンドをプロアクティブに一時停止するかリダイレクトすることができます。

### <a name="client-network-configuration-changes"></a>クライアントのネットワーク構成の変更

クライアント側の特定のネットワーク構成の変更によって、"接続が使用できません" というエラーが発生することがあります。 このような変更には、次のものが含まれます。

- クライアント アプリケーションの仮想 IP アドレスをステージング スロットと運用スロット間で交換しました。
- お使いのアプリケーションのインスタンスのサイズまたは数をスケーリングしました。

このような変更では、1 分未満の接続の問題が発生する可能性があります。 お使いのクライアント アプリケーションからは、Azure Cache for Redis サービスに加え、他の外部のネットワークのリソースにも接続できなくなる場合があります。

## <a name="build-in-resiliency"></a>回復性を組み込む

フェールオーバーを完全に回避することはできません。 接続の中断や要求の失敗への回復性を備えるようにクライアント アプリケーションを作成してください。 ほとんどのクライアント ライブラリは自動的にキャッシュ エンドポイントに再接続できますが、少数は失敗した要求を再試行します。 アプリケーションのシナリオによっては、バックオフによる再試行ロジックが理にかなっている場合があります。

### <a name="how-do-i-make-my-application-resilient"></a>アプリケーションの回復性を高める方法

回復性があるクライアントを作成するには、これらの設計パターン、特にサーキット ブレーカー パターンや再試行パターンを参照してください。

- [信頼性パターン - クラウド設計パターン](/azure/architecture/framework/resiliency/reliability-patterns#resiliency)
- [Azure サービスの再試行ガイダンス - クラウド アプリケーションのベスト プラクティス](/azure/architecture/best-practices/retry-service-specific)
- [エクスポネンシャル バックオフを含む再試行を実装する](/dotnet/architecture/microservices/implement-resilient-applications/implement-retries-exponential-backoff)

クライアント アプリケーションの回復性をテストするには、接続が切断された場合の手動トリガーとして、[再起動](cache-administration.md#reboot)を使用します。

加えて、キャッシュに対する[更新のスケジュール設定](cache-administration.md#schedule-updates)で、毎週の時間枠を指定して Redis ランタイムのパッチを適用することをお勧めします。 通常これらの時間枠は、発生の可能性のある事柄を回避する、クライアント アプリケーションのトラフィックが少ない時間です。

詳細については、「[接続の回復力](cache-best-practices-connection.md)」を参照してください。

## <a name="next-steps"></a>次のステップ

- キャッシュの[更新をスケジュールする](cache-administration.md#schedule-updates)。
- [再起動](cache-administration.md#reboot)を使用して、アプリケーションの回復性をテストする。
- メモリの予約とポリシーを[構成する](cache-configure.md#memory-policies)。
