---
title: ¿Qué es el servicio Azure Metrics Advisor?
titleSuffix: Azure Applied AI Services
description: ¿Qué es Metrics Advisor?
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: metrics-advisor
ms.topic: overview
ms.date: 09/14/2020
ms.author: mbullwin
ms.openlocfilehash: 34ab524cfc00cf4f74d632883ffeb18b904342a6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376854"
---
# <a name="what-is-azure-metrics-advisor-preview"></a>¿Qué es Azure Metrics Advisor (versión preliminar)? 

Metrics Advisor forma parte de [Azure Applied AI Services](../../applied-ai-services/what-are-applied-ai-services.md), que usa inteligencia artificial para realizar la supervisión de datos y la detección de anomalías en los datos de series temporales. El servicio automatiza el proceso de aplicación de modelos a los datos y proporciona un conjunto de API y un área de trabajo basada en web para la ingesta de datos, la detección de anomalías y los diagnósticos, sin necesidad de conocer el aprendizaje automático. Los desarrolladores pueden crear aplicaciones de AIOps, mantenimiento de predictivo y supervisión empresarial basadas en el servicio. Use Metrics Advisor para:

* Analizar los datos multidimensionales de varios orígenes de datos
* Identificar y correlacionar las anomalías
* Configurar y ajustar el modelo de detección de anomalías usado en los datos
* Diagnosticar las anomalías y ayudar con el análisis de la causa principal

:::image type="content" source="media/metrics-advisor-overview.png" alt-text="Introducción a Metrics Advisor":::

Esta documentación contiene los siguientes tipos de artículos:
* Los [inicios rápidos](./Quickstarts/web-portal.md) son instrucciones paso a paso que permiten realizar llamadas al servicio y obtener los resultados en un breve período de tiempo. 
* Las [guías paso a paso](./how-tos/onboard-your-data.md) contienen instrucciones para usar el servicio de maneras más específicas o personalizadas.
* Los [artículos conceptuales](glossary.md) proporcionan explicaciones detalladas de la funcionalidad y las características del servicio.

## <a name="connect-to-a-variety-of-data-sources"></a>Conexión a diferentes orígenes de datos

Metrics Advisor puede conectarse e [ingerir datos de métricas multidimensionales](how-tos/onboard-your-data.md) de muchos almacenes de datos, entre los que se incluyen lo siguientes: SQL Server, Azure Blob Storage, MongoDB y otros.

## <a name="easy-to-use-and-customizable-anomaly-detection"></a>Detección de anomalías fácil de usar y personalizable

* Metrics Advisor selecciona automáticamente el mejor modelo para los datos, sin necesidad de realizar ningún aprendizaje automático.
* Supervise automáticamente cada serie temporal dentro de las [métricas multidimensionales](glossary.md#multi-dimensional-metric).
* Use la [optimización de parámetros](how-tos/configure-metrics.md) y los [comentarios interactivos](how-tos/anomaly-feedback.md) para personalizar el modelo que se aplica a los datos y los futuros resultados de la detección de anomalías.

## <a name="real-time-alerts-through-multiple-channels"></a>Alertas en tiempo real en varios canales

Cada vez que se detecten anomalías, Metrics Advisor puede [enviar alertas en tiempo real](how-tos/alerts.md) mediante varios canales con enlaces, como son los de correo electrónico, los canales Web y los de Azure DevOps. Las reglas de alerta flexibles permiten personalizar las alertas que se envían y su destino.

## <a name="smart-diagnostic-insights-by-analyzing-anomalies"></a>Información de diagnóstico inteligente mediante el análisis de anomalías

Analice las anomalías detectadas en las métricas multidimensionales y genere [información de diagnóstico inteligente](how-tos/diagnose-incident.md) como las causas más probables, los árboles de diagnóstico, la exploración de métricas, etc. Al configurar el [grafo de métricas](how-tos/metrics-graph.md), puede habilitarse el análisis de las métricas cruzadas para ayudarle a visualizar los incidentes.


## <a name="typical-workflow"></a>Flujo de trabajo típico

El flujo de trabajo es sencillo: después de incorporar los datos, puede ajustar la detección de anomalías y crear configuraciones para que se adapten a su escenario.

1. [Cree un recurso de Azure](https://go.microsoft.com/fwlink/?linkid=2142156) para Metrics Advisor. 
2. Cree su primer monitor con el portal web.
    1. Incorporación de los datos
    2. Ajuste de la detección de anomalías
    3. Suscripción a alertas
    4. Visualización de la Información de diagnóstico
3. Use la API de REST para personalizar la instancia.

## <a name="next-steps"></a>Pasos siguientes

* Explore una guía de inicio rápido: [Supervisión de la primera métrica en la web](quickstarts/web-portal.md).
* Explore una guía de inicio rápido: [Uso de las API de REST para personalizar su solución](./quickstarts/rest-api-and-client-library.md).
