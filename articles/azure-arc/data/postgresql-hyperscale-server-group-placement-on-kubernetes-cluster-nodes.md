---
title: Posicionamento de um grupo de servidores de hiperescala PostgreSQL nos nós de cluster kubernetes
description: Explica como as instâncias do PostgreSQL que formam um grupo de servidores de hiperescala PostgreSQL são colocadas nos nós de cluster kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 1fc768890e932d1f17ad111b4681b75721ae1e06
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148106"
---
# <a name="azure-arc-enabled-postgresql-hyperscale-server-group-placement"></a>Posicionamento do grupo de servidores de hiperescala do PostgreSQL habilitado para o Azure Arc

Neste artigo, vamos usar um exemplo para ilustrar como as instâncias do PostgreSQL do grupo de servidores de hiperescala PostgreSQL habilitados para o Azure ARC são colocadas nos nós físicos do cluster kubernetes que os hospeda. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="configuration"></a>Configuração

Neste exemplo, estamos usando um cluster AKS (serviço kubernetes do Azure) que tem quatro nós físicos. 

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/1_cluster_portal.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

Liste os nós físicos do cluster kubernetes executando o comando:

```console
kubectl get nodes
```

Que mostra os quatro nós físicos dentro do cluster kubernetes:

```output
NAME                                STATUS   ROLES   AGE   VERSION
aks-agentpool-42715708-vmss000000   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000001   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000002   Ready    agent   11h   v1.17.9
aks-agentpool-42715708-vmss000003   Ready    agent   11h   v1.17.9
```

A arquitetura pode ser representada como:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/2_logical_cluster.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

O cluster kubernetes hospeda um controlador de dados de arco do Azure e um grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc. Esse grupo de servidores é constituído de três instâncias do PostgreSQL: um coordenador e dois trabalhadores.

Liste o pods com o comando:

```console
kubectl get pods -n arc3
```
Que produz esta saída:

```output
NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01-0         3/3     Running   0          9h
postgres01-1         3/3     Running   0          9h
postgres01-2         3/3     Running   0          9h
```
Cada um desses pods hospeda uma instância do PostgreSQL. Juntos, eles formam o grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc:

```output
Pod name        Role in the server group
postgres01-0  Coordinator
postgres01-1    Worker
postgres01-2    Worker
```

## <a name="placement"></a>Posicionamento
Vejamos como o kubernetes coloca o pods do grupo de servidores. Descreva cada pod e identifique em qual nó físico do cluster kubernetes eles são colocados. Por exemplo, para o coordenador, execute o seguinte comando:

```console
kubectl describe pod postgres01-0 -n arc3
```

Que produz esta saída:

```output
Name:         postgres01-0
Namespace:    arc3
Priority:     0
Node:         aks-agentpool-42715708-vmss000000
Start Time:   Thu, 17 Sep 2020 00:40:33 -0700
…
```

À medida que executamos esse comando para cada um dos pods, resumimos o posicionamento atual como:

| Função de grupo de servidores|Pod do grupo de servidores|Nó físico kubernetes que hospeda o Pod |
|--|--|--|
| Trabalho|postgres01-1|AKs-agentpool-42715708-vmss000002 |
| Trabalho|postgres01-2|AKs-agentpool-42715708-vmss000003 |

Além disso, observe também, na descrição do pods, os nomes dos contêineres que cada pod hospeda. Por exemplo, para o segundo trabalho, execute o seguinte comando:

```console
kubectl describe pod postgres01-2 -n arc3
```

Que produz esta saída:

```output
…
Node:         aks-agentpool-42715708-vmss000003/10.240.0.7
..
Containers:
  Fluentbit:
…
  Postgres:
…
  Telegraf:
…
```

Cada pod que faz parte do grupo de servidores de hiperescala PostgreSQL habilitado para Arc do Azure hospeda os três contêineres a seguir:

|Contêineres|Description
|----|----|
|`Fluentbit` |Data * coletor de logs: https://fluentbit.io/
|`Postgres`|Parte da instância PostgreSQL do grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc
|`Telegraf` |Coletor de métricas: https://www.influxdata.com/time-series-platform/telegraf/

A arquitetura é semelhante ao:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/3_pod_placement.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

