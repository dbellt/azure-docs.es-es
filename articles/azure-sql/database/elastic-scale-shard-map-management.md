---
title: Escalado horizontal de una base de datos
description: Cómo usar ShardMapManager, la biblioteca de cliente de bases de datos elásticas
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/25/2019
ms.openlocfilehash: 39f1286f0bd6075d3d083e74a15f5cd67766c0d8
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708675"
---
# <a name="scale-out-databases-with-the-shard-map-manager"></a>Escalado horizontal de las bases de datos mediante Shard Map Manager
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Para escalar fácilmente bases de datos de Azure SQL Database, utilice Shard Map Manager. Shard Map Manager es una base de datos especial que mantiene la información de asignación global acerca de todas las particiones (bases de datos) de un conjunto de particiones. Los metadatos permiten que una aplicación se conecte a la base de datos correcta en función del valor de la **clave de particionamiento**. Además, cada partición del conjunto contiene asignaciones que realizan el seguimiento de los datos de particiones locales (conocidos como **shardlets**).

![Administración de mapas de particiones.](./media/elastic-scale-shard-map-management/glossary.png)

Comprender cómo se construyen estos mapas es fundamental para la administración de los mapas de particiones. Esto se realiza mediante la clase ShardMapManager ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager)), que se encuentra en la [biblioteca de cliente de Elastic Database](elastic-database-client-library.md) para administrar los mapas de particiones.  

## <a name="shard-maps-and-shard-mappings"></a>Mapas de particiones y asignaciones de particiones

Para cada partición, debe seleccionar el tipo de mapa de particiones que desea crear. La elección depende de la arquitectura de base de datos:

1. Un solo inquilino por base de datos  
2. Varios inquilinos por base de datos (dos tipos):
   1. Asignación de lista
   2. Asignación de intervalo

Para un modelo de un solo inquilino, cree un mapa de particiones de **asignación de lista**. El modelo de inquilino único asigna una base de datos por inquilino. Se trata de un modelo eficaz para desarrolladores de SaaS, puesto que simplifica la administración de mapas de particiones.

![Asignación de lista][1]

El modelo de varios inquilinos asigna varios inquilinos a una base de datos individual (además, puede distribuir grupos de inquilinos entre varias bases de datos). Use este modelo si espera que cada inquilino tenga pequeñas necesidades de datos. En este modelo, asigne un intervalo de inquilinos a una base de datos mediante **asignación de intervalo**.

![Asignación de intervalo][2]

O bien puede implementar un modelo de base de datos de varios inquilinos mediante una *asignación de lista* para asignar varios inquilinos a una base de datos individual. Por ejemplo, DB1 se usa para almacenar información sobre el identificador de inquilino 1 y 5, mientras que DB2 almacena datos sobre el inquilino 7 y 10.

![Varios inquilinos en una sola base de datos][3]

### <a name="supported-types-for-sharding-keys"></a>Tipos admitidos para claves de particionamiento

El escalado elástico admite los siguientes tipos como claves de particionamiento:

| .NET | Java |
| --- | --- |
| integer |integer |
| long |long |
| guid |uuid |
| byte[]  |byte[] |
| datetime | timestamp |
| timespan | duration|
| datetimeoffset |offsetdatetime |

### <a name="list-and-range-shard-maps"></a>Mapas de particiones de lista y de intervalo

Los mapas de particiones se pueden construir mediante **listas de valores individuales de clave de particionamiento**, o por medio de **intervalos de valores de clave de particionamiento**.

### <a name="list-shard-maps"></a>Mapas de particiones de lista

Las **particiones** contienen **shardlets** y la asignación de shardlets a particiones se mantiene mediante un mapa de particiones. Un **mapa de particiones de lista** es una asociación entre los valores de clave individuales que identifican los shardlets y las bases de datos que funcionan como particiones.  **asignaciones de lista** son valores de clave explícitos y diferentes que se pueden asignar a la misma base de datos. Por ejemplo, el valor de clave 1 se asigna a la Base de datos A y los valores de clave 3 y 6 se asignan ambos a la Base de datos B.

| Clave | Ubicación de la partición |
| --- | --- |
| 1 |Database_A |
| 3 |Database_B |
| 4 |Database_C |
| 6 |Database_B |
| ... |... |

### <a name="range-shard-maps"></a>Mapas de particiones de intervalo

