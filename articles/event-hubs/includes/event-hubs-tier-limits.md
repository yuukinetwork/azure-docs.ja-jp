---
title: インクルード ファイル description: インクルード ファイル services: event-hubs author: spelluru ms.service: event-hubs ms.topic: include ms.date: 11/04/2021 ms.author: spelluru ms.custom: "include file","fasttrack-edit","iot","event-hubs"

---

次の表は、Basic、Standard、Dedicated の各レベルで異なる可能性がある制限を示しています。 この表で、CU は[容量ユニット](../event-hubs-dedicated-overview.md)、PU は[処理装置](../event-hubs-scalability.md#processing-units)、TU は[スループット ユニット](../event-hubs-scalability.md#throughput-units)です。 

| 制限 | Basic | Standard | Premium |  専用 |
| ----- | ----- | -------- | -------- | --------- | 
| Event Hubs 発行の最大サイズ | 256 KB | 1 MB | 1 MB |  1 MB |
| イベント ハブあたりのコンシューマー グループの数 | 1 | 20 | 100 | 1000<br/>CU あたりの制限なし  |
| 名前空間あたりの仲介型接続の数 | 100 | 5,000 | PU あたり処理装置あたり 10000 | CU あたり 100, 000 |
| イベント データの最大リテンション期間 | 1 日 | 7 日 | 90 日間<br/>PU あたり 1 TB | 90 日間<br/>CU あたり 10 TB |
| 最大 TU または PU または CU | 40 TU | 40 TU | 16 PU | 20 CU |
| イベント ハブあたりのパーティションの数 | 32 | 32 | イベントハブあたり 100、PU あたり 200 | イベント ハブあたり 1,024<br/> CU あたり 2,000 |
| サブスクリプションあたりの名前空間の数 | 1000 | 1000 | 1000 | 1000 (CU あたり 50) |
| 名前空間あたりのイベント ハブの数 | 10 | 10 | PU あたり 100 | 1000 |
| キャプチャ | なし | 1 時間ごとの課金 | Included | Included |
| スキーマ レジストリ (名前空間) のサイズ (MB) | なし | 25 | 100 | 1024 |
| スキーマ レジストリまたは名前空間内のスキーマ グループの数 | なし | 1 (既定のグループは除く) | 100 <br/>スキーマあたり 1 MB | 1000<br/>スキーマあたり 1 MB |
| スキーマ グループ全体におけるスキーマ バージョンの数 | なし | 25 | 1000 | 10000 |
| ユニットあたりのスループット | イングレス - 1 MB/秒または 1000 イベント/秒<br/>エグレス - 2 MB/秒または 4096 イベント/秒 | イングレス - 1 MB/秒または 1000 イベント/秒<br/>エグレス - 2 MB/秒または 4096 イベント/秒 | PU あたりの制限なし * | CU あたりの制限なし * |

\* リソースの割り当て、パーティションの数、ストレージなど、さまざまな要因によって異なります。 
 

> [!NOTE]
> イベントは、個別に発行することもバッチ処理することもできます。 発行の制限 (SKU に基づく) は、1 つのイベントであるかバッチであるかに関係なく適用されます。 最大しきい値を超えるイベントの発行は拒否されます。

