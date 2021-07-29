---
title: Alta disponibilidad en Azure Cache for Redis
description: Más información sobre las características y opciones de alta disponibilidad de Azure Cache for Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.date: 02/08/2021
ms.author: yegu
ms.openlocfilehash: e06ced47829beeab46edbc98ddeb92fa4e959ec0
ms.sourcegitcommit: 6323442dbe8effb3cbfc76ffdd6db417eab0cef7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110614768"
---
# <a name="high-availability-for-azure-cache-for-redis"></a>Alta disponibilidad en Azure Cache for Redis

Azure Cache for Redis tiene alta disponibilidad integrada.  Se usa una arquitectura de alta disponibilidad para asegurarse de que la instancia de Redis administrada funciona incluso cuando las interrupciones afectan a las máquinas virtuales (VM) subyacentes, tanto las interrupciones planeadas como las no planeadas. Ofrece tasas de porcentaje mucho mayores que las que se logran al hospedar Redis en una sola máquina virtual.

Azure Cache for Redis implementa alta disponibilidad usando varias máquinas virtuales, llamadas *nodos*, para una caché. Los nodos se configuran de forma que la replicación de datos y la conmutación por error se realizan de forma coordinada. La alta disponibilidad también ayuda en las operaciones de mantenimiento, como la aplicación de revisiones de software de Redis. Hay varias opciones de alta disponibilidad disponibles en los niveles Estándar, Premium y Enterprise:

