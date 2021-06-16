---
title: Contadores de rendimiento para realizar un seguimiento del administrador de mapas de particiones
description: Clase ShardMapManager y contadores de rendimiento de enrutamiento dependiente de los datos
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: seoapril2019, seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 02/07/2019
ms.openlocfilehash: 8dcda09a73b86551ee636bd564477262dad5c916
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706295"
---
# <a name="create-performance-counters-to-track-performance-of-shard-map-manager"></a>Creación de contadores de rendimiento para realizar el seguimiento del rendimiento del administrador de mapa de particiones
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Los contadores de rendimiento se usan para realizar el seguimiento del rendimiento de las operaciones de [enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md). Dichos contadores son accesibles en el Monitor de rendimiento, en la categoría "Elastic Database: Administración de particiones".

Puede capturar el rendimiento de una instancia de [Shard Map Manager](elastic-scale-shard-map-management.md), en especial cuando se utiliza el [enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md). Los contadores se crean con métodos de la clase Microsoft.Azure.SqlDatabase.ElasticScale.Client.  


**En la versión más reciente**: vaya a [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/). Consulte también [Actualización de una aplicación para usar la biblioteca de cliente de base de datos elástica más reciente](elastic-scale-upgrade-client-library.md).

## <a name="prerequisites"></a>Requisitos previos

* Para crear la categoría y los contadores de rendimiento, el usuario debe formar parte del grupo **Administradores** local en el equipo que hospeda la aplicación.  
* Para crear una instancia de contador de rendimiento y actualizar los contadores, el usuario debe ser miembro del grupo **Administradores** o del grupo **Usuarios del monitor de sistema**.

## <a name="create-performance-category-and-counters"></a>Creación de la categoría y los contadores de rendimiento

Para crear los contadores, llame al método CreatePerformanceCategoryAndCounters de la [clase ShardMapManagementFactory](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory). Solo un administrador puede ejecutar el método:

`ShardMapManagerFactory.CreatePerformanceCategoryAndCounters()`

También puede usar [este](https://gallery.technet.microsoft.com/scriptcenter/Elastic-DB-Tools-for-Azure-17e3d283) script de PowerShell para ejecutar el método.
El método crea los siguientes contadores de rendimiento:  

* **Asignaciones en caché**: número de asignaciones almacenadas en caché para el mapa de particiones.
* **Operaciones de DDR/s**: tasa de operaciones de enrutamiento dependientes de los datos para el mapa de particiones. Este contador se actualiza cuando una llamada a [OpenConnectionForKey()](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey) da lugar a una conexión correcta con la partición de destino.
* **Aciertos/s de caché de búsqueda de asignaciones**: tasa de operaciones correctas de búsqueda en la caché de asignaciones en el mapa de particiones.
* **Errores/s de caché de búsqueda de asignaciones**: tasa de operaciones erróneas de búsqueda en la caché de asignaciones en el mapa de particiones.
* **Asignaciones agregadas o actualizadas en la caché/s**: velocidad a la que se agregan o actualizan las asignaciones en la caché en el mapa de particiones.
* **Asignaciones eliminadas de la caché/s**: velocidad a la que se eliminan las asignaciones de la caché en el mapa de particiones.

Los contadores de rendimiento se crean para cada mapa de particiones en caché por proceso.  

## <a name="notes"></a>Notas

Los siguientes eventos desencadenan la creación de los contadores de rendimiento:  

* Inicialización de [ShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager) con [carga diligente](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerloadpolicy), si ShardMapManager no contiene ningún mapa de particiones. Aquí se incluyen los métodos [GetSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager) y [TryGetSqlShardMapManager](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager).
* Búsqueda correcta de un mapa de particiones (mediante [GetShardMap()](/previous-versions/azure/dn824215(v=azure.100)), [GetListShardMap()](/previous-versions/azure/dn824212(v=azure.100)) o [GetRangeShardMap()](/previous-versions/azure/dn824173(v=azure.100))).
* Creación correcta de un mapa de particiones con CreateShardMap().

Los contadores de rendimiento se actualizarán con todas las operaciones de caché realizadas en el mapa de particiones y las asignaciones. La eliminación correcta del mapa de particiones mediante DeleteShardMap() da lugar a la eliminación de la instancia de contadores de rendimiento.  

## <a name="best-practices"></a>Procedimientos recomendados

* La creación de la categoría y los contadores de rendimiento debe realizarse una sola vez antes de la creación del objeto ShardMapManager. Cada ejecución del comando CreatePerformanceCategoryAndCounters() borra los contadores anteriores (se pierden los datos notificados por todas las instancias) y crea otros nuevos.  
* Las instancias de contadores de rendimiento se crean por proceso. Cualquier bloqueo de la aplicación o eliminación de un mapa de particiones de la caché da lugar a la eliminación de las instancias de contadores de rendimiento.  

### <a name="see-also"></a>Consulte también

[Información general de las características de Elastic Database](elastic-scale-introduction.md)  

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->