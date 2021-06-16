---
title: 'Inicio rápido: Minería de datos mediante la biblioteca cliente de Text Analytics'
titleSuffix: Azure Cognitive Services
description: Use este inicio rápido para realizar un análisis de sentimiento, entre otras funciones, mediante la API Text Analytics desde Azure Cognitive Services.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: quickstart
ms.date: 06/11/2021
ms.author: aahi
keywords: text mining, sentiment analysis, text analytics
ms.custom: devx-track-python, devx-track-js, devx-track-csharp, cog-serv-seo-aug-2020
zone_pivot_groups: programming-languages-text-analytics
ms.openlocfilehash: dadf293369071ff4f6dd95c3167754f16e952864
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112026965"
---
# <a name="quickstart-use-the-text-analytics-client-library-and-rest-api"></a>Inicio rápido: Uso de la biblioteca cliente y la API REST de Text Analytics

Use este artículo para empezar a trabajar con la biblioteca cliente y la API REST de Text Analytics. Siga estos pasos para probar códigos de ejemplo para realizar tareas de minería de texto:

* análisis de opiniones
* Minería de opiniones
* Detección de idiomas
* Reconocimiento de entidades
* Reconocimiento de información de identificación personal
* Extracción de la frase clave


::: zone pivot="programming-language-csharp"

> [!IMPORTANT]
> * La versión estable más reciente de Text Analytics API es `3.0`.
>    * Asegúrese de seguir únicamente las instrucciones de la versión que está usando.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. En escenarios de producción, se recomienda usar los métodos asincrónicos por lotes para mayor rendimiento y escalabilidad. Para más información, consulte la siguiente documentación de referencia.
> * Si desea usar Text Analytics for Health u operaciones asincrónicas, consulte en GitHub los ejemplos para [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) o [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

[!INCLUDE [C# quickstart](../includes/quickstarts/csharp-sdk.md)]

::: zone-end

::: zone pivot="programming-language-java"

> [!IMPORTANT]
> * La versión estable más reciente de Text Analytics API es `3.0`.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. En escenarios de producción, se recomienda usar los métodos asincrónicos por lotes para mayor rendimiento y escalabilidad. Para más información, consulte la siguiente documentación de referencia.
Si desea usar Text Analytics for Health u operaciones asincrónicas, consulte en GitHub los ejemplos para [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) o [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

[!INCLUDE [Java quickstart](../includes/quickstarts/java-sdk.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

> [!IMPORTANT]
> * La versión estable más reciente de Text Analytics API es `3.0`.
>    * Asegúrese de seguir únicamente las instrucciones de la versión que está usando.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. En escenarios de producción, se recomienda usar los métodos asincrónicos por lotes para mayor rendimiento y escalabilidad. Para más información, consulte la siguiente documentación de referencia.
> * Esta versión de la biblioteca cliente de Text Analytics también se puede ejecutar en el [explorador](https://github.com/Azure/azure-sdk-for-js/blob/master/documentation/Bundling.md).

[!INCLUDE [NodeJS quickstart](../includes/quickstarts/nodejs-sdk.md)]

::: zone-end

::: zone pivot="programming-language-python"

> [!IMPORTANT]
> * La versión estable más reciente de Text Analytics API es `3.0`.
>    * Asegúrese de seguir únicamente las instrucciones de la versión que está usando.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. En escenarios de producción, se recomienda usar los métodos asincrónicos por lotes para mayor rendimiento y escalabilidad. Para más información, consulte la siguiente documentación de referencia. Si desea usar Text Analytics for Health u operaciones asincrónicas, consulte en GitHub los ejemplos para [C#](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/textanalytics/Azure.AI.TextAnalytics), [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/textanalytics/azure-ai-textanalytics/) o [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/textanalytics/azure-ai-textanalytics)

[!INCLUDE [Python quickstart](../includes/quickstarts/python-sdk.md)]

::: zone-end

::: zone pivot="rest-api"

> [!IMPORTANT]
> * La versión estable más reciente de Text Analytics API es `3.0`.
>    * Asegúrese de seguir únicamente las instrucciones de la versión que está usando.

[!INCLUDE [REST API quickstart](../includes/quickstarts/rest-api.md)]

::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Exploración de una solución](../text-analytics-user-scenarios.md#analyze-recorded-inbound-customer-calls)

* [Información general de Text Analytics](../overview.md)
* [Análisis de opiniones](../how-tos/text-analytics-how-to-sentiment-analysis.md)
* [Reconocimiento de entidades](../how-tos/text-analytics-how-to-entity-linking.md)
* [Detección de idioma](../how-tos/text-analytics-how-to-keyword-extraction.md)
* [Reconocimiento de idioma](../how-tos/text-analytics-how-to-language-detection.md)
