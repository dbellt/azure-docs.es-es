---
title: Optimización de cualquier aplicación de Azure Cosmos DB mediante la limitación de velocidad
description: En este artículo se proporciona a los desarrolladores una metodología para limitar la velocidad de las solicitudes a Azure Cosmos DB. La implementación de este patrón puede reducir los errores y mejorar el rendimiento general de las cargas de trabajo que superan el rendimiento aprovisionado de la base de datos o el contenedor de destino.
author: plasne
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/07/2021
ms.author: pelasne
ms.openlocfilehash: 01bde3cdcb5ee39dfc3e1959165523cfd05c3481
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110105447"
---
# <a name="optimize-your-azure-cosmos-db-application-using-rate-limiting"></a>Optimización de cualquier aplicación de Azure Cosmos DB mediante la limitación de velocidad

En este artículo se proporciona a los desarrolladores una metodología para limitar la velocidad de las solicitudes a Azure Cosmos DB. La implementación de este patrón puede reducir los errores y mejorar el rendimiento general de las cargas de trabajo que superan el rendimiento aprovisionado de la base de datos o el contenedor de destino.

Las solicitudes que superan el rendimiento aprovisionado en Azure Cosmos DB pueden dar lugar a errores transitorios como [TooManyRequests](troubleshoot-request-rate-too-large.md), [Timeout](troubleshoot-request-timeout.md) y [ServiceUnavailable](troubleshoot-service-unavailable.md). Lo habitual es que estas solicitudes se reintenten cuando la capacidad esté disponible y se realicen correctamente. Sin embargo, este enfoque puede provocar que haya un gran número de solicitudes que sigan la ruta de acceso del error en el código y, habitualmente, reduce el rendimiento.

El rendimiento óptimo del sistema, medido por costo y tiempo, se puede lograr haciendo coincidir el tráfico de cargas de trabajo del cliente con el rendimiento aprovisionado del servidor.

Considere el siguiente escenario:

* Aprovisiona Azure Cosmos DB con 20 000 RU/segundo.
* La aplicación procesa un trabajo de ingesta que contiene 10 000 registros, cada uno de los cuales cuesta 10 RU. La capacidad total necesaria para completar este trabajo es de 100 000 RU.
* Si envía todo el trabajo a Azure Cosmos DB, es muy probable que se produzcan un gran número de errores transitorios y un gran búfer de solicitudes que debe reintentar. Esta condición se debe a que el número total de RU necesarias para el trabajo (100 000) es mucho mayor que el máximo aprovisionado (20 000). Aproximadamente 2000 de los registros se aceptarán en la base de datos, pero se rechazarán aproximadamente 8000. Enviará aproximadamente 8000 registros a Azure Cosmos DB en el reintento, de los que se aceptarán aproximadamente 2000, y así sucesivamente. Cabría esperar que este patrón enviara aproximadamente 30 000 registros, en lugar de 10 000.
* En su lugar, si envía esas solicitudes uniformemente a lo largo de 5 segundos, no debería esperar ningún error y un rendimiento general más rápido, ya que cada lote estaría en los 20 000 aprovisionados, o por debajo de ellos.

La propagación de las solicitudes en un período de tiempo se puede lograr mediante la introducción de un mecanismo de limitación de velocidad en el código.

Las RU aprovisionadas para un contenedor se compartirán de forma uniforme entre el número de particiones físicas. En el ejemplo anterior, si Azure Cosmos DB aprovisionara dos particiones físicas, cada una de ellas tendría 10 000 RU.

