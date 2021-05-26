---
title: Oferta sin servidor basada en el consumo en Azure Cosmos DB
description: Más información sobre la oferta sin servidor basada en el consumo de Azure Cosmos DB.
author: ThomasWeiss
ms.author: thweiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/25/2021
ms.openlocfilehash: 146a1dba3e13ac594e3b4d9edc358d46b0d54201
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110378497"
---
# <a name="azure-cosmos-db-serverless"></a>Azure Cosmos DB sin servidor
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Azure Cosmos DB sin servidor le permite usar su cuenta de Azure Cosmos en un modo basado en el consumo, donde solo se cobran las unidades de solicitud que las operaciones de base de datos consumen y el almacenamiento consumido por los datos. Los contenedores sin servidor pueden servir miles de solicitudes por segundo sin cargos mínimos y no es necesario planear la capacidad.

> [!IMPORTANT] 
> ¿Tiene comentarios sobre el modo sin servidor? Queremos conocerlos. No dude en enviar un mensaje al equipo de Azure Cosmos DB sin servidor: [azurecosmosdbserverless@service.microsoft.com](mailto:azurecosmosdbserverless@service.microsoft.com).

Cuando se usa Azure Cosmos DB, cada operación de base de datos tiene un costo expresado en [unidades de solicitud](request-units.md). La forma en que se le cobra por este costo depende del tipo de cuenta de Azure Cosmos que use:

- En modo de [rendimiento aprovisionado](set-throughput.md), debe confirmar una cantidad determinada de rendimiento (expresado en unidades de solicitud por segundo) que se aprovisiona en las bases de datos y los contenedores. El costo de las operaciones de base de datos se deduce a partir del número de unidades de solicitud disponibles cada segundo. Al final del período de facturación, se le cobrará la cantidad de rendimiento aprovisionado.
- En el modo sin servidor, no tiene que aprovisionar ningún rendimiento al crear contenedores en su cuenta de Azure Cosmos. Al final del período de facturación, se le facturará el número de unidades de solicitud consumidas por las operaciones de base de datos.

## <a name="use-cases"></a>Casos de uso

Azure Cosmos DB sin servidor se adapta mejor a aquellos escenarios en los que se espera **tráfico intermitente e imprevisible** con tiempos de inactividad prolongados. Dado que la capacidad de aprovisionamiento en estas situaciones no es necesaria y su costo puede ser prohibitivo, Azure Cosmos DB sin servidor debe considerarse en los casos de uso siguientes:

- Introducción a Azure Cosmos DB
- Ejecución de aplicaciones con
    - tráfico intermitente a ráfagas que es difícil de predecir, o
    - baja relación de tráfico de promedio a pico (<10 %)
- Desarrollo, prueba, creación de prototipos y ejecución en producción de nuevas aplicaciones cuyo patrón de tráfico es desconocido
- Integración con servicios de proceso sin servidor como [Azure Functions](../azure-functions/functions-overview.md)

Consulte el artículo sobre la [elección entre el rendimiento aprovisionado y el modo sin servidor](throughput-serverless.md) para obtener más información sobre cómo elegir la oferta que mejor se adapte a su caso de uso.

## <a name="using-serverless-resources"></a>Uso de recursos sin servidor

Sin servidor es un nuevo tipo de cuenta de Azure Cosmos, lo que significa que tiene que elegir entre el **rendimiento aprovisionado** y el modo **sin servidor** al crear una nueva cuenta. Debe crear una nueva cuenta sin servidor para empezar a trabajar sin servidor. Durante la versión preliminar, la única manera admitida para crear una nueva cuenta sin servidor es mediante [Azure Portal](create-cosmosdb-resources-portal.md). Actualmente no se admite la migración de las cuentas existentes a o desde el modo sin servidor.

Cualquier contenedor que se cree en una cuenta sin servidor será un contenedor sin servidor. Los contenedores sin servidor exponen las mismas funcionalidades que los contenedores creados en el modo de rendimiento aprovisionado, por lo que puede leer, escribir y consultar los datos de la misma manera. Sin embargo, las cuentas y los contenedores sin servidor también tienen características específicas:

- Una cuenta sin servidor solo puede ejecutarse en una única región de Azure. No es posible agregar más regiones de Azure a una cuenta sin servidor después de crearla.
- No es posible habilitar la [Característica en vista previa (GB) de Synapse Link](synapse-link.md) en una cuenta sin servidor.
- El aprovisionamiento de rendimiento no es necesario en los contenedores sin servidor, por lo que se aplican las siguientes instrucciones:
    - No se puede pasar ningún rendimiento al crear un contenedor sin servidor y, si se hace, se devuelve un error.
    - No se puede leer ni actualizar el rendimiento en un contenedor sin servidor y, si se hace, se devuelve un error.
    - No se puede crear una base de datos de rendimiento compartido en una cuenta sin servidor y, si se hace, se devuelve un error.
- Los contenedores sin servidor pueden almacenar un máximo de 50 GB de datos e índices.

## <a name="monitoring-your-consumption"></a>Supervisión del consumo

Si ya ha usado Azure Cosmos DB en el modo de rendimiento aprovisionado, observará que sin servidor es más rentable cuando el tráfico no justifica la capacidad aprovisionada. La desventaja es que los costos serán menos predecibles porque se facturan en función del número de solicitudes procesadas por la base de datos. Por eso, es importante estar atento al consumo actual.

Al examinar el panel **Métricas** de su cuenta, encontrará un gráfico denominado **Request Units consumed** (Unidades de solicitud consumidas) en la pestaña **Información general**. En este gráfico se muestra el número de unidades de solicitud que ha consumido su cuenta:

:::image type="content" source="./media/serverless/request-units-consumed.png" alt-text="Gráfico que muestra las unidades de solicitud consumidas" border="false":::

Puede encontrar el mismo gráfico al usar Azure Monitor, tal y como se describe [aquí](monitor-request-unit-usage.md). Tenga en cuenta que Azure Monitor le permite configurar [alertas](../azure-monitor/alerts/alerts-metric-overview.md), que se pueden usar para recibir una notificación cuando el consumo de unidades de solicitud ha superado un umbral determinado.

## <a name="performance"></a><a id="performance"></a>Rendimiento

Los recursos sin servidor brindan características de rendimiento específicas que son diferentes a las que ofrecen los recursos de rendimiento aprovisionados. La latencia de los contenedores sin servidor se cubrirá mediante un objetivo de nivel de servicio (SLO) de 10 milisegundos o menos para las lecturas puntuales y de 30 milisegundos o menos para las escrituras. Una operación de lectura de punto consiste en capturar un solo elemento por su identificador y el valor de clave de partición.

## <a name="next-steps"></a>Pasos siguientes

Comience a trabajar con el modo sin servidor con los siguientes artículos:

- [Unidades de solicitud en Azure Cosmos DB](request-units.md)
- [Elija entre el rendimiento aprovisionado y sin servidor](throughput-serverless.md)
- [Modelo de precios de Azure Cosmos DB](how-pricing-works.md)
