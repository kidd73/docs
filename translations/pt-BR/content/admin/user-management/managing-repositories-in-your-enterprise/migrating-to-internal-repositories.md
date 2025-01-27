---
title: Migrando para repositórios internos
intro: 'Você pode migrar para repositórios internos para unificar a experiência interna para desenvolvedores usando {% data variables.product.prodname_ghe_server %} e {% data variables.product.prodname_ghe_cloud %}.'
redirect_from:
  - /enterprise/admin/installation/migrating-to-internal-repositories
  - /enterprise/admin/user-management/migrating-to-internal-repositories
  - /admin/user-management/migrating-to-internal-repositories
permissions: Site administrators can migrate to internal repositories.
versions:
  ghes: '*'
type: how_to
topics:
  - Enterprise
  - Privacy
  - Repositories
  - Security
shortTitle: Internal repository migration
ms.openlocfilehash: 66a535d8fd2e20cbcc78791588ca2b50ae8ede79
ms.sourcegitcommit: 47bd0e48c7dba1dde49baff60bc1eddc91ab10c5
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 09/05/2022
ms.locfileid: '145094870'
---
## Sobre repositórios internos

Os repositórios internos estão disponíveis em {% data variables.product.prodname_ghe_server %} 2.20+. {% data reusables.repositories.about-internal-repos %} Para obter mais informações, confira "[Sobre os repositórios](/repositories/creating-and-managing-repositories/about-repositories#about-repository-visibility)".

Em versões futuras do {% data variables.product.prodname_ghe_server %}, ajustaremos como a visibilidade do repositório funciona para que os termos público, interno e privado tenham significados uniformes para desenvolvedores em {% data variables.product.prodname_ghe_server %} e {% data variables.product.prodname_ghe_cloud %}.

Para se preparar para essas alterações, se você tiver o modo privado ativado, é possível executar uma migração em sua instância para converter repositórios públicos em internos. Essa migração é atualmente opcional, para permitir que você teste as mudanças em uma instância não produtiva. A migração será obrigatória no futuro.

Quando você efetuar a migração, todos os repositórios públicos pertencentes a organizações na sua instância se tornarão repositórios internos. Se qualquer um desses repositórios tiver bifurcações, as bifurcações vão se tornar privadas. Repositórios privados permanecerão privados.

Todos os repositórios públicos pertencentes a contas de usuário na sua instância se tornarão repositórios privados. Se qualquer um desses repositórios tiver bifurcações, as bifurcações vão se tornar privadas. O proprietário de cada bifurcação receberá permissões de leitura para o principal da bifurcação.

O acesso de leitura anônimo Git será desativado para cada repositório público que se tornar interno ou privado.

Se sua visibilidade padrão atual for pública, o padrão se tornará interno. Se o padrão atual for privado, o padrão não será alterado. Você pode alterar o padrão a qualquer momento. Para obter mais informações, confira "[Como impor políticas de gerenciamento de repositório na sua empresa](/admin/policies/enforcing-repository-management-policies-in-your-enterprise#configuring-the-default-visibility-of-new-repositories-in-your-enterprise)".

A política de criação de repositórios para a instância mudará para desativar repositórios públicos e permitir repositórios privados e internos. Você pode atualizar a política a qualquer momento. Para obter mais informações, confira "[Como restringir a criação de repositórios nas suas instâncias](/enterprise/admin/user-management/restricting-repository-creation-in-your-instance)".

Se você não tiver o modo privado ativado, o script de migração não terá efeito.

## Executando a migração

1. Conecte-se ao shell administrativo. Para obter mais informações, confira "[Como acessar o shell administrativo (SSH)](/enterprise/admin/installation/accessing-the-administrative-shell-ssh)".
{% ifversion ghes or ghae %}
2. Execute o comando de migração.

   ```shell
   github-env bin/safe-ruby lib/github/transitions/20191210220630_convert_public_ghes_repos_to_internal.rb --verbose -w |  tee -a /tmp/convert_public_ghes_repos_to_internal.log
   ```

{% else %}
2. Navegue até o diretório `/data/github/current`.
   ```shell
   cd /data/github/current
   ```
3. Execute o comando de migração.
   ```shell
   sudo bin/safe-ruby lib/github/transitions/20191210220630_convert_public_ghes_repos_to_internal.rb --verbose -w | tee -a /tmp/convert_public_ghes_repos_to_internal.log
   ```
{% endif %}

A saída do log será exibida no terminal e em `/tmp/convert_public_ghes_repos_to_internal.log`.

## Leitura adicional

- "[Como habilitar o modo privado](/enterprise/admin/installation/enabling-private-mode)"
