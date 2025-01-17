---
title: 言語サポート - カスタム質問と回答
titleSuffix: Azure Cognitive Services
description: ナレッジ ベースのカスタム質問と回答でサポートされるカルチャと自然言語の一覧。 同じナレッジ ベースに複数の言語を混在させないでください。
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: language-service
ms.topic: reference
ms.date: 11/02/2021
ms.custom: language-service-question-answering, ignite-fall-2021
ms.openlocfilehash: dbef13bdb39085c650a1fc5cedceb6c143eea4ad
ms.sourcegitcommit: 702df701fff4ec6cc39134aa607d023c766adec3
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/03/2021
ms.locfileid: "131450117"
---
# <a name="language-support-for-custom-question-answering-and-knowledge-bases"></a>カスタム質問と回答、およびナレッジ ベースの言語サポート

この記事では、カスタム質問と回答が有効なリソースとナレッジ ベースの言語サポート オプションについて説明します。

カスタム質問と回答では、複数の言語をサポートしているリソースに新しいプロジェクトを追加するたびに言語を選択するか、リソースの今後のすべてのプロジェクトに適用される言語を選択するかの、どちらかを選ぶことができます。

## <a name="supporting-multiple-languages-in-one-custom-question-answering-enabled-resource"></a>1 つのカスタム質問と回答が有効なリソースでの複数の言語のサポート

> [!div class="mx-imgBorder"]
> ![多言語ナレッジベースの選択](./media/language-support/choose-language.png)

* サービスで最初のプロジェクトを作成している場合は、新しいプロジェクトを作成するたびに言語を選択できます。 1 つのサービス内でさまざまな言語に属するナレッジ ベースを作成するには、このオプションをオンにします。
* 最初のナレッジ ベースが作成されたら、そのサービスの言語設定オプションを変更することはできません。
* ナレッジ ベースで複数言語の使用を有効にする場合、サービスに 1 つではなく、ナレッジ ベースごとに 1 つのテスト インデックスを使用します。

## <a name="supporting-multiple-languages-in-one-knowledge-base"></a>1 つのナレッジ ベースで複数の言語をサポートする

複数の言語を含むナレッジ ベース システムをサポートする必要がある場合は、次を行うことができます。

* ナレッジ ベースに質問を送信する前に、[Translator サービス](../../translator/translator-info-overview.md)を使用して、質問を 1 つの言語に翻訳します。 これで、1 つの言語の品質と、代替の質問と回答の品質に集中できるようになります。
* すべての言語に対して、カスタム質問と回答が有効な言語リソースと、そのリソース内のナレッジ ベースを作成します。 これで、言語ごとに、より微妙な個別の代替の質問と回答のテキストを管理できるようになります。 これにより、柔軟性が大幅に向上しますが、すべての言語で質問または回答が変わると、メンテナンス コストが大幅に増加します。

## <a name="single-language-per-resource"></a>リソースごとに 1 つの言語

**リソースに関連付けられているすべてのプロジェクトで使用される言語を設定するオプションを選択** する場合は、次の点を考慮してください。 
* 言語リソースとそのすべてのプロジェクトとナレッジ ベースは、1 つの言語のみをサポートします。
* 言語は、サービスの最初のプロジェクトが作成されるときに明示的に設定されます。
* リソースに関連付けられている他のプロジェクトでは、言語を変更できません。
* 言語は、Cognitive Search サービス (ランカー #1) とカスタム質問と回答 (ランカー #2) によって、クエリに対するベスト アンサーを生成するために使用されます。

## <a name="languages-supported"></a>サポートされている言語

次の一覧では、質問応答リソースに対してサポートされている言語を示します。

| Language |
|--|
| アラビア語 |
| アルメニア語 |
| ベンガル語 |
| バスク語 |
| ブルガリア語 |
| カタロニア語 |
| 簡体中国語 |
| 繁体字中国語 |
| クロアチア語 |
| チェコ語 |
| デンマーク語 |
| オランダ語 |
| 英語 |
| エストニア語 |
| フィンランド語 |
| フランス語 |
| ガリシア語 |
| ドイツ語 |
| ギリシャ語 |
| グジャラート語 |
| ヘブライ語 |
| ヒンディー語 |
| ハンガリー語 |
| アイスランド語 |
| インドネシア語 |
| アイルランド語 |
| イタリア語 |
| 日本語 |
| カンナダ語 |
| 韓国語 |
| ラトビア語 |
| リトアニア語 |
| マラヤーラム語 |
| マレー語 |
| ノルウェー語 |
| ポーランド語 |
| Portuguese |
| パンジャーブ語 |
| ルーマニア語 |
| ロシア語 |
| セルビア語 (キリル) |
| セルビア語 (ラテン) |
| スロバキア語 |
| スロベニア語 |
| スペイン語 |
| スウェーデン語 |
| タミル語 |
| テルグ語 |
| タイ語 |
| トルコ語 |
| ウクライナ語 |
| ウルドゥ語 |
| ベトナム語 |

## <a name="query-matching-and-relevance"></a>クエリの一致と関連性
カスタム質問と回答による結果の提供は、[Azure Cognitive Search の言語アナライザー](/rest/api/searchservice/language-support)に依存しています。

Azure Cognitive Search の機能はサポートされている言語に従いますが、質問と回答には Azure Search の結果より上位に追加のランカーがあります。 このランカー モデルでは、以下の言語においていくつかの特別なセマンティックとワード ベースの機能が使用されます。

|追加のランカーがある言語|
|--|
|中国語|
|チェコ語|
|オランダ語|
|英語|
|フランス語|
|ドイツ語|
|ハンガリー語|
|イタリア語|
|日本語|
|韓国語|
|ポーランド語|
|Portuguese|
|スペイン語|
|スウェーデン語|

この追加のランク付けは、カスタム質問と回答のランカーの内部的な作業です。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [言語の選択](../index.yml)
