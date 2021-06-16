---
title: Migración de bases de datos existentes al escalado horizontal
description: Conversión de bases de datos particionadas para usar herramientas para bases de datos elásticas creando un administrador de mapas de particiones
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: 18a78ef71143968327b290548b3307a8bbcfdfd2
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112021598"
---
# <a name="migrate-existing-databases-to-scale-out"></a>Migración de bases de datos existentes al escalado horizontal
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Administre fácilmente sus bases de datos existentes particionadas con escalado horizontal mediante herramientas (como la [biblioteca cliente de base de datos elástica](elastic-database-client-library.md)). Primero convierta un conjunto existente de bases de datos para utilizar [Shard Map Manager](elastic-scale-shard-map-management.md).

## <a name="overview"></a>Información general

Para migrar una base de datos particionada existente:

1. Prepare la [base de datos de Shard Map Manager](elastic-scale-shard-map-management.md).
2. Crear el mapa de particiones.
3. Preparar las particiones individuales.  
4. Agregue asignaciones al mapa de particiones.

Estas técnicas pueden implementarse mediante la [biblioteca cliente de .NET Framework](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/) o los scripts de PowerShell que se encuentran en [Azure SQL Database: scripts de herramientas para bases de datos elásticas](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db). Los ejemplos siguientes usan los scripts de PowerShell.

Para más información sobre ShardMapManager, consulte [Administración de mapas de particiones](elastic-scale-shard-map-management.md). Si desea más información general de las herramientas para bases de datos elásticas, consulte [Información general de las características de las bases de datos elásticas](elastic-scale-introduction.md).

## <a name="prepare-the-shard-map-manager-database"></a>Prepare la base de datos de Shard Map Manager.

Shard Map Manager es una base de datos especial que contiene los datos para administrar bases de datos con escalado horizontal. Puede utilizar una base de datos existente o crear una nueva. Tenga en cuenta que una base de datos que actúa como Shard Map Manager no debe ser la misma base de datos que la de una partición. El script de PowerShell no crea la base de datos por usted.

## <a name="step-1-create-a-shard-map-manager"></a>Paso 1: crear un administrador de mapas de particiones

```powershell
# Create a shard map manager
New-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
#<server_name> and <smm_db_name> are the server name and database name
# for the new or existing database that should be used for storing
# tenant-database mapping information.
```

### <a name="to-retrieve-the-shard-map-manager"></a>Para recuperar el administrador de mapas de particiones

Después de la creación, puede recuperar el administrador de mapas de particiones con este cmdlet. Este paso es necesario cada vez que tiene que usar el objeto ShardMapManager.

```powershell
# Try to get a reference to the Shard Map Manager  
$ShardMapManager = Get-ShardMapManager -UserName '<user_name>' -Password '<password>' -SqlServerName '<server_name>' -SqlDatabaseName '<smm_db_name>'
```

## <a name="step-2-create-the-shard-map"></a>Paso 2: crear el mapa de particiones.

Seleccione el tipo de mapa de particiones que desea crear. La elección depende de la arquitectura de base de datos:

1. Un solo inquilino por base de datos (para ver los términos, consulte el [glosario](elastic-scale-glossary.md)).
2. Varios inquilinos por base de datos (dos tipos):
   1. Asignación de lista
   2. Asignación de intervalo

Para un modelo de un solo inquilino, cree un mapa de particiones de **asignación de lista** . El modelo de inquilino único asigna una base de datos por inquilino. Se trata de un modelo eficaz para desarrolladores de SaaS, pues simplifica la administración.

![Asignación de lista][1]

El modelo de varios inquilinos asigna varios inquilinos a una base de datos individual (además, puede distribuir grupos de inquilinos entre varias bases de datos). Use este modelo si espera que cada inquilino tenga pequeñas necesidades de datos. En este modelo, asigne un intervalo de inquilinos a una base de datos mediante **asignación de intervalo**.

![Asignación de intervalo][2]

O bien puede implementar un modelo de base de datos de varios inquilinos mediante una *asignación de lista* para asignar varios inquilinos a una base de datos individual. Por ejemplo, DB1 se usa para almacenar información sobre el identificador de inquilino 1 y 5, mientras que DB2 almacena datos sobre el inquilino 7 y 10.

![Varios inquilinos en una sola base de datos][3]

**En función de su elección, elija una de estas opciones:**

### <a name="option-1-create-a-shard-map-for-a-list-mapping"></a>Opción 1: crear un mapa de particiones para una asignación de lista

Cree un mapa de particiones con el objeto ShardMapManager.

