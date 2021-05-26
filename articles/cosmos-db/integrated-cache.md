---
title: Caché integrada de Azure Cosmos DB
description: La caché integrada de Azure Cosmos DB es una caché en memoria que le ayuda a garantizar costos administrables y baja latencia a medida que crece el volumen de solicitudes.
author: timsander1
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: conceptual
ms.date: 05/25/2021
ms.author: tisande
ms.openlocfilehash: f0a0556ce2a46f922e387d96d20b6425ab362580
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386447"
---
# <a name="azure-cosmos-db-integrated-cache---overview-preview"></a>Caché integrada de Azure Cosmos DB: información general (versión preliminar)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La caché integrada de Azure Cosmos DB es una caché en memoria que le ayuda a garantizar costos administrables y baja latencia a medida que crece el volumen de solicitudes. La memoria caché integrada es fácil de configurar y no es necesario dedicar tiempo a escribir código personalizado para la invalidación de caché ni administrar la infraestructura de back-end. La memoria caché integrada usa una [puerta de enlace dedicada](dedicated-gateway.md) dentro de la cuenta de Azure Cosmos DB. La caché integrada es la primera de muchas características de Azure Cosmos DB que usarán una puerta de enlace dedicada para mejorar el rendimiento. Puede elegir entre tres posibles tamaños de puerta de enlace dedicada en función del número de núcleos y memoria necesarios para la carga de trabajo.

Una caché integrada se configura automáticamente dentro de la puerta de enlace dedicada. La memoria caché integrada tiene dos partes: 

* Una caché de elementos para lecturas puntuales 
* Una caché de consulta para consultas

La caché integrada es una caché de lectura y escritura a través con una directiva de expulsión de menos usados recientemente (LRU). La caché de elementos y la caché de consulta comparten la misma capacidad dentro de la caché integrada y la directiva de expulsión de LRU se aplica a ambas. En otras palabras, los datos se expulsan de la memoria caché estrictamente en función de los menos usados recientemente, independientemente de si se trata de una consulta o una lectura puntual.

## <a name="workloads-that-benefit-from-the-integrated-cache"></a>Cargas de trabajo que se benefician de la memoria caché integrada

El objetivo principal de la caché integrada es reducir los costos de las cargas de trabajo con mucha actividad de lectura. La baja latencia, aunque útil, no es la principal ventaja de la caché integrada porque Azure Cosmos DB ya es rápido sin almacenamiento en caché.

Las lecturas puntuales y las consultas que alcanzaron la memoria caché integrada no usarán ninguna RU. En otras palabras, los aciertos de caché tendrán un cargo de RU de 0. Los aciertos de caché tendrán un costo por operación mucho menor que las lecturas de la base de datos back-end.

Las cargas de trabajo que se ajusten a las siguientes características deben evaluar si la memoria caché integrada ayudará a reducir los costos:

-   Cargas de trabajo con mucha actividad de lectura
-   Muchas lecturas puntuales repetidas en elementos grandes
-   Muchas consultas de RU elevadas repetidas
-   Clave de partición activa para lecturas

El factor más importante en el ahorro esperado es el grado en que las lecturas se repiten. Si la carga de trabajo ejecuta de forma coherente las mismas lecturas puntuales o consultas en un breve período de tiempo, es una excelente candidata para la caché integrada. Cuando se usa la memoria caché integrada para lecturas repetidas, solo se usan RU para la primera lectura. Las lecturas posteriores enrutadas a través del mismo nodo de puerta de enlace dedicado (dentro de la ventana `MaxIntegratedCacheStaleness` y siempre que no se hayan expulsado los datos) no usarán el rendimiento.

Algunas cargas de trabajo no deben tener en cuenta la memoria caché integrada, entre las que se incluyen:

-   Cargas de trabajo con mucha actividad de escritura
-  Consultas o lecturas puntuales que rara vez se repiten

## <a name="item-cache"></a>Caché de elementos

Puede usar la caché de elementos para las lecturas puntuales (es decir, las búsquedas de clave-valor basadas en el identificador de elemento y la clave de partición).

