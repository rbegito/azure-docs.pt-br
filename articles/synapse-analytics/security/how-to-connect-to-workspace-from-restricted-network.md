---
title: Conectar-se ao recurso de espaço de trabalho do Synapse Studio de uma rede restrita
description: Este artigo ensinará como se conectar aos recursos de espaço de trabalho do Azure Synapse Studio de uma rede restrita
author: xujxu
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 10/25/2020
ms.author: xujiang1
ms.reviewer: jrasnick
ms.openlocfilehash: f2d8953ccae1057d7a7aa2d786fb7b641b3f6284
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392496"
---
# <a name="connect-to-synapse-studio-workspace-resources-from-a-restricted-network"></a>Conectar-se a recursos de espaço de trabalho do Synapse Studio de uma rede restrita

O leitor de destino deste artigo é o administrador de ti da empresa que está gerenciando a rede restrita da empresa. O administrador de ti está prestes a habilitar a conexão de rede entre o Azure Synapse Studio e a estação de trabalho dentro dessa rede restrita.

Neste artigo, você aprenderá a se conectar ao seu espaço de trabalho Synapse do Azure de um ambiente de rede restrito. 

## <a name="prerequisites"></a>Pré-requisitos

* **Assinatura do Azure** : Caso você não tenha uma assinatura do Azure, crie uma [conta gratuita do Azure](https://azure.microsoft.com/free/) antes de começar.
* **Espaço de trabalho Synapse do Azure** : se você não tiver um Synapse Studio, crie um espaço de trabalho do Synapse do Azure Synapse Analytics. O nome do espaço de trabalho será necessário na etapa 4 a seguir.
* **Uma rede restrita** : a rede restrita é mantida pelo administrador de ti da empresa. o administrador de ti tem a permissão para configurar a política de rede. O nome da rede virtual e sua sub-rede serão necessários na etapa 3 a seguir.


## <a name="step-1-add-network-outbound-security-rules-to-the-restricted-network"></a>Etapa 1: adicionar regras de segurança de saída de rede à rede restrita

Você precisará adicionar quatro regras de segurança de saída de rede com quatro marcas de serviço. Saiba mais sobre a [visão geral das marcas de serviço](/azure/virtual-network/service-tags-overview.md) 
* AzureResourceManager
* AzureFrontDoor.Frontend
* AzureActiveDirectory
* AzureMonitor (opcional. Adicione este tipo de regra somente quando desejar compartilhar os dados para a Microsoft.)

**Azure Resource Manager** detalhes da regra de saída como abaixo. Quando você estiver criando as outras três regras, substitua o valor de " **marca de serviço de destino** " por escolher o nome da marca de serviço " **AzureFrontDoor. frontend** ", " **AzureActiveDirectory** ", " **AzureMonitor** " na lista suspensa de seleção.

![AzureResourceManager](./media/how-to-connect-to-workspace-from-restricted-network/arm-servicetag.png)


## <a name="step-2-create-azure-synapse-analytics-private-link-hubs"></a>Etapa 2: criar a análise de Synapse do Azure (hubs de link privado)

Você precisará criar uma análise de Synapse do Azure (hubs de link privado) de portal do Azure. Pesquise " **Azure Synapse Analytics (hubs de link privado)** " por meio do portal do Azure e, em seguida, preencha o campo necessário e crie-o. 

> [!Note]
> A região deve ser a mesma que a do seu espaço de trabalho Synapse.

![Criando hubs de link privado do Synapse Analytics](./media/how-to-connect-to-workspace-from-restricted-network/private-links.png)

## <a name="step-3-create-private-endpoint-for-synapse-studio-gateway"></a>Etapa 3: criar um ponto de extremidade privado para o gateway do Synapse Studio

Para acessar o gateway do Synapse Studio, você precisará criar um ponto de extremidade privado de portal do Azure. Pesquise " **link privado** " por meio do portal do Azure. Selecione " **criar ponto de extremidade privado** " no " **centro de vínculo privado** " e, em seguida, preencha o campo necessário e crie-o. 

> [!Note]
> A região deve ser a mesma que a do seu espaço de trabalho Synapse.

![Criando ponto de extremidade privado para o Synapse Studio 1](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-1.png)

Na próxima guia de " **recurso** ", escolha o Hub de link privado, que foi criado na etapa 2 acima.

![Criando um ponto de extremidade privado para o Synapse Studio 2](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-2.png)

Na próxima guia de " **configuração** ", 
* Escolha o nome da rede virtual restrita que você tem para " **rede virtual** ".
* Escolha a sub-rede da rede virtual restrita para " **subnet** ". 
* Selecione " **Sim** " para " **integrar com a zona DNS privada** ".

![Criando um ponto de extremidade privado para o Synapse Studio 3](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-3.png)

Depois que o ponto de extremidade do link privado for criado, você poderá acessar a página de entrada da ferramenta Web do Synapse Studio. No entanto, você não poderá acessar os recursos dentro do seu espaço de trabalho Synapse ainda até que seja necessário concluir a próxima etapa.

## <a name="step-4-create-private-endpoints-for-synapse-studio-workspace-resource"></a>Etapa 4: criar pontos de extremidade privados para o recurso de espaço de trabalho do Synapse Studio

Para acessar os recursos dentro de seu recurso de espaço de trabalho do Synapse Studio, você precisará criar pelo menos um ponto de extremidade de link privado com o tipo " **dev** " de " **target subresource** " e dois outros pontos de extremidades de link privado opcionais com tipos de " **SQL** " ou " **sqldemand** " depende de quais recursos no espaço de trabalho do Synapse Studio você gostaria de acessar. Essa criação de ponto de extremidade de link privado para o Synapse Studio Workspace é semelhante à criação de ponto de extremidade acima.  

Preste atenção às áreas abaixo na guia " **Resource** ":
* Selecione " **Microsoft. Synapse/Workspaces** " para " **tipo de recurso** ".
* Selecione " **YourWorkSpaceName** " para " **recurso** " que você criou anteriormente.
* Selecione o tipo de ponto de extremidade em " **sub-recurso de destino** ":
  * **SQL** : é para a execução da consulta SQL no pool do SQL.
  * **OnDemand** : é para execução de consulta interna do SQL.
  * **Dev** : destina-se a acessar tudo o mais dentro dos espaços de trabalho do Synapse Studio. Você precisa criar pelo menos o ponto de extremidade do link privado com esse tipo.

![Criando ponto de extremidade privado para o espaço de trabalho do Synapse Studio](./media/how-to-connect-to-workspace-from-restricted-network/plinks-endpoint-ws-1.png)


## <a name="step-5-create-private-endpoints-for-synapse-studio-workspace-linked-storage"></a>Etapa 5: criar pontos de extremidade privados para o armazenamento vinculado do espaço de trabalho do Synapse Studio

Para acessar o armazenamento vinculado com o Gerenciador de armazenamento no espaço de trabalho do Synapse Studio, você precisará criar um ponto de extremidade privado com as etapas semelhantes na etapa 3 acima. 

Preste atenção às áreas abaixo na guia " **Resource** ":
* Selecione " **Microsoft. Synapse/storageAccounts** " para " **tipo de recurso** ".
* Selecione " **YourWorkSpaceName** " para " **recurso** " que você criou anteriormente.
* Selecione o tipo de ponto de extremidade em " **sub-recurso de destino** ":
  * **blob** : é para o armazenamento de BLOBs do Azure.
  * **DFS** : é para Azure data Lake Storage Gen2.

![Criando ponto de extremidade privado para o armazenamento vinculado do espaço de trabalho do Synapse Studio](./media/how-to-connect-to-workspace-from-restricted-network/plink-endpoint-storage.png)

Agora, você pode acessar o recurso de armazenamento vinculado do Gerenciador de armazenamento em seu espaço de trabalho do Synapse Studio na vNet.

Se o seu espaço de trabalho tiver " **Habilitar rede virtual gerenciada** " durante a criação do espaço de trabalho, conforme mostrado abaixo,

![Criando ponto de extremidade privado para o armazenamento vinculado do espaço de trabalho do Synapse Studio 1](./media/how-to-connect-to-workspace-from-restricted-network/ws-network-config.png)

E você deseja que seu bloco de anotações acesse os recursos de armazenamento vinculados em determinada conta de armazenamento, você precisa adicionar um " **pontos de extremidade privados gerenciados** " em seu Synapse Studio. O " **nome da conta de armazenamento** " deve ser aquele que seu bloco de anotações precisa acessar. Conheça as etapas detalhadas de [criar um ponto de extremidade privado gerenciado para sua fonte de dados](./how-to-create-managed-private-endpoints.md).

Depois que esse ponto de extremidade for criado, o " **estado de aprovação** " será " **pendente** ", você precisa solicitar ao proprietário dessa conta de armazenamento para aprová-la na guia "conexões de **ponto de extremidade privado** " dessa conta de armazenamento no portal do Azure. Depois de aprovado, seu notebook poderá acessar os recursos de armazenamento vinculados nessa conta de armazenamento.

Agora, tudo definido. Você pode acessar o recurso de espaço de trabalho do Synapse Studio.

## <a name="next-steps"></a>Próximas etapas

Saiba mais sobre a [rede virtual do espaço de trabalho gerenciado](./synapse-workspace-managed-vnet.md)

Saiba mais sobre [Pontos de extremidade privados gerenciados](./synapse-workspace-managed-private-endpoints.md)
