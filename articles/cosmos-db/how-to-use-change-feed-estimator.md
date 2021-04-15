---
title: 'Uso del calculador de la fuente de cambios: Azure Cosmos DB'
description: Obtenga información acerca de cómo usar el calculador de la fuente de cambios para analizar el progreso del procesador de la fuente de cambios
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 04/01/2021
ms.author: maquaran
ms.custom: devx-track-csharp
ms.openlocfilehash: 5d4e461b25a25ecdf0d4d89ee7f1c82b9d4a0737
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106220171"
---
# <a name="use-the-change-feed-estimator"></a>Uso del calculador de la fuente de cambios
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En este artículo se describe cómo puede supervisar el progreso de las instancias del [procesador de la fuente de cambios](./change-feed-processor.md) a medida que leen la fuente de cambios.

## <a name="why-is-monitoring-progress-important"></a>¿Por qué es importante el progreso de la supervisión?

El procesador de la fuente de cambios actúa como un puntero que avanza por [la fuente de cambios](./change-feed.md) y entrega los cambios a una implementación de delegado.

La implementación del procesador de la fuente de cambios puede procesar los cambios a una velocidad determinada en función de los recursos disponibles, como la CPU, la memoria, la red, etc.

Si esta velocidad es más lenta que la velocidad a la que se producen los cambios en el contenedor de Azure Cosmos, el procesador comenzará a retrasarse.

La identificación de este escenario ayuda a comprender si es necesario escalar la implementación del procesador de la fuente de cambios.

## <a name="implement-the-change-feed-estimator"></a>Implementación del calculador de la fuente de cambios

### <a name="as-a-push-model-for-automatic-notifications"></a>Como modelo de inserción para las notificaciones automáticas

Al igual que el [procesador de la fuente de cambios](./change-feed-processor.md), el estimador de la fuente de cambios funciona como un modelo de inserción. El calculador medirá la diferencia entre el último elemento procesado (definido por el estado del contenedor de concesiones) y el cambio más reciente en el contenedor, e insertará este valor en un delegado. El intervalo en el que se toma la medida también puede personalizarse con un valor predeterminado de 5 segundos.

Por ejemplo, si el procesador de la fuente de cambios se define de la siguiente manera:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartProcessorEstimator)]

La manera correcta de inicializar un calculador para medir ese procesador sería usar `GetChangeFeedEstimatorBuilder` como se explica a continuación:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartEstimator)]

Donde tanto el procesador como el calculador comparten el mismo `leaseContainer` y el mismo nombre.

Los otros dos parámetros son el delegado, que recibirá un número que representa **cuántos cambios están pendientes de ser leídos** por el procesador y el intervalo de tiempo en el que desea que se realice esta medida.

Un ejemplo de un delegado que recibe la estimación es:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=EstimationDelegate)]

Puede enviarla a su solución de supervisión y usarla para saber cómo se comporta el progreso con el tiempo.

### <a name="as-an-on-demand-detailed-estimation"></a>Como una estimación detallada a petición

A diferencia del modelo de inserción, hay una alternativa que le permite obtener la estimación a petición. Este modelo también proporciona información más detallada:

* Retardo estimado por concesión.
* Instancia que posee y procesa cada concesión, de modo que puede identificar si hay algún problema en una instancia.

Si el procesador de la fuente de cambios se define de la siguiente manera:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartProcessorEstimatorDetailed)]

Puede crear el estimador con la misma configuración de concesión:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=StartEstimatorDetailed)]

Y siempre que quiera, con la frecuencia que necesite, puede obtener la estimación detallada:

[!code-csharp[Main](~/samples-cosmosdb-dotnet-v3/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed/Program.cs?name=GetIteratorEstimatorDetailed)]

Cada `ChangeFeedProcessorState` contendrá la información de la concesión y del retardo, y también quién es la instancia actual propietaria. 

> [!NOTE]
> No es necesario implementar el calculador de la fuente de cambios como parte del procesador de la fuente de cambios ni que formen parte del mismo proyecto. Puede ser independiente y ejecutarse en una instancia completamente diferente, que es lo recomendado. Solo debe usar el mismo nombre y la misma configuración de concesión.

## <a name="additional-resources"></a>Recursos adicionales

* [SDK de Azure Cosmos DB](sql-api-sdk-dotnet.md)
* [Ejemplos de uso en GitHub](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage/ChangeFeed)
* [Ejemplos adicionales en GitHub](https://github.com/Azure-Samples/cosmos-dotnet-change-feed-processor)

## <a name="next-steps"></a>Pasos siguientes

Puede obtener más información sobre el procesador de la fuente de cambios en los siguientes artículos:

* [Introducción a la fuente de cambios](change-feed-processor.md)
* [Hora de inicio del procesador de la fuente de cambios](./change-feed-processor.md#starting-time)