### <a name="populating-the-item-cache"></a>Rellenado de la caché de elementos

- Las nuevas escrituras, actualizaciones y eliminaciones se rellenan automáticamente en la caché de elementos
- Si la aplicación intenta leer un elemento específico que no estaba anteriormente en la caché (error de caché), el elemento se almacenaría ahora en la caché de elementos.

### <a name="item-cache-invalidation-and-eviction"></a>Invalidación y expulsión de la caché de elementos

- Actualización o eliminación del elemento
- Menos usados recientemente (LRU)
- Tiempo de retención en caché (es decir, `MaxIntegratedCacheStaleness`)

## <a name="query-cache"></a>Caché de consultas

La caché de consulta se puede usar para almacenar en caché las consultas. La caché de consulta transforma una consulta en una búsqueda clave-valor donde la clave es el texto de la consulta y el valor son los resultados de la consulta. La memoria caché integrada no tiene un motor de consultas, solo almacena la búsqueda de clave-valor para cada consulta.

### <a name="populating-the-query-cache"></a>Rellenado de la caché de consulta

- Si la caché no tiene un resultado para esa consulta (error de caché), la consulta se envía al back-end. Una vez ejecutada la consulta, la caché almacenará sus resultados.

### <a name="query-cache-eviction"></a>Expulsión de la caché de consulta

- Menos usados recientemente (LRU)
- Tiempo de retención en caché (es decir, `MaxIntegratedCacheStaleness`)

### <a name="working-with-the-query-cache"></a>Uso de la caché de consulta

No necesita código especial al trabajar con la caché de consulta, incluso si las consultas tienen varias páginas de resultados. Los procedimientos recomendados y el código para la paginación de consultas son los mismos, tanto si la consulta alcanza la caché integrada como si se ejecuta en el motor de consulta back-end.

La caché de consulta almacenará en caché automáticamente los tokens de continuación de consultas, si procede. Si tiene una consulta con varias páginas de resultados, las páginas almacenadas en la caché integrada tendrán un cargo de RU de 0. Si las páginas posteriores de los resultados de la consulta requieren la ejecución de back-end, tendrán un token de continuación de la página anterior para que puedan evitar duplicar el trabajo anterior.

> [!NOTE]
> Las instancias de caché integradas dentro de distintos nodos de puerta de enlace dedicados tienen cachés independientes entre sí. Si los datos se almacenan en caché dentro de un nodo, no necesariamente se almacenan en caché en los demás.

## <a name="integrated-cache-consistency"></a>Coherencia de la caché integrada

La caché integrada solo admite la [coherencia](consistency-levels.md) final. Si una lectura tiene un prefijo coherente, una sesión, obsolescencia limitada o coherencia fuerte, siempre omitirá la caché integrada.

