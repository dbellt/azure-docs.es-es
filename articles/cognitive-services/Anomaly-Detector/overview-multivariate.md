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
ms.openlocfilehash: e913cc4738b0f58a411e80f3e2b602a072f28665
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112005994"
---
# <a name="multivariate-time-series-anomaly-detection-public-preview"></a>Detección de anomalías de serie temporal multivariante (versión preliminar pública)

Las nuevas API de **detección de anomalías multivariante** permiten a los desarrolladores integrar fácilmente la inteligencia artificial avanzada para detectar anomalías de grupos de métricas, sin necesidad de tener conocimientos de aprendizaje automático ni datos etiquetados. Las dependencias y las correlaciones de hasta 300 señales diferentes ahora se cuentan automáticamente como factores clave. Esta nueva funcionalidad le permite proteger de forma proactiva los sistemas complejos, como aplicaciones de software, servidores, máquinas de fábrica, vehículos espaciales o incluso su empresa, frente a errores.

![Varios gráficos de líneas de serie temporal para variables de vibración, temperatura, presión, velocidad, velocidad de rotación con anomalías resaltadas en naranja.](./media/multivariate-graph.png)

Imagine que tiene 20 sensores de un motor automático que generan 20 señales diferentes, como vibración, temperatura, presión de combustible, etc. Es posible que las lecturas individuales de esas señales no le informen mucho sobre los problemas de nivel del sistema, pero juntos pueden representar el estado del motor. Cuando la interacción de esas señales se desvía fuera del intervalo habitual, la característica de detección de anomalías multivariante puede detectar la anomalía como un experto con experiencia. Los modelos de IA subyacentes se entrenan y personalizan con los datos, de forma que comprendan las necesidades únicas del negocio. Con las nuevas API de Anomaly Detector, los desarrolladores ahora pueden integrar fácilmente las funcionalidades de detección de anomalías de serie temporal multivariante en soluciones de mantenimiento predictivo, soluciones de supervisión de AIOps para software empresarial complejo o herramientas de inteligencia empresarial.

## <a name="when-to-use-multivariate-versus-univariate"></a>Cuándo usar un elemento **multivariante** frente a uno **univariante**

Si su objetivo es detectar anomalías en un patrón normal en cada serie temporal individual basándose únicamente en sus propios datos históricos, utilice las API de detección de anomalías univariante. Por ejemplo, quiere detectar anomalías en los ingresos diarios en función de los propios datos de ingresos o quiere detectar un pico de CPU en función de los datos de la CPU solamente.

Si su objetivo es detectar anomalías en el nivel del sistema a partir de un grupo de datos de serie temporal, utilice las API de detección de anomalías multivariante. En particular, cuando una serie temporal individual no le proporciona muchos datos y tiene que ver todas las señales (un grupo de series temporales) holísticamente para determinar un problema de nivel de sistema. Por ejemplo, tiene un recurso físico costoso, como un avión, el equipo de una plataforma petrolífera o un satélite. Cada uno de estos recursos tiene decenas o cientos de tipos diferentes de sensores. Tendría que ver todas esas señales de serie temporal de esos sensores para decidir si hay un problema de nivel de sistema.

## <a name="notebook"></a>Notebook

Para aprender a llamar a Anomaly Detector API (multivariante), pruebe este [cuaderno](https://github.com/Azure-Samples/AnomalyDetector/blob/master/ipython-notebook/Multivariate%20API%20Demo%20Notebook.ipynb). En esta instancia de Jupyter Notebook se muestra cómo enviar una solicitud de API y visualizar el resultado.

Para ejecutar el cuaderno, debe obtener una **clave de suscripción** de Anomaly Detector API y un **punto de conexión de API**. En el cuaderno, agregue la clave de suscripción de Anomaly Detector API válida a la variable `subscription_key` y cambie la variable `endpoint` al punto de conexión.


## <a name="region-support"></a>Regiones admitidas

La versión preliminar pública de Anomaly Detector multivariante está disponible actualmente en seis regiones: Oeste de EE. UU. 2, Oeste de Europa, Este de EE. UU. 2, Centro-sur de EE. UU., Este de EE. UU. y Sur de Reino Unido.

## <a name="algorithms"></a>Algoritmos

Consulte los siguientes documentos técnicos para obtener información acerca de los algoritmos usados:

* Blog: [Introducción a la detección de anomalía multivariante](https://techcommunity.microsoft.com/t5/azure-ai/introducing-multivariate-anomaly-detection/ba-p/2260679)
* Documento: [Detección de anomalías de serie temporal multivariante a través de la red de atención de Graph](https://arxiv.org/abs/2009.02040)


> [!VIDEO https://www.youtube.com/embed/FwuI02edclQ]


## <a name="join-the-anomaly-detector-community"></a>Únase a la comunidad de Anomaly Detector

- Únase al [grupo de asesores de Anomaly Detector en Microsoft Teams](https://aka.ms/AdAdvisorsJoin)

## <a name="next-steps"></a>Pasos siguientes

- [Inicios rápidos](./quickstarts/client-libraries-multivariate.md).
- [Procedimientos recomendados](./concepts/best-practices-multivariate.md): este artículo trata sobre los patrones recomendados para usar con las API multivariantes.
