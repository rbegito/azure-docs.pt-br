---
title: Limitações
description: Limitações de código para recursos do SDK
author: erscorms
ms.author: erscor
ms.date: 02/11/2020
ms.topic: reference
ms.openlocfilehash: 33f5314c80dc33dbec50dc21a71f4cb507979e12
ms.sourcegitcommit: 0dcafc8436a0fe3ba12cb82384d6b69c9a6b9536
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94427421"
---
# <a name="limitations"></a>Limitações

Vários recursos têm limitações de tamanho, contagem ou outras limitações.

## <a name="azure-frontend"></a>Azure Frontend

As seguintes limitações se aplicam à API de front-end (C++ e C#):
* Total de instâncias [AzureFrontend](/dotnet/api/microsoft.azure.remoterendering.azurefrontend) por processo: 16.
* Total de instâncias de [AzureSession](/dotnet/api/microsoft.azure.remoterendering.azuresession) por [AzureFrontend](/dotnet/api/microsoft.azure.remoterendering.azurefrontend): 16.

## <a name="objects"></a>Objetos

* Total de objetos permitidos de um único tipo ([entidade](../concepts/entities.md), [CutPlaneComponent](../overview/features/cut-planes.md), etc.): 16.777.215.
* Total de planos de corte ativos permitidos: 8.

## <a name="geometry"></a>Geometria

* **Animação:** As animações são limitadas à animação de transformações individuais de [objetos de jogos](../concepts/entities.md). Não há suporte para animações de esqueleto com animações de aparência ou de vértice. As faixas de animação do arquivo de ativo de origem não são preservadas. Em vez disso, as animações de transformação de objeto precisam ser orientadas pelo código do cliente.
* **Sombreadores personalizados:** Não há suporte para a criação de sombreadores personalizados. Somente [materiais de cores](../overview/features/color-materials.md) internos ou [materiais PBR](../overview/features/pbr-materials.md) podem ser usados.
* **Número máximo de materiais distintos** em um ativo: 65.535. Para obter mais informações sobre a redução automática de contagem de material, consulte o capítulo eliminação [de duplicação de material](../how-tos/conversion/configure-model-conversion.md#material-de-duplication) .
* **Dimensão máxima de uma única textura** : 16.384 x 16.384. Texturas de origem maiores serão reduzidas em tamanho pelo processo de conversão.

### <a name="overall-number-of-polygons"></a>Número total de polígonos

O número permitido de polígonos para todos os modelos carregados depende do tamanho da VM, conforme passado para [a API REST de gerenciamento de sessão](../how-tos/session-rest-api.md#create-a-session):

| Tamanho do servidor | Número máximo de polígonos |
|:--------|:------------------|
|padrão| 20 milhões |
|premium| nenhum limite |

Para obter informações detalhadas sobre essa limitação, consulte o capítulo [tamanho do servidor](../reference/vm-sizes.md) .

## <a name="platform-limitations"></a>Limitações da plataforma

**Windows 10 Desktop**

* Win32/x64 é a única plataforma Win32 com suporte. Não há suporte para Win32/x86.

**HoloLens 2**

* Não há suporte para [renderização de câmeras de foto/vídeo](/windows/mixed-reality/mixed-reality-capture-for-developers#render-from-the-pv-camera-opt-in).