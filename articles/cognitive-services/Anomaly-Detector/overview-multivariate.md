---
title: ¿Qué es la API multivariante de Anomaly Detector?
titleSuffix: Azure Cognitive Services
description: Información general sobre las API multivariantes de la versión preliminar pública de Anomaly Detector.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 04/01/2021
ms.author: mbullwin
keywords: anomaly detection, machine learning, algorithms
ms.openlocfilehash: 63ad63cd57dce5f503e317e8c6330dca0325ba05
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107318855"
---
# <a name="multivariate-time-series-anomaly-detection-public-preview"></a>Detección de anomalías de serie temporal multivariante (versión preliminar pública)

La primera versión del detector de anomalías de Azure Cognitive Services permite compilar soluciones de supervisión de métricas mediante la [serie temporal univariante](overview.md) fácil de usar de las API de Anomaly Detector. Al permitir el análisis de serie temporal individualmente, la versión univariante de Anomaly Detector proporciona simplicidad y escalabilidad.

Las nuevas API de **detección de anomalías multivariante** permiten a los desarrolladores integrar fácilmente la inteligencia artificial avanzada para detectar anomalías de grupos de métricas, sin necesidad de tener conocimientos de aprendizaje automático ni datos etiquetados. Las dependencias y las correlaciones de hasta 300 señales diferentes ahora se cuentan automáticamente como factores clave. Esta nueva funcionalidad le permite proteger de forma proactiva los sistemas complejos, como aplicaciones de software, servidores, máquinas de fábrica, vehículos espaciales o incluso su empresa, frente a errores.

Imagine que tiene 20 sensores de un motor automático que generan 20 señales diferentes, como vibración, temperatura, presión de combustible, etc. Es posible que las lecturas individuales de esas señales no le informen mucho sobre los problemas de nivel del sistema, pero juntos pueden representar el estado del motor. Cuando la interacción de esas señales se desvía fuera del intervalo habitual, la característica de detección de anomalías multivariante puede detectar la anomalía como un experto con experiencia. Los modelos de IA subyacentes se entrenan y personalizan con los datos, de forma que comprendan las necesidades únicas del negocio. Con las nuevas API de Anomaly Detector, los desarrolladores ahora pueden integrar fácilmente las funcionalidades de detección de anomalías de serie temporal multivariante en soluciones de mantenimiento predictivo, soluciones de supervisión de AIOps para software empresarial complejo o herramientas de inteligencia empresarial.

## <a name="when-to-use-multivariate-versus-univariate"></a>Cuándo usar un elemento **multivariante** frente a uno **univariante**

Use las API de detección de anomalías univariantes si su objetivo es detectar anomalías fuera de un patrón normal en cada serie temporal individual basándose únicamente en sus propios datos históricos. Ejemplos: digamos que quiere detectar anomalías de ingresos diarias en función de los propios datos de ingresos, o que quiere detectar un pico de CPU basado exclusivamente en los datos de la CPU.
- `POST /anomalydetector/v1.0/timeseries/last/detect`
- `POST /anomalydetector/v1.0/timeseries/batch/detect`
- `POST /anomalydetector/v1.0/timeseries/changepoint/detect`

![Gráfico de línea de serie temporal con valores fluctuantes de una sola variable capturados por una línea azul con anomalías identificadas con círculos naranjas.](./media/anomaly_detection2.png)

Use las API de detección de anomalías multivariantes a continuación, si su objetivo es detectar anomalías en el nivel del sistema a partir de un grupo de datos de serie temporal. En particular, cuando una serie temporal individual no le proporciona muchos datos y tiene que ver todas las señales (un grupo de series temporales) holísticamente para determinar un problema de nivel de sistema. Ejemplo: tiene un recurso físico costoso, como un avión, un equipo en una plataforma de petróleo o un satélite. Cada uno de estos recursos tiene decenas o cientos de tipos diferentes de sensores. Tendría que ver todas esas señales de serie temporal de esos sensores para decidir si hay un problema de nivel de sistema.

- `POST /anomalydetector/v1.1-preview/multivariate/models`
- `GET /anomalydetector/v1.1-preview/multivariate/models[?$skip][&$top]`
- `GET /anomalydetector/v1.1-preview/multivariate/models/{modelId}`
- `POST/anomalydetector/v1.1-preview/multivariate/models/{modelId}/detect`
- `GET /anomalydetector/v1.1-preview/multivariate/results/{resultId}`
- `DELETE /anomalydetector/v1.1-preview/multivariate/models/{modelId}`
- `GET /anomalydetector/v1.1-preview/multivariate/models/{modelId}/export`

![Varios gráficos de líneas de serie temporal para variables de vibración, temperatura, presión, velocidad, velocidad de rotación con anomalías resaltadas en naranja.](./media/multivariate-graph.png)

## <a name="region-support"></a>Regiones admitidas

La versión preliminar pública de Anomaly Detector multivariante está disponible actualmente en tres regiones: Oeste de EE. UU. 2, Este de EE. UU. 2 y Oeste de Europa.

## <a name="algorithms"></a>Algoritmos

- [Detección de anomalías de serie temporal multivariante a través de la red de atención de Graph](https://arxiv.org/abs/2009.02040)

## <a name="join-the-anomaly-detector-community"></a>Únase a la comunidad de Anomaly Detector

- Únase al [grupo de asesores de Anomaly Detector en Microsoft Teams](https://aka.ms/AdAdvisorsJoin)

## <a name="next-steps"></a>Pasos siguientes

- [Guías de inicio rápido](./quickstarts/client-libraries-multivariate.md).
- [Procedimientos recomendados:](./concepts/best-practices-multivariate.md)este artículo trata sobre los patrones recomendados para usar con las API multivariantes.
