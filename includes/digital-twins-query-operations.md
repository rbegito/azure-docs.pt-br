---
author: baanders
description: arquivo de inclusão para operações de consulta do Azure digital gêmeos
ms.service: digital-twins
ms.topic: include
ms.date: 7/28/2020
ms.author: baanders
ms.openlocfilehash: 333a7ec4ae0e5c8cbc94a603e2ccf81ee92e7d48
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078452"
---
## <a name="query-language-features"></a>Recursos de linguagem de consulta

O Azure digital gêmeos fornece recursos de consulta extensivos em relação ao grafo de entrelaçamento. As consultas são descritas usando a sintaxe do tipo SQL, em uma linguagem de consulta semelhante à [linguagem de consulta do Hub IOT](../articles/iot-hub/iot-hub-devguide-query-language.md) com muitos recursos comparáveis.

> [!NOTE]
> Todas as operações de consulta do Azure digital gêmeos diferenciam maiúsculas de minúsculas.

Aqui estão as operações disponíveis na linguagem de consulta do Azure digital gêmeos.

Obter gêmeos digital por seus...
* modelo (usando o `IS_OF_MODEL` operador)
* Propriedades (incluindo [Propriedades de marca](../articles/digital-twins/how-to-use-tags.md))
* interfaces
* relacionamentos
  - Propriedades das relações

Você pode aprimorar ainda mais suas consultas com as seguintes operações:
* Obter gêmeos em vários tipos de relação ( `JOIN` consultas). 
  - Durante a visualização, são permitidos até cinco níveis de `JOIN` .
* Selecionar somente os resultados da consulta superior ( `Select TOP` operador)
* Contar o número de itens em um conjunto de resultados usando `Select COUNT`
* Use projeções para escolher as colunas que uma consulta retornará
* Usar funções escalares:,,,,,, `IS_BOOL` `IS_DEFINED` `IS_NULL` `IS_NUMBER` `IS_OBJECT` `IS_PRIMITIVE` `IS_STRING` , `STARTSWITH` , `ENDSWITH` .
* Usar operadores de comparação de consulta:,,,,, `IN` / `NIN` `=` `!=` `<` `>` `<=` , `>=` .
* Use qualquer combinação ( `AND` , `OR` , `NOT` operador) de `IS_OF_MODEL` , funções escalares e operadores de comparação.
* Use a continuação: o objeto de consulta é instanciado com um tamanho de página (até 100). Você pode recuperar o digital gêmeos uma página por vez fornecendo o token de continuação em chamadas subsequentes para a API.