En un **mapa de particiones de intervalo**, el intervalo de claves se describe mediante un par **[valor bajo, valor alto)** donde el *valor bajo* es la clave mínima de el intervalo y el *valor alto* es el primer valor superior del intervalo.

Por ejemplo **[0, 100)** incluye todos los números enteros que son mayores o iguales que 0 y menores que 100. Tenga en cuenta que varios intervalos pueden apuntar a la misma base de datos y que se admiten intervalos separados (por ejemplo, tanto [100,200) como [400,600) apuntan a la base de datos C en el ejemplo siguiente).

| Clave | Ubicación de la partición |
| --- | --- |
| [1,50) |Database_A |
| [50,100) |Database_B |
| [100,200) |Database_C |
| [400,600) |Database_C |
| ... |... |

Cada una de las tablas mostradas anteriormente es un ejemplo conceptual de un objeto **ShardMap** . Cada fila es un ejemplo simplificado de un objeto **PointMapping** (para el mapa de particiones de lista) o **RangeMapping** (para el mapa de particiones de intervalo).

## <a name="shard-map-manager"></a>Administrador de mapas de particiones

En biblioteca de cliente, el administrador de mapas de particiones es una colección de mapas de particiones. Los datos administrados por una instancia **ShardMapManager** se mantienen en tres lugares:

1. **Mapa de particiones global (GSM)** : se especifica una base de datos que funcione como repositorio de todos los mapas de particiones y sus asignaciones. Automáticamente se crean procedimientos almacenados y tablas especiales para administrar la información. Suele ser una base de datos pequeña y con menos acceso, pero no debe utilizarse para otras necesidades de la aplicación. Las tablas se encuentran en un esquema especial llamado **__ShardManagement**.
2. **Mapa de particiones local (LSM)** : cada base de datos que especifique para que sea una partición se modificará para que contenga varias tablas pequeñas y procedimientos almacenados especiales que contienen y administran información de mapa de particiones específica de esa partición. Esta información es redundante con la información del GSM, pero permite que la aplicación valide información de mapa de particiones en caché sin colocar ninguna carga en el GSM; la aplicación usa el LSM para determinar si una asignación en caché sigue siendo válida. Las tablas correspondientes al LSM en cada partición se encuentran también en el esquema **__ShardManagement**.
3. **Caché de aplicación**: cada instancia de la aplicación que accede a un objeto **ShardMapManager** mantiene una caché en memoria local de sus asignaciones. Almacena información de enrutamiento que recientemente se ha recuperado.

## <a name="constructing-a-shardmapmanager"></a>Construcción de un ShardMapManager

Un objeto **ShardMapManager** se construye mediante un patrón de fábrica ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory)). El método **ShardMapManagerFactory.GetSqlShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.getsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager)) toma las credenciales (lo que incluye el nombre del servidor y el nombre de la base de datos que contiene el GSM) en forma de una instancia de **ConnectionString** y devuelve una instancia de **ShardMapManager**.  

**Tenga en cuenta lo siguiente:** solo se debe crear una instancia de **ShardMapManager** una vez por dominio de aplicación, dentro del código de inicialización de una aplicación. La creación de instancias adicionales de ShardMapManager en el mismo dominio de aplicación da lugar a un aumento en el uso de memoria y CPU de la aplicación. Un objeto **ShardMapManager** puede contener cualquier número de mapas de particiones. Aunque un solo mapa de particiones puede ser suficiente para muchas aplicaciones, hay veces en que se usan diferentes conjuntos de bases de datos para un esquema diferente o con fines únicos y, en esos casos, puede que sea preferible usar varios mapas de particiones.

En este código, una aplicación intenta abrir una instancia de **ShardMapManager** existente con el método TryGetSqlShardMapManager ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.trygetsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager)). Si aún no existen objetos que representan una instancia de **ShardMapManager** (GSM) global dentro de la base de datos, la biblioteca de cliente los crea mediante el método CreateSqlShardMapManager ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.createsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager)).

