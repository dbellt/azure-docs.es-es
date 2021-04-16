---
title: 'Análisis de opiniones: LUIS'
titleSuffix: Azure Cognitive Services
description: Si el análisis de sentimiento está configurado, la respuesta JSON de LUIS incluye el análisis de sentimiento.
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: reference
ms.date: 04/06/2021
ms.openlocfilehash: 7524644b34a6fd479c08b9ce6418c547c836add5
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554032"
---
# <a name="sentiment-analysis"></a>análisis de opiniones
Si el análisis de sentimiento está configurado, la respuesta JSON de LUIS incluye el análisis de sentimiento. Obtenga más información sobre el análisis de sentimiento en la documentación de [Text Analytics](../text-analytics/index.yml).

LUIS usa Text Analytics v2. 

Análisis de sentimiento se configura al publicar la aplicación. Vea [cómo publicar una aplicación](./luis-how-to-publish-app.md) para obtener más información.

## <a name="resolution-for-sentiment"></a>Resolución de opiniones

Los datos de opinión son una puntuación entre 1 y 0 que indica el valor de opinión positiva (más cercano a 1) o negativa (más cercano a 0) de los datos.

#### <a name="english-language"></a>[Lengua inglesa](#tab/english)

Cuando la referencia cultural es `en-us`, la respuesta es:

```JSON
"sentimentAnalysis": {
  "label": "positive",
  "score": 0.9163064
}
```

#### <a name="other-languages"></a>[Otros idiomas](#tab/other-languages)

Para todas las demás referencias culturales, la respuesta es:

```JSON
"sentimentAnalysis": {
  "score": 0.9163064
}
```
* * *

## <a name="next-steps"></a>Pasos siguientes

Más información acerca del [punto de conexión de predicción de V3](luis-migration-api-v3.md).