| Opción | Descripción | Disponibilidad | Estándar | Premium | Enterprise |
| ------------------- | ------- | ------- | :------: | :---: | :---: |
| [Replicación estándar](#standard-replication)| Configuración replicada de dos nodos en un único centro de recursos con conmutación automática por error | 99,9 % (consulte los [detalles](https://azure.microsoft.com/support/legal/sla/cache/v1_1/)) |✔|✔|-|
| [Redundancia de zona](#zone-redundancy) | Configuración replicada de varios nodos en varias zonas de disponibilidad, con conmutación automática por error | Hasta el 99,99 % (consulte los [detalles](https://azure.microsoft.com/support/legal/sla/cache/v1_1/)) |-|✔|✔|
| [Replicación geográfica](#geo-replication) | Instancias de caché vinculadas en dos regiones, con conmutación por error controlada por el usuario | Hasta el 99,999 % (consulte los [detalles](https://azure.microsoft.com/support/legal/sla/cache/v1_1/)) |-|✔|Vista previa|

## <a name="standard-replication"></a>Replicación estándar

De forma predeterminada, Azure Cache for Redis en el nivel Estándar o Premium se ejecuta en un par de servidores de Redis. Los dos servidores se hospedan en máquinas virtuales dedicadas. Redis de código abierto solo permite que un servidor controle las solicitudes de escritura de datos. Este servidor es el nodo *principal*, mientras que el otro es el de *réplica*. Después de aprovisionar los nodos de servidor, Azure Cache for Redis les asigna roles principales y de réplica. Normalmente, el nodo principal es responsable del mantenimiento de las solicitudes de escritura y de lectura de los clientes de Redis. En una operación de escritura, confirma una nueva clave y una actualización de claves en su memoria interna y responde inmediatamente al cliente. Además, reenvía la operación al nodo de réplica de forma asincrónica.

:::image type="content" source="media/cache-high-availability/replication.png" alt-text="Configuración de la replicación de datos":::

>[!NOTE]
>Normalmente, un cliente de Redis se comunica con el nodo principal en una caché en Redis para controlar todas las solicitudes de lectura y escritura. Algunos clientes de Redis se pueden configurar para que lleven a cabo operaciones de lectura desde el nodo de réplica.
>
>

Si el nodo principal de una caché en Redis no está disponible, el de réplica se promueve para convertirse en el nodo principal automáticamente. Este proceso se denomina *conmutación por error*. El nodo de réplica esperará durante el tiempo suficiente antes de tomar el control, por si el principal se recupera rápidamente. Cuando se produce una conmutación por error, Azure Cache for Redis aprovisiona una nueva máquina virtual y la une a la caché como el nodo de réplica. El nodo de réplica realiza una sincronización completa de datos con el principal para que tenga otra copia de los datos de caché.

Un nodo principal puede retirarse del servicio como parte de una actividad de mantenimiento planeado, como la actualización del sistema operativo o del software de Redis. También puede dejar de funcionar debido a eventos no planeados como errores en el hardware, el software o la red subyacentes. En el artículo [Conmutación por error y aplicación de revisión de Azure Cache for Redis](cache-failover.md) se proporciona una explicación detallada sobre los tipos de conmutaciones por error de Redis. Una instancia de Azure Cache for Redis pasa por muchas conmutaciones por error durante su vigencia. El diseño de la arquitectura de alta disponibilidad realiza estos cambios en una caché de la forma más transparente posible para sus clientes.

Además, Azure Cache for Redis proporciona más nodos de réplica en el nivel Prémium. Una [caché de varias réplicas](cache-how-to-multi-replicas.md) puede configurarse con hasta tres nodos de réplica. Normalmente, la existencia de más réplicas mejora la resistencia porque hay nodos que hacen copia de seguridad del principal. Incluso con más réplicas, una instancia de Azure Cache for Redis se puede ver seriamente afectada por una interrupción a nivel de las zonas de disponibilidad o del centro de datos. Puede aumentar la disponibilidad de la caché usando varias réplicas con [redundancia de zona](#zone-redundancy).

## <a name="zone-redundancy"></a>Redundancia de zona

Azure Cache for Redis admite configuraciones con redundancia de zona en los niveles Premium y Enterprise. Una [caché con redundancia de zona](cache-how-to-zone-redundancy.md) puede colocar sus nodos en diferentes [zonas de disponibilidad de Azure](../availability-zones/az-overview.md) dentro de la misma región. Además, acaba con el problema de que las interrupciones de las zonas de disponibilidad o del centro de datos sean el único punto de error, y aumenta la disponibilidad general de la caché.

### <a name="premium-tier"></a>Nivel Premium

En el diagrama siguiente se muestra la configuración de redundancia de zona para el nivel Premium:

:::image type="content" source="media/cache-high-availability/zone-redundancy.png" alt-text="Configuración de redundancia de zona":::

Azure Cache for Redis distribuye mediante un mecanismo round-robin los nodos de una caché con redundancia de zona por las zonas de disponibilidad que ha seleccionado. También determina qué nodo actuará como el principal inicialmente.

Una caché con redundancia de zona proporciona conmutación automática por error. Cuando el nodo principal actual no está disponible, una de las réplicas tomará el control. Es posible que la aplicación experimente un mayor tiempo de respuesta de la caché si el nuevo nodo principal se encuentra en otra zona de disponibilidad. Las zonas de disponibilidad están separadas geográficamente. Si se cambia de una zona de disponibilidad a otra, se altera la distancia física entre el lugar en el que se hospedan la aplicación y la caché. Este cambio afecta a las latencias de red bidireccionales entre la aplicación y la caché. La latencia adicional debería estar dentro de un intervalo aceptable en la mayoría de las aplicaciones. Se recomienda probar la aplicación para asegurarse de que funciona correctamente con una caché con redundancia de zona.

### <a name="enterprise-tiers"></a>Niveles de Enterprise

Una memoria caché en el nivel Enterprise se ejecuta en un clúster de Redis Enterprise. Siempre se requiere un número impar de nodos de servidor para que haya cuórum. De forma predeterminada, tiene tres nodos, cada uno hospedado en una máquina virtual dedicada. Una memoria caché de Enterprise tiene dos *nodos de datos* del mismo tamaño y un *nodo de cuórum* más pequeño. Una caché de Enterprise Flash tiene tres nodos de datos del mismo tamaño. El clúster de Enterprise divide los datos de Redis en particiones internamente. Cada partición tiene un elemento *principal* y al menos una *réplica*. Cada nodo de datos contiene una o más particiones. El clúster de Enterprise garantiza que el elemento principal y las réplicas de cualquier partición no se coloquen nunca en el mismo nodo de datos. Las particiones replican datos de forma asincrónica desde los elementos principales a sus réplicas correspondientes.

Cuando un nodo de datos deja de estar disponible o se produce una división de red, se realiza una conmutación por error similar a la descrita en [Replicación estándar](#standard-replication). El clúster de Enterprise usa un modelo basado en cuórum para determinar qué nodos disponibles participarán en un nuevo cuórum. También promueve las particiones de réplica dentro de estos nodos en los elementos principales, según sea necesario.

## <a name="geo-replication"></a>Replicación geográfica

La [replicación geográfica](cache-how-to-geo-replication.md) es un mecanismo para vincular dos instancias o más de Azure Cache for Redis, normalmente abarcando dos regiones de Azure.

### <a name="premium-tier"></a>Nivel Premium

>[!NOTE]
>La replicación geográfica en el nivel Premium está diseñada principalmente para la recuperación ante desastres.
>
>

Dos instancias de caché de nivel Premium se pueden conectar a través de la [replicación geográfica](cache-how-to-geo-replication.md) para que pueda realizar copias de seguridad de los datos de la memoria caché en otra región. Cuando se vinculan entre ellas, una instancia se nombra caché vinculada principal y la otra caché vinculada secundaria. Solo la caché principal acepta solicitudes de lectura y escritura. Los datos escritos en la caché principal se replican en la caché secundaria.

Una aplicación accede a la caché a través de puntos de conexión independientes para las cachés principal y secundaria. La aplicación debe enviar todas las solicitudes de escritura a la caché principal cuando se implemente en varias regiones de Azure. Puede leer desde la caché principal o secundaria. En general, desea que las instancias de proceso de la aplicación lean de las cachés más cercanas para reducir la latencia. TLS protege la transferencia de datos entre las instancias de caché principal y secundaria.

La replicación geográfica no proporciona conmutación por error automática debido a problemas de un tiempo de ida y vuelta de red agregado entre regiones si el resto de la aplicación permanece en la región principal. Deberá administrar e iniciar la conmutación por error desvinculando la caché secundaria. La desvinculación la promueve para ser la nueva instancia principal.

### <a name="enterprise-tiers"></a>Niveles de Enterprise

>[!NOTE]
>Opción disponible en versión preliminar.
>
>

Los niveles de Enterprise admiten una forma más avanzada de replicación geográfica. Lo llamamos [replicación geográfica activa](cache-how-to-active-geo-replication.md). Mediante el uso de los tipos de datos replicados sin conflictos, el software de Redis Enterprise admite la escritura en varias instancias de caché y se encarga de la combinación de los cambios y la resolución de conflictos. Se pueden unir dos o más instancias de caché de nivel de Enterprise en diferentes regiones de Azure para formar una caché de replicación geográfica activa. 

Una aplicación que usa dicha caché puede leer y escribir en las instancias de caché distribuidas geográficamente a través de los puntos de conexión correspondientes. La caché debe usar lo que esté más cerca de cada instancia de proceso, lo que ofrece la menor latencia. La aplicación también necesita supervisar las instancias de caché y cambiar a otra región si una de las instancias deja de estar disponible. Para más información sobre cómo funciona la replicación geográfica activa, consulte [Distribución geográfica activa-activa (basada en CRDT)](https://redislabs.com/redis-enterprise/technology/active-active-geo-distribution/).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo configurar las opciones de alta disponibilidad de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)
* [Incorporación de réplicas a Azure Cache for Redis](cache-how-to-multi-replicas.md)
* [Habilitación de la redundancia de zona para Azure Cache for Redis](cache-how-to-zone-redundancy.md)
* [Configuración de la replicación geográfica para Azure Cache for Redis](cache-how-to-geo-replication.md)
