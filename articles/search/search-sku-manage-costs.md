---
title: Cálculo de costos
titleSuffix: Azure Cognitive Search
description: Obtenga información acerca de los eventos que se facturan y del modelo de precios, así como sugerencias para administrar el costo de ejecutar un servicio Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2021
ms.openlocfilehash: e2ab7a007a92bd2306a7a97781af5653f932d0b7
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106580607"
---
# <a name="how-to-estimate-and-manage-costs-of-an-azure-cognitive-search-service"></a>Procedimientos para el cálculo y administración de los costos de un servicio Azure Cognitive Search

En este artículo obtendrá información sobre el modelo de precios y los eventos que se facturan, así como sugerencias para administrar el costo de ejecutar un servicio Azure Cognitive Search.

## <a name="pricing-model"></a>Modelo de precios

La arquitectura de escalabilidad de Azure Cognitive Search se basa en combinaciones flexibles de réplicas y particiones que permiten variar la capacidad en función de si se necesita una mayor potencia de indexación o consulta, y pagar solo por lo que se necesite.

Para determinar el costo del servicio hay que multiplicar la cantidad de recursos que utiliza el servicio de búsqueda por la tarifa de facturación que establece por el nivel de servicio. Los costos y la capacidad están estrechamente enlazados. En el momento de calcular costos, conocer la capacidad necesaria para ejecutar las cargas de trabajo de indexación y consulta permite hacerse una idea muy exacta de cuáles serán los costos previstos.

A efectos de facturación, hay dos fórmulas simples que se deben tener en cuenta:

| Fórmula | Descripción |
|---------|-------------|
| **R x P = SU** | Para saber cuántas *unidades de búsqueda* (SU) usa un servicio, hay que multiplicar el número de réplicas usadas por el número de particiones usadas. Una unidad de búsqueda es una unidad de recurso y puede ser una partición o una réplica. |
| **SU * tarifa de facturación = gasto mensual** | El elemento que tiene más peso al determinar la factura mensual general es el número de unidades de búsqueda multiplicado por la tarifa de facturación del nivel en el que se aprovisionó el servicio. Algunas características o cargas de trabajo tienen dependencias en otros servicios de Azure, lo que puede aumentar el costo de la solución en el nivel de suscripción. La sección de eventos facturables que encontrará a continuación identifica las características que se pueden agregar a una factura. |

