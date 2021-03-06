---
title: 'Conceitos: fluxo de dados no conector IoT do Azure para o recurso FHIR (versão prévia) da API do Azure para FHIR'
description: Entenda o conector de IoT do Azure para o fluxo de dados de FHIR (visualização). O conector do Azure IoT para ingestão de FHIR (visualização), normalizações, grupos, transformações e persiste os dados de IoMT para a API do Azure para FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 07/31/2020
ms.author: punagpal
ms.openlocfilehash: 3cae648e3c2bddbafec555621d97575a007cfeb4
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394859"
---
# <a name="azure-iot-connector-for-fhir-preview-data-flow"></a>Fluxo de dados do Conector IoT do Azure para FHIR (versão prévia)

Este artigo fornece uma visão geral do fluxo de dados no conector IoT do Azure para FHIR *. Você aprenderá sobre os diferentes estágios de processamento de dados no conector IoT do Azure para FHIR que transformam dados de dispositivo em recursos de [Observação](https://www.hl7.org/fhir/observation.html) baseados em FHIR.

![Fluxo de dados do Conector IoT do Azure para FHIR](media/concepts-iot-data-flow/iot-connector-data-flow.png)

O diagrama acima mostra fluxos de dados comuns usando o conector IoT do Azure para FHIR. 

Abaixo estão os estágios diferentes pelos quais os dados passam quando recebidos pelo conector do Azure IoT para FHIR.

## <a name="ingest"></a>Ingerir
Ingestão é o primeiro estágio em que os dados do dispositivo são recebidos no conector do Azure IoT para FHIR. O ponto de extremidade de ingestão para dados do dispositivo é hospedado em um [Hub de eventos do Azure](../event-hubs/index.yml). A plataforma do hub de eventos do Azure dá suporte a alta escala e taxa de transferência com a capacidade de receber e processar milhões de mensagens por segundo. Ele também permite que o conector IoT do Azure para FHIR consuma mensagens de forma assíncrona, eliminando a necessidade de dispositivos aguardarem enquanto os dados do dispositivo são processados.

> [!NOTE]
> JSON é o único formato com suporte no momento para dados do dispositivo.

## <a name="normalize"></a>Normaliza
Normalize é o próximo estágio em que os dados do dispositivo são recuperados do hub de eventos do Azure acima e processados usando modelos de mapeamento de dispositivo. Esse processo de mapeamento resulta na transformação de dados do dispositivo em um esquema normalizado. 

O processo de normalização não apenas simplifica o processamento de dados em estágios posteriores, mas também fornece a capacidade de projetar uma mensagem de entrada em várias mensagens normalizadas. Por exemplo, um dispositivo poderia enviar vários sinais vitais para temperatura do corpo, taxa de pulso, pressão de sangue e taxa de Respiration em uma única mensagem. Essa mensagem de entrada criaria quatro recursos de FHIR separados. Cada recurso representaria um sinal vital diferente, com a mensagem de entrada projetada em quatro mensagens normalizadas diferentes.

## <a name="group"></a>Agrupar
O grupo é o próximo estágio em que as mensagens normalizadas disponíveis no estágio anterior são agrupadas usando três parâmetros diferentes: identidade do dispositivo, tipo de medida e período de tempo.

A identidade do dispositivo e o agrupamento do tipo de medição habilitam o uso do tipo de medição [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData) . Esse tipo fornece uma maneira concisa de representar uma série de medidas com base em tempo de um dispositivo no FHIR. E o período de tempo controla a latência na qual os recursos de observação gerados pelo conector do Azure IoT para FHIR são gravados na API do Azure para FHIR.

> [!NOTE]
> O valor do período de tempo é padronizado para 15 minutos e não pode ser configurado para visualização.

## <a name="transform"></a>Transformar
No estágio de transformação, as mensagens de normalização agrupadas são processadas por meio de modelos de mapeamento FHIR. As mensagens que correspondem a um tipo de modelo são transformadas em recursos de observação com base em FHIR conforme especificado por meio do mapeamento.

Neste ponto, o recurso de [dispositivo](https://www.hl7.org/fhir/device.html) , juntamente com seu recurso de [paciente](https://www.hl7.org/fhir/patient.html) associado, também é recuperado do servidor FHIR usando o identificador de dispositivo presente na mensagem. Esses recursos são adicionados como uma referência ao recurso de observação que está sendo criado.

> [!NOTE]
> Todas as pesquisas de identidade são armazenadas em cache uma vez resolvidos para diminuir a carga no servidor FHIR. Se você planeja reutilizar dispositivos com vários pacientes, é aconselhável criar um recurso de dispositivo virtual que seja específico ao paciente e enviar o identificador de dispositivo virtual na carga da mensagem. O dispositivo virtual pode ser vinculado ao recurso de dispositivo real como um pai.

Se nenhum recurso de dispositivo para um determinado identificador de dispositivo existir no servidor FHIR, o resultado dependerá do valor `Resolution Type` definido no momento da criação. Quando definido como `Lookup` , a mensagem específica é ignorada e o pipeline continuará processando outras mensagens de entrada. Se definido como `Create` , o conector do Azure IOT para FHIR criará um dispositivo básico e recursos de paciente no servidor FHIR.  

## <a name="persist"></a>Persist
Quando o recurso de observação FHIR é gerado no estágio de transformação, o recurso é salvo na API do Azure para FHIR. Se o recurso FHIR for novo, ele será criado no servidor. Se o recurso FHIR já existir, ele será atualizado.

## <a name="next-steps"></a>Próximas etapas

Clique abaixo da próxima etapa para aprender a criar modelos de mapeamento de dispositivo e FHIR.

>[!div class="nextstepaction"]
>[Modelos de mapeamento do Conector IoT do Azure para FHIR](iot-mapping-templates.md)

*No portal do Azure, o Conector IoT do Azure para FHIR é chamado de Conector IoT (versão prévia).

FHIR é uma marca registrada da HL7, usada com permissão da HL7.