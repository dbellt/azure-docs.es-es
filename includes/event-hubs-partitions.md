---
title: archivo de inclusión
description: archivo de inclusión
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 03/15/2021
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 05a54a5e240f528ca5662b05e37127741ce4722e
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110367103"
---
Event Hubs organiza las secuencias de eventos que se envían a un centro de eventos en una o más particiones. A medida que llegan eventos más recientes, se agregan al final de esta secuencia. 

![Event Hubs](./media/event-hubs-partitions/multiple-partitions.png)

Una partición puede considerarse como un "registro de confirmación". Las particiones almacenan datos de eventos que contienen el cuerpo del evento, un contenedor de propiedades definido por el usuario que describe el evento y metadatos, como su desplazamiento en la partición, su número en la secuencia de la transmisión y la marca de tiempo del servicio en el momento que se aceptó.

![Diagrama que muestra la secuencia de eventos, de más antiguo o más reciente.](./media/event-hubs-partitions/partition.png)

### <a name="advantages-of-using-partitions"></a>Ventajas de usar particiones
Event Hubs está diseñado para ayudar en el procesamiento de grandes volúmenes de eventos y la creación de particiones ayuda a hacerlo de dos maneras:

- Aunque Event Hubs sea un servicio PaaS, hay una realidad física subyacente, y el mantenimiento de un registro que conserva el orden de los eventos requiere que estos eventos se mantengan juntos en el almacenamiento subyacente y sus réplicas, y esto crea un límite máximo de rendimiento para este tipo de registro. La creación de particiones permite que se usen varios registros paralelos para el mismo centro de eventos y, por lo tanto, multiplicar la capacidad de rendimiento de E/S sin procesar disponible.
- Sus propias aplicaciones deben poder mantener el procesamiento del volumen de eventos que se envía a un centro de eventos. Esto puede ser complejo y requiere una capacidad de procesamiento en paralelo importante con escalabilidad horizontal. La capacidad de un único proceso para controlar eventos es limitada, por lo que se necesitan varios procesos. Las particiones son el modo en que la solución alimenta esos procesos y, además, garantiza que cada evento tenga un propietario del proceso claro. 

### <a name="number-of-partitions"></a>Número de particiones
El número de particiones se especifica en el momento de la creación y debe estar comprendido entre 1 y 32 en Event Hubs Estándar. El número de particiones puede llegar hasta 2000 particiones por unidad de capacidad en Event Hubs dedicado. 

Se recomienda elegir al menos tantas particiones como [unidades de procesamiento (TU)](../articles/event-hubs/event-hubs-scalability.md#throughput-units) sostenidas que espera necesitar durante la carga máxima de la aplicación para ese centro de eventos específico. Debe hacer los cálculos con una sola partición que tenga una capacidad de procesamiento de 1 TU (1 MB de entrada, 2 MB de salida). Puede escalar las unidades de procesamiento del espacio de nombres o las unidades de capacidad del clúster independientemente del número de particiones. Un centro de eventos con 32 particiones y un centro de eventos con 1 partición incurren en el mismo costo cuando el espacio de nombres se establece en una capacidad de 1 TU. 

El número de particiones de un centro de eventos de un [clúster de Event Hubs dedicado](../articles/event-hubs/event-hubs-dedicated-overview.md) se puede [aumentar](../articles/event-hubs/dynamically-add-partitions.md) tras su creación, pero la distribución de flujos entre las particiones cambiará cuando se realice como la asignación de claves de partición a cambios de las particiones, por lo que es preciso evitar a toda costa ese tipo de cambios si el orden relativo de los es importante en la aplicación.

Establecer el número de particiones en el valor máximo permitido es tentador, pero siempre debe tener en cuenta que los flujos de eventos deben estar estructurados de manera que se puedan aprovechar las ventajas de tener varias particiones. Si necesita conservar el orden absoluto en todos los eventos o solo en un puñado de subsecuencias, es posible que no pueda aprovechar las ventajas de tener muchas particiones. Además, muchas particiones hacen que el procesamiento sea más complejo. 


### <a name="mapping-of-events-to-partitions"></a>Asignación de eventos a particiones
Puede usar una clave de partición para asignar datos de eventos entrantes a particiones concretas con fines de organización de los datos. La clave de partición es un valor proporcionado por el remitente que se pasa a un centro de eventos. Se procesa a través de una función hash estática que crea la asignación de la partición. Si no especifica una clave de partición cuando se publica un evento, se usa una asignación de tipo round robin.

El publicador de eventos solo conoce su clave de partición, no la partición en la que se publican los eventos. Este desacoplamiento de la clave y la partición evita al remitente la necesidad de conocer demasiado sobre el procesamiento de bajada. Una identidad única por cada dispositivo o usuario es una buena clave de partición, pero otros atributos como la geografía también pueden usarse para agrupar eventos relacionados en una única partición.

La especificación de una clave de partición permite mantener los eventos relacionados en la misma partición y en el orden exacto en el que se enviaron. La clave de partición es una cadena que se deriva del contexto de la aplicación e identifica la interrelación de los eventos. Una secuencia de eventos identificados por una clave de partición es un *flujo*. Una partición es un almacén de registros multiplexado para muchos de estos flujos. 

> [!NOTE]
> Aunque puede enviar eventos directamente a las particiones, no se recomienda, sobre todo si le da una gran importancia a la alta disponibilidad. Esto degrada la disponibilidad de un centro de eventos en el nivel de partición. Para más información, consulte [Disponibilidad y coherencia](../articles/event-hubs/event-hubs-availability-and-consistency.md).

