---
title: Enrutamiento dependiente de los datos
description: Procedimientos a fin de usar la clase ShardMapManager en aplicaciones .NET para el enrutamiento dependiente de los datos, una característica de las bases de datos con particiones en Azure SQL Database
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
ms.openlocfilehash: d393c43d59e2ffd0041c13e02bd8fafe7764a8aa
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694333"
---
# <a name="use-data-dependent-routing-to-route-a-query-to-an-appropriate-database"></a>Uso del enrutamiento dependiente de datos para enrutar una consulta a la base de datos adecuada
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

El **enrutamiento dependiente de los datos** es la posibilidad de utilizar los datos de una consulta para enrutar la solicitud a una base de datos adecuada. El enrutamiento dependiente de los datos es un patrón fundamental cuando se trabaja con bases de datos con particiones. El contexto de solicitud también puede utilizarse para enrutar la solicitud, en especial si la clave de particionamiento no forma parte de la consulta. Cada consulta o transacción específica en una aplicación que usa enrutamiento dependiente de los datos tiene restringido el acceso a una base de datos por solicitud. En las herramientas elásticas de Azure SQL Database, este enrutamiento se efectúa con la clase **ShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager)).

La aplicación no necesita realizar el seguimiento de las distintas cadenas de conexión o ubicaciones de base de datos asociadas con diferentes segmentos de datos en el entorno particionado. Por el contrario, el [Administrador de mapas de particiones](elastic-scale-shard-map-management.md) abre las conexiones a las bases de datos correctas cuando es necesario, en función de los datos del mapa de particiones y del valor de la clave de particionamiento que es el destino de la solicitud de la aplicación. Esta clave normalmente es el valor de *customer_id*, *tenant_id*, *date_key* o algún otro identificador específico que sea un parámetro fundamental de la solicitud de la base de datos.

Para más información, consulte [Scaling Out SQL Server with Data Dependent Routing](/previous-versions/sql/sql-server-2005/administrator/cc966448(v=technet.10)) (Escalado horizontal de SQL Server con enrutamiento dependiente de los datos).

## <a name="download-the-client-library"></a>Descarga de la biblioteca de cliente

Para descargar:

* La versión de Java de la biblioteca, consulte el [repositorio central de Maven](https://search.maven.org/#search%7Cga%7C1%7Celastic-db-tools).
* La versión de .NET de la biblioteca, consulte [NuGet](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/).

## <a name="using-a-shardmapmanager-in-a-data-dependent-routing-application"></a>Uso de ShardMapManager en una aplicación de enrutamiento dependiente de los datos

Las aplicaciones deben crear una instancia de **ShardMapManager** durante la inicialización, mediante la llamada de fábrica **GetSQLShardMapManager** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanagerfactory.getsqlshardmapmanager), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager)). En este ejemplo, se inicializan **ShardMapManager** y un elemento **ShardMap** específico que contiene. En este ejemplo se muestran los métodos GetSqlShardMapManager y GetRangeShardMap ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapmanager.shardmapmanager.getrangeshardmap), [.NET](/previous-versions/azure/dn824173(v=azure.100))).

```Java
ShardMapManager smm = ShardMapManagerFactory.getSqlShardMapManager(connectionString, ShardMapManagerLoadPolicy.Lazy);
RangeShardMap<int> rangeShardMap = smm.getRangeShardMap(Configuration.getRangeShardMapName(), ShardKeyType.Int32);
```

```csharp
ShardMapManager smm = ShardMapManagerFactory.GetSqlShardMapManager(smmConnectionString, ShardMapManagerLoadPolicy.Lazy);
RangeShardMap<int> customerShardMap = smm.GetRangeShardMap<int>("customerMap"); 
```

### <a name="use-lowest-privilege-credentials-possible-for-getting-the-shard-map"></a>Uso de credenciales de menor privilegio posible para obtener el mapa de particiones

