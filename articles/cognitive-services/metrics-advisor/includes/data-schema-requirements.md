---
title: Requisitos do esquema de dados
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: c754efef02cdad6edbf047c5de9f1af6d758f137
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043181"
---
O Monitoramento de Métricas é um serviço para detecção de anomalias, diagnóstico e análise de séries temporais. Como um serviço da plataforma AI, ele usa os dados para treinar o modelo usado. O serviço aceita tabelas de dados agregados com as seguintes colunas:

* **Medida** (obrigatória): uma ou mais colunas contendo valores numéricos.
* **Carimbo de data/hora** (opcional): zero ou uma coluna com o tipo `DateTime` ou `String`. Quando essa coluna não for definida, o carimbo de data/hora será designado como a hora de início de cada período de ingestão. Formatar o carimbo de data/hora como: `yyyy-MM-ddTHH:mm:ssZ`. 
  * **O carimbo de data/hora deve corresponder à granularidade da métrica. Por exemplo, uma métrica diária deve garantir que a hora, o minuto e o segundo do carimbo de data/hora sejam rotulados como `00:00:00`** .
* **Dimensão** (opcional): as colunas podem ser de qualquer tipo de dados. Tenha cuidado ao trabalhar com grandes volumes de colunas e valores, para impedir que números excessivos de dimensões sejam processados.

> [!Note]
> Para cada métrica, deve haver apenas um carimbo de data/hora por medida, correspondendo a uma combinação de dimensão. Agregue os dados antes da integração ou use uma consulta para especificar os dados a serem ingeridos.