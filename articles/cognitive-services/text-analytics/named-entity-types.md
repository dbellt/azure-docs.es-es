---
title: Categorías admitidas para Reconocimiento de entidades con nombre
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre las categorías de entidad admitidas en Text Analytics API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 06/03/2021
ms.author: aahi
ms.openlocfilehash: 9c2cdcaa5aaadf9b1314b436fce4ab747bebdf8c
ms.sourcegitcommit: 70ce9237435df04b03dd0f739f23d34930059fef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2021
ms.locfileid: "111527611"
---
# <a name="supported-entity-categories-in-the-text-analytics-api-v3"></a>Categorías de entidad admitidas en Text Analytics API v3.

Use este artículo para buscar las categorías de entidad que [Reconocimiento de entidades con nombre](how-tos/text-analytics-how-to-entity-linking.md) (NER) puede devolver. NER ejecuta un modelo predictivo para identificar y categorizar entidades con nombre a partir de un documento de entrada.

También está disponible una versión preliminar de NER v3.1, que incluye la capacidad de detectar información personal (`PII`) y de salud (`PHI`). Además, haga clic en la pestaña de **estado** para ver una lista de las categorías admitidas en Text Analytics para el estado. 

Puede encontrar una lista de los tipos devueltos por la versión 2.1 en la [guía de migración](migration-guide.md?tabs=named-entity-recognition).

## <a name="entity-categories"></a>Categorías de entidad

#### <a name="general"></a>[General](#tab/general)

[!INCLUDE [supported entity types - general](./includes/entity-types/general-entities.md)]

#### <a name="pii"></a>[PII](#tab/personal)

[!INCLUDE [supported entity types - personally identifying information](./includes/entity-types/personal-information-entities.md)]

#### <a name="health"></a>[Estado](#tab/health)

[!INCLUDE [biomedical entity types](./includes/entity-types/health-entities.md)]

***

## <a name="next-steps"></a>Pasos siguientes

* [Uso del reconocimiento de entidades con nombre en Text Analytics](how-tos/text-analytics-how-to-entity-linking.md)