Si una aplicación no manipula el propio mapa de particiones, las credenciales utilizadas en el método de fábrica deben tener permisos de solo lectura en la base de datos del **mapa de particiones global**. Estas credenciales normalmente son distintas de las credenciales que se usan para abrir conexiones con el administrador de mapa de particiones. Consulte también [Credenciales usadas para acceder a la biblioteca de cliente de Elastic Database](elastic-scale-manage-credentials.md).

## <a name="call-the-openconnectionforkey-method"></a>Llamada al método OpenConnectionForKey

El método **ShardMap.OpenConnectionForKey** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper.listshardmapper.openconnectionforkey), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey)) devuelve una conexión lista para emitir comandos a la base de datos adecuada según el valor del parámetro **key**. La información de particionamiento la almacena **ShardMapManager** en la caché de la aplicación, de modo que las solicitudes no implican normalmente una búsqueda de base de datos contra la base de datos **Mapa de particiones global**.

```Java
// Syntax:
public Connection openConnectionForKey(Object key, String connectionString, ConnectionOptions options)
```

```csharp
// Syntax:
public SqlConnection OpenConnectionForKey<TKey>(TKey key, string connectionString, ConnectionOptions options)
```

* El parámetro **key** se usa como clave de búsqueda en el mapa de particiones para determinar la base de datos adecuada para la solicitud.
* El elemento **connectionString** se usa para pasar únicamente las credenciales de usuario para la conexión deseada. Ningún nombre de base de datos o de servidor se incluye en esta *connectionString*, dado que el método determina la base de datos y el servidor a través de **ShardMap**.
* El valor de **connectionOptions** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper.connectionoptions), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.connectionoptions)) se debe establecer en **ConnectionOptions.Validate** si se trata de un entorno donde las asignaciones de las particiones pueden cambiar y las filas pueden moverse a otras bases de datos como resultado de operaciones de división o combinación. Esta validación implica una breve consulta al mapa de particiones local en la base de datos de destino (no al mapa de particiones global) antes de que se entregue la conexión a la aplicación.

Si se produce un error de validación de la asignación de particiones local (lo que indica que el caché es incorrecto), Shard Map Manager consulta la asignación de particiones global para obtener el nuevo valor correcto de la consulta, actualizar el caché, y obtener y devolver la conexión de base de datos adecuada.

Utilice **ConnectionOptions.None** solo cuando no se esperen cambios de asignación de particiones mientras una aplicación está en línea. En ese caso, los valores en caché se pueden asumir como correctos siempre y la llamada de validación de ida y vuelta adicional a la base de datos de destino se puede omitir sin problemas. Esto reduce el tráfico de la base de datos. **connectionOptions** también se puede definir a través de un valor en un archivo de configuración para indicar si se esperan o no cambios en el particionamiento durante un período.  

En este ejemplo se utiliza el valor de una clave de entero **CustomerID**, mediante un objeto **ShardMap** denominado **customerShardMap**.  

```Java
int customerId = 12345;
int productId = 4321;
// Looks up the key in the shard map and opens a connection to the shard
try (Connection conn = shardMap.openConnectionForKey(customerId, Configuration.getCredentialsConnectionString())) {
    // Create a simple command that will insert or update the customer information
    PreparedStatement ps = conn.prepareStatement("UPDATE Sales.Customer SET PersonID = ? WHERE CustomerID = ?");

    ps.setInt(1, productId);
    ps.setInt(2, customerId);
    ps.executeUpdate();
} catch (SQLException e) {
    e.printStackTrace();
}
```

```csharp
int customerId = 12345;
int newPersonId = 4321;

// Connect to the shard for that customer ID. No need to call a SqlConnection
// constructor followed by the Open method.
using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate))
{
    // Execute a simple command.
    SqlCommand cmd = conn.CreateCommand();
    cmd.CommandText = @"UPDATE Sales.Customer
                        SET PersonID = @newPersonID WHERE CustomerID = @customerID";

    cmd.Parameters.AddWithValue("@customerID", customerId);cmd.Parameters.AddWithValue("@newPersonID", newPersonId);
    cmd.ExecuteNonQuery();
}  
```