Para más información sobre las unidades de solicitud, consulte [Unidades de solicitud en Azure Cosmos DB](request-units.md).
Para más información sobre cómo calcular el número de unidades de solicitud consumidas por la carga de trabajo, consulte [Consideraciones de la unidad de solicitud](request-units.md#request-unit-considerations).
Para más información sobre la creación de particiones en Azure Cosmos DB, consulte [Creación de particiones y escalado horizontal en Azure Cosmos DB](partitioning-overview.md).

## <a name="methodology"></a>Metodología

Este podría ser un enfoque para implementar la limitación de velocidad:

1. Genere un perfil de su aplicación para tener datos sobre las solicitudes de escritura y lectura que se usan.
1. Defina todos los índices.
1. Rellene la colección con una cantidad razonable de datos (pueden ser datos de ejemplo). Si espera que la aplicación normalmente tenga millones de registros, rellénela con millones de registros.
1. Escriba los documentos representativos y registre el costo de la RU.
1. Ejecute las consultas representativas y registre el costo de la RU.
1. Implemente una función en la aplicación para determinar el costo de cualquier solicitud determinada en función de sus conclusiones.
1. Implemente un mecanismo de limitación de velocidad en el código para asegurarse de que la suma de todas las operaciones enviadas a Azure Cosmos DB en un segundo no supera el rendimiento aprovisionado.
1. Realice una prueba de carga de la aplicación y compruebe que no supera el rendimiento aprovisionado.
1. Vuelva a probar los costos de RU periódicamente y actualice la función de costo según sea necesario.

## <a name="indexing"></a>Indexación

A diferencia de otras bases de datos SQL y NoSQL con las que puede estar familiarizado, la directiva de indexación predeterminada de Azure Cosmos DB para los contenedores recién creados indexa **todas las** propiedades. Cada propiedad indexada aumenta el costo de RU de las escrituras.

La directiva de indexación predeterminada puede reducir la latencia en sistemas con muchas operaciones de lectura donde las condiciones de filtro de consulta están bien distribuidas entre todos los campos almacenados. Por ejemplo, podrían beneficiarse los sistemas en los Azure Cosmos DB dedica la mayor parte de su tiempo a atender las búsquedas ad hoc diseñadas por el usuario final.

Si lo desea, es posible excluir de la indexación aquellas propiedades en las que nunca se busca. La eliminación de propiedades del índice podría mejorar el rendimiento general del sistema (costo y tiempo) en los sistemas con muchas operaciones de escritura y en los que los patrones de recuperación de registros están más restringidos.

Antes de medir los costos, es preciso configurar intencionadamente una directiva de índice adecuada para los casos de uso. Además, si posteriormente cambia los índices, tendrá que volver a realizar todos los cálculos de costos. 

Siempre que sea posible, el uso de una carga que refleje consultas típicas en condiciones normales y de máxima demanda para probar cualquier sistema en desarrollo le ayudará a revelar qué directiva de indexación se debe usar.

Para más información sobre los índices, consulte [Directivas de indexación de Azure Cosmos DB](index-policy.md).

## <a name="measuring-cost"></a>Medición del costo

Hay algunos conceptos clave que deben tenerse en cuenta al medir el costo:

* Tenga en cuenta todos los factores que afectan al uso de RU, como se describe en la sección [Consideraciones de la unidad de solicitud](request-units.md#request-unit-considerations).
* Todas las operaciones de lectura y escritura de una base de datos o de un contenedor compartirán el mismo rendimiento aprovisionado.
* Se incurre en el consumo de RU, independientemente de las API de Azure Cosmos DB que se usen.
* La estrategia de partición de una colección puede tener un impacto significativo en el costo de un sistema. Para más información, consulte [Creación de particiones y escalado horizontal en Azure Cosmos DB](partitioning-overview.md#choose-partitionkey).
* Use documentos y consultas representativos.
  * Son aquellos documentos y consultas que crea que se acercan a lo que encontrará el sistema operativo.
  * La mejor manera de obtener estos documentos y consultas es instrumentar el uso de la aplicación. Siempre es mejor tomar una decisión controlada por datos.
* Mida los costos de forma periódica.
  * Cambios en el índice, el tamaño de los índices puede afectar al costo. 
  * Será útil crear alguna prueba repetible (quizás incluso automatizada) de los documentos y consultas representativos.
  * Asegúrese de que los documentos y consultas representativos siguen siendo representativos.

El método para determinar el costo de una solicitud es diferente en cada API:

* [Core API](find-request-unit-charge.md)
* [Cassandra API](find-request-unit-charge-cassandra.md)
* [Gremlin API](find-request-unit-charge-gremlin.md)
* [API de Mongo DB](find-request-unit-charge-mongodb.md)
* [Table API](find-request-unit-charge-table.md)

## <a name="write-requests"></a>Solicitudes de escritura

El costo de las operaciones de escritura suele ser fácil de predecir. Insertará en registros y documentará el costo que Azure Cosmos ha notificado.

Si tiene documentos de distinto tamaño o documentos que usarán índices diferentes, es importante medirlos todos.
Es posible que los documentos representativos estén lo suficientemente cerca, en costo, como para que pueda asignar un solo valor a todas las operaciones de escritura.
Por ejemplo, si encontró los costos de 13,14 RU; 16,01 RU y 12,63 RU; podría promediar esos costos a 14 RU.

## <a name="read-requests"></a>Solicitudes de lectura

El costo de las operaciones de consulta puede ser mucho más difícil de predecir por los siguientes motivos:

* Si el sistema admite consultas definidas por el usuario, será preciso que asigne las consultas entrantes a las consultas representativas para ayudarle a determinar el costo. Este proceso puede adoptar varias formas:
  * Es posible que coincida exactamente con las consultas. Si no hay coincidencia directa, es posible que tenga que encontrar la consulta representativa de la que esté más cerca.
  * Es posible que pueda calcular un costo en función de las características de la consulta. Por ejemplo, puede percatarse de que cada cláusula de la consulta tiene un costo determinado o de que una propiedad indexada cuesta "x", mientras que una no indexada cuesta "y", etc.
* El número de resultados puede variar y, salvo que tenga estadísticas, no puede predecir el impacto de una unidad de solicitud a partir de la carga de retorno.

Es probable que no tenga un costo individual de las operaciones de consulta, sino alguna función que evalúe la consulta y calcule un costo.
Si usa Core API, puede evaluar el costo real de la operación y determinar la precisión de la estimación (el ajuste de esta estimación podría incluso producirse automáticamente dentro del código).

## <a name="transient-fault-handling"></a>Control de errores transitorios

La aplicación necesitará un control de errores transitorios aunque se implemente un mecanismo de limitación de velocidad por los siguientes motivos:

* El costo real de una solicitud puede ser diferente del previsto.
* Los errores transitorios pueden producirse por motivos distintos de TooManyRequests.

Sin embargo, la implementación correcta de un mecanismo de limitación de velocidad en la aplicación reducirá considerablemente el número de errores transitorios.

## <a name="alternate-and-related-techniques"></a>Técnicas alternativas y relacionadas

Aunque en este artículo se describe la coordinación en el cliente y el procesamiento por lotes de cargas de trabajo, hay otras técnicas que se pueden emplear para administrar el rendimiento general del sistema.

### <a name="autoscaling"></a>Escalado automático

El rendimiento aprovisionado de escalado automático de Azure Cosmos DB permite escalar el rendimiento (RU/s) de la base de datos o el contenedor de forma automática e instantánea. El rendimiento se escala en función del uso, sin afectar a la disponibilidad, la latencia o el rendimiento de la carga de trabajo.

El rendimiento aprovisionado con escalabilidad automática es adecuado para cargas de trabajo críticas que tienen patrones de tráfico variables o imprevisibles y requieren SLA para el alto rendimiento y la escala.

Para más información sobre la escalabilidad automática, consulte [Creación de contenedores y bases de datos de Azure Cosmos con rendimiento de escalabilidad automática](provision-throughput-autoscale.md).

### <a name="queue-based-load-leveling-pattern"></a>Patrón Queue-Based Load Leveling

Puede emplear una cola que actúe como búfer entre un cliente y Azure Cosmos DB para suavizar las cargas pesadas intermitentes que pueden provocar errores en el servicio o que se agote el tiempo de espera de la tarea.

Este modelo es útil en cualquier aplicación que use servicios que estén sujetos a la sobrecarga. Sin embargo, este patrón no es útil si la aplicación espera una respuesta del servicio con una latencia mínima.

Este patrón suele ser adecuado para las operaciones de ingesta.

Para más información sobre este patrón, consulte [Patrón Queue-Based Load Leveling](/azure/architecture/patterns/queue-based-load-leveling).

### <a name="cache-aside-pattern"></a>Patrón Cache-Aside

Puede considerar la posibilidad de cargar datos a petición en una caché, en lugar de consultar Azure Cosmos DB cada vez. El uso de una caché puede mejorar el rendimiento y también ayuda a mantener la coherencia entre los datos contenidos en la caché y los datos del almacén de datos subyacente.

Para más información, consulte el artículo [Patrón Cache-Aside](/azure/architecture/patterns/cache-aside).

### <a name="materialized-view-pattern"></a>Patrón Materialized View

Puede rellenar previamente las vistas en otras colecciones después de almacenar los datos en Azure Cosmos DB cuando los datos no tienen el formato ideal para las operaciones de consulta necesarias. Este patrón puede ayudar a que la realización de consultas y la extracción de los datos se lleven a cabo de manera eficaz, y a la mejora del rendimiento de la aplicación.

Para más información, consulte [Patrón Materialized View](/azure/architecture/patterns/materialized-view).

## <a name="next-steps"></a>Pasos siguientes

* Aprenda a [solucionar errores de TooManyRequests](troubleshoot-request-rate-too-large.md) en Azure Cosmos DB.
* Aprenda a [solucionar errores de Timeout](troubleshoot-request-timeout.md) en Azure Cosmos DB.
* Aprenda a [solucionar errores de ServiceUnavailable](troubleshoot-service-unavailable.md) en Azure Cosmos DB.
* Obtenga información sobre [Unidades de solicitud](request-units.md) en Azure Cosmos DB.
* Obtenga más información sobre la [creación de particiones y el escalado horizontal](partitioning-overview.md) en Azure Cosmos DB.
* Obtenga información sobre las [directivas de indexación](index-policy.md) de Azure Cosmos DB.
* Obtenga información sobre la [escalabilidad](provision-throughput-autoscale.md) en Azure Cosmos DB.
