---
title: Uso de Recovery Manager para solucionar problemas del mapa de particiones
description: Uso de la clase RecoveryManager para solucionar problemas con los mapas de particiones
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/03/2019
ms.openlocfilehash: 821d63ae48c2062578f7f75390dd47407bf70f9c
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708864"
---
# <a name="using-the-recoverymanager-class-to-fix-shard-map-problems"></a>Uso de la clase RecoveryManager para solucionar problemas de mapas de particiones
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

La clase [RecoveryManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager) proporciona a las aplicaciones ADO.NET la capacidad de detectar y corregir fácilmente las incoherencias entre el mapa de particiones global y el mapa de particiones local en un entorno de base de datos con particiones.

Los mapas de particiones global y local realizan un seguimiento de la asignación de cada base de datos en un entorno con particiones. En ocasiones, se produce una interrupción entre el GSM y el LSM. En ese caso, utilice la clase RecoveryManager para detectar y reparar la interrupción.

La clase RecoveryManager forma parte de la [biblioteca de cliente de Elastic Database](elastic-database-client-library.md).

![Mapa de particiones][1]

Para definiciones de términos, consulte el [Glosario de herramientas de Elastic Database](elastic-scale-glossary.md). Para comprender cómo se usa **ShardMapManager** para administrar los datos en una solución con particiones, consulte [Administración de mapas de particiones](elastic-scale-shard-map-management.md).

## <a name="why-use-the-recovery-manager"></a>¿Por qué usar Recovery Manager?

En un entorno de base de datos particionada, hay un inquilino por base de datos y muchas bases de datos por servidor. También puede haber muchos servidores en el entorno. Cada base de datos se asigna en el mapa de particiones, de forma que las llamadas se puedan enrutar al servidor y a la base de datos correctos. Se realiza un seguimiento de las bases de datos según una **clave de particionamiento** y se asigna un **intervalo de valores de clave** a cada partición. Por ejemplo, una clave de particionamiento puede representar los nombres de los clientes de "D" a "F". La asignación de todas las particiones (también conocidas como bases de datos) y sus intervalos de asignación se incluye en el **mapa de particiones global (GSM)** . Cada base de datos también contiene un mapa de los intervalos contenidos en la partición, lo que se conoce como **mapa de particiones local (LSM)** . Cuando una aplicación se conecta a una partición, la asignación se almacena en caché con la aplicación para una rápida recuperación. El mapa de particiones local se usa para validar los datos almacenados en caché.

Puede que GSM y LSM no estén sincronizados por los motivos siguientes:

1. La eliminación de una partición cuyo intervalo se considera que ya no está en uso o el cambio de nombre de una partición. La eliminación de una partición da como resultado una **asignación de particiones huérfana**. De igual forma, una base de datos cuyo nombre cambió puede provocar una asignación de particiones huérfanas. En función de cuál sea el objetivo del cambio, puede que tenga que quitar la partición o simplemente actualizar la ubicación de la partición. Para recuperar una base de datos eliminada, consulte el artículo que explica cómo [restaurar una base de datos eliminada](recovery-using-backups.md).
2. Se produce un evento de conmutación por error geográfica. Para continuar, se debe actualizar el nombre del servidor y el nombre de la base de datos del administrador de mapas de particiones en la aplicación y luego actualizar los detalles de la asignación de particiones de todas las particiones de un mapa de particiones. Si hay una conmutación por error geográfica, se debería automatizar esa lógica de recuperación en el flujo de trabajo de conmutación por error. La automatización de las acciones de recuperación permite una capacidad de administración sin contacto para bases de datos habilitadas geográficamente y evita acciones humanas manuales. Para descubrir las opciones de recuperación de una base de datos tras una posible interrupción del centro de datos, consulte los temas sobre la [continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md) y la [recuperación ante desastres](disaster-recovery-guidance.md).
3. Se restaura la partición o la base de datos de ShardMapManager al anterior punto de tiempo. Para obtener información sobre la recuperación a un momento dado mediante copias de seguridad, consulte [este artículo](recovery-using-backups.md).

Para obtener más información sobre las herramientas de Elastic Database de Azure SQL Database, la replicación y restauración geográficas, consulte lo siguiente:

* [Información general: continuidad del negocio en la nube y recuperación ante desastres con SQL Database](business-continuity-high-availability-disaster-recover-hadr-overview.md).
* [Introducción a las herramientas de base de datos elástica](elastic-scale-get-started.md)  
* [Administración de ShardMap](elastic-scale-shard-map-management.md)

## <a name="retrieving-recoverymanager-from-a-shardmapmanager"></a>Recuperación de RecoveryManager desde un ShardMapManager

El primer paso es crear una instancia de RecoveryManager. El [método GetRecoveryManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getrecoverymanager) devuelve Recovery Manager para la instancia de [ShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager) actual. Para resolver las incoherencias en la asignación de particiones, primero debe recuperar RecoveryManager para el mapa de partición particular.

   ```java
    ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnectionString,  
             ShardMapManagerLoadPolicy.Lazy);
             RecoveryManager rm = smm.GetRecoveryManager();
   ```

