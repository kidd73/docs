---
title: アクションの終了コードの設定
shortTitle: Setting exit codes
intro: '終了コードを使って、アクションのステータスを設定できます。 {% data variables.product.prodname_dotcom %}は、パスした、あるいは失敗したアクションを示すステータスを表示します。'
redirect_from:
  - /actions/building-actions/setting-exit-codes-for-actions
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: how_to
ms.openlocfilehash: 28aecc646814736beb8c814dfe4b8385a6605cd2
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2022
ms.locfileid: '145088631'
---
{% data reusables.actions.enterprise-beta %} {% data reusables.actions.enterprise-github-hosted-runners %}

## 終了コードについて

{% data variables.product.prodname_dotcom %} では、終了コードを使用してアクションのチェック実行ステータスを設定します。ステータスは `success` か `failure` です。

終了ステータス | チェック実行ステータス | 説明
------------|------------------|------------
`0` | `success` | アクションが正常に完了し、それに依存する他のタスクを開始できます。
ゼロ以外の値 (0 以外の任意の整数)| `failure` | その他の終了コードは、アクションの失敗を表します。 アクションが失敗すると、同時に実行されていたアクションはすべてキャンセルされ、今後のアクションはスキップされます。 チェック実行とチェック スイートの両方に `failure` ステータスが与えられます。

## JavaScript アクションで失敗終了を設定する

JavaScript のアクションを作成する場合は、アクション ツールキット [`@actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) パッケージを使用してメッセージをログに記録し、エラー終了コードを設定できます。 次に例を示します。

```javascript
try {
  // something
} catch (error) {
  core.setFailed(error.message);
}
```

詳細については、「[JavaScript アクションを作成する](/articles/creating-a-javascript-action)」を参照してください。

## Docker コンテナアクションで失敗終了コードを設定する

Docker コンテナー アクションを作成する場合は、`entrypoint.sh` スクリプトにエラー終了コードを設定できます。 次に例を示します。

```
if <condition> ; then
  echo "Game over!"
  exit 1
fi
```

詳細については、「[Docker コンテナー アクションを作成する](/articles/creating-a-docker-container-action)」を参照してください。