Isso significa que, neste ponto, cada instância PostgreSQL que constituem o grupo de servidores de hiperescala PostgreSQL habilitado para o Azure Arc é hospedado em um host físico específico dentro do contêiner kubernetes. Esta é a melhor configuração para ajudar a obter o máximo desempenho do grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc, uma vez que cada função (coordenador e trabalhadores) usa os recursos de cada nó físico. Esses recursos não são compartilhados entre várias funções do PostgreSQL.

## <a name="scale-out-azure-arc-enabled-postgresql-hyperscale"></a>Escalar horizontalmente a hiperescala do PostgreSQL habilitada para o Arc do Azure

Agora, vamos expandir para adicionar um terceiro nó de trabalho ao grupo de servidores e observar o que acontece. Ele criará uma quarta instância PostgreSQL que será hospedada em um quarto Pod.
Para escalar horizontalmente, execute o comando:

```console
azdata arc postgres server edit --name postgres01 --workers 3
```

Que produz a seguinte saída:

```output
Updating postgres01 in namespace `arc3`
postgres01 is Ready
```

Liste os grupos de servidores implantados no controlador de dados Arc do Azure e verifique se o grupo de servidores agora é executado com três trabalhadores. Execute o comando:

```console
azdata arc postgres server list
```

Observe que ele foi dimensionado de dois trabalhadores para três trabalhadores:

```output
Name        State    Workers
----------  -------  ---------
postgres01  Ready    3
```

Como fizemos anteriormente, observe que o grupo de servidores agora usa um total de quatro pods:

```console
kubectl get pods -n arc3
```

```output
NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01-0         3/3     Running   0          11h
postgres01-1         3/3     Running   0          11h
postgres01-2         3/3     Running   0          11h
postgres01-3         3/3     Running   0          5m2s
```

E descreva o novo pod para identificar em quais dos nós físicos do cluster kubernetes ele está hospedado.
Execute o comando:

```console
kubectl describe pod postgres01-3 -n arc3
```

Para identificar o nome do nó de hospedagem:

```output
Name:         postgres01-3
Namespace:    arc3
Priority:     0
Node:         aks-agentpool-42715708-vmss000000
```

O posicionamento das instâncias do PostgreSQL nos nós físicos do cluster agora é:

|Função de grupo de servidores|Pod do grupo de servidores|Nó físico kubernetes que hospeda o Pod
|-----|-----|-----
|Pelos|postgres01-0|AKs-agentpool-42715708-vmss000000
|Trabalho|postgres01-1|AKs-agentpool-42715708-vmss000002
|Trabalho|postgres01-2|AKs-agentpool-42715708-vmss000003
|Trabalho|postgres01-3|AKs-agentpool-42715708-vmss000000

E observe que o Pod do novo trabalhador (postgres01-3) foi colocado no mesmo nó que o coordenador. 

A arquitetura é semelhante ao:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/4_pod_placement_.png" alt-text="cluster AKS de 4 nós no portal do Azure" do controlador que fica atento à disponibilidade do controlador de dados.|AKs-agentpool-42715708-vmss000000
|logsdb-0|Esta é uma instância de pesquisa elástica que é usada para armazenar todos os logs coletados em todos os pods dos serviços de dados de arco. Elasticsearch, recebe dados do `Fluentbit` contêiner de cada pod|AKs-agentpool-42715708-vmss000003
|logsui-5fzv5|Essa é uma instância Kibana que fica sobre o banco de dados de pesquisa elástica para apresentar uma GUI do log Analytics.|AKs-agentpool-42715708-vmss000003
|metricsdb-0|Essa é uma instância de InfluxDB que é usada para armazenar todas as métricas coletadas em todos os pods de serviços de dados de arco. InfluxDB, recebe dados do `Telegraf` contêiner de cada pod|AKs-agentpool-42715708-vmss000000
|metricsdc-47d47|Este é um daemonset implantado em todos os nós kubernetes no cluster para coletar métricas no nível de nó sobre os nós.|AKs-agentpool-42715708-vmss000002
|metricsdc-864kj|Este é um daemonset implantado em todos os nós kubernetes no cluster para coletar métricas no nível de nó sobre os nós.|AKs-agentpool-42715708-vmss000001
|metricsdc-l8jkf|Este é um daemonset implantado em todos os nós kubernetes no cluster para coletar métricas no nível de nó sobre os nós.|AKs-agentpool-42715708-vmss000003
|metricsdc-nxm4l|Este é um daemonset implantado em todos os nós kubernetes no cluster para coletar métricas no nível de nó sobre os nós.|AKs-agentpool-42715708-vmss000000
|metricsui-4fb7l|Essa é uma instância Grafana que fica na parte superior do banco de dados InfluxDB para apresentar uma GUI do painel de monitoramento.|AKs-agentpool-42715708-vmss000003
|mgmtproxy-4qppp|Essa é uma camada de proxy de aplicativo Web que fica na frente das instâncias Grafana e Kibana.|AKs-agentpool-42715708-vmss000002

