---
title: Azure Database for MySQL フレキシブル サーバーでのバックアップと復元
description: Azure Database for MySQL フレキシブル サーバーでのバックアップと復元の概念について説明します
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 0199d0742bd3c90c206a14b34300c02780e69246
ms.sourcegitcommit: 677e8acc9a2e8b842e4aef4472599f9264e989e7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/11/2021
ms.locfileid: "132345189"
---
# <a name="backup-and-restore-in-azure-database-for-mysql-flexible-server"></a>Azure Database for MySQL フレキシブル サーバーでのバックアップと復元

[!INCLUDE[applies-to-mysql-flexible-server](../includes/applies-to-mysql-flexible-server.md)]

Azure Database for MySQL フレキシブル サーバーを使用すると、サーバーのバックアップが自動的に作成されて、リージョン内のローカル冗長ストレージに安全に格納されます。 バックアップを使用すると、サーバーを特定の時点に復元できます。 不慮の破損または削除からデータを保護するバックアップと復元は、ビジネス継続性戦略の最も重要な部分です。

## <a name="backup-overview"></a>Backup の概要

フレキシブル サーバーを使用すると、データ ファイルのスナップショット バックアップが取得されて、ローカル冗長ストレージに格納されます。 また、サーバーによりトランザクション ログのバックアップも実行され、それもローカル冗長ストレージに格納されます。 これらのバックアップを使用すると、サーバーを、バックアップの構成済みリテンション期間内の任意の時点に復元できます。 バックアップの既定のリテンション期間は 7 日です。 必要に応じて、1 から 35 日の範囲でデータベース バックアップを構成できます。 すべてのバックアップの保存データは、AES 256 ビット暗号化を使用して暗号化されます。

