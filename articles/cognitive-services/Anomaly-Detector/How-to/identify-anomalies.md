---
title: Uso de Anomaly Detector API en datos de serie temporal
titleSuffix: Azure Cognitive Services
description: Aprenda a detectar anomalías en los datos como un lote, o en la transmisión de datos.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 10/01/2019
ms.author: mbullwin
ms.openlocfilehash: 6657147d044d49ff72532ad270552a175c197c8f
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110078227"
---
# <a name="how-to-use-the-anomaly-detector-api-on-your-time-series-data"></a>Procedimientos: Uso de Anomaly Detector API en datos de serie temporal  

[Anomaly Detector API](https://westus2.dev.cognitive.microsoft.com/docs/services/AnomalyDetector/operations/post-timeseries-entire-detect) proporciona dos métodos de detección de anomalías. Las anomalías se pueden detectar como un lote a lo largo de la serie temporal, o a medida que se generan los datos mediante la detección del estado de anomalía del punto de datos más reciente. El modelo de detección devuelve resultados de anomalía junto con el valor esperado de cada punto de datos, y los límites de detección de anomalías superior e inferior. Puede usar estos valores para visualizar el intervalo de valores normales y las anomalías en los datos.

## <a name="anomaly-detection-modes"></a>Modos de detección de anomalías 

Anomaly Detector API proporciona dos modos de detección: por lotes y transmisión.

> [!NOTE]
> Las siguientes direcciones URL de solicitud se deben combinar con el punto de conexión adecuado para su suscripción. Por ejemplo: `https://<your-custom-subdomain>.api.cognitive.microsoft.com/anomalydetector/v1.0/timeseries/entire/detect`


### <a name="batch-detection"></a>Detección por lotes

Para detectar anomalías a lo largo de un lote de puntos de datos durante un intervalo de tiempo determinado, use el siguiente URI de solicitud con datos de serie temporal: 

`/timeseries/entire/detect`. 

Al enviarse los datos de serie temporal a la vez, la API genera un modelo con toda la serie y analiza cada punto de datos con ella.  

### <a name="streaming-detection"></a>Detección mediante transmisión

Para detectar continuamente anomalías al transmitir datos, use el siguiente URI de solicitud con el punto de datos más reciente: 

`/timeseries/last/detect`. 

Al enviar nuevos puntos de datos a medida que se generan, puede supervisar los datos en tiempo real. Se genera un modelo con los puntos de datos que envíe, y la API determina si el punto más reciente de la serie temporal es una anomalía.

## <a name="adjusting-lower-and-upper-anomaly-detection-boundaries"></a>Ajuste de los límites de detección de anomalías inferior y superior

De forma predeterminada, los límites superior e inferior de la detección de anomalías se calculan con `expectedValue`, `upperMargin` y `lowerMargin`. Si necesita límites diferentes, se recomienda aplicar `marginScale` a `upperMargin` o `lowerMargin`. Los límites se calcularían del siguiente modo:

|Límite  |Cálculo  |
|---------|---------|
|`upperBoundary` | `expectedValue + (100 - marginScale) * upperMargin`        |
|`lowerBoundary` | `expectedValue - (100 - marginScale) * lowerMargin`        |

En los ejemplos siguientes se muestra un resultado de Anomaly Detector API en distintas sensibilidades.

### <a name="example-with-sensitivity-at-99"></a>Ejemplo con sensibilidad en 99

![Sensibilidad predeterminada](../media/sensitivity_99.png)

### <a name="example-with-sensitivity-at-95"></a>Ejemplo con sensibilidad en 95

![Sensibilidad de 99](../media/sensitivity_95.png)

### <a name="example-with-sensitivity-at-85"></a>Ejemplo con sensibilidad en 85

![Sensibilidad de 85:](../media/sensitivity_85.png)

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Anomaly Detector API?](../overview.md)
* [Inicio rápido: Uso de la biblioteca cliente de Anomaly Detector](../quickstarts/client-libraries.md)