> \* O sufixo nos nomes de Pod variará em outras implantações. Além disso, estamos listando apenas os pods hospedados no namespace kubernetes do controlador de dados de arco do Azure.

A arquitetura é semelhante ao:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/5_full_list_of_pods.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

Isso significa que os nós de coordenador (Pod 1) do grupo de servidores postgres de hiperescala habilitados para o Azure Arc compartilham os mesmos recursos físicos que o terceiro nó de trabalho (Pod 4) do grupo de servidores. Isso é aceitável, pois o nó de coordenador normalmente está usando muito poucos recursos em comparação com o que um nó de trabalho pode estar usando. A partir disso, você pode inferir que você deve escolher cuidadosamente:
- o tamanho do cluster kubernetes e as características de cada um de seus nós físicos (memória, vCore)
- o número de nós físicos dentro do cluster kubernetes
- os aplicativos ou as cargas de trabalho que você hospeda no cluster kubernetes.

A implicação de hospedar muitas cargas de trabalho no cluster kubernetes é a limitação pode ocorrer para o grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc. Se isso acontecer, você não se beneficiará muito com sua capacidade de dimensionar horizontalmente. O desempenho que você sai do sistema não se trata apenas do posicionamento ou das características físicas dos nós físicos ou do sistema de armazenamento. O desempenho que você obtém também é sobre como configurar cada um dos recursos em execução dentro do cluster kubernetes (incluindo a hiperescala do PostgreSQL habilitada para Arc do Azure), por exemplo, as solicitações e os limites definidos para a memória e vCore. A quantidade de carga de trabalho que você pode hospedar em um determinado cluster kubernetes é relativa às características do cluster kubernetes, a natureza das cargas de trabalho, o número de usuários, como as operações do cluster kubernetes são feitas...

## <a name="scale-out-aks"></a>Expandir AKS

Vamos demonstrar que dimensionar horizontalmente o cluster AKS e o servidor de hiperescala PostgreSQL habilitado para Arc do Azure é uma maneira de aproveitar ao máximo o alto desempenho da hiperescala do PostgreSQL habilitada para Arc do Azure.
Vamos adicionar um quinto nó ao cluster AKS:

:::row:::
    :::column:::
        Antes
    :::column-end:::
    :::column:::
        Depois
    :::column-end:::
:::row-end:::
:::row:::
    :::column:::
        :::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/6_layout_before.png" alt-text="cluster AKS de 4 nós no portal do Azure":::
    :::column-end:::
    :::column:::
        :::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/7_layout_after.png" alt-text="cluster AKS de 4 nós no portal do Azure":::
    :::column-end:::
:::row-end:::

A arquitetura é semelhante ao:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/8_logical_layout_after.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

Vejamos o que o pods do namespace do controlador de dados Arc está hospedado no novo nó físico AKS executando o comando:

```console
kubectl describe node aks-agentpool-42715708-vmss000004
```

E vamos atualizar a representação da arquitetura do nosso sistema:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/9_updated_list_of_pods.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

Podemos observar que o novo nó físico do cluster kubernetes está hospedando apenas o pod de métricas necessário para os serviços de dados de arco do Azure. Observe que, neste exemplo, estamos nos concentrando apenas no namespace do controlador de dados Arc, não estamos representando os outros pods.

## <a name="scale-out-azure-arc-enabled-postgresql-hyperscale-again"></a>Escalar horizontalmente habilitada para o Arc do Azure com o hiperescala do PostgreSQL

O quinto nó físico ainda não está hospedando nenhuma carga de trabalho. À medida que expandirmos a hiperescala do PostgreSQL habilitada para o Azure Arc, o kubernetes otimizará o posicionamento do novo pod PostgreSQL e não deverá colocar o local em nós físicos que já hospedam mais cargas de trabalho. Execute o comando a seguir para dimensionar a hiperescala do PostgreSQL habilitada para o Arc do Azure de 3 a 4 trabalhadores. No final da operação, o grupo de servidores será constituído e distribuído entre cinco instâncias do PostgreSQL, um coordenador e quatro trabalhadores.