```Java
// Try to get a reference to the Shard Map Manager in the shardMapManager database.
// If it doesn't already exist, then create it.
ShardMapManager shardMapManager = null;
boolean shardMapManagerExists = ShardMapManagerFactory.tryGetSqlShardMapManager(shardMapManagerConnectionString,ShardMapManagerLoadPolicy.Lazy, refShardMapManager);
shardMapManager = refShardMapManager.argValue;

if (shardMapManagerExists) {
    ConsoleUtils.writeInfo("Shard Map %s already exists", shardMapManager);
}
else {
    // The Shard Map Manager does not exist, so create it
    shardMapManager = ShardMapManagerFactory.createSqlShardMapManager(shardMapManagerConnectionString);
    ConsoleUtils.writeInfo("Created Shard Map %s", shardMapManager);
}
```

```csharp
// Try to get a reference to the Shard Map Manager via the Shard Map Manager database.  
// If it doesn't already exist, then create it.
ShardMapManager shardMapManager;
bool shardMapManagerExists = ShardMapManagerFactory.TryGetSqlShardMapManager(
                                        connectionString,
                                        ShardMapManagerLoadPolicy.Lazy,
                                        out shardMapManager);

if (shardMapManagerExists)
{
    Console.WriteLine("Shard Map Manager already exists");
}
else
{
    // Create the Shard Map Manager.
    ShardMapManagerFactory.CreateSqlShardMapManager(connectionString);
    Console.WriteLine("Created SqlShardMapManager");

    shardMapManager = ShardMapManagerFactory.GetSqlShardMapManager(
            connectionString,
            ShardMapManagerLoadPolicy.Lazy);

// The connectionString contains server name, database name, and admin credentials for privileges on both the GSM and the shards themselves.
}
```

