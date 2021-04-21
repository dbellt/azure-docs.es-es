---
title: 'Límites y limitaciones: Hiperescala (Citus) para Azure Database for PostgreSQL'
description: Límites actuales de los grupos de servidores de Hiperescala (Citus)
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 04/07/2021
ms.openlocfilehash: cfd57c60c4bfd60976eb28822c696412cabda212
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107023906"
---
# <a name="azure-database-for-postgresql--hyperscale-citus-limits-and-limitations"></a>Azure Database for PostgreSQL: límites y limitaciones de Hiperescala (Citus)

En la siguiente sección se describen los límites de capacidad y funcionales en el servicio de Hiperescala (Citus).

## <a name="maximum-connections"></a>Número máximo de conexiones

Cada conexión de PostgreSQL (incluso inactiva) usa al menos 10 MB de memoria, por lo que es importante limitar las conexiones simultáneas. Estos son los límites que elegimos para mantener los nodos en buen estado:

* Nodo de coordinación
   * Número máximo de conexiones: 300
   * Número máximo de conexiones de usuario: 297
* Nodo de trabajo
   * Número máximo de conexiones: 600
   * Número máximo de conexiones de usuario: 597

> [!NOTE]
> En un grupo de servidores con [características en versión preliminar](hyperscale-preview-features.md) habilitadas, los límites de conexión al coordinador son ligeramente diferentes:
>
> * Conexiones máximas de nodo de coordinación
>    * 300 para una cantidad de 0 a 3 núcleos virtuales
>    * 500 para una cantidad de 4 a 15 núcleos virtuales
>    * 1000 para una cantidad superior a 16 núcleos virtuales

Se producirá un error al intentar conectarse más allá de estos límites. El sistema reserva tres conexiones para supervisar los nodos, por lo que hay tres menos conexiones disponibles para las consultas de usuario que el total de conexiones.

### <a name="connection-pooling"></a>Agrupación de conexiones

El establecimiento de las nuevas conexiones lleva su tiempo. Esto funciona con la mayoría de las aplicaciones, que solicitan muchas conexiones de corta duración. Se recomienda usar un agrupador de conexiones para reducir las transacciones inactivas y reutilizar las conexiones existentes. Para más información, visite nuestra [entrada de blog](https://techcommunity.microsoft.com/t5/azure-database-for-postgresql/not-all-postgres-connection-pooling-is-equal/ba-p/825717).

Puede ejecutar su propio concentrador de conexión o usar PgBouncer administrado por Azure.

#### <a name="managed-pgbouncer-preview"></a>PgBouncer administrado (versión preliminar)

> [!IMPORTANT]
> El concentrador de conexión de PgBouncer administrado de Hiperescala (Citus) está actualmente en versión preliminar. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
>
> Puede ver una lista completa de otras características nuevas en [Características en versión preliminar para Hiperescala (Citus)](hyperscale-preview-features.md).

Los concentradores de conexión, como PgBouncer, permiten que más clientes se conecten al nodo de coordinación a la vez. Las aplicaciones se conectan al concentrador, que retransmite comandos a la base de datos de destino.

Cuando los clientes se conectan a través de PgBouncer, el número de conexiones que se pueden ejecutar activamente en la base de datos no cambia. En su lugar, PgBouncer pone en cola el exceso de conexiones y las ejecuta cuando la base de datos está lista.

Hiperescala (Citus) ahora ofrece una instancia administrada de PgBouncer para grupos de servidores (en versión preliminar). Admite hasta 2000 conexiones de cliente simultáneas.
Para conectarse a través de PgBouncer, siga estos pasos:

1. Vaya a la página **Cadenas de conexión** del grupo de servidores en Azure Portal.
2. Active la casilla **Cadenas de conexión de PgBouncer**. (Las cadenas de conexión enumeradas cambiarán).
3. Actualice las aplicaciones cliente para conectarse con la nueva cadena.

## <a name="storage-scaling"></a>Escalado de almacenamiento

El almacenamiento en el coordinador y los nodos de trabajo se pueden escalar verticalmente (aumentar), pero no se puede reducir verticalmente (disminuir).

## <a name="storage-size"></a>Tamaño de almacenamiento

Se admite hasta 2 TiB de almacenamiento en los nodos de coordinador y de trabajo. Consulte las opciones de almacenamiento disponibles y el cálculo de IOPS [anterior](concepts-hyperscale-configuration-options.md#compute-and-storage) para los tamaños de nodo y clúster.

## <a name="database-creation"></a>Creación de base de datos

Azure Portal proporciona credenciales para conectarse exactamente a una base de datos por grupo de servidores de Hiperescala (Citus), la base de datos de `citus`. Actualmente no se permite crear otra base de datos, y el comando CREATE DATABASE producirá un error.

## <a name="columnar-storage"></a>Almacenamiento en columnas

Actualmente, Hiperescala (Citus) tiene estas limitaciones con las [tablas en columnas](concepts-hyperscale-columnar.md):

* La compresión está en disco, no en memoria.
* Solo anexar (sin compatibilidad con UPDATE/DELETE).
* No hay recuperación de espacio (por ejemplo, las transacciones de reversión pueden seguir consumiendo espacio en disco).
* Sin compatibilidad con índices, exámenes de índices o exámenes de índices de mapa de bits.
* Sin tidscans.
* Sin exámenes de ejemplo.
* Sin compatibilidad con TOAST (se admiten valores grandes insertados).
* No se admiten instrucciones ON CONFLICT (excepto las acciones DO NOTHING sin ningún destino especificado).
* No se admiten bloqueos de tupla (SELECT ... FOR SHARE, SELECT ... FOR UPDATE).
* No se admite el nivel de aislamiento serializable.
* Compatibilidad solo con las versiones 12 y posteriores del servidor PostgreSQL.
* No se admiten claves externas, restricciones de unicidad ni restricciones de exclusión.
* No se admite la descodificación lógica.
* No se admiten exámenes paralelos dentro del nodo.
* No se admiten los desencadenadores AFTER ... FOR EACH ROW.
* Sin tablas en columnas UNLOGGED.
* Sin tablas en columnas TEMPORARY.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo [crear un grupo de servidores Hiperescala (Citus) en el portal](quickstart-create-hyperscale-portal.md).
