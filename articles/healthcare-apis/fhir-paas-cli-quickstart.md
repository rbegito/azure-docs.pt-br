---
title: 'Início Rápido: Implantar a API do Azure para FHIR usando a CLI do Azure'
description: Neste guia de início rápido, você aprenderá a implantar a API do Azure para FHIR no Azure usando a CLI do Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.date: 10/15/2019
ms.author: cavoeg
ms.custom: devx-track-azurecli
ms.openlocfilehash: b09a72538dd7a6886811b9a23c915316627da093
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92339434"
---
# <a name="quickstart-deploy-azure-api-for-fhir-using-azure-cli"></a>Início Rápido: Implantar a API do Azure para FHIR usando a CLI do Azure

Neste guia de início rápido, você aprenderá a implantar a API do Azure para FHIR no Azure usando a CLI do Azure.

Se você não tiver uma assinatura do Azure, crie uma [conta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de começar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="add-healthcareapis-extension"></a>Adicionar a extensão HealthcareAPIs

```azurecli-interactive
az extension add --name healthcareapis
```

Obtenha uma lista de comandos para HealthcareAPIs:

```azurecli-interactive
az healthcareapis --help
```

## <a name="create-azure-resource-group"></a>Criar um grupo de recursos do Azure

Escolha um nome para o grupo de recursos que conterá a API do Azure para FHIR e crie-o:

```azurecli-interactive
az group create --name "myResourceGroup" --location westus2
```

## <a name="deploy-the-azure-api-for-fhir"></a>Implantar a API do Azure para FHIR

```azurecli-interactive
az healthcareapis create --resource-group myResourceGroup --name nameoffhiraccount --kind fhir-r4 --location westus2 
```

## <a name="fetch-fhir-api-capability-statement"></a>Instrução de funcionalidade Buscar API de FHIR

Obtenha uma declaração de capacidade da API de FHIR com:

```azurecli-interactive
curl --url "https://nameoffhiraccount.azurehealthcareapis.com/metadata"
```

## <a name="clean-up-resources"></a>Limpar os recursos

Se você não quiser continuar usando este aplicativo, exclua o grupo de recursos por meio das seguintes etapas:

```azurecli-interactive
az group delete --name "myResourceGroup"
```

## <a name="next-steps"></a>Próximas etapas

Neste guia de início rápido, você implantou a API do Azure para FHIR em sua assinatura. Para definir configurações adicionais em sua API do Azure para FHIR, vá para o guia de instruções de configurações adicionais. Se você estiver pronto para começar a usar a API do Azure para FHIR, leia mais sobre como registrar aplicativos.

>[!div class="nextstepaction"]
>[Configurações adicionais na API do Azure para FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Visão geral sobre como Registrar os Aplicativos](fhir-app-registration.md)