En este ejemplo, se inicializa RecoveryManager desde ShardMapManager. También se inicializa un ShardMapManager que contiene un ShardMap.

Como este código de aplicación manipula el mapa de particiones propiamente dicho, las credenciales que se usan en el método de fábrica (en el ejemplo anterior, smmConnectionString) deben ser credenciales con permisos de lectura-escritura en la base de datos del mapa de particiones global a la que hace referencia la cadena de conexión. Estas credenciales normalmente son distintas de las credenciales que se usan a fin de abrir conexiones para el enrutamiento dependiente de datos. Para más información, consulte [Uso de credenciales en las bibliotecas de cliente de base de datos elástica](elastic-scale-manage-credentials.md).

## <a name="removing-a-shard-from-the-shardmap-after-a-shard-is-deleted"></a>Supresión de una partición de ShardMap después de eliminar una partición

El [método DetachShard](/previous-versions/azure/dn842083(v=azure.100)) desasocia la partición determinada del mapa de particiones y elimina las asignaciones asociadas a la partición.  

* El parámetro location es la ubicación de la partición, específicamente el nombre del servidor y el nombre de la base de datos de la partición que se va a desasociar.
* El parámetro shardMapName es el nombre del mapa de particiones. Solo es necesario cuando se administran varios mapas de particiones por el mismo administrador de mapas de particiones. Opcional.

> [!IMPORTANT]
> Use esta técnica solo si está seguro de que el intervalo para la asignación actualizada está vacío. Los métodos anteriores no comprueban los datos para el intervalo que se va a mover, por lo que es mejor incluir comprobaciones en el código.

En este ejemplo se eliminan particiones del mapa de particiones.

   ```java
   rm.DetachShard(s.Location, customerMap);
   ```

El mapa de particiones refleja la ubicación de partición en el GSM antes de la eliminación de la partición. Como se eliminó la partición, se supone esto fue intencionado y el rango con clave de particionamiento ya no está en uso. De lo contrario, puede ejecutar la restauración a un momento dado. para recuperar la partición de un momento anterior. (En ese caso, revise la sección siguiente para detectar las incoherencias de partición). Para recuperar, consulte el artículo sobre la [recuperación a un momento dado](recovery-using-backups.md).

Puesto que se supone que la eliminación de la base de datos era intencionada, la acción de limpieza administrativas final consiste en eliminar la entrada a la partición en el administrador de mapas de particiones. Esto impide que la aplicación escriba accidentalmente información en un rango que no se espera.

## <a name="to-detect-mapping-differences"></a>Para detectar las diferencias de asignación

El [método DetectMappingDifferences](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.detectmappingdifferences) selecciona y devuelve uno de los mapas de particiones (local o global) como el origen de datos y reconcilia las asignaciones en ambos mapas de particiones (global y local).

   ```java
   rm.DetectMappingDifferences(location, shardMapName);
   ```

* La *ubicación* especifica el nombre del servidor y el nombre de la base de datos.
* El parámetro *shardMapName* es el nombre del mapa de particiones. Solo es necesario si un mismo administrador de mapas de particiones administra varios mapas de particiones. Opcional.

## <a name="to-resolve-mapping-differences"></a>Para resolver diferencias de asignación

El [método ResolveMappingDifferences](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.resolvemappingdifferences) selecciona uno de los mapas de particiones (local o global) como origen de datos y concilia las asignaciones en ambos mapas de particiones (global y local).

   ```java
   ResolveMappingDifferences (RecoveryToken, MappingDifferenceResolution.KeepShardMapping);
   ```

* El parámetro *RecoveryToken* enumera las diferencias en las asignaciones entre los mapas de particiones global y local para la partición específica.
* La enumeración [MappingDifferenceResolution](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.mappingdifferenceresolution) se usa para indicar el método para resolver la diferencia entre las asignaciones de particiones.
* Se recomienda **MappingDifferenceResolution.KeepShardMapping** cuando el mapa de particiones local contenga la asignación precisa y, por tanto, se deba usar la asignación en la partición. Suele ser el caso de una conmutación por error: la partición ahora reside en un servidor nuevo. Como se debe quitar primero la partición del mapa de particiones global (mediante el método RecoveryManager.DetachShard), ya no existe una asignación en el mapa de particiones global. Por lo tanto, debe usarse el mapa de particiones local para volver a establecer la asignación de particiones.

## <a name="attach-a-shard-to-the-shardmap-after-a-shard-is-restored"></a>Anexión de una partición al mapa de particiones después de restaurar una partición

