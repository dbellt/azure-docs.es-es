---
title: 'Configuración de la persistencia de datos: versión Premium de Azure Cache for Redis'
description: Aprenda a configurar y administrar la persistencia de datos en las instancias del nivel Prémium de Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 02/08/2021
ms.openlocfilehash: d115495f56a9e64672682a92d5837db48dbf052d
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110099818"
---
# <a name="configure-data-persistence-for-a-premium-azure-cache-for-redis-instance"></a>Configuración de la persistencia de datos en la instancia prémium de Azure Cache for Redis

La [persistencia de Redis](https://redis.io/topics/persistence) permite conservar los datos almacenados en Redis. También puede tomar instantáneas y hacer una copia de seguridad de los datos. Si se produce un error de hardware, cargue los datos. La capacidad de conservar los datos es una gran ventaja con respecto a los niveles Básico o Estándar, donde todos los datos se almacenan en memoria. La pérdida de datos es posible si se produce un error en el que los nodos de la caché están fuera de servicio.

Azure Cache for Redis ofrece persistencia de Redis mediante la base de datos de Redis (RDB) y el archivo de solo anexión (AOF):

* **Persistencia de RDB**: cuando se utiliza la persistencia de RDB, Azure Cache for Redis conserva una instantánea de Azure Cache for Redis en un disco Redis en formato binario. La instantánea se guarda en una cuenta de Azure Storage. La frecuencia de copia de seguridad configurable determina la frecuencia con la que se conserva la instantánea. Si se produce un evento catastrófico que deshabilita tanto la caché de réplica como la principal, se reconstruye la memoria caché con la instantánea más reciente. Más información sobre las [ventajas](https://redis.io/topics/persistence#rdb-advantages) y [desventajas](https://redis.io/topics/persistence#rdb-disadvantages) de la persistencia de RDB.
* **Persistencia de AOF**: cuando se usa la persistencia de AOF, Azure Cache for Redis guarda todas las operaciones de escritura en un registro. El registro se guarda al menos una vez por segundo en una cuenta de Azure Storage. Si se produce un evento catastrófico que deshabilita tanto la caché de réplica como la principal, se reconstruye la memoria caché con las operaciones de escritura almacenadas. Más información sobre las [ventajas](https://redis.io/topics/persistence#aof-advantages) y [desventajas](https://redis.io/topics/persistence#aof-disadvantages) de la persistencia de AOF.

La persistencia escribe los datos de Redis en una cuenta de Azure Storage que posea y administre. Configure el **nuevo Azure Cache for Redis** a la izquierda durante la creación de la memoria caché. Para las cachés prémium existentes, use el **menú Recurso**.

> [!NOTE]
>
> Azure Storage cifra automáticamente los datos cuando se conservan. Puede usar sus propias claves para el cifrado. Para más información, consulte [Claves administradas por el cliente con Azure Key Vault](../storage/common/storage-service-encryption.md).
>
>

## <a name="set-up-data-persistence"></a>Configuración de la persistencia de datos

1. Para crear una instancia de caché prémium, inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione **Crear un recurso**. Puede crear cachés en Azure Portal. También se pueden crear mediante las plantillas de Resource Manager, PowerShell o la CLI de Azure. Para más información sobre cómo crear una instancia de Azure Cache for Redis, consulte [Creación de una caché](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

    :::image type="content" source="media/cache-private-link/1-create-resource.png" alt-text="Crear recurso.":::
  
2. En la página **Nuevo**, seleccione **Base de datos** y, a continuación, seleccione **Azure Cache for Redis**.

    :::image type="content" source="media/cache-private-link/2-select-cache.png" alt-text="Selección de Azure Cache for Redis.":::

3. En la página **Nueva instancia de Redis Cache**, configure las opciones de la nueva caché prémium.
  
   | Parámetro      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Nombre DNS** | Escriba un nombre único global. | El nombre de la memoria caché debe ser una cadena de entre 1 y 63 caracteres, y solo puede contener números, letras o guiones. El nombre debe comenzar y terminar por un número o una letra y no puede contener guiones consecutivos. El *nombre de host* de la instancia de caché será *\<DNS name>.redis.cache.windows.net*. |
   | **Suscripción** | Desplácese hacia abajo y seleccione su suscripción. | La suscripción en la que se creará esta nueva instancia de Azure Cache for Redis. |
   | **Grupos de recursos** | Desplácese hacia abajo y seleccione un grupo de recursos o **Crear nuevo** y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. |
   | **Ubicación** | Desplácese hacia abajo y seleccione una ubicación. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la memoria caché. |
   | **Tipo de caché** | Desplácese hacia abajo y seleccione una caché prémium para configurar las características prémium. Para más información, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/). |  El plan de tarifa determina el tamaño, el rendimiento y las características disponibles para la memoria caché. Para más información, consulte la [introducción a Azure Redis Cache](cache-overview.md). |

4. Seleccione la pestaña **Redes** o elija el botón **Redes** situado en la parte inferior de la página.

5. En la pestaña **Redes**, seleccione el método de conectividad. En el caso de las instancias de caché premium, se conecta de forma pública mediante puntos de conexión de servicio o direcciones IP públicas. Se conecta de forma privada mediante un punto de conexión privado.

6. Seleccione la pestaña **Siguiente: Opciones avanzadas** o seleccione el botón **Siguiente: Opciones avanzadas** en la parte inferior de la página.

7. En la pestaña **Opciones avanzadas** de la instancia de caché prémium, configure el puerto no TLS, la agrupación en clústeres y la persistencia de datos. Para la persistencia de datos, puede elegir la persistencia de **RDB** o **AOF**.

8. Para habilitar la persistencia de RDB, seleccione **RDB** y configure las opciones.
  
   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Frecuencia de copia de seguridad** | Desplácese hacia abajo y seleccione un intervalo de copia de seguridad. Las opciones disponibles son: **15 minutos**, **30 minutos**, **60 minutos**, **6 horas**, **12 horas** y **24 horas**. | Este intervalo empieza la cuenta atrás cuando se completa correctamente la operación de copia de seguridad anterior y se inicia cuando se produce una nueva copia de seguridad. |
   | **Storage Account** | Desplácese hacia abajo y seleccione la cuenta de almacenamiento. | Elija una cuenta de almacenamiento en la misma región y suscripción que la memoria caché; es recomendable que use una cuenta de **Premium Storage**, ya que tiene un mayor rendimiento.  |
   | **Clave de almacenamiento** | Desplácese hacia abajo y seleccione si se usará la **Clave principal** o la **Clave secundaria**. | Si se vuelve a generar la clave de almacenamiento para su cuenta de persistencia, debe volver a configurar la clave en la lista desplegable **Clave de almacenamiento**. |

    La primera copia de seguridad se inicia cuando transcurre el intervalo de frecuencia de copia de seguridad.
  
   > [!NOTE]
   > Cuando se realizan copias de seguridad de los archivos RDB en el almacenamiento, se almacenan en forma de blobs en páginas.
  
9. Para habilitar la persistencia de AOF, seleccione **AOF** y configure las opciones.

   | Configuración      | Valor sugerido  | Descripción |
   | ------------ |  ------- | -------------------------------------------------- |
   | **Primera cuenta de almacenamiento** | Desplácese hacia abajo y seleccione la cuenta de almacenamiento. | Esta cuenta de almacenamiento debe estar en la misma región y suscripción que la memoria caché; es recomendable que use una cuenta de **Premium Storage**, ya que el almacenamiento Premium tiene un mayor rendimiento. |
   | **Primera clave de almacenamiento** | Desplácese hacia abajo y seleccione si se usará la **Clave principal** o la **Clave secundaria**. | Si se vuelve a generar la clave de almacenamiento para su cuenta de persistencia, debe volver a configurar la clave en la lista desplegable **Clave de almacenamiento**. |
   | **Segunda cuenta de almacenamiento** | (Opcional) Desplácese hacia abajo y seleccione la cuenta de almacenamiento secundaria. | Opcionalmente, puede configurar otra cuenta de almacenamiento. Si se configura una segunda cuenta de almacenamiento, las operaciones de escritura en la caché de réplica se realizan en esta segunda cuenta de almacenamiento. |
   | **Segunda clave de almacenamiento** | (Opcional) Desplácese hacia abajo y seleccione si se usará la **Clave principal** o la **Clave secundaria**. | Si se vuelve a generar la clave de almacenamiento para su cuenta de persistencia, debe volver a configurar la clave en la lista desplegable **Clave de almacenamiento**. |

    Cuando se habilita la persistencia de AOF, las operaciones de escritura en la memoria caché se guardan en la cuenta de almacenamiento designada (o las cuentas si ha configurado una segunda cuenta de almacenamiento). Si se produce un error catastrófico que daña la caché principal y de réplica, el registro de AOF almacenado se usa para regenerar la memoria caché.

10. Seleccione el botón **Siguiente: Opciones avanzadas** o elija el botón **Siguiente: Etiquetas** situado en la parte inferior de la página.

11. Opcionalmente, en la pestaña **Etiquetas**, escriba el nombre y el valor si desea clasificar el recurso.

12. Seleccione **Revisar + crear**. Pasará a la pestaña Revisar y crear, donde Azure validará la configuración.

13. Tras aparecer el mensaje verde Validación superada, seleccione **Crear**.

La caché tarda un tiempo en crearse. Puede supervisar el progreso en la página **Información general** de Azure Cache for Redis. Cuando **Estado** se muestra como **En ejecución**, la memoria caché está lista para su uso.

## <a name="persistence-faq"></a>P+F de persistencia

La lista siguiente contiene respuestas a las preguntas frecuentes sobre la persistencia de Azure Cache for Redis.

* [¿Puedo habilitar la persistencia en una memoria caché creada anteriormente?](#can-i-enable-persistence-on-a-previously-created-cache)
* [¿Puedo habilitar la persistencia de AOF y RDB al mismo tiempo?](#can-i-enable-aof-and-rdb-persistence-at-the-same-time)
* [¿Qué modelo de persistencia debería elegir?](#which-persistence-model-should-i-choose)
* [¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?](#what-happens-if-ive-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)
* [¿Puedo usar la misma cuenta de almacenamiento para la persistencia de dos memorias caché diferentes?](#can-i-use-the-same-storage-account-for-persistence-across-two-different-caches)
* [¿Se me cobrará por el almacenamiento que se usa en la persistencia de datos?](#will-i-be-charged-for-the-storage-being-used-in-data-persistence)

### <a name="rdb-persistence"></a>Persistencia de RDB

* [¿Puedo cambiar la frecuencia de copia de seguridad de RDB después de crear la memoria caché?](#can-i-change-the-rdb-backup-frequency-after-i-create-the-cache)
* [¿Por qué hay más de 60 minutos entre copias de seguridad cuando tengo una frecuencia de copia de seguridad de RDB de 60 minutos?](#why-is-there-more-than-60-minutes-between-backups-when-i-have-an-rdb-backup-frequency-of-60-minutes)
* [¿Qué ocurre con las copias de seguridad de RDB antiguas cuando se realiza una nueva copia de seguridad?](#what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made)

### <a name="aof-persistence"></a>Persistencia de AOF

* [¿Cuándo debo usar una segunda cuenta de almacenamiento?](#when-should-i-use-a-second-storage-account)
* [¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)
* [¿Cómo puedo quitar la segunda cuenta de almacenamiento?](#how-can-i-remove-the-second-storage-account)
* [¿Qué es una reescritura y cómo afecta a la memoria caché?](#what-is-a-rewrite-and-how-does-it-affect-my-cache)
* [¿Qué debo esperar al escalar una memoria caché con AOF habilitado?](#what-should-i-expect-when-scaling-a-cache-with-aof-enabled)
* [¿Cómo se organizan los datos de AOF en el almacenamiento?](#how-is-my-aof-data-organized-in-storage)

### <a name="can-i-enable-persistence-on-a-previously-created-cache"></a>¿Puedo habilitar la persistencia en una memoria caché creada anteriormente?

Sí, la persistencia de Redis puede configurarse tanto durante la creación de la memoria caché como en las memorias caché premium existente.

### <a name="can-i-enable-aof-and-rdb-persistence-at-the-same-time"></a>¿Puedo habilitar la persistencia de AOF y RDB al mismo tiempo?

No, puede habilitar RDB o AOF, pero no ambos a la vez.

### <a name="which-persistence-model-should-i-choose"></a>¿Qué modelo de persistencia debería elegir?

La persistencia de AOF guarda cada escritura en un registro, lo que tiene un efecto significativo en el rendimiento. Se ha comparado AOF con la persistencia de RDB, que guarda las copias de seguridad en función del intervalo de copia de seguridad configurado con un efecto mínimo en el rendimiento. Elija la persistencia de AOF si el objetivo principal es minimizar la pérdida de datos y si puede controlar un menor rendimiento de la memoria caché. Elija la persistencia de RDB si quiere mantener un rendimiento óptimo en la memoria caché pero todavía quiere un mecanismo para la recuperación de datos.

* Más información sobre las [ventajas](https://redis.io/topics/persistence#rdb-advantages) y [desventajas](https://redis.io/topics/persistence#rdb-disadvantages) de la persistencia de RDB.
* Más información sobre las [ventajas](https://redis.io/topics/persistence#aof-advantages) y [desventajas](https://redis.io/topics/persistence#aof-disadvantages) de la persistencia de AOF.

Para más información sobre el rendimiento al usar la persistencia de AOF, vea [¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)

### <a name="what-happens-if-ive-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation"></a>¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?

Para la persistencia de RDB y AOF:

* Si ha escalado a un tamaño mayor, no hay ningún efecto.
* Si ha escalado a un tamaño menor y tiene una configuración de [bases de datos](cache-configure.md#databases) personalizada que es mayor que el [límite de bases de datos](cache-configure.md#databases) del nuevo tamaño, los datos de esas bases de datos no se restauran. Para más información, consulte [¿Mi configuración de bases de datos personalizada se ve afectada durante el escalado?](cache-how-to-scale.md#is-my-custom-databases-setting-affected-during-scaling)
* Si ha escalado a un tamaño menor y no hay suficiente espacio en el tamaño más pequeño para contener todos los datos desde la última copia de seguridad, las claves se expulsarán durante el proceso de restauración.  Normalmente, las claves se expulsan mediante la directiva de expulsión [allkeys-lru](https://redis.io/topics/lru-cache).

### <a name="can-i-use-the-same-storage-account-for-persistence-across-two-different-caches"></a>¿Puedo usar la misma cuenta de almacenamiento para la persistencia de dos memorias caché diferentes?

Sí, puede usar la misma cuenta de almacenamiento para la persistencia de dos memorias caché diferentes.

### <a name="can-i-change-the-rdb-backup-frequency-after-i-create-the-cache"></a>¿Puedo cambiar la frecuencia de copia de seguridad de RDB después de crear la memoria caché?

Sí, puede cambiar la frecuencia de copia de seguridad de la persistencia de RDB en la **Persistencia de los datos** a la izquierda. Para ver las instrucciones, consulte Configurar la persistencia de Redis.

### <a name="why-is-there-more-than-60-minutes-between-backups-when-i-have-an-rdb-backup-frequency-of-60-minutes"></a>¿Por qué hay más de 60 minutos entre copias de seguridad cuando tengo una frecuencia de copia de seguridad de RDB de 60 minutos?

El intervalo de frecuencia de copia de seguridad de la persistencia de RDB no se inicia hasta que el proceso de copia de seguridad anterior se ha completado correctamente. Si la frecuencia de copia de seguridad es de 60 minutos y realiza un proceso de copia de seguridad en 15 minutos para completarse, la siguiente copia de seguridad no se iniciará hasta pasados 75 minutos de la hora de inicio de la copia de seguridad anterior.

### <a name="what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made"></a>¿Qué ocurre con las copias de seguridad de RDB antiguas cuando se realiza una nueva copia de seguridad?

Todas las copias de seguridad de persistencia de RDB excepto la más reciente se eliminan automáticamente. Es posible que esta eliminación no se produzca inmediatamente, pero las copias de seguridad anteriores no se guardan de manera indefinida.

### <a name="when-should-i-use-a-second-storage-account"></a>¿Cuándo debo usar una segunda cuenta de almacenamiento?

Use una segunda cuenta de almacenamiento para la persistencia de AOF cuando crea que tiene más operaciones de conjunto de las esperadas en la memoria caché.  La configuración de la cuenta de almacenamiento secundaria ayuda a garantizar que la memoria caché no alcance los límites de ancho de banda de almacenamiento.

### <a name="does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache"></a>¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?

La persistencia de AOF afecta a la productividad aproximadamente entre un 15 y un 20 % cuando la memoria caché está por debajo de la carga máxima (carga de CPU y servidor ambas por debajo del 90 %). No debe haber problemas de latencia cuando la memoria caché está dentro de estos límites. Pero la memoria caché alcanza estos límites antes con AOF habilitado.

### <a name="how-can-i-remove-the-second-storage-account"></a>¿Cómo puedo quitar la segunda cuenta de almacenamiento?

Puede quitar la cuenta de almacenamiento secundaria de persistencia de AOF si establece la segunda cuenta de almacenamiento de modo que sea la misma que la primera cuenta de almacenamiento. Para las memorias caché existentes, a la **Persistencia de los datos** de la izquierda se accede desde el **menú Recursos** de la caché. Para deshabilitar la persistencia de AOF, seleccione **Deshabilitada**.

### <a name="what-is-a-rewrite-and-how-does-it-affect-my-cache"></a>¿Qué es una reescritura y cómo afecta a la memoria caché?

Cuando el archivo AOF es lo suficientemente grande, en la memoria caché se pone en cola automáticamente una reescritura. La reescritura cambia el tamaño del archivo AOF con el conjunto mínimo de operaciones necesario para crear el conjunto de datos actual. Durante las operaciones de reescritura, puede esperar que se alcancen los límites de rendimiento antes, especialmente cuando se trabaja con grandes conjuntos de datos. Las operaciones de reescritura se producen con menos frecuencia a medida que el archivo AOF crece, aunque duran mucho tiempo cuando ocurren.

### <a name="what-should-i-expect-when-scaling-a-cache-with-aof-enabled"></a>¿Qué debo esperar al escalar una memoria caché con AOF habilitado?

Si el archivo AOF en el momento del escalado es considerablemente grande, se espera que la operación de escalado tarda más de lo esperado porque volverá a cargar el archivo una vez finalizado el escalado.

Para más información sobre el escalado, vea [¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?](#what-happens-if-ive-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### <a name="how-is-my-aof-data-organized-in-storage"></a>¿Cómo se organizan los datos de AOF en el almacenamiento?

Los datos almacenados en archivos AOF se dividen en varios blobs en páginas por nodo para aumentar el rendimiento al guardar los datos en el almacenamiento. En la siguiente tabla se muestra cuántos blobs en páginas se usan en cada plan de tarifa:

| Nivel Premium | Datos BLOB |
|--------------|-------|
| P1           | 4 por partición    |
| P2           | 8 por partición    |
| P3           | 16 por partición   |
| P4           | 20 por partición   |

Cuando la agrupación en clústeres está habilitada, cada partición de la memoria caché tiene su propio conjunto de blobs en páginas, como se ha indicado en la tabla anterior. Por ejemplo, una caché P2 con tres particiones distribuye su archivo AOF entre 24 blobs en páginas (ocho blobs por partición, con tres particiones).

Después de una reescritura, hay dos conjuntos de archivos AOF en el almacenamiento. Las reescrituras se producen en segundo plano y se asocian al primer conjunto de archivos. Las operaciones establecidas, enviadas a la caché durante la reescritura, se anexan al segundo conjunto. Una copia de seguridad se almacena temporalmente durante las reescrituras si se produce un error. La copia de seguridad se elimina rápidamente una vez que finaliza una reescritura.

### <a name="will-i-be-charged-for-the-storage-being-used-in-data-persistence"></a>¿Se me cobrará por el almacenamiento que se usa en la persistencia de datos?

Sí, se le cobrará por el almacenamiento usado según el modelo de precios de la cuenta de almacenamiento que se esté utilizando.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-rdb-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-rdb-persistence.png

[redis-cache-aof-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-aof-persistence.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png