```powershell
# $ShardMapManager is the shard map manager object
$ShardMap = New-ListShardMap -KeyType $([int]) -ListShardMapName 'ListShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-2-create-a-shard-map-for-a-range-mapping"></a>Opción 2: crear un mapa de particiones para una asignación de intervalo

Para usar este patrón de asignación, los valores del identificador de inquilino deben consistir en intervalos continuos y es aceptable que haya una brecha en los intervalos al omitir el intervalo al crear las bases de datos.

```powershell
# $ShardMapManager is the shard map manager object
# 'RangeShardMap' is the unique identifier for the range shard map.  
$ShardMap = New-RangeShardMap -KeyType $([int]) -RangeShardMapName 'RangeShardMap' -ShardMapManager $ShardMapManager
```

### <a name="option-3-list-mappings-on-an-individual-database"></a>Opción 3: Asignaciones de lista en una base de datos individual

La configuración de este patrón también requiere la creación de un mapa de lista, tal como se muestra en el paso 2, opción 1.

## <a name="step-3-prepare-individual-shards"></a>Paso 3: preparar particiones individuales

Agregue cada partición (base de datos) al administrador de mapas de particiones. Esto prepara a las bases de datos individuales para almacenar información de asignación. Ejecute este método en cada partición.

```powershell
Add-Shard -ShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
# The $ShardMap is the shard map created in step 2.
```

## <a name="step-4-add-mappings"></a>Paso 4: agregar asignaciones

La adición de asignaciones depende del tipo de mapa de particiones creado. Si ha creado un mapa de lista, agregue asignaciones de lista. Si ha creado un mapa de intervalo, agregue asignaciones de intervalo.

### <a name="option-1-map-the-data-for-a-list-mapping"></a>Opción 1: asignar los datos de una asignación de lista

Asigne los datos agregando una asignación de lista para cada inquilino.  

```powershell
# Create the mappings and associate it with the new shards
Add-ListMapping -KeyType $([int]) -ListPoint '<tenant_id>' -ListShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="option-2-map-the-data-for-a-range-mapping"></a>Opción 2: asignar los datos de una asignación de intervalo

Agregue las asignaciones de intervalo para todas las asociaciones de base de datos de intervalo de identificador de inquilino:

```powershell
# Create the mappings and associate it with the new shards
Add-RangeMapping -KeyType $([int]) -RangeHigh '5' -RangeLow '1' -RangeShardMap $ShardMap -SqlServerName '<shard_server_name>' -SqlDatabaseName '<shard_database_name>'
```

### <a name="step-4-option-3-map-the-data-for-multiple-tenants-on-an-individual-database"></a>Paso 4, opción 3: asignar los datos para varios inquilinos en una base de datos individual

Para cada inquilino, ejecute Add-ListMapping (opción 1).

## <a name="checking-the-mappings"></a>Comprobación de las asignaciones

Información sobre las particiones existentes y las asignaciones asociadas a ellas pueden consultarse mediante los comandos siguientes:  

```powershell
# List the shards and mappings
Get-Shards -ShardMap $ShardMap
Get-Mappings -ShardMap $ShardMap
```

## <a name="summary"></a>Resumen

Una vez haya completado la configuración, puede empezar a usar la biblioteca de cliente de Elastic Database. También puede usar las características de [enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md) y [consulta a través de particiones múltiples](elastic-scale-multishard-querying.md).

## <a name="next-steps"></a>Pasos siguientes

Obtenga los scripts de PowerShell de [Azure SQL Database: scripts de herramientas para bases de datos elásticas](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

Las herramientas también se encuentran en GitHub: [Azure/elastic-db-tools](https://github.com/Azure/elastic-db-tools).

Use la herramienta de división y combinación para mover datos a un modelo de varios inquilinos o desde este a un modelo de inquilino único. Consulte el artículo sobre la [herramienta de división y combinación](elastic-scale-configure-deploy-split-and-merge.md).

## <a name="additional-resources"></a>Recursos adicionales

Para más información sobre los patrones comunes de la arquitectura de datos de aplicaciones de base de datos de software como servicio (SaaS) multiinquilino, consulte [Modelos de diseño para las aplicaciones SaaS multiinquilino y Azure SQL Database](saas-tenancy-app-design-patterns.md).

## <a name="questions-and-feature-requests"></a>Preguntas y solicitudes de características

Si tiene alguna pregunta, utilice la [Página de preguntas y respuestas de Microsoft sobre SQL Database](/answers/topics/azure-sql-database.html). Para efectuar solicitudes de características, agréguelas en el [foro de comentarios sobre SQL Database](https://feedback.azure.com/forums/217321-sql-database/).

<!--Image references-->
[1]: ./media/elastic-convert-to-use-elastic-tools/listmapping.png
[2]: ./media/elastic-convert-to-use-elastic-tools/rangemapping.png
[3]: ./media/elastic-convert-to-use-elastic-tools/multipleonsingledb.png