El [método AttachShard](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.recovery.recoverymanager.attachshard) asocia una partición determinada con el mapa de particiones. Detecta entonces cualquier inconsistencia en el mapa de particiones y actualiza las asignaciones para que coincidan con la partición en el punto de la restauración de las particiones. Se supone que la base de datos se cambia también para reflejar el nombre de la base de datos original (antes de la restauración de la partición), dado que el valor predeterminado de la restauración en un momento dado es una nueva base de datos anexada con la marca de tiempo.

   ```java
   rm.AttachShard(location, shardMapName)
   ```

* El parámetro *location* es el nombre del servidor y el nombre de la base de datos de la partición que se va a asociar.
* El parámetro *shardMapName* es el nombre del mapa de particiones. Solo es necesario cuando se administran varios mapas de particiones por el mismo administrador de mapas de particiones. Opcional.

En este ejemplo se agrega una partición al mapa de particiones que se restauró recientemente desde un momento dado anterior. Puesto que la partición (es decir, la asignación de la partición del mapa de particiones local) se restauró, es potencialmente incoherente con la entrada de partición en el mapa de particiones global. Fuera de este código de ejemplo, la partición se restauró y cambió el nombre al nombre original de la base de datos. Como se restauró, se asume que la asignación del mapa de particiones local es la asignación de confianza.

   ```java
   rm.AttachShard(s.Location, customerMap);
   var gs = rm.DetectMappingDifferences(s.Location);
      foreach (RecoveryToken g in gs)
       {
       rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping);
       }
   ```

## <a name="updating-shard-locations-after-a-geo-failover-restore-of-the-shards"></a>Actualización de las ubicaciones de partición después de una conmutación de error geográfica (restauración) de las particiones

Si se produce una conmutación por error geográfica, a la base de datos secundaria se le da permiso de escritura y se convierte en la nueva base de datos principal. El nombre del servidor y, posiblemente de la base de datos (según la configuración), puede ser diferente de la réplica principal original. Por lo tanto, deben corregirse las entradas de asignación para la partición en el mapa de particiones local y global. De igual forma, si la base de datos se restaura en una ubicación o nombre diferente o a un momento anterior en el tiempo, podría provocar inconsistencias en los mapas de particiones. El administrador de mapas de particiones controla la distribución de las conexiones abiertas a la base de datos correcta. La distribución se basa en los datos del mapa de particiones y en el valor de la clave de particionamiento, que es el destino de la solicitud de la aplicación. Después de una conmutación por error geográfica, esta información debe actualizarse con el nombre preciso del servidor, el nombre de la base de datos y la asignación de particiones de la base de datos recuperada.

## <a name="best-practices"></a>Procedimientos recomendados

La conmutación por error y la recuperación geográficas son operaciones que normalmente controla un administrador de nube de la aplicación al usar a propósito características de continuidad empresarial de Azure SQL Database. La planeación de la continuidad de negocio requiere procesos, procedimientos y medidas para garantizar que las operaciones empresariales puedan continuar sin interrupción. Deben usarse los métodos disponibles como parte de la clase RecoveryManager dentro de este flujo de trabajo para garantizar que los mapas de particiones local y global se actualizan según la acción de recuperación realizada. Existen cinco pasos básicos para asegurar correctamente que los mapas de particiones local y global reflejan la información precisa después de un evento de conmutación por error. El código de la aplicación para ejecutar estos pasos se puede integrar en el flujo de trabajo y en las herramientas existentes.

1. Recupere RecoveryManager de ShardMapManager.
2. Desasocie la partición anterior del mapa de particiones.
3. Asociar la nueva partición al mapa de particiones, incluida la nueva ubicación de la partición.
4. Detecte incoherencias en la asignación entre los mapas de particiones global y local.
5. Resuelva las diferencias entre los mapas de particiones global y local, confiando en el mapa de particiones local.

En este ejemplo se realizan los siguientes pasos:

1. Quita las particiones del mapa de particiones que reflejan las ubicaciones de particiones anteriores al evento de conmutación por error.
2. Asocia particiones al mapa de particiones que refleja las nuevas ubicaciones de las particiones (el parámetro "Configuration.SecondaryServer" es el nuevo nombre del servidor, pero el mismo nombre de base de datos).
3. Recupera los tokens de recuperación mediante la detección de diferencias de asignación entre los mapas de particiones global y local para cada partición.
4. Resuelve las incoherencias al confiar en la asignación del mapa de particiones local de cada partición.

   ```java
   var shards = smm.GetShards();
   foreach (shard s in shards)
   {
     if (s.Location.Server == Configuration.PrimaryServer)
         {
          ShardLocation slNew = new ShardLocation(Configuration.SecondaryServer, s.Location.Database);
          rm.DetachShard(s.Location);
          rm.AttachShard(slNew);
          var gs = rm.DetectMappingDifferences(slNew);
          foreach (RecoveryToken g in gs)
            {
               rm.ResolveMappingDifferences(g, MappingDifferenceResolution.KeepShardMapping);
            }
        }
    }
   ```

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/elastic-database-recovery-manager/recovery-manager.png