---
title: Introducción a Event Hubs Premium (versión preliminar)
description: En este artículo se proporciona información general sobre Azure Event Hubs Premium, que ofrece implementaciones multiinquilino de Event Hubs para necesidades de streaming de alto nivel.
ms.topic: article
ms.date: 5/25/2021
ms.openlocfilehash: 5b55816d3e75fcbd8af070f07735b8d8ed081c4f
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388611"
---
# <a name="overview-of-event-hubs-premium-preview"></a>Introducción a Event Hubs Premium (versión preliminar)

El nivel Event Hubs Premium está diseñado para escenarios de streaming de alto nivel que requieren un rendimiento elástico y superior con latencia previsible. El rendimiento se logra al proporcionar recursos reservados de proceso, memoria y almacenamiento, lo que minimiza las interferencias entre inquilinos en un entorno PaaS multiinquilino administrado. 

Event Hubs Premium (versión preliminar) incorpora un nuevo motor de registro de código nativo de dos niveles que proporciona latencias de envío y paso a través mucho más bajas y previsibles que la generación anterior, sin ninguna renuncia de durabilidad. Event Hubs Premium replica cada evento en tres réplicas distribuidas entre zonas de disponibilidad de Azure si están disponibles, y todas las réplicas se vacían de manera sincrónica en el almacenamiento rápido subyacente antes de que se comunique que la operación de envío está completada. Los eventos que no se leen inmediatamente o que deben volver a leerse más adelante se pueden conservar hasta 90 días de forma transparente en una capa de almacenamiento con redundancia de zona de disponibilidad. Los eventos de los niveles de almacenamiento rápido y de retención se cifran; en Event Hubs Premium el usuario puede proporcionar las claves de cifrado. 

Además de estas características relacionadas con el almacenamiento y todas las capacidades y compatibilidad con protocolos de la oferta estándar de Event Hubs, el modelo de aislamiento de Event Hubs Premium habilita nuevas características como el escalado vertical de particiones dinámicas y capacidades futuras aún por agregar. También se obtienen asignaciones de cuota mucho más generosas. Event Hubs Capture está incluido sin costo adicional.

> [!IMPORTANT]
> Event Hubs Premium se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
 
La oferta Premium se factura por [unidades de procesamiento (PU)](event-hubs-scalability.md#processing-units), que corresponden a un recurso compartido de recursos aislados (CPU, memoria y almacenamiento) de la infraestructura subyacente. 

En comparación con la oferta dedicada, dado que Event Hubs Premium proporciona aislamiento dentro de un entorno multiinquilino muy grande que puede desplazar recursos rápidamente, se puede escalar de una forma mucho más elástica y rápida, y las PU se pueden ajustar de manera dinámica. Por lo tanto, Event Hubs Premium normalmente va a ser una opción más rentable para los requisitos de rendimiento de gama media (<120 MB/s), especialmente con cargas variables a lo largo del día o la semana, en comparación con Event Hubs dedicado. 

En el caso de la solidez adicional que se obtiene con la compatibilidad con zonas de disponibilidad, la escala de implementación mínima de Event Hubs dedicado es de 8 unidades de capacidad (CU), pero con Event Hubs Premium se obtiene desde la primera PU en todas las regiones de AZ. 

Puede adquirir 1, 2, 4, 8 y 16 unidades de procesamiento para cada espacio de nombres. Como Event Hubs Premium es una oferta basada en la capacidad, el rendimiento factible no se establece mediante una limitación como en Event Hubs estándar, sino que depende del trabajo que se pida a Event Hubs que realice, de forma similar a Event Hubs dedicado. El rendimiento de ingesta y streaming real por PU depende de varios factores, entre los que se incluyen:

* Número de productores y consumidores
* Tamaño de carga 
* Recuento de particiones
* Velocidad de solicitudes de salida 
* Uso de Event Hubs Capture, registro de esquema y otras características avanzadas

Vea la [comparación entre SKU de Event Hubs](event-hubs-quotas.md) para obtener más detalles.


> [!NOTE]
> Todos los espacios de nombres de Event Hubs están habilitados para el protocolo RPC de Apache Kafka, de modo que las aplicaciones existentes basadas en Kafka puedan usarlos de manera predeterminada. El hecho de tener habilitado Kafka en el clúster no afectará a otros casos de uso que no estén relacionados con Kafka, por lo que no existe la opción ni la necesidad de deshabilitar Kafka en un clúster.

## <a name="why-premium"></a>¿Por qué Premium?

Event Hubs Premium ofrece tres atractivas ventajas para los clientes que requieren un mejor aislamiento en un entorno multiinquilino con necesidades de ingesta de datos de baja latencia y alto rendimiento.

#### <a name="superior-performance-with-the-new-two-tier-storage-engine"></a>Rendimiento superior con el nuevo motor de almacenamiento de dos niveles

Event Hubs Premium usa un nuevo motor de almacenamiento de registros de dos niveles que mejora drásticamente el rendimiento de entrada de datos con una latencia general y una oscilación de latencia considerablemente reducidas sin renunciar a las garantías de durabilidad. 

#### <a name="better-isolation-and-predictability"></a>Mejor aislamiento y previsibilidad

Event Hubs Premium ofrece una capacidad de proceso y memoria aislada para lograr una latencia más previsible y un riesgo de impacto de *vecino ruidoso* mucho menor en una implementación multiinquilino.

Event Hubs Premium implementa un modelo *clúster en clúster* en sus clústeres multiinquilino para proporcionar previsibilidad y rendimiento, a la vez que conserva todas las ventajas de un entorno PaaS multiinquilino administrado. 


#### <a name="cost-savings-and-scalability"></a>Ahorros de costo y escalabilidad
Como Event Hubs Premium es una oferta multiinquilino, puede escalar dinámicamente de forma más flexible y muy rápida. La capacidad se asigna en unidades de procesamiento que asignan pods aislados de CPU/memoria dentro del clúster. El número de esos pods se puede escalar o reducir verticalmente por espacio de nombres. Por lo tanto, Event Hubs Premium es una opción de bajo costo para escenarios de mensajería con el rango de rendimiento general inferior a 120 MB/s, pero superior a lo que se puede lograr con el SKU estándar.  

## <a name="quotas-and-limits"></a>Cuotas y límites
El nivel Premium ofrece todas las características del plan estándar, pero con mejor rendimiento, aislamiento y cuotas más generosas. Para obtener más información sobre cuotas y límites, consulte [Cuotas y límites de Azure Event Hubs](event-hubs-quotas.md).


## <a name="faqs"></a>Preguntas más frecuentes

[!INCLUDE [event-hubs-dedicated-clusters-faq](../../includes/event-hubs-premium-faq.md)]

## <a name="next-steps"></a>Pasos siguientes

Puede empezar a usar Event Hubs Premium (versión preliminar) mediante [Azure Portal](https://aka.ms/eventhubsclusterquickstart). Vea [Precios de Event Hubs Premium](https://azure.microsoft.com/pricing/details/event-hubs/) para obtener más detalles sobre los precios, y [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.yml) para encontrar respuestas a algunas preguntas frecuentes sobre Event Hubs. 