El método **OpenConnectionForKey** devuelve una nueva conexión ya abierta a la base de datos correcta. Las conexiones utilizadas de esta manera seguirán aprovechando completamente las agrupaciones de conexiones.

El método **OpenConnectionForKeyAsync** ([Java](/java/api/com.microsoft.azure.elasticdb.shard.mapper.listshardmapper.openconnectionforkeyasync), [.NET](/dotnet/api/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkeyasync))también está disponible si la aplicación utiliza programación asincrónica.

## <a name="integrating-with-transient-fault-handling"></a>Integración con el control de errores transitorios

Un procedimiento recomendado para el desarrollo de aplicaciones de acceso a datos en la nube es tener la seguridad de que la aplicación es capaz de capturar los errores transitorios y que las operaciones se reintentan varias veces antes de presentarse un error. El control de errores transitorios para aplicaciones en la nube se describe en el artículo sobre el control de errores transitorios ([Java](/java/api/com.microsoft.azure.elasticdb.core.commons.transientfaulthandling), [.NET](/previous-versions/msp-n-p/dn440719(v=pandp.60))).

El control de errores transitorios puede coexistir naturalmente con el patrón de Enrutamiento dependiente de los datos. El requisito clave es volver a intentar la solicitud de acceso a los datos completa, incluido el bloque **using** que obtuvo la conexión de enrutamiento dependiente de los datos. El ejemplo anterior se podría volver a escribir como se indica a continuación.

### <a name="example---data-dependent-routing-with-transient-fault-handling"></a>Ejemplo: enrutamiento dependiente de los datos con control de errores transitorios

```Java
int customerId = 12345;
int productId = 4321;
try {
    SqlDatabaseUtils.getSqlRetryPolicy().executeAction(() -> {
        // Looks up the key in the shard map and opens a connection to the shard
        try (Connection conn = shardMap.openConnectionForKey(customerId, Configuration.getCredentialsConnectionString())) {
            // Create a simple command that will insert or update the customer information
            PreparedStatement ps = conn.prepareStatement("UPDATE Sales.Customer SET PersonID = ? WHERE CustomerID = ?");

            ps.setInt(1, productId);
            ps.setInt(2, customerId);
            ps.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    });
} catch (Exception e) {
    throw new StoreException(e.getMessage(), e);
}
```

```csharp
int customerId = 12345;
int newPersonId = 4321;

Configuration.SqlRetryPolicy.ExecuteAction(() -> {

    // Connect to the shard for a customer ID.
    using (SqlConnection conn = customerShardMap.OpenConnectionForKey(customerId, Configuration.GetCredentialsConnectionString(), ConnectionOptions.Validate))
    {
        // Execute a simple command
        SqlCommand cmd = conn.CreateCommand();

        cmd.CommandText = @"UPDATE Sales.Customer
                            SET PersonID = @newPersonID
                            WHERE CustomerID = @customerID";

        cmd.Parameters.AddWithValue("@customerID", customerId);
        cmd.Parameters.AddWithValue("@newPersonID", newPersonId);
        cmd.ExecuteNonQuery();

        Console.WriteLine("Update completed");
    }
});
```

Los paquetes necesarios para implementar el control de errores transitorio se descargan automáticamente cuando crea la aplicación  de ejemplo de base de datos elástica.

## <a name="transactional-consistency"></a>Coherencia de las transacciones

Las propiedades de las transacciones están garantizadas para todas las operaciones locales respecto a una partición. Por ejemplo, las transacciones enviadas a través del enrutamiento dependiente de los datos se ejecutan dentro del ámbito de la partición de destino de la conexión. En este momento, no se proporcionan funcionalidades para alistar conexiones múltiples en una transacción y, por lo tanto, no hay garantías de transacciones para las operaciones realizadas a través de las particiones.

## <a name="next-steps"></a>Pasos siguientes

Para desasociar o volver a asociar una partición, vea [Uso de la clase RecoveryManager para solucionar problemas de mapas de particiones](elastic-database-recovery-manager.md).

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]