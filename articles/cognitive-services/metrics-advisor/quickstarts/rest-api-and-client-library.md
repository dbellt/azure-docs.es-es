---
title: API REST de las bibliotecas cliente de Metrics Advisor
titleSuffix: Azure Cognitive Services
description: Use este inicio rápido para conectar las aplicaciones a la API de Metrics Advisor desde Azure Cognitive Services.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: quickstart
ms.date: 10/14/2020
ms.author: mbullwin
zone_pivot_groups: programming-languages-metrics-monitor
ms.openlocfilehash: dc13aec8cd5bab7df93c25275331b34ea8093206
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894518"
---
# <a name="quickstart-use-the-client-libraries-or-rest-apis-to-customize-your-solution"></a>Inicio rápido: Uso de las API REST o las bibliotecas cliente para personalizar una solución

Introducción a las API REST o las bibliotecas cliente de Metrics Advisor. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Use Metrics Advisor para realizar lo siguiente:

* Incorporación de una fuente de distribución de datos desde un origen de datos
* Comprobación del estado de la ingesta
* Configuración de la detección y las alertas 
* Consulta de los resultados de la detección de anomalías
* Diagnóstico de las anomalías


::: zone pivot="programming-language-csharp"

[!INCLUDE [REST API quickstart](../includes/quickstarts/csharp.md)]

::: zone-end

::: zone pivot="programming-language-java"

[!INCLUDE [REST API quickstart](../includes/quickstarts/java.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

[!INCLUDE [REST API quickstart](../includes/quickstarts/javascript.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [REST API quickstart](../includes/quickstarts/python.md)]

::: zone-end

::: zone pivot="programming-language-rest-api"

[!INCLUDE [REST API quickstart](../includes/quickstarts/rest-api.md)]

::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

- [Uso del portal web](web-portal.md)
- [Incorporación de las fuentes de distribución de datos](../how-tos/onboard-your-data.md)
    - [Administración de las fuentes de distribución de datos](../how-tos/manage-data-feeds.md)
    - [Configuraciones para distintos orígenes de datos](../data-feeds-from-different-sources.md)
- [Configuración de métricas y ajuste de la configuración de la detección](../how-tos/configure-metrics.md)
- [Ajuste de la detección de anomalías mediante los comentarios](../how-tos/anomaly-feedback.md)