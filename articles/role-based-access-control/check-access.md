---
title: Início rápido – Ver o acesso que um usuário tem aos recursos do Azure – RBAC do Azure
description: Neste Início rápido, você aprende a ver o acesso que um usuário ou outra entidade de segurança tem aos recursos do Azure usando o portal do Azure e o RBAC do Azure (controle de acesso baseado em função do Azure).
services: role-based-access-control
documentationCenter: ''
author: rolyon
manager: mtillman
editor: ''
ms.service: role-based-access-control
ms.devlang: ''
ms.topic: quickstart
ms.tgt_pltfrm: ''
ms.workload: identity
ms.date: 11/30/2018
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 9be53aa964e75bab0b90495640537fe927a5af0e
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/05/2020
ms.locfileid: "82734154"
---
# <a name="quickstart-view-the-access-a-user-has-to-azure-resources"></a>Início Rápido: Exibir o acesso que um usuário tem aos recursos do Azure

Use a folha **Controle de acesso (IAM)** no [RBAC do Azure (controle de acesso baseado em função do Azure)](overview.md) para ver o acesso que um usuário ou outra entidade de segurança tem aos recursos do Azure. No entanto, às vezes, você apenas precisa exibir rapidamente o acesso de um único usuário ou outra entidade de segurança. A maneira mais fácil de fazer isso é usar o recurso **Verificar acesso** no portal do Azure.

## <a name="view-role-assignments"></a>Exibir atribuições de função

 A forma como o acesso de um usuário é exibido é uma lista de suas atribuições de funções. Siga essas etapas para exibir as atribuições de função para um único usuário, grupo, entidade de serviço ou identidade gerenciada no escopo de assinaturas.

1. No portal do Microsoft Azure, clique em **Todos os serviços** e, em seguida, em **Assinaturas**.

1. Clique em sua assinatura.

1. Clique em **Controle de acesso (IAM)** .

1. Clique na guia **Verificar acesso**.

    ![Controle de acesso - verificar a guia de acesso](./media/check-access/access-control-check-access.png)

1. Na lista **Localizar**, selecione o tipo de entidade de segurança para a qual você deseja verificar o acesso.

1. Na caixa de pesquisa, insira uma cadeia de caracteres para pesquisar no diretório nomes de exibição, endereços de e-mail ou identificadores de objetos.

    ![Verifique a lista de seleção de acesso](./media/check-access/check-access-select.png)

1. Clique na entidade de segurança para abrir o painel **atribuições**.

    ![painel atribuições](./media/check-access/check-access-assignments.png)

    Nesse painel, você pode ver as funções atribuídas à entidade de segurança selecionada e o escopo. Se houver alguma atribuição de negação nesse escopo ou herdada para esse escopo, ela será listada.

## <a name="next-steps"></a>Próximas etapas

> [!div class="nextstepaction"]
> [Tutorial: Conceder acesso aos recursos do Azure para um usuário usando o portal do Azure](quickstart-assign-role-user-portal.md)