Cada servicio se inicia como mínimo con una SU (una réplica multiplicada por una partición). El máximo para cualquier servicio es 36 unidades de búsqueda. Este valor máximo se puede conseguir de varias maneras: por ejemplo, 6 particiones por 6 réplicas, o 3 particiones por 12 réplicas. Es habitual usar menos capacidad total (por ejemplo, un servicio de 3 réplicas y 3 particiones facturado como 9 SU). Vea el gráfico [Combinaciones de particiones y réplicas](search-capacity-planning.md#chart) para ver las combinaciones válidas.

La tarifa de facturación es de una hora por unidad. Cada nivel tiene una tarifa mayor de forma progresiva. Los niveles superiores incluyen particiones mayores y más rápidas, y esto contribuye a un total superior para ese nivel. Puede ver las tarifas de cada nivel en la página [detalles de precios](https://azure.microsoft.com/pricing/details/search/).

La mayoría de los clientes solo ponen en línea una parte de la capacidad total, manteniendo el resto en la reserva. Para la facturación, el número de particiones y réplicas que tiene en línea, calculado mediante la fórmula de SU, determina lo que se paga por hora. 

## <a name="billable-events"></a>Eventos facturables

Una solución basada en Azure Cognitive Search puede incurrir en costos de las siguientes maneras:

+ [Costo del propio servicio](#service-costs), que se ejecuta de forma ininterrumpida con una configuración mínima (una partición y una réplica) a la tasa base. Puede considerar esto como el costo fijo de ejecutar el servicio.

+ La adición de capacidad (réplicas o particiones), donde los costos aumentan en incrementos de la tarifa facturable. Si la empresa requiere alta disponibilidad, se necesitarán tres réplicas.

+ Cargos de ancho de banda (transferencia de datos salientes)

+ Servicios complementarios necesarios para funcionalidades o características específicas:

  + Enriquecimiento con inteligencia artificial (requiere [Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/))
  + almacén de conocimiento (requiere [Azure Storage](https://azure.microsoft.com/pricing/details/storage/))
  + enriquecimiento incremental (requiere [Azure Storage](https://azure.microsoft.com/pricing/details/storage/), se aplica al enriquecimiento con inteligencia artificial)
  + claves administradas por el cliente y cifrado doble (requiere [Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/))
  + puntos de conexión privados para un modelo de acceso sin Internet (requiere [Azure Private Link](https://azure.microsoft.com/pricing/details/private-link/))

### <a name="service-costs"></a>Costos de servicio

A diferencia de las máquinas virtuales u otros recursos que se pueden "pausar" para evitar cargos, un servicio de Azure Cognitive Search siempre está disponible en el hardware dedicado para su uso exclusivo. Así, la creación de un servicio es un evento facturable que se inicia cuando se crea el servicio y termina cuando se elimina este. 

El cargo mínimo es la primera unidad de búsqueda (una réplica x una partición) a la tasa facturable. Este valor mínimo es fijo durante la vigencia del servicio porque el servicio no se puede ejecutar en ninguna otra configuración menor que esta. 

Más allá del mínimo, puede agregar réplicas y particiones por separado. Los aumentos incrementales de la capacidad mediante réplicas y particiones aumentan la factura en función de la siguiente fórmula: **(réplicas x particiones x tarifa de facturación)** , donde la tarifa que se le cobra depende del plan que seleccione.

Al calcular el coste de una solución de búsqueda, tenga en cuenta que los precios y la capacidad no son lineales (el doble de capacidad supone más del doble del coste en el mismo nivel). Además, en un momento dado, cambiar a un nivel superior puede ofrecerle un mejor rendimiento y mayor rapidez aproximadamente al mismo precio. Puede obtener más información y consultar un ejemplo en [Actualización a un nivel estándar S2](search-performance-tips.md#tip-upgrade-to-a-standard-s2-tier).

### <a name="bandwidth-charges"></a>Cargos de ancho de banda

El uso de [indexadores](search-indexer-overview.md) puede afectar a la facturación si el origen de datos de Azure está en una región diferente de Azure Cognitive Search. En este escenario, mover datos salientes desde el origen de datos de Azure a Azure Cognitive Search puede implicar un costo. Para más información, consulte las páginas de precios de la plataforma de datos de Azure en cuestión.

Puede eliminar por completo los cargos de salida de datos si crea el servicio Azure Cognitive Search en la misma región que los datos. Para saber más, eche un vistazo a la [página de precios de ancho de banda](https://azure.microsoft.com/pricing/details/bandwidth/):

+ Datos de entrada: Microsoft no cobra por los datos de entrada a ningún servicio de Azure. 

+ Datos de salida:  los datos de salida son los resultados de la consulta. Cognitive Search no cobra los datos de salida, pero Azure puede cobrarlos si los servicios se encuentran en regiones diferentes.

  Estos cargos realmente no forman parte de la factura de Azure Cognitive Search. Se mencionan aquí porque si los resultados se envían a otras regiones o aplicaciones que no son de Azure, podría ver los costos reflejados en la factura global.

### <a name="ai-enrichment-with-cognitive-services"></a>Enriquecimientos de inteligencia artificial con Cognitive Services

Para el [enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md), debe planear [asociar un recurso facturable de Azure Cognitive Services](cognitive-search-attach-cognitive-services.md), en la misma región que Azure Cognitive Search, en el plan de tarifa S0 para el procesamiento de pago por uso. No hay ningún costo fijo por asociar Cognitive Services. Solo se paga por el procesamiento que necesita.

| Operación | Impacto en la facturación |
|-----------|----------------|
| Descifrado de documentos, extracción de texto | Gratuito |
| Descifrado de documentos, extracción de imágenes | Se factura según el número de imágenes extraídas de los documentos. En una [configuración de indexador](/rest/api/searchservice/create-indexer#indexer-parameters), **imageAction** es el parámetro que desencadena la extracción de imágenes. Si **imageAction** está establecido en "none" (valor predeterminado), no se le cobrará por la extracción de imágenes. La tarifa por extracción de imágenes está documentada en la página de [detalles de precios](https://azure.microsoft.com/pricing/details/search/) de Azure Cognitive Search.|
| [Aptitudes cognitivas integradas](cognitive-search-predefined-skills.md) | Se facturan a la misma tarifa que si se hubiera realizado la tarea mediante Cognitive Services directamente. |
| Aptitudes personalizadas | Una aptitud personalizada es funcionalidad que proporciona el usuario. El costo de usar una aptitud personalizada depende completamente de si el código personalizado está llamando a otros servicios de uso medido. |

La característica [enriquecimiento incremental (versión preliminar)](cognitive-search-incremental-indexing-conceptual.md) le posibilita proporcionar una memoria caché que permite que el indexador sea más eficaz al ejecutar solo las aptitudes cognitivas necesarias si modifica el conjunto de aptitudes en el futuro, lo que le ahorra tiempo y dinero.

## <a name="tips-for-managing-costs"></a>Sugerencias para administrar costos

Las siguientes instrucciones pueden ayudarle a reducir costos, o al menos a administrarlos de forma más eficaz:

+ Cree todos los recursos en la misma región, o en el menor número de regiones posible, para minimizar o eliminar los cargos de ancho de banda.

+ Consolide todos los servicios en un grupo de recursos, como Azure Cognitive Search, Cognitive Services y cualquier otro servicio de Azure que se use en la solución. En Azure Portal, busque el grupo de recursos y use los comandos de **Administración de costos** para obtener información sobre el gasto real y previsto.

+ Considere Azure Web App como aplicación de front-end para que las solicitudes y las respuestas permanezcan dentro del límite del centro de datos.

+ Escale verticalmente las operaciones que consumen muchos recursos, como la indización, y vuelva a ajustar hacia abajo para las cargas de trabajo de consulta normales. Comience con la configuración mínima de Azure Cognitive Search (una SU compuesta de una partición y una réplica) y luego supervise la actividad de los usuarios para identificar patrones de uso que indiquen la necesidad de más capacidad. Si hay un patrón predecible, es posible que pueda sincronizar la escala con la actividad (necesitaría escribir código para automatizar esto).

+ Cost Management está integrado en la infraestructura de Azure. En el artículo sobre [Facturación y administración de costos](../cost-management-billing/cost-management-billing-overview.md) encontrará más información sobre el seguimiento de costos, así como sobre las herramientas y API disponibles para tal fin.

No es posible quitar un servicio de búsqueda de forma temporal. Los recursos dedicados siempre están operativos y se asignan para su uso exclusivo mientras el servicio esté vigente. La eliminación de un servicio es permanente y también elimina los datos asociados.

En lo que respecta al propio servicio, la única manera de reducir la factura es reducir las réplicas y las particiones a un nivel que aún proporcione rendimiento aceptable y [cumplimiento del SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/), o crear un servicio en un nivel inferior (las tarifas por hora de S1 son inferiores a las de S2 o S3). Supongamos que aprovisiona su servicio con la menor de las previsiones de carga; si se supera el servicio, puede crear un segundo servicio con un nivel mayor, regenerar los índices en el segundo servicio y eliminar el primero.

## <a name="next-steps"></a>Pasos siguientes

Aprenda a supervisar y administrar los costos de su suscripción de Azure.

> [!div class="nextstepaction"]
> [Documentación de Azure Cost Management y facturación](../cost-management-billing/cost-management-billing-overview.md)