---
title: Otimizar o cache Gen2
description: Saiba como monitorar seu cache Gen2 usando o portal do Azure.
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.topic: conceptual
ms.date: 09/06/2018
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: fa5025e0a2bd260adeb23b4ab7c4d5f8bd83a43a
ms.sourcegitcommit: daab0491bbc05c43035a3693a96a451845ff193b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/29/2020
ms.locfileid: "93026795"
---
# <a name="how-to-monitor-the-gen2-cache"></a>Como monitorar o cache Gen2

Este artigo descreve como monitorar e solucionar problemas de desempenho de consultas lentas determinando se sua carga de trabalho está aproveitando da forma ideal o cache Gen2.

A arquitetura de armazenamento Gen2 divide automaticamente em camadas seus segmentos columnstore consultados com mais frequência em um cache que reside em SSDs baseados em NVMe projetado para data warehouses Gen2. O melhor desempenho é alcançado quando suas consultas recuperam segmentos que residem no cache.
 
## <a name="troubleshoot-using-the-azure-portal"></a>Solucionar problemas usando o Portal do Azure

Você pode usar o Azure Monitor para exibir métricas do cache Gen2 para solucionar problemas de desempenho de consulta. Primeiro, acesse a portal do Azure e clique em **Monitor** , **métricas** e **+ Selecione um escopo** :

![Captura de tela mostra selecionar um escopo selecionado de métricas no portal do Azure.](./media/sql-data-warehouse-how-to-monitor-cache/cache-0.png)

Use as barras de pesquisa e lista suspensa para localizar o data warehouse. Em seguida, selecione aplicar.

![Captura de tela mostra o painel Selecionar um escopo onde você pode selecionar seu data warehouse.](./media/sql-data-warehouse-how-to-monitor-cache/cache-1.png)

As principais métricas para solucionar problemas do cache Gen2 são **Percentual de ocorrência no cache** e **Percentual de uso do cache** . Selecione **porcentagem de acesso ao cache** e, em seguida, use o botão **Adicionar métrica** para adicionar o **percentual de cache usado** . 

![Métricas de cache](./media/sql-data-warehouse-how-to-monitor-cache/cache-2.png)

## <a name="cache-hit-and-used-percentage"></a>Ocorrência no cache e percentual de uso

A matriz a seguir descreve cenários baseados nos valores das métricas de cache:

|                                | **Alto percentual de ocorrência no cache** | **Baixo percentual de ocorrência no cache** |
| :----------------------------: | :---------------------------: | :--------------------------: |
| **Alto percentual de uso do cache** |          Cenário 1           |          Cenário 2          |
| **Baixo percentual de uso do cache**  |          Cenário 3           |          Cenário 4          |

**Cenário 1:** você está usando o cache de forma ideal. [Solucionar problemas](sql-data-warehouse-manage-monitor.md) de outras áreas que podem estar atrasando suas consultas.

**Cenário 2:** seu conjunto de dados de trabalho atual não cabe no cache, o que causa um percentual baixo de ocorrência no cache devido a leituras físicas. Considere aumentar o nível de desempenho e execute novamente sua carga de trabalho para popular o cache.

**Cenário 3:** é provável que sua consulta esteja sendo executada lentamente por motivos não relacionados ao cache. [Solucionar problemas](sql-data-warehouse-manage-monitor.md) de outras áreas que podem estar atrasando suas consultas. Você também pode considerar [reduzir verticalmente sua instância](sql-data-warehouse-manage-monitor.md) para reduzir o tamanho do cache e economizar custos. 

**Cenário 4:** você tinha um cache frio que poderia ser o motivo de sua consulta ser lenta. Considere a possibilidade de executar novamente a consulta, pois seu conjunto de dados de trabalho agora deve estar armazenado em cache. 

> [!IMPORTANT]
> Se a porcentagem de acesso ao cache ou o percentual de cache usado não estiver atualizando após a reexecução da carga de trabalho, seu conjunto funcional já poderá estar residindo na memória. Somente as tabelas columnstore clusterizadas são armazenadas em cache.

## <a name="next-steps"></a>Próximas etapas
Para obter mais informações sobre o ajuste de desempenho de consultas geral, confira [Monitorar a execução de consulta](sql-data-warehouse-manage-monitor.md#monitor-query-execution).
