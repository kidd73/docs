---
ms.openlocfilehash: 3492de2cd163b4bbb59b912c17d152b7d2af5c68
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/05/2022
ms.locfileid: "145113753"
---
### Búsqueda basada en la operación

Use el calificador `operation` para limitar las acciones a tipos específicos de operaciones. Por ejemplo:

  * `operation:access` busca todos los eventos en los que se ha accedido a un recurso.
  * `operation:authentication` busca todos los eventos en los que se ha realizado un evento de autenticación.
  * `operation:create` busca todos los eventos en los que se ha creado un recurso.
  * `operation:modify` busca todos los eventos en los que se ha modificado un recurso existente.
  * `operation:remove` busca todos los eventos en los que se ha quitado un recurso existente.
  * `operation:restore` busca todos los eventos en los que se ha restaurado un recurso existente.
  * `operation:transfer` busca todos los eventos en los que se ha transferido un recurso existente.
