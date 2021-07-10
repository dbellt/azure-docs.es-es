---
title: 'Mensajería de Azure Service Bus: características avanzadas'
description: En este artículo se proporciona información general de alto nivel de las características avanzadas de Azure Service Bus.
ms.topic: overview
ms.date: 06/11/2021
ms.openlocfilehash: 80fab956c182b6f934a767f2dffc09ca2d0c0214
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112034914"
---
# <a name="azure-service-bus---advanced-features"></a>Azure Service Bus: características avanzadas
Service Bus incluye características avanzadas que le permiten resolver problemas de mensajería más complejos. En este artículo se describen varias de estas características.

## <a name="message-sessions"></a>Sesiones de mensajes
Para crear una garantía primero en entrar/primero en salir (FIFO) en Service Bus, use sesiones. Las sesiones de mensajes permiten la administración ordenada y exclusiva de secuencias de mensajes relacionados no enlazadas. Para permitir la administración de sesiones en sistemas de alta disponibilidad y a gran escala, la característica de sesión también permite almacenar el estado de la sesión, lo que hace que las sesiones se muevan de forma segura entre controladores. Para más información, consulte [Sesiones de mensajes: primera en entrar, primero en salir (FIFO)](message-sessions.md).

## <a name="autoforwarding"></a>Artículo reenvío automático
La característica de reenvío automático encadena una cola o suscripción a otra cola o tema dentro del mismo espacio de nombres. Al usar esta característica, Service Bus mueve automáticamente los mensajes de una cola o suscripción a una cola o tema de destino. Todos estos movimientos se realizan de manera transaccional. Para más información, consulte [Encadenamiento de Service Bus entidades con el reenvío](service-bus-auto-forwarding.md).

## <a name="dead-letter-queue"></a>Cola de mensajes fallidos
Todas las suscripciones a temas y colas de Service Bus llevan asociadas colas de mensajes con problemas de entrega (DLQ). Esta cola contiene mensajes que cumplen los criterios siguientes: 

- No se pueden entregar correctamente a ningún receptor.
- Se les ha agotado el tiempo de espera.
- La aplicación receptora los deja a un lado explícitamente. 

Los mensajes de la cola de mensajes con problemas de entrega se anotan con el motivo de que se colocaran allí. La cola de mensajes con problemas de entregas tiene un punto de conexión especial, pero, por lo demás, funciona como cualquier cola normal. Una aplicación o herramienta puede examinar una cola de mensajes con problemas de entrega o quitarlos de esta. También puede reenviar automáticamente una cola de mensajes con problemas de entrega. Para más información, consulte [Introducción a las colas de mensajes fallidos de Service Bus](service-bus-dead-letter-queues.md).

## <a name="scheduled-delivery"></a>Entrega programada
Puede enviar mensajes a una cola o tema para procesarlos más tarde y establecer una hora a la que el mensaje estará disponible para su consumo. Los mensajes programados también se pueden cancelar. Para más información, consulte [Mensajes programados](message-sequencing.md#scheduled-messages).

## <a name="message-deferral"></a>Aplazamiento de mensajes
El cliente de una cola o suscripción puede aplazar la recuperación de un mensaje recibido para otro momento. Es posible que el mensaje se haya publicado fuera del orden esperado y que el cliente desee esperar hasta recibir otro mensaje. Los mensajes aplazados permanecen en la cola o suscripción y se deben reactivar explícitamente mediante el número de secuencia asignado por el servicio. Para más información, consulte [ Aplazamiento de mensajes](message-deferral.md).

## <a name="batching"></a>Lotes
El procesamiento por lotes del lado cliente permite que el cliente de una cola o tema acumule un conjunto de mensajes y los transfiera juntos. Este sistema se suele emplear para ahorrar ancho de banda o para aumentar el rendimiento. Para más información, consulte [Procesamiento por lotes del lado cliente](service-bus-performance-improvements.md#client-side-batching).

## <a name="transactions"></a>Transacciones
Una transacción agrupa dos o más operaciones en un ámbito de ejecución. Service Bus permite agrupar las operaciones en varias entidades de mensajería dentro del ámbito de una única transacción. Una entidad de mensaje puede ser una cola, un tema o una suscripción. Para más información, consulte [Información general sobre el procesamiento de transacciones de Service Bus](service-bus-transactions.md).

## <a name="autodelete-on-idle"></a>Eliminación automática en estado inactivo
La eliminación automática en modo inactivo le permite especificar un intervalo de inactividad después del cual se elimina automáticamente la suscripción a una cola o tema. La duración mínima es de 5 minutos. 

## <a name="duplicate-detection"></a>Detección de duplicados
La característica de detección de duplicados permite al remitente volver a enviar el mismo mensaje y al agente quitar un posible duplicado. Para más información, consulte [Detección de duplicados](duplicate-detection.md).

## <a name="geo-disaster-recovery"></a>Recuperación ante desastres geográfica
Cuando una región de Azure experimenta tiempo de inactividad, la característica de recuperación ante desastres permite que el procesamiento de mensajes siga funcionando en otra región o centro de datos. La característica mantiene un reflejo estructural de un espacio de nombres disponible en la región secundaria y permite que la identidad del espacio de nombres cambie al espacio de nombres secundario. Los mensajes que ya se han publicado permanecen en el espacio de nombres principal anterior a la espera de recuperarse una vez que mejora el incidente de disponibilidad. Para obtener más información, consulte [Recuperación ante desastres con localización geográfica de Azure Service Bus](service-bus-geo-dr.md).

## <a name="security"></a>Seguridad
Service Bus admite los protocolos estándar [AMQP 1,0](service-bus-amqp-overview.md) y [HTTP o REST](/rest/api/servicebus/) y sus respectivas utilidades de seguridad, incluida la seguridad de nivel de transporte (TLS). Los clientes pueden tener acceso autorizado mediante [Firma de acceso compartido](service-bus-sas.md) o la seguridad basada en roles de [Azure Active Directory](service-bus-authentication-and-authorization.md). 

Para proteger contra el tráfico no deseado, Service Bus proporciona [características de seguridad](network-security.md), como el firewall de IP y la integración con redes virtuales. 

## <a name="next-steps"></a>Pasos siguientes
Consulte [Ejemplos de mensajería de Service Bus](service-bus-samples.md) que muestran cómo usar estas características de Service Bus.
