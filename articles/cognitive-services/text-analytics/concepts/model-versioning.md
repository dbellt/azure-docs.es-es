---
title: Especificación de la versión de modelo en Text Analytics v3
titleSuffix: Azure Cognitive Services
description: Obtenga información sobre cómo especificar el modelo de Text Analytics API que se usa en los datos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: article
ms.date: 06/03/2021
ms.author: aahi
ms.openlocfilehash: 60b9b7ace0f788c7ae5b248c4011ccde52811d2b
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111757146"
---
# <a name="model-versioning-in-the-text-analytics-api"></a>Control de versiones de modelos en Text Analytics API

La versión 3 de Text Analytics API le permite elegir la versión del modelo que se usará en los datos. Use el parámetro opcional `model-version` para seleccionar la versión del modelo en sus solicitudes de API. Por ejemplo: `<resource-url>/text/analytics/v3.0/sentiment?model-version=2020-04-01`. Si no se especifica este parámetro, la API usará de forma predeterminada la versión estable más reciente. 

## <a name="available-versions"></a>Versiones disponibles

Use la siguiente tabla para averiguar qué modelos de versiones admite cada punto de conexión hospedado.


| Punto de conexión                        | Versiones admitidas                                     | Versión más reciente |
|---------------------------------|--------------------------------------------------------|----------------|
| `/sentiment`                    | `2019-10-01`, `2020-04-01`                             | `2020-04-01`   |
| `/languages`                    | `2019-10-01`, `2020-07-01`, `2020-09-01`, `2021-01-05` | `2021-01-05`   |
| `/entities/linking`             | `2019-10-01`, `2020-02-01`                             | `2020-02-01`   |
| `/entities/recognition/general` | `2019-10-01`, `2020-02-01`, `2020-04-01`,`2021-01-15`,`2021-06-01`  | `2021-06-01`   |
| `/entities/recognition/pii`     | `2019-10-01`, `2020-02-01`, `2020-04-01`,`2020-07-01`, `2021-01-15`  | `2021-01-15`   |
| `/entities/health`              | `2021-05-15`                           | `2021-05-15`   |
| `/keyphrases`                   | `2019-10-01`, `2020-07-01`, `2021-06-01`  | `2021-06-01`   |


Encontrará más detalles sobre las actualizaciones de estos modelos en [Novedades](../whats-new.md).

## <a name="text-analytics-for-health"></a>Text Analytics for Health

El contenedor de [Text Analytics for Health](../how-tos/text-analytics-for-health.md) usa versiones de modelo independientes de los puntos de conexión de API anteriores.  Tenga en cuenta que solo hay una versión de modelo disponible por cada imagen de contenedor.

| Punto de conexión                        | Etiqueta de la imagen de contenedor                     | Versión del modelo |
|---------------------------------|-----------------------------------------|---------------|
| `/entities/health`              | `3.0.015370001-onprem-amd64` o más reciente          | `2021-03-01`  |
| `/entities/health`              | `1.1.013530001-amd64-preview`           | `2020-09-03`  |
| `/entities/health`              | `1.1.013150001-amd64-preview`           | `2020-07-24`  |
| `/domains/health`               | `1.1.012640001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012420001-amd64-preview`           | `2020-05-08`  |
| `/domains/health`               | `1.1.012070001-amd64-preview`           | `2020-04-16`  |


## <a name="next-steps"></a>Pasos siguientes

* [Información general de Text Analytics](../overview.md)
* [Análisis de opiniones](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Reconocimiento de entidades](../how-tos/text-analytics-how-to-entity-linking.md)