これらのバックアップ ファイルをエクスポートすることはできません。 バックアップは、フレキシブル サーバーでの復元操作にのみ使用できます。 MySQL クライアントから [mysqldump](../concepts-migrate-dump-restore.md#dump-and-restore-using-mysqldump-utility) を使用してデータベースをコピーすることもできます。

## <a name="backup-frequency"></a>バックアップ頻度

フレキシブル サーバーでのバックアップはスナップショット ベースです。 初回のスナップショット バックアップは、サーバーの作成直後にスケジュールされます。 スナップショット バックアップは、毎日 1 回作成されます。 トランザクション ログ バックアップは 5 分ごとに実行されます。

## <a name="backup-redundancy-options"></a>バックアップ冗長オプション

Azure Database for MySQL では、計画されたイベントや計画外のイベント (一時的なハードウェア障害、ネットワークの停止や停電、大規模な自然災害など) からデータを保護するため、バックアップのコピーが複数格納されます。 Azure Database for MySQL では、Basic、General Purpose、およびメモリ最適化のレベルで、ローカル冗長、ゾーン冗長、geo 冗長の各バックアップ ストレージから柔軟に選択できます。 既定では、Azure Database for MySQL サーバー バックアップ ストレージは、同一ゾーンの高可用性 (HA) が設定されているサーバー、または高可用性構成のないサーバーの場合はローカル冗長で、ゾーン冗長の HA 構成を持つサーバーの場合はゾーン冗長です。

バックアップの冗長性により、障害が発生した場合でもデータベースが可用性と永続性の目標を満たすことができ、また Azure Database for MySQL ユーザーに 3 つのオプションを提供できます。 

- **ローカル冗長バックアップ ストレージ**: バックアップがローカル冗長バックアップ ストレージに格納されている場合、バックアップのコピーが同じデータセンターに複数格納されます。 このオプションは、サーバー ラックとドライブの障害からデータを保護します。 また、バックアップ オブジェクトに年間 99.999999999% (イレブン ナイン) 以上の持続性を提供します。 既定では、同一ゾーンの高可用性 (HA) を使用しているサーバー、または高可用性構成のないサーバーのバックアップ ストレージは、ローカル冗長に設定されます。  

- **ゾーン冗長バックアップ ストレージ**: バックアップがゾーン冗長バックアップ ストレージに格納されている場合、その複数のコピーが、サーバーがホストされている可用性ゾーン内に格納されるだけでなく、同じリージョン内の別の可用性ゾーンにもレプリケートされます。 このオプションは、高可用性を必要とするシナリオや、データ所在地の要件を満たすためデータのレプリケーションを国/地域内に制限する目的で利用できます。 また、バックアップ オブジェクトに年間 99.9999999999% (トゥエルブ ナイン) 以上の持続性を提供します。 サーバーの作成時に [ゾーン冗長の高可用性] オプションを選択して、ゾーン冗長バックアップ ストレージを確保することができます。 サーバーの高可用性は作成後に無効にすることができますが、バックアップ ストレージは引き続きゾーン冗長になります。  

- **geo 冗長バックアップ ストレージ**: バックアップが geo 冗長バックアップ ストレージに格納されている場合、その複数のコピーが、サーバーがホストされているリージョンに格納されるだけでなく、geo ペア リージョンにもレプリケートされます。 これにより、災害発生時に、より適切な保護が提供され、別のリージョンにサーバーを復元することができます。 また、バックアップ オブジェクトに年間 99.99999999999999% (シックスティーン ナイン) 以上の持続性を提供します。 サーバーの作成時に [Geo 冗長性] オプションを有効にして、geo 冗長バックアップ ストレージを確保することができます。 Geo 冗長性は、[Azure のペアになっているリージョン](../../best-practices-availability-paired-regions.md)のいずれかでホストされているサーバーでサポートされます。 

> [!NOTE]
> Geo 冗長性と、ゾーン冗長性をサポートするためのゾーン冗長の高可用性は、現在、作成時の操作としてのみ表示されます。

## <a name="moving-from-other-backup-storage-options-to-geo-redundant-backup-storage"></a>他のバックアップ ストレージ オプションから geo 冗長バックアップ ストレージへの移行 

バックアップに対して geo 冗長ストレージを構成できるのは、サーバーの作成時だけです。 一度サーバーがプロビジョニングされると、バックアップ ストレージ冗長オプションを変更することはできません。 ただし、次の推奨される方法を使用して、既存のバックアップ ストレージを geo 冗長ストレージに移行することもできます。 

- **ローカル冗長から geo 冗長バックアップ ストレージへの移行**: バックアップ ストレージをローカル冗長ストレージから geo 冗長ストレージに移行するには、ポイントインタイム リストア操作を実行し、[コンピューティングとストレージ] サーバー構成を変更して、ローカル冗長ソース サーバーの geo 冗長性を有効にします。 同一ゾーンの冗長 HA サーバーについても、基になるバックアップ ストレージが同様にローカル冗長であるため、同じやり方で geo 冗長サーバーとして復元することができます。 

- **ゾーン冗長から geo 冗長バックアップ ストレージへの移行**: Azure Database for MySQL では、[コンピューティングとストレージ] 設定の変更またはポイントインタイム リストア操作による、ゾーン冗長ストレージから geo 冗長ストレージの変換はサポートされていません。 バックアップ ストレージをゾーン冗長ストレージから geo 冗長ストレージに移動するには、新しいサーバーを作成し、[ダンプと復元](../concepts-migrate-dump-restore.md)を使用してデータを移行することが、サポートされている唯一のオプションです。


## <a name="backup-retention"></a>バックアップ保有期間

バックアップは、サーバーのバックアップ保有期間の設定に基づいて保持されます。 保有期間は 1 から 35 日の範囲で選択でき、既定の保有期間は 7 日です。 Azure portal を使用してバックアップ構成を更新することで、サーバーの作成中またはその後で保有期間を設定することができます。

ポイントインタイム リストアは使用可能なバックアップに基づいているため、その操作を現在からどのくらい遡って実行できるかは、バックアップの保有期間によって管理されます。 バックアップの保有期間は、復元の観点から回復期間として扱うこともできます。 バックアップ保有期間内の特定の時点に復旧するために必要なすべてのバックアップは、バックアップ ストレージに保持されます。 たとえば、バックアップの保有期間が 7 日に設定されている場合、回復期間は過去 7 日間と見なされます。 このシナリオでは、過去 7 日間のサーバーを復元するために必要なすべてのバックアップが保持されます。 バックアップ保有期間が 7 日の場合、データベース スナップショットとトランザクション ログ バックアップは、過去 8 日間 (期間の 1 日前まで) について格納されます。

## <a name="backup-storage-cost"></a>バックアップ ストレージのコスト

フレキシブル サーバーを使用すると、プロビジョニングされているサーバー ストレージの最大 100% までが、バックアップ ストレージとして追加コストなしで提供されます。 バックアップ ストレージを追加で使用した場合は、1 か月ごとに GB 単位で請求されます。 たとえば、サーバーに 250 GB のストレージをプロビジョニングした場合は、250 GB のストレージを追加料金なしでサーバーのバックアップに利用できます。 毎日のバックアップ使用量が 25 GB の場合は、無料バックアップ ストレージを最大 10 日間使用できます。 250 GB を超えてバックアップに使用されているストレージについては、[価格モデル](https://azure.microsoft.com/pricing/details/mysql/)に従って請求されます。

Azure portal で使用可能な Azure Monitor の[使用されたバックアップ ストレージ](./concepts-monitoring.md) メトリックを使用して、サーバーによって使用されるバックアップ ストレージを監視できます。 使用済み **バックアップ ストレージ** メトリックは、サーバーに設定されているバックアップ保有期間に基づいて保持されるすべてのデータベース バックアップとログ バックアップによって使用されたストレージの合計を表します。 サーバーで大量のトランザクション アクティビティが発生すると、データベースの合計サイズに関係なく、バックアップ ストレージの使用量が増加する可能性があります。 Geo 冗長サーバーに使用されるバックアップ ストレージは、ローカル冗長サーバーの 2 倍になります。

バックアップ ストレージ コストを制御する主な方法は、適切なバックアップ保有期間を設定することです。 保有期間は 1 日から 35 日の範囲で選択できます。

> [!IMPORTANT]
> ゾーン冗長高可用性構成で構成されたデータベース サーバーからのバックアップは、スナップショット バックアップでのオーバーヘッドが最小限に抑えられるため、プライマリ データベース サーバーから行われます。

## <a name="view-available-full-backups"></a>使用可能な完全バックアップを表示する

Azure portal の [バックアップと復元] ブレードに、毎日 1 回実行される自動完全バックアップが一覧表示されます。 このブレードを使用すると、サーバーの保有期間中に使用可能なすべての完全バックアップの完了タイムスタンプを表示し、これらの完全バックアップを使用して復元操作を実行できます。 使用可能なバックアップの一覧には、保有期間中のすべての自動バックアップ、正常に完了したことを示すタイムスタンプ、バックアップの保持期間を示すタイムスタンプ、復元アクションが含まれます。

## <a name="restore"></a>復元

Azure Database for MySQL で復元を実行すると、元のサーバーのバックアップから新しいサーバーが作成されます。 使用できる復元には 2 つの種類があります。 

- ポイントインタイム リストア: いずれのバックアップ冗長オプションでも使用でき、元のサーバーと同じリージョンに新しいサーバーが作成されます。
- geo リストア: サーバーを geo 冗長ストレージ用に構成した場合にのみ使用でき、ご利用のサーバーを geo ペア リージョンに復元できます。 他のリージョンへの geo リストアは現在サポートされていません。 

サーバーの復旧にかかる推定時間は、次のいくつかの要因によって異なります。 

- データベースのサイズ 
- 関連するトランザクション ログ数 
- 復元時点に復旧するために再生の必要があるアクティビティ量 
- 別のリージョンへの復元の場合は、ネットワーク帯域幅 
- ターゲット リージョンで処理される、同時実行される復元要求の数 
- データベース内のテーブルに主キーが存在するかどうか。 高速復旧を行うには、データベース内のすべてのテーブルに主キーを追加することを検討してください。  


> [!NOTE]
> 高可用性が有効になっているサーバーは、ポイントインタイム リストアと geo リストアの両方を復元した後に、非 HA (高可用性が無効になっている) サーバーになります。 

## <a name="point-in-time-restore"></a>ポイントインタイム リストア

Azure Database for MySQL フレキシブル サーバーでは、ポイントインタイム リストアを実行すると、ソース サーバーと同じリージョンに、フレキシブル サーバーのバックアップから新しいサーバーが作成されます。 コンピューティング レベル、仮想コア数、ストレージ サイズ、バックアップ保有期間、バックアップ冗長オプションについては、元のサーバーの構成で作成されます。 また、仮想ネットワークやファイアウォールなどのタグと設定は、ソース サーバーから継承されます。 復元されたサーバーのコンピューティング レベルとストレージ レベル、構成、セキュリティの設定は、復元が完了した後で変更できます。

> [!NOTE]
> 復元操作の後で既定値にリセットされる (そして、プライマリ サーバーからコピーで上書きされない) サーバー パラメーターが 2 つあります
> *   time_zone - この値は、既定値 SYSTEM に設定されます
> *   event_scheduler - 復元されたサーバーでは event_scheduler は OFF に設定されます

ポイントインタイム リストアは複数のシナリオで役に立ちます。 一般的なユース ケースとしては、次のようなものがあります。
-   ユーザーがデータベース内のデータを誤って削除した場合
-   ユーザーが重要なテーブルまたはデータベースを削除する
-   ユーザー アプリケーションの欠陥のため、正しいデータが不適切なデータで誤って上書きされる。

最新の復元ポイント、カスタム復元ポイント、[Azure portal](how-to-restore-server-portal.md) を介した最速の復元ポイント (完全バックアップを使用した復元) から選択できます。

-   **最新の復元ポイント**: 最新の復元ポイントのオプションは、復元操作がトリガーされたときのタイムスタンプにサーバーを復元するために役立ちます。 このオプションは、サーバーを最も更新された状態にすばやく復元するのに役立ちます。
-   **カスタム復元ポイント**: これを使用すると、このフレキシブル サーバーに定義されている保有期間内の特定の時点を選択できます。 このオプションは、ユーザー エラーから復旧するために、サーバーを正確な時点まで復元する場合に便利です。
-   **最速の復元ポイント**: このオプションを使用すると、ユーザーは、フレキシブル サーバーに対して定義された保有期間内に、可能な限り最速の時間でサーバーを復元できます。 最速の復元は、完全バックアップが完了した時点の復元ポイントを選択することで可能です。 この復元操作では、スナップショットの完全バックアップが復元されるだけで、高速になるログの復元や復旧は保証されません。 復元操作を正常に実行するには、最も古い復元ポイントを超える完全バックアップ タイムスタンプを選択することをお勧めします。

復旧にかかる推定時間は、データベースのサイズ、トランザクション ログのバックアップ サイズ、SKU のコンピューティング サイズ、復元の時刻など、いくつかの要因によって異なります。 トランザクション ログの復旧は、復元プロセスの中で最も時間がかかる部分です。 スナップショット バックアップのスケジュールに近い復元時刻を選択すると、トランザクション ログの適用が最小限になるため、復元操作の速度が速くなります。 サーバーの復旧時間を正確に見積もるには、環境に固有の変数が多すぎるため、実際の環境でテストすることを強くお勧めします。

> [!IMPORTANT]
> ゾーン冗長の高可用性を使用して構成されたフレキシブル サーバーを復元する場合、復元されたサーバーは、プライマリ サーバーと同じリージョンおよびゾーン内に構成され、非 HA モードで単一のフレキシブル サーバーとしてデプロイされます。 フレキシブル サーバーに対する[ゾーン冗長高可用性](concepts-high-availability.md)に関するページを参照してください。

> [!IMPORTANT]
> 削除された MySQL フレキシブル サーバー リソースは、サーバーを削除した時点から 5 日以内であれば復元できます。 削除されたサーバーの復元方法については、手順を示したドキュメントを参照してください (../flexible-server/how-to-restore-dropped-server.md)。 管理者は、デプロイ後の誤削除や予期せぬ変更からサーバーのリソースを保護するために、[管理ロック](../../azure-resource-manager/management/lock-resources.md)を利用できます。

## <a name="geo-restore"></a>geo リストア

geo 冗長バックアップ用にサーバーを構成した場合は、サービスを使用できる [geo ペア リージョン](../../best-practices-availability-paired-regions.md)にサーバーを復元できます。 他のリージョンへの geo リストアは現在サポートされていません。 

geo リストアは、サーバーがホストされているリージョンでのインシデントが原因でサーバーが利用できない場合の既定の復旧オプションです。 リージョン内の大規模なインシデントにより、データベース アプリケーションが使用できなくなった場合、geo 冗長バックアップから他の任意のリージョン内のサーバーに、サーバーを復元できます。 geo リストアでは、サーバーの最新のバックアップが利用されます。 バックアップが取得される時刻と、別のリージョンにそのバックアップがレプリケートされる時刻には時間差があります。 この時間差は最大 1 時間なので、障害が発生した場合、最大 1 時間分のデータが損失する可能性があります。 

geo リストア中に変更できるサーバー構成には、セキュリティ構成 (ファイアウォール規則と仮想ネットワーク設定) のみが含まれます。 geo リストア中に、コンピューティング、ストレージ、価格レベル (Basic、General Purpose、またはメモリ最適化) など、他のサーバー構成を変更することはできません。 

復旧の推定所要時間は、データベースのサイズ、トランザクション ログのサイズ、ネットワーク帯域幅、同じリージョン内で同時に復旧するデータベースの合計数など、複数の要因によって異なります。 

> [!NOTE]
> ゾーン冗長の高可用性を使用して構成されたフレキシブル サーバーを geo 復元する場合、復元されたサーバーは、geo ペア リージョンと、プライマリ サーバーと同じゾーン内に構成され、非 HA モードで単一フレキシブル サーバーとしてデプロイされます。 フレキシブル サーバーに対する[ゾーン冗長高可用性](concepts-high-availability.md)に関するページを参照してください。

> [!IMPORTANT]
> プライマリ リージョンがダウンすると、プライマリ リージョンでストレージをプロビジョニングできなくなるため、geo 冗長サーバーをそれぞれの geo ペア リージョンに作成できません。 geo ペア リージョンで geo 冗長サーバーをプロビジョニングするには、プライマリ リージョンが稼働するまで待機する必要があります。 プライマリ リージョンがダウンしていても、ソース サーバーを geo ペア リージョンに geo リストアすることはできます。それには、復元ポータル操作により [コンピューティングとストレージ] の [サーバーの構成] 設定で geo 冗長オプションを無効にし、ローカル冗長サーバーとして復元してビジネス継続性を確保します。  

## <a name="perform-post-restore-tasks"></a>復元後のタスクの実行

**最新の復元ポイント** または **カスタム復元ポイント** のいずれかの復旧メカニズムで復元した後、ユーザーとアプリケーションを元に戻して実行するには、次のタスクを実行する必要があります。

-   元のサーバーを新しいサーバーで置き換える場合は、クライアントとクライアント アプリケーションを新しいサーバーにリダイレクトします。
-   ユーザーが接続できるように、サーバー レベルの適切なファイアウォール規則と仮想ネットワーク規則が適用されていることを確認します。
-   適切なログインとデータベース レベルのアクセス許可が指定されていることを確認します。
-   必要に応じて、アラートを構成する。

## <a name="frequently-asked-questions-faqs"></a>よく寄せられる質問 (FAQ)

### <a name="backup-related-questions"></a>バックアップに関連する質問

- **サーバーをバックアップするにはどうすればよいですか。**
既定では、Azure Database for MySQL によって、サーバー全体 (作成されたすべてのデータベースを含む) の自動バックアップが有効になります。保有期間は、既定では 7 日間です。 バックアップを手動で作成する唯一の方法は、コミュニティ ツールを使用することです。たとえば、mysqldump (ドキュメントは[こちら](../concepts-migrate-dump-restore.md#dump-and-restore-using-mysqldump-utility))、mydumper (ドキュメントは[こちら](../concepts-migrate-mydumper-myloader.md#create-a-backup-using-mydumper)) などを使用します。 Azure Database for MySQL を BLOB ストレージにバックアップする場合は、「[Azure Database for MySQL を BLOB ストレージにバックアップする](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/backup-azure-database-for-mysql-to-a-blob-storage/ba-p/803830)」という Tech Community ブログを参照してください。

- **自動バックアップが長期間保有されるように構成できますか。**
いいえ。現在サポートされているのは、最大 35 日間の自動バックアップ保有のみです。 手動バックアップを作成して、それを長期保有の要件に対して使用することができます。

- **サーバーのバックアップの時間帯はどうなっていますか。カスタマイズできますか。**
初回のスナップショット バックアップは、サーバーの作成直後にスケジュールされます。 スナップショット バックアップは、毎日 1 回作成されます。 トランザクション ログ バックアップは 5 分ごとに実行されます。 バックアップの時間帯は本質的に Azure によって管理されており、カスタマイズできません。

- **バックアップは暗号化されますか?**
クエリの実行中に作成されたすべての Azure Database for MySQL データ、バックアップ、一時ファイルは、AES 256 ビット暗号化を使用して暗号化されます。 ストレージの暗号化は常にオンになっており、無効にすることはできません。

- **単一または少数のデータベースを復元できますか。**
単一または少数のデータベースおよびテーブルの復元はサポートされていません。 特定のデータベースを復元する場合は、ポイントインタイム リストアを実行してから、必要なテーブルまたはデータベースを抽出します。

- **バックアップの時間帯にサーバーを使用できますか。**
はい。 バックアップはオンライン操作であり、スナップショットベースです。 スナップショット操作には数秒しかかからず、運用環境のワークロードには影響しないため、サーバーの高可用性が確保されます。

- **サーバーのメンテナンス期間を設定するときに、バックアップの時間帯を考慮する必要がありますか。**
いいえ。バックアップは管理サービスの一部として内部的にトリガーされ、管理されたメンテナンス期間には関係しません。

- **自動バックアップはどこに格納され、その保有期間はどのように管理できますか。**
Azure Database for MySQL では、サーバーのバックアップが自動的に作成され、ユーザーが構成したローカル冗長ストレージまたは geo 冗長ストレージに格納されます。 これらのバックアップ ファイルをエクスポートすることはできません。 バックアップの既定のリテンション期間は 7 日です。 必要に応じて、1 から 35 日の範囲でデータベース バックアップを構成できます。

- **バックアップを検証するにはどうすればよいですか。**
正常に完了したバックアップの可用性を検証する最善の方法は、[バックアップと復元] ブレードで、保有期間中に作成された完全な自動バックアップを表示する方法です。 バックアップが失敗した場合、バックアップは使用可能なバックアップの一覧に表示されません。バックアップ サービスは、バックアップが正常に作成されるまで 20 分ごとにバックアップを試します。 これらのバックアップ エラーは、サーバーでのトランザクションの実稼働負荷が高い場合に発生します。

- **バックアップの使用量はどこで確認できますか。**
Azure portal の [監視] タブの [メトリック] セクションに、[使用済みバックアップ ストレージ](./concepts-monitoring.md)のメトリックがあります。これは、バックアップの使用量の合計を監視するために役立ちます。

- **サーバーを削除した場合、バックアップはどうなりますか。**
サーバーを削除すると、そのサーバーに属するバックアップもすべて削除され、復元できなくなります。 管理者は、デプロイ後の誤削除や予期せぬ変更からサーバーのリソースを保護するために、[管理ロック](../../azure-resource-manager/management/lock-resources.md)を利用できます。

- **バックアップの使用に対して、どのように課金および請求が行われますか。**
フレキシブル サーバーを使用すると、プロビジョニングされているサーバー ストレージの最大 100% までが、バックアップ ストレージとして追加コストなしで提供されます。 バックアップ ストレージを追加で使用した場合は、[価格モデル](https://azure.microsoft.com/pricing/details/mysql/server/)に基づき、1 か月ごとに GB 単位で請求されます。 また、バックアップ ストレージの課金には、使用されるバックアップ ストレージの合計使用量に直接影響するサーバー上のトランザクション アクティビティとは別に、選択されたバックアップ保有期間と、選択されたバックアップ冗長性オプションも関与します。

- **停止したサーバーのバックアップはどのように保有されますか。**
停止したサーバーに対して、新しいバックアップは実行されません。 サーバーを停止した時点でのすべての古いバックアップ (保有期間内のもの) は、サーバーが再起動されるまで保有されます。その後、アクティブなサーバーのバックアップの保有は、そのサーバーのバックアップ保有期間によって決まります。

- **停止しているサーバーのバックアップについては、どのように請求されますか。**
サーバー インスタンスが停止している間は、プロビジョニングされたストレージ (プロビジョニングされた IOPS を含む) とバックアップ ストレージ (指定した保有期間内の格納されているバックアップ) に対して課金されます。 無料バックアップ ストレージは、プロビジョニングしたデータベースのサイズに制限され、アクティブなサーバーにのみ適用されます。

### <a name="restore-related-questions"></a>復元に関連する質問

- **サーバーを復元するにはどうすればよいですか。**
Azure portal ではポイントインタイム リストアがサポートされ (すべてのサーバーが対象)、ユーザーは最新またはカスタムの復元ポイントに復元できます。 mysqldump/myDumper によって作成されたバックアップからサーバーを手動で復元するには、「[myLoader を使用してデータベースを復元する](../concepts-migrate-mydumper-myloader.md#restore-your-database-using-myloader)」を参照してください。

- **復元に長時間かかるのはなぜですか。**
サーバーの復旧にかかる推定時間は、次のいくつかの要因によって異なります。
   - データベースのサイズ。 復旧プロセスの一環として、データベースを最新の物理バックアップからハイドレートする必要があるため、復旧にかかる時間はデータベースのサイズに比例します。
   - 復旧のために再生する必要があるトランザクション アクティビティのアクティブな部分。 最後に成功したチェックポイント以降の追加のトランザクション アクティビティによっては、復旧に時間がかかる場合があります。
   - 別のリージョンへの復元の場合は、ネットワーク帯域幅
   - ターゲット リージョンで処理される、同時実行される復元要求の数
   - データベース内のテーブルに主キーが存在するかどうか。 高速復旧を行うには、データベース内のすべてのテーブルに主キーを追加することを検討してください。


## <a name="next-steps"></a>次のステップ

-   [ビジネス継続性](./concepts-business-continuity.md)について確認する
-   [ゾーン冗長による高可用性](./concepts-high-availability.md)について確認する
-   [バックアップと復旧](./concepts-backup-restore.md)について確認する