Para la versión .NET, puede usar PowerShell para crear un nueva instancia de Shard Map Manager. Hay un ejemplo disponible [aquí](https://gallery.technet.microsoft.com/scriptcenter/Azure-SQL-DB-Elastic-731883db).

## <a name="get-a-rangeshardmap-or-listshardmap"></a>Obtención de las clases RangeShardMap o ListShardMap

Después de crear un administrador de asignación de partición, puede obtener una clase RangeShardMap ([.NET](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [Java](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) o ListShardMap ([.NET](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [Java](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) utilizando el método TryGetRangeShardMap ([.NET](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetrangeshardmap), [Java](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetrangeshardmap)), TryGetListShardMap ([.NET](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.trygetlistshardmap), [Java](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.trygetlistshardmap)) o GetShardMap ([.NET](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getshardmap), [Java](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.getshardmap)).

```Java
// Creates a new Range Shard Map with the specified name, or gets the Range Shard Map if it already exists.
static <T> RangeShardMap<T> createOrGetRangeShardMap(ShardMapManager shardMapManager,
            String shardMapName,
            ShardKeyType keyType) {
    // Try to get a reference to the Shard Map.
    ReferenceObjectHelper<RangeShardMap<T>> refRangeShardMap = new ReferenceObjectHelper<>(null);
    boolean isGetSuccess = shardMapManager.tryGetRangeShardMap(shardMapName, keyType, refRangeShardMap);
    RangeShardMap<T> shardMap = refRangeShardMap.argValue;

    if (isGetSuccess && shardMap != null) {
        ConsoleUtils.writeInfo("Shard Map %1$s already exists", shardMap.getName());
    }
    else {
        // The Shard Map does not exist, so create it
        try {
            shardMap = shardMapManager.createRangeShardMap(shardMapName, keyType);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
        ConsoleUtils.writeInfo("Created Shard Map %1$s", shardMap.getName());
    }

    return shardMap;
}
```

```csharp
// Creates a new Range Shard Map with the specified name, or gets the Range Shard Map if it already exists.
public static RangeShardMap<T> CreateOrGetRangeShardMap<T>(ShardMapManager shardMapManager, string shardMapName)
{
    // Try to get a reference to the Shard Map.
    RangeShardMap<T> shardMap;
    bool shardMapExists = shardMapManager.TryGetRangeShardMap(shardMapName, out shardMap);

    if (shardMapExists)
    {
        ConsoleUtils.WriteInfo("Shard Map {0} already exists", shardMap.Name);
    }
    else
    {
        // The Shard Map does not exist, so create it
        shardMap = shardMapManager.CreateRangeShardMap<T>(shardMapName);
        ConsoleUtils.WriteInfo("Created Shard Map {0}", shardMap.Name);
    }

    return shardMap;
}
```

### <a name="shard-map-administration-credentials"></a>Credenciales de administración de mapas de particiones

Las aplicaciones que administran y manipulan mapas de particiones son diferentes de las que usan los mapas de particiones para enrutar conexiones.

Para administrar asignaciones de partición (agregar o cambiar particiones, asignaciones de particiones, etc.), se deben crear instancias de **ShardMapManager** mediante **credenciales que tengan privilegios de lectura y escritura en la base de datos GSM y en cada base de datos que funcione como partición**. Las credenciales deben permitir escrituras en las tablas del GSM y el LSM cuando se especifica o se cambia información de mapa de particiones, así como al crear tablas del LSM en nuevas particiones.  

Consulte [Credenciales usadas para acceder a la biblioteca de cliente de Elastic Database](elastic-scale-manage-credentials.md).

### <a name="only-metadata-affected"></a>Solo los metadatos afectados

Los métodos usados para rellenar o cambiar datos de **ShardMapManager** no modifican los datos de usuario almacenados en las propias particiones. Por ejemplos, métodos como **CreateShard**, **DeleteShard**, **UpdateMapping**, etc. afectan solo a los metadatos del mapa de particiones. No quitan, agregan ni modificar los datos contenidos en las particiones. En su lugar, estos métodos están diseñados para usarse conjuntamente con operaciones independientes que se lleven a cabo para crear o quitar bases de datos reales, o para mover filas de una partición a otra con el fin de reequilibrar un entorno particionado.  (La herramienta de **división y combinación** incluida en las herramientas de bases de datos elásticas usa estas API junto con la coordinación del movimiento de datos real entre particiones). Consulte el artículo sobre [escalado con la herramienta de división y combinación de Elastic Database](elastic-scale-overview-split-and-merge.md).

## <a name="data-dependent-routing"></a>Enrutamiento dependiente de los datos

El administrador de mapas de particiones se usa principalmente en aplicaciones que requieren conexiones de base de datos para realizar las operaciones de datos específicas de la aplicación. Esas conexiones deben estar asociadas con la base de datos correcta. Esto se conoce como **Enrutamiento dependiente de los datos**. En estas aplicaciones, cree instancias de un objeto de administrador de mapas de particiones predeterminado con credenciales que tengan acceso de solo lectura en la base de datos GSM. Las solicitudes individuales de conexiones posteriores proporcionan las credenciales necesarias para conectarse a la base de datos de particiones adecuada.

Tenga en cuenta que estas aplicaciones (que usan **ShardMapManager** abierto con credenciales de solo lectura) no podrán realizar cambios en los mapas ni las asignaciones. Para cubrir esas necesidades, cree aplicaciones específicas de administración o scripts de PowerShell que proporcionen credenciales con más privilegios, como se explicó anteriormente. Consulte [Credenciales usadas para acceder a la biblioteca de cliente de Elastic Database](elastic-scale-manage-credentials.md).

Para obtener más información, vea [Enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md).

## <a name="modifying-a-shard-map"></a>Modificación de un mapa de particiones

Un mapa de particiones puede cambiarse de distintas maneras. Los siguientes métodos modifican los metadatos que describen las particiones y sus asignaciones, pero no modifican físicamente los datos incluidos en las particiones ni crean o eliminan las bases de datos reales.  Algunas de las operaciones en el mapa de particiones que se describen a continuación, deben coordinarse con acciones administrativas que muevan físicamente los datos o que agreguen y quiten bases de datos que actúan como particiones.

Estos métodos funcionan en conjunto como los bloques de creación disponibles para modificar la distribución global de los datos en el entorno de base de datos particionada.  

* Para agregar o quitar particiones, use **CreateShard** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.createshard), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.createshard)) y **DeleteShard** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap.deleteshard), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.deleteshard)) de la clase shardmap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.shardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap)).
  
    El servidor y la base de datos que representa la partición de destino deben existir ya para que estas operaciones se ejecuten. Estos métodos no tienen ningún efecto en las propias bases de datos, solo en los metadatos del mapa de particiones.
* Para crear o quitar puntos o intervalos que se asignan a las particiones, use **CreateRangeMapping** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.createrangemapping), [.NET](/previous-versions/azure/dn841993(v=azure.100))), **DeleteMapping** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.deletemapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) de la clase RangeShardMapping ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) y **CreatePointMapping** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap.createpointmapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)) de la clase ListShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.listshardmap), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.listshardmap-1)).
  
    Pueden asignarse muchos puntos o intervalos distintos a la misma partición. Estos métodos solo afectan a los metadatos, no a los datos que puedan existir ya en particiones. Si es necesario quitar datos de la base de datos para mantener la coherencia con operaciones de **DeleteMapping**, realizará esas operaciones por separado pero conjuntamente con el uso de estos métodos.  