La manera más fácil de configurar la coherencia final para todas las lecturas es [establecerla en el nivel de cuenta](consistency-levels.md#configure-the-default-consistency-level). Sin embargo, si solo desea que algunas de las lecturas tengan coherencia final, también puede configurar la coherencia en el [nivel de solicitud](how-to-manage-consistency.md#override-the-default-consistency-level).

## <a name="integrated-cache-retention-time"></a>Tiempo de retención en la caché integrada

El tiempo de retención en caché es la retención máxima de los datos almacenados en caché. Durante la versión preliminar, `MaxIntegratedCacheStaleness` siempre se establece en 5 minutos y no es posible personalizarlo.

## <a name="metrics"></a>Métricas

Al usar la memoria caché integrada, resulta útil supervisar algunas métricas clave. Las métricas de la caché integrada incluyen:

- `DedicatedGatewayCpuUsage`: uso de CPU por cada nodo de puerta de enlace dedicado
- `DedicatedGatewayMemoryUsage`: uso de memoria por cada nodo de puerta de enlace dedicado para las solicitudes de enrutamiento y el almacenamiento en caché
- `DedicatedGatewayRequests`: número de solicitudes enrutadas a través de cada nodo de puerta de enlace dedicado
- `IntegratedCacheEvictedEntriesSize`: cantidad de datos expulsada de la memoria caché integrada
- `IntegratedCacheTTLExpirationCount`: número de entradas expulsadas de la memoria caché integrada específicamente debido a que los datos en caché superan el tiempo de `MaxIntegratedCacheStaleness`.
- `IntegratedCacheHitRate`: proporción de consultas y lecturas puntuales que usaban la caché integrada (de todas las solicitudes de puerta de enlace dedicadas que intentaron usar la caché integrada).

Todas las métricas existentes están disponibles, de manera predeterminada, en la hoja **Métricas** (no en la hoja de métricas clásica):

   :::image type="content" source="./media/integrated-cache/integrated-cache-metrics.png" alt-text="Imagen que muestra la ubicación de las métricas de la caché integrada" border="false":::

Todas las métricas se exponen como promedio en todos los nodos de puerta de enlace dedicados. Por ejemplo, si aprovisiona un clúster de puerta de enlace dedicado con cinco nodos, las métricas reflejan el valor medio en los cinco nodos.

## <a name="troubleshooting-common-issues"></a>Solucionar los problemas comunes

En los ejemplos siguientes, se muestra cómo depurar algunos escenarios comunes:

### <a name="i-cant-tell-if-my-application-is-using-the-dedicated-gateway"></a>No sé si mi aplicación usa la puerta de enlace dedicada

Active `DedicatedGatewayRequests`. Esta métrica incluye todas las solicitudes que usan la puerta de enlace dedicada, independientemente de si han alcanzado la caché integrada. Si la aplicación usa la puerta de enlace estándar o el modo directo con la cadena de conexión original, no verá un mensaje de error, pero `DedicatedGatewayRequests` será cero.

### <a name="i-cant-tell-if-my-requests-are-hitting-the-integrated-cache"></a>No sé si mis solicitudes alcanzan la memoria caché integrada

Active `IntegratedCacheHitRate`. Si este valor es cero, las solicitudes no alcanzan la memoria caché integrada. Compruebe que usa la cadena de conexión de puerta de enlace dedicada, que se conecta con el modo de puerta de enlace, y que ha establecido la coherencia final.

### <a name="i-want-to-understand-if-my-dedicated-gateway-is-too-small"></a>Quiero saber si mi puerta de enlace dedicada es demasiado pequeña

Active `IntegratedCacheHitRate`. Si este valor es alto (por ejemplo, por encima de 0,5-0,6), esto es una buena señal de que la puerta de enlace dedicada es lo suficientemente grande.

Si `IntegratedCacheHitRate` es bajo, observe `IntegratedCacheEvictedEntriesSize`. Si `IntegratedCacheEvictedEntriesSize` es alto, puede significar que un mayor tamaño de puerta de enlace dedicada sería beneficioso.

### <a name="i-want-to-understand-if-my-dedicated-gateway-is-too-large"></a>Quiero saber si mi puerta de enlace dedicada es demasiado grande

Esto es más difícil de medir. En general, debe empezar por valores pequeños y aumentar lentamente el tamaño de la puerta de enlace dedicada hasta que `IntegratedCacheHitRate` deje de mejorar.

Si la mayoría de los datos se expulsa de la memoria caché debido a que se supera `MaxIntegratedCacheStaleness`, en lugar de LRU, la memoria caché podría ser mayor de lo necesario. Compruebe si `IntegratedCacheTTLExpirationCount` es casi tan grande como `IntegratedCacheEvictedEntriesSize`. Si es así, puede experimentar con un tamaño de puerta de enlace dedicado más pequeño y comparar el rendimiento.

Compruebe `DedicatedGatewayMemoryUsage` y compárelo con el tamaño de la puerta de enlace dedicada. Si `DedicatedGatewayMemoryUsage` es menor que el tamaño de puerta de enlace dedicado, debe probar un tamaño de puerta de enlace dedicado más pequeño.

## <a name="next-steps"></a>Pasos siguientes

- [Preguntas más frecuentes sobre caché integrada](integrated-cache-faq.md)
- [Configuración de la memoria caché integrada](how-to-configure-integrated-cache.md)
- [Puerta de enlace dedicada](dedicated-gateway.md)