```console
azdata arc postgres server edit --name postgres01 --workers 4
```

Que produz a seguinte saída:

```output
Updating postgres01 in namespace `arc3`
postgres01 is Ready
```

Liste os grupos de servidores implantados no controlador de dados e verifique se o grupo de servidores agora é executado com quatro trabalhadores:

```console
azdata arc postgres server list
```

Observe que ele foi dimensionado de três para quatro trabalhadores. 

```console
Name        State    Workers
----------  -------  ---------
postgres01  Ready    4
```

Como fizemos anteriormente, observe que o grupo de servidores agora usa quatro pods:

```output
kubectl get pods -n arc3

NAME                 READY   STATUS    RESTARTS   AGE
…
postgres01-0         3/3     Running   0          13h
postgres01-1         3/3     Running   0          13h
postgres01-2         3/3     Running   0          13h
postgres01-3         3/3     Running   0          179m
postgres01-4         3/3     Running   0          3m13s
```

A forma do grupo de servidores agora é:

|Função de grupo de servidores|Pod do grupo de servidores
|----|-----
|Pelos|postgres01-0
|Trabalho|postgres01-1
|Trabalho|postgres01-2
|Trabalho|postgres01-3
|Trabalho|postgres01-4

Vamos descrever o Pod postgres01-4 para identificar em qual nó físico ele está hospedado:

```console
kubectl describe pod postgres01-4 -n arc3
```

E observe em que pods ele é executado:

|Função de grupo de servidores|Pod do grupo de servidores| Pod
|----|-----|------
|Pelos|postgres01-0|AKs-agentpool-42715708-vmss000000
|Trabalho|postgres01-1|AKs-agentpool-42715708-vmss000002
|Trabalho|postgres01-2|AKs-agentpool-42715708-vmss000003
|Trabalho|postgres01-3|AKs-agentpool-42715708-vmss000000
|Trabalho|postgres01-4|AKs-agentpool-42715708-vmss000004

E a arquitetura é semelhante a:

:::image type="content" source="media/migrate-postgresql-data-into-postgresql-hyperscale-server-group/10_kubernetes_schedules_newest_pod.png" alt-text="cluster AKS de 4 nós no portal do Azure":::

O kubernetes agendou o novo pod PostgreSQL no nó físico menos carregado do cluster kubernetes.

## <a name="summary"></a>Resumo

Para aproveitar ao máximo a escalabilidade e o desempenho do dimensionamento horizontal do grupo de servidores habilitados para Arc do Azure, você deve evitar a contenção de recursos dentro do cluster kubernetes:
- entre o grupo de servidores de hiperescala do PostgreSQL habilitado para o Azure Arc e outras cargas de trabalho hospedadas no mesmo cluster kubernetes
- entre todas as instâncias do PostgreSQL que constituem o grupo de servidores de hiperescala PostgreSQL habilitado para Arc do Azure

Você pode conseguir isso de várias maneiras:
- Dimensione horizontalmente o kubernetes e o Arc do Azure habilitados para postgres hiperescala: considere a possibilidade de dimensionar verticalmente o cluster kubernetes da mesma maneira que está dimensionando o grupo de servidores de hiperescala PostgreSQL habilitados para o Azure Arc. Adicione um nó físico ao cluster para cada trabalho que você adicionar ao grupo de servidores.
- Escalar horizontalmente o postgres de expansão habilitado para Arc do Azure sem escalar horizontalmente o kubernetes: definindo as restrições de recursos certas (solicitação e limites de memória e vCore) nas cargas de trabalho hospedadas em kubernetes (o Azure Arc habilitou o hiperescala do PostgreSQL incluído), você habilitará a colocação de cargas de trabalho no kubernetes e reduzirá o risco de contenção de recursos. Você precisa certificar-se de que as características físicas dos nós físicos do cluster kubernetes possam respeitar as restrições de recursos que você definir. Você também deve garantir que o equilíbrio permaneça à medida que as cargas de trabalho evoluem ao longo do tempo ou à medida que mais cargas de trabalho são adicionadas ao cluster kubernetes.
- Use os mecanismos de kubernetes (seletor de Pod, afinidade, antiafinidade) para influenciar o posicionamento do pods.

## <a name="next-steps"></a>Próximas etapas

[Escalar horizontalmente seu grupo de servidores de hiperescala PostgreSQL habilitado para o Azure Arc adicionando mais nós de trabalho](scale-out-postgresql-hyperscale-server-group.md)