* Para dividir en dos los intervalos existentes o combinar en uno los intervalos adyacentes, use **SplitMapping** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.splitmapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) y **MergeMappings** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.mergemappings), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)).  
  
    Tenga en cuenta que las operaciones de división y combinación **no cambian la partición a la que se asignan los valores de clave**. Una división divide un rango existente en dos partes, pero deja ambas partes como asignadas a la misma partición. Una combinación opera en dos intervalos adyacentes que ya están asignados a la misma partición, fusionándolos en un solo intervalo.  El movimiento de puntos o intervalos propiamente dichos entre particiones debe coordinarse mediante **UpdateMapping** junto con el movimiento de datos real.  Puede utilizar el servicio de **División y combinación** que forma parte de la vista previa de Escalado elástico para coordinar los cambios de mapas de particiones con el movimiento de datos, cuando se requiere movimiento.
* Para volver a asignar (o mover) puntos o intervalos individuales a particiones diferentes, use **UpdateMapping** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.updatemapping), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)).  
  
    Puesto que es posible que necesite mover datos de una partición a otra para ser coherente con las operaciones de **UpdateMapping**, debe realizar dicho movimiento por separado pero conjuntamente con el uso de estos métodos.

* Para conectar y desconectar las asignaciones, use **MarkMappingOffline** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.markmappingoffline), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) and **MarkMappingOnline** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.map.rangeshardmap.markmappingonline), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.rangeshardmap-1)) para controlar el estado de conexión de una asignación.
  
    Solo se permite realizar determinadas operaciones en las asignaciones de particiones cuando una asignación se encuentra en un estado "sin conexión", incluidas **UpdateMapping** y **DeleteMapping**. Cuando una asignación está sin conexión, una solicitud dependiente de datos basada en una clave incluida en esa asignación devuelve un error. Además, cuando un intervalo se queda sin conexión por primera vez, todas las conexiones a la partición afectada se terminan automáticamente para evitar resultados incoherentes o incompletos para las consultas dirigidas a los intervalos que se están cambiando.

Las asignaciones son objetos inmutables en .NET.  Todos los métodos mencionados que cambian las asignaciones también invalidan las referencias a ellos en el código. Para facilitar la ejecución de secuencias de operaciones que cambian el estado de una asignación, todos los métodos que cambian una asignación devuelven una referencia de la asignación nueva, por lo que las operaciones se pueden encadenar. Por ejemplo, para eliminar una asignación existente en shardmap sm que contiene la clave 25, puede ejecutar lo siguiente:

```
    sm.DeleteMapping(sm.MarkMappingOffline(sm.GetMappingForKey(25)));
```

## <a name="adding-a-shard"></a>Incorporación de una partición

Con frecuencia, las aplicaciones necesitan agregar nuevas particiones para controlar los datos que se esperan de nuevas claves o intervalos de claves, en un mapa de particiones que ya existe. Por ejemplo, es posible que una aplicación particionada por identificador de inquilino necesite aprovisionar una nueva partición para un nuevo inquilino o que datos particionados mensualmente necesiten que se aprovisione una nueva partición antes del inicio de cada nuevo mes.

Si el nuevo intervalo de valores de clave ya no forma parte de una asignación existente y no se requiere ningún movimiento de datos, es sencillo agregar la nueva partición y asociar la nueva clave o el nuevo intervalo a dicha partición. Para obtener más información sobre cómo agregar nuevas particiones, consulte [Incorporación de una nueva partición](elastic-scale-add-a-shard.md).

Sin embargo, para escenarios que requieren movimiento de datos, se necesita la herramienta de división y combinación para coordinar el movimiento de datos entre particiones con las actualizaciones necesarias de mapas de particiones. Para obtener más información sobre cómo usar la herramienta de división y combinación, consulte [Información general sobre división y combinación](elastic-scale-overview-split-and-merge.md).

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/elastic-scale-shard-map-management/listmapping.png
[2]: ./media/elastic-scale-shard-map-management/rangemapping.png
[3]: ./media/elastic-scale-shard-map-management/multipleonsingledb.png
