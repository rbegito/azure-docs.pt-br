---
title: Acessar logs de consulta lentos-CLI do Azure-banco de dados do Azure para MySQL
description: Este artigo descreve como acessar os logs de consulta lenta no banco de dados do Azure para MySQL usando o CLI do Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 945a67f81010a61adf814f6f6f422eba5001b48d
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541565"
---
# <a name="configure-and-access-slow-query-logs-by-using-azure-cli"></a>Configurar e acessar logs de consulta lentos usando CLI do Azure
Você pode baixar os logs de consulta lenta do banco de dados do Azure para MySQL usando CLI do Azure, o utilitário de linha de comando do Azure.

## <a name="prerequisites"></a>Pré-requisitos
Para seguir este guia de instruções, você precisa:
- [Banco de dados do Azure para servidor MySQL](quickstart-create-mysql-server-database-using-azure-cli.md)
- A [CLI do Azure](/cli/azure/install-azure-cli) ou o Azure Cloud Shell no navegador

## <a name="configure-logging"></a>Configurar o registro em log
É possível configurar o servidor para acessar os logs de consulta lenta do MySQL executando as seguintes etapas:
1. Ative o log de consultas lentas definindo o parâmetro de **\_ \_ log de consulta lenta** como ativado.
2. Selecione para onde os logs são gerados usando **a \_ saída do log**. Para enviar logs para o armazenamento local e Azure Monitor logs de diagnóstico, selecione **arquivo**. Para enviar logs somente para Azure Monitor logs, selecione **nenhum**
3. Ajuste outros parâmetros como **long\_query\_time** e **log\_slow\_admin\_statements**.

Para saber como definir o valor desses parâmetros por meio da CLI do Azure, consulte [Como configurar parâmetros do servidor](howto-configure-server-parameters-using-cli.md).

Por exemplo, o comando da CLI a seguir ativa o log de consultas lentas, define o tempo de consulta longo para 10 segundos e desativa o registro em log da instrução de admin lenta. Por fim, ele lista as opções de configuração para sua análise.
```azurecli-interactive
az mysql server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mysql server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mysql server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mysql server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mysql server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mysql-server"></a>Listar os logs para o servidor do Banco de Dados do Azure para MySQL
Se **log_output** estiver configurado como "arquivo", você poderá acessar os logs diretamente do armazenamento local do servidor. Para listar os arquivos de log de consulta lenta disponíveis para seu servidor, execute o comando [AZ MySQL Server-logs List](/cli/azure/mysql/server-logs#az-mysql-server-logs-list) .

Você pode listar os arquivos de log para o servidor **mydemoserver.mysql.database.azure.com** no Grupo de Recursos **myresourcegroup**. Em seguida, direcione-os para um arquivo de texto chamado **log\_files\_list.txt**.
```azurecli-interactive
az mysql server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Baixar logs do servidor
Se **log_output** estiver configurado como "arquivo", você poderá baixar arquivos de log individuais do seu servidor com o comando [AZ MySQL Server-logs download](/cli/azure/mysql/server-logs#az-mysql-server-logs-download) .

Use o exemplo a seguir para baixar o arquivo de log específico para o servidor **mydemoserver.mysql.database.azure.com** no grupo de recursos **myresourcegroup** para seu ambiente local.
```azurecli-interactive
az mysql server-logs download --name 20170414-mydemoserver-mysql.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Próximas etapas
- Saiba mais sobre [logs de consulta lentos no banco de dados do Azure para MySQL](concepts-server-logs.md).
