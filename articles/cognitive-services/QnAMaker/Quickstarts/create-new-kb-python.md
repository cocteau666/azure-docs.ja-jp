---
title: 'クイック スタート: ナレッジ ベースを作成する - REST (Python) - QnA Maker'
description: この Python REST ベースのクイック スタートでは、Cognitive Services API アカウントの Azure ダッシュボードに表示される QnA Maker ナレッジ ベースのサンプルをプログラムから作成する手順を紹介しています。
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 12/16/2019
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-python
ms.topic: how-to
ms.openlocfilehash: 9f3f433742ec25a1ee1abb2ede32a38e6b611f14
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352288"
---
# <a name="quickstart-create-a-knowledge-base-in-qna-maker-using-python"></a>クイック スタート: Python を使用して QnA Maker のナレッジ ベースを作成する

このクイック スタートでは、QnA Maker ナレッジ ベースのサンプルをプログラムから作成して発行する手順を紹介しています。 QnA Maker は、[データ ソース](../index.yml)の FAQ などの半構造化コンテンツから質問とその回答を自動的に抽出します。 ナレッジ ベースのモデルは、API 要求の本文で送信される JSON で定義されます。

このクイック スタートで呼び出す QnA Maker API は次のとおりです。
* [KB の作成](/rest/api/cognitiveservices/qnamaker/knowledgebase/create)
* [取得操作の詳細](/rest/api/cognitiveservices/qnamaker/operations/getdetails)

[リファレンス ドキュメント](/rest/api/cognitiveservices/qnamaker/knowledgebase) | [Python サンプル](https://github.com/Azure-Samples/cognitive-services-qnamaker-python/blob/master/documentation-samples/quickstarts/create-knowledge-base/create-new-knowledge-base-3x.py)

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>前提条件

* [Python 3.7](https://www.python.org/downloads/)
* [QnA Maker サービス](../How-To/set-up-qnamaker-service-azure.md)が必要です。 キーと (リソース名を含む) エンドポイントを取得するには、Azure portal で対象のリソースの **[クイックスタート]** を選択します。

## <a name="create-a-knowledge-base-python-file"></a>ナレッジ ベースの Python ファイルを作成する

`create-new-knowledge-base-3x.py` という名前でファイルを作成します。

## <a name="add-the-required-dependencies"></a>必要な依存関係を追加する

`create-new-knowledge-base-3x.py` の先頭に次の行を追加して、プロジェクトに必要な依存関係を追加します。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="dependencies":::

## <a name="add-the-required-constants"></a>必要な定数を追加する
上記の必要な依存関係の後に、QnA Maker にアクセスするために必要な定数を追加します。 `<your-qna-maker-subscription-key>` と `<your-resource-name>` の値を対象の QnA Maker のキーとリソース名に置き換えます。

Program クラスの上部に、QnA Maker にアクセスするために必要な定数を追加します。

次の値を設定します。

* `<your-qna-maker-subscription-key>` - この **キー** は 32 文字の文字列で、Azure portal の [クイックスタート] ページの QnA Maker リソースで入手できます。 これは、予測エンドポイント キーと同じではありません。
* `<your-resource-name>` - この **リソース名** を使用して、`https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com` 形式の作成エンドポイントの URL が構築されます。 これは、予測エンドポイントを照会するときに使用した URL と同じではありません。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="constants":::

## <a name="add-the-kb-model-definition"></a>KB モデル定義を追加する

定数の後に、次の KB モデル定義を追加します。 モデルは、定義の後に文字列に変換されます。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="model":::

## <a name="add-supporting-function"></a>補助的な関数を追加する

JSON を読みやすい形式で出力するために、次の関数を追加します。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="pretty":::

## <a name="add-function-to-create-kb"></a>KB を作成するための関数を追加する

ナレッジ ベースを作成するための HTTP POST 要求を行う次の関数を追加します。
この API 呼び出しは、ヘッダー フィールド **Location** に操作 ID を含んだ JSON 応答を返します。 この操作 ID を使用して、KB が正常に作成されたかどうかを判断します。 `Ocp-Apim-Subscription-Key` は、認証に使用される QnA Maker サービス キーです。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="create_kb":::

この API 呼び出しは、操作 ID を含んだ JSON 応答を返します。 この操作 ID を使用して、KB が正常に作成されたかどうかを判断します。

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:19:01Z",
  "lastActionTimestamp": "2018-09-26T05:19:01Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "8dfb6a82-ae58-4bcb-95b7-d1239ae25681"
}
```

## <a name="add-function-to-check-creation-status"></a>作成状態を確認するための関数を追加する

次の関数は、URL ルートの最後に操作 ID を追加して送信することにより、作成状態を確認します。 `check_status` 呼び出しは、main の _while_ ループ内に存在します。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="get_status":::

この API 呼び出しは、操作の状態を含んだ JSON 応答を返します。

```JSON
{
  "operationState": "NotStarted",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:22:53Z",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

成功または失敗が返されるまで、この呼び出しを繰り返します。

```JSON
{
  "operationState": "Succeeded",
  "createdTimestamp": "2018-09-26T05:22:53Z",
  "lastActionTimestamp": "2018-09-26T05:23:08Z",
  "resourceLocation": "/knowledgebases/XXX7892b-10cf-47e2-a3ae-e40683adb714",
  "userId": "XXX9549466094e1cb4fd063b646e1ad6",
  "operationId": "177e12ff-5d04-4b73-b594-8575f9787963"
}
```

## <a name="add-main-code-block"></a>main コード ブロックを追加する
次のループは、作成操作が完了するまで、その操作状態を定期的にポーリングします。

:::code language="python" source="~/cognitive-services-quickstart-code/python/QnAMaker/rest/create-kb.py" id="main":::

## <a name="build-and-run-the-program"></a>プログラムをビルドして実行する

コマンド ラインで次のコマンドを入力して、プログラムを実行します。 QnA Maker API に要求が送信され、KB が作成された後、30 秒おきに結果がポーリングされます。 それぞれの応答は、コンソール ウィンドウに出力されます。

```bash
python create-new-knowledge-base-3x.py
```

作成されたナレッジ ベースは、QnA Maker ポータルの [[My knowledge bases]\(マイ ナレッジ ベース\)](https://www.qnamaker.ai/Home/MyServices) ページで確認できます。 ナレッジ ベースの名前 (QnA Maker FAQ など) を選択すると表示されます。

[!INCLUDE [Clean up files and KB](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [QnA Maker (V4) REST API リファレンス](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)