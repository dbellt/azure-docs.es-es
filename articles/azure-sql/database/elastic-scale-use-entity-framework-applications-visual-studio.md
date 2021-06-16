---
title: Uso de la biblioteca cliente de bases de datos elásticas con Entity Framework
description: Uso de la biblioteca de cliente de Elastic Database y Entity Framework para la codificación de bases de datos
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: sample
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 01/04/2019
ms.openlocfilehash: 5dbbbb639058f55c2a99a054bf9569f9b7d3d0b6
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690134"
---
# <a name="elastic-database-client-library-with-entity-framework"></a>Biblioteca de cliente de Elastic Database con Entity Framework
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Este documento muestra los cambios que es necesario realizar en una aplicación de Entity Framework para su integración con las [herramientas de Elastic Database](elastic-scale-introduction.md). Se centra en la composición de la [administración de mapas de particiones](elastic-scale-shard-map-management.md) y el [enrutamiento dependiente de los datos](elastic-scale-data-dependent-routing.md) con el enfoque **Code First** de Entity Framework. El tutorial [Code First – Nueva base de datos](/ef/ef6/modeling/code-first/workflows/new-database) para EF sirve como ejemplo en ejecución en este documento. El código de ejemplo que acompaña a este documento forma parte del conjunto de ejemplos de las herramientas de bases de datos elásticas en los ejemplos de código de Visual Studio.

## <a name="downloading-and-running-the-sample-code"></a>Descarga y ejecución del código de ejemplo

Para descargar el código de este artículo:

* Se requiere Visual Studio 2012 o posterior.
* Descargue el [ ejemplo Elastic DB Tools for Azure SQL - Entity Framework Integration (Herramientas de Elastic DB para SQL Azure: Integración con Entity Framework)](https://github.com/Azure/elastic-db-tools/). Descomprima el ejemplo en una ubicación de su elección.
* Inicie Visual Studio.
* En Visual Studio, seleccione Archivo -> Abrir proyecto/solución.
* En el cuadro de diálogo **Abrir proyecto**, vaya al ejemplo que descargó y seleccione **EntityFrameworkCodeFirst.sln** para abrir el ejemplo.

Para ejecutar el ejemplo, debe crear tres bases de datos vacías en Azure SQL Database:

* Base de datos de administrador de mapas de particiones
* Base de datos de partición 1
* Base de datos de partición 2

Cuando haya creado estas bases de datos, rellene los marcadores de posición en **Program.cs** con el nombre del servidor, los nombres de base de datos y las credenciales para conectarse a las bases de datos. Compile la solución en Visual Studio. Visual Studio descarga los paquetes de NuGet necesarios para la biblioteca de cliente de bases de datos elásticas, Entity Framework y control de errores transitorios como parte del proceso de compilación. Asegúrese de que la restauración de paquetes de NuGet está habilitada para la solución. Puede habilitar esta configuración haciendo clic con el botón derecho en el archivo de solución en el Explorador de soluciones de Visual Studio.

## <a name="entity-framework-workflows"></a>Flujos de trabajo de Entity Framework

Los desarrolladores de Entity Framework se basan en uno de los cuatro flujos de trabajo siguientes para compilar aplicaciones y garantizar la persistencia de los objetos de la aplicación:

* **Code First (nueva base de datos)** : el desarrollador de EF crea el modelo en el código de aplicación y luego EF genera la base de datos a partir de él.
* **Code First (base de datos existente)** : el desarrollador permite que EF genere el código de aplicación para el modelo de una base de datos existente.
* **Model First**: el desarrollador crea el modelo en el diseñador de EF y luego EF crea la base de datos a partir del modelo.
* **Database First**: el desarrollador usa las herramientas de EF para deducir el modelo a partir de una base de datos existente.

Todos estos métodos se basan en la clase DbContext para administrar de forma transparente las conexiones de base de datos y el esquema de base de datos de una aplicación. Diferentes constructores de la clase base DbContext permiten distintos niveles de control sobre la creación de la conexión, el arranque de base de datos y la creación del esquema. Los problemas surgen principalmente del hecho de que la administración de conexiones de base de datos proporcionada por EF interfiere con la funcionalidad de administración de conexiones de las interfaces de enrutamiento dependientes de datos proporcionadas por la biblioteca de cliente de bases de datos elásticas.

## <a name="elastic-database-tools-assumptions"></a>Suposiciones de herramientas de bases de datos elásticas

Para definiciones de términos, consulte el [Glosario de herramientas de Elastic Database](elastic-scale-glossary.md).

Con la biblioteca de cliente de bases de datos elásticas, se definen particiones de los datos de la aplicación, denominadas shardlets. Los shardlets se identifican mediante una clave de particionamiento y se asignan a bases de datos específicas. Una aplicación puede tener tantas bases de datos como sea necesario y distribuir los shardlets para proporcionar suficiente capacidad o rendimiento en función de los requisitos del negocio actuales. La asignación de valores de clave de particionamiento a las bases de datos se almacena en un mapa de particiones que proporcionan las API de cliente de bases de datos elásticas. A esta capacidad la denominamos **Administración de mapas de particiones** o, para abreviar, SMM. El mapa de particiones también funciona como el agente de conexiones de base de datos para las solicitudes que llevan una clave de particionamiento. Esta funcionalidad se conoce como **enrutamiento dependiente de los datos**.

El Administrador de mapas de particiones protege a los usuarios de vistas incoherentes en datos de shardlet que se pueden producir cuando se producen operaciones de administración de shardlets simultáneas (por ejemplo, la reubicación de datos de una partición a otra). Para ello, los mapas de particiones administrados por la biblioteca de cliente negocian las conexiones de base de datos de una aplicación. Esto permite que la funcionalidad de asignación de mapas elimine automáticamente una conexión de base de datos cuando las operaciones de administración de particiones puedan afectar al shardlet para el que se ha creado la conexión. Este enfoque debe integrarse con parte de la funcionalidad de EF, como la creación de nuevas conexiones a partir de otra existente, para comprobar la existencia de la base de datos. En general, hemos observado que los constructores de DbContext estándar solo funcionan de forma confiable con las conexiones de base de datos cerradas que pueden clonarse para el funcionamiento de EF. En lugar de ello, el principio de diseño de la base de datos elástica solo negocia conexiones abiertas. Se podría pensar que cerrando una conexión negociada por la biblioteca de cliente antes de entregarla a DbContext de EF resolvería este problema. Sin embargo, al cerrar la conexión y depender de EF para volver a abrirla, se renuncia a las comprobaciones de validación y coherencia que realiza la biblioteca. Sin embargo, la funcionalidad de migraciones de EF usa estas conexiones para administrar el esquema de base de datos subyacente de forma transparente a la aplicación. Lo ideal sería conservar y combinar todas estas funcionalidades de la biblioteca de cliente de bases de datos elásticas y EF en la misma aplicación. En la siguiente sección se describen estas propiedades y los requisitos con más detalle.

## <a name="requirements"></a>Requisitos

Cuando se trabaja con la biblioteca de cliente de bases de datos elásticas y las API de Entity Framework, se pretende conservar las propiedades siguientes:

* **Escalado horizontal**: es necesario poder agregar o quitar bases de datos de la capa de datos de la aplicación particionada cuando las demandas de capacidad de la aplicación lo requieran. Esto se traduce en control sobre la creación y eliminación de bases de datos y el uso de las API del administrador de mapas de particiones de bases de datos elásticas para administrar bases de datos y las asignaciones de shardlets.
* **Coherencia**: la aplicación emplea particionamiento y usa la funcionalidad de enrutamiento dependiente de datos de la biblioteca de cliente. Para evitar daños o resultados de la consulta incorrectos, las conexiones se negocian mediante el administrador de mapas de particiones. También mantiene la validación y la coherencia.
* **Code First**: permite conservar la ventaja del paradigma de Code First de EF. En Code First, las clases de la aplicación se asignan de manera transparente a las estructuras de base de datos subyacente. El código de aplicación interactúa con DbSets que enmascaran la mayoría de los aspectos implicados en el procesamiento de la base de datos subyacente.
* **Esquema**: Entity Framework controla la creación del esquema de base de datos inicial y la evolución del esquema posterior a través de migraciones. Al conservar estas capacidades, la adaptación de la aplicación se simplifica a medida que los datos evolucionan.

Las instrucciones siguientes indican cómo satisfacer estos requisitos para las aplicaciones de Code First con las herramientas de bases de datos elásticas.

## <a name="data-dependent-routing-using-ef-dbcontext"></a>Enrutamiento dependiente de datos con DbContext de EF

Las conexiones de base de datos con Entity Framework se suelen administrar mediante subclases de **DbContext**. Cree estas subclases basándose en **DbContext**. Aquí es donde define los **DbSets** que implementan las colecciones de objetos CLR con respaldo de base de datos para la aplicación. En el contexto de enrutamiento dependiente de datos, se pueden identificar varias propiedades útiles que no necesariamente se incluyen en otros escenarios de aplicación EF Code First:

* La base de datos ya existe y se ha registrado en el mapa de particiones de bases de datos elásticas.
* El esquema de la aplicación ya se ha implementado en la base de datos (que se explica más adelante).
* Las conexiones de enrutamiento dependiente de datos con la base de datos se negocian con el mapa de particiones.

Para integrar **DbContexts** con enrutamiento dependiente de datos para escalado horizontal:

1. Cree conexiones de base de datos física a través de las interfaces de cliente de bases de datos elásticas del administrador de mapas de particiones.
2. Ajuste la conexión con la subclase **DbContext** .
3. Transfiera la conexión a las clases base **DbContext** para asegurarse de que también se produce todo el procesamiento del lado EF.

En el ejemplo de código siguiente se muestra este método. (Este código también se encuentra en el proyecto de Visual Studio adjunto).

```csharp
public class ElasticScaleContext<T> : DbContext
{
public DbSet<Blog> Blogs { get; set; }
...

    // C'tor for data-dependent routing. This call opens a validated connection
    // routed to the proper shard by the shard map manager.
    // Note that the base class c'tor call fails for an open connection
    // if migrations need to be done and SQL credentials are used. This is the reason for the
    // separation of c'tors into the data-dependent routing case (this c'tor) and the internal c'tor for new shards.
    public ElasticScaleContext(ShardMap shardMap, T shardingKey, string connectionStr)
        : base(CreateDDRConnection(shardMap, shardingKey, connectionStr),
        true /* contextOwnsConnection */)
    {
    }

    // Only static methods are allowed in calls into base class c'tors.
    private static DbConnection CreateDDRConnection(
    ShardMap shardMap,
    T shardingKey,
    string connectionStr)
    {
        // No initialization
        Database.SetInitializer<ElasticScaleContext<T>>(null);

        // Ask shard map to broker a validated connection for the given key
        SqlConnection conn = shardMap.OpenConnectionForKey<T>
                            (shardingKey, connectionStr, ConnectionOptions.Validate);
        return conn;
    }
```

## <a name="main-points"></a>Puntos principales

* Un nuevo constructor reemplaza al constructor predeterminado en la subclase DbContext
* El nuevo constructor adopta los argumentos necesarios para el enrutamiento dependiente de datos a través de la biblioteca de cliente de bases de datos elásticas:
  
  * El mapa de particiones para acceder a las interfaces de enrutamiento dependiente de datos.
  * La clave de particionamiento para identificar el shardlet.
  * Una cadena de conexión con las credenciales para la conexión de enrutamiento dependiente de datos a la partición.
* La llamada al constructor de clase base se desvía a un método estático que realiza todos los pasos necesarios para enrutamiento dependiente de datos.
  
  * Usa la llamada OpenConnectionForKey de las interfaces de cliente de bases de datos elásticas en el mapa de particiones para establecer una conexión abierta.
  * El mapa de particiones crea la conexión abierta con la partición que contiene el shardlet para la clave de particionamiento especificada.
  * Esta conexión abierta se transfiere de nuevo al constructor de la clase base DbContext para indicar que se va a usar esta conexión en EF en lugar de dejar que EF cree automáticamente una conexión. De este modo, la API de cliente de bases de datos elásticas etiqueta la conexión para que pueda garantizar la coherencia en las operaciones de administración de mapas de particiones.

Use en el código el nuevo constructor para la subclase DbContext en lugar del constructor predeterminado. Este es un ejemplo:

```csharp
// Create and save a new blog.

Console.Write("Enter a name for a new blog: ");
var name = Console.ReadLine();

using (var db = new ElasticScaleContext<int>(
                        sharding.ShardMap,  
                        tenantId1,  
                        connStrBldr.ConnectionString))
{
    var blog = new Blog { Name = name };
    db.Blogs.Add(blog);
    db.SaveChanges();

    // Display all Blogs for tenant 1
    var query = from b in db.Blogs
                orderby b.Name
                select b;
    …
}
```

El nuevo constructor abre la conexión con la partición que contiene los datos para el shardlet identificado por el valor de **tenantid1**. El código en el bloque **using** permanece sin cambios para acceder al **DbSet** de los blogs que usan EF en la partición de **tenantid1**. Esto cambia la semántica para el código del bloque using de modo que todas las operaciones de base de datos ahora se limitan a la partición donde se guarda **tenantid1** . Por ejemplo, una consulta LINQ en los blogs de **DbSet** solo devolvería los blogs almacenados en la partición actual, pero no los almacenados en otras particiones.  

### <a name="transient-faults-handling"></a>Control de errores transitorios

El equipo de Microsoft Patterns & Practices publicó el artículo sobre el [bloque de aplicación de gestión de errores transitorios](/previous-versions/msp-n-p/dn440719(v=pandp.60)). La biblioteca se usa con la biblioteca de cliente de escalado elástico conjuntamente con EF. Sin embargo, asegúrese de que cualquier excepción transitoria se devuelve a un lugar donde pueda garantizar que después de un error transitorio se usa el nuevo constructor para que cualquier nuevo intento de conexión se realice con los constructores que ha ajustado. De lo contrario, no se garantiza una conexión a la partición correcta y no hay ninguna certeza de que la conexión se mantiene cuando se producen cambios en el mapa de particiones.

El ejemplo de código siguiente muestra cómo se puede usar una directiva de reintentos SQL en torno a los nuevos constructores de subclase **DbContext** :

```csharp
SqlDatabaseUtils.SqlRetryPolicy.ExecuteAction(() =>
{
    using (var db = new ElasticScaleContext<int>(
                            sharding.ShardMap,  
                            tenantId1,  
                            connStrBldr.ConnectionString))
        {
                var blog = new Blog { Name = name };
                db.Blogs.Add(blog);
                db.SaveChanges();
        …
        }
    });
```

**SqlDatabaseUtils.SqlRetryPolicy** en el código anterior se define como **SqlDatabaseTransientErrorDetectionStrategy** con un número de reintentos de 10 y un tiempo de espera de 5 segundos entre reintentos. Este enfoque es parecido a las instrucciones para EF y las transacciones iniciadas por el usuario (consulte las [limitaciones con las estrategias de ejecución de reintentos [EF6 en adelante])](/ef/ef6/fundamentals/connection-resiliency/retry-logic). En ambas situaciones es necesario que el programa de la aplicación controle el ámbito en el que se devuelve la excepción transitoria: para volver a abrir la transacción o (como se muestra) volver a crear el contexto a partir del constructor adecuado que usa la biblioteca de cliente de bases de datos elásticas.

La necesidad de controlar dónde las excepciones transitorias nos llevan de vuelta en el ámbito también impide el uso de la **SqlAzureExecutionStrategy** integrada que se incluye con EF. **SqlAzureExecutionStrategy** volvería a abrir la conexión pero no usa **OpenConnectionForKey** y, por tanto, pasa por alto toda la validación que se realiza como parte de la llamada **OpenConnectionForKey**. En su lugar, el código de ejemplo usa **DefaultExecutionStrategy** integrada que también se incluye con EF. Al contrario que **SqlAzureExecutionStrategy**, funciona correctamente en combinación con la directiva de reintentos de gestión de errores transitorios. La directiva de ejecución se establece en la clase **ElasticScaleDbConfiguration** . Tenga en cuenta que hemos decidido no usar **DefaultSqlExecutionStrategy** dado que sugiere el uso de **SqlAzureExecutionStrategy** si se producen excepciones transitorias, lo que podría llevar a un comportamiento erróneo, como hemos comentado. Para obtener más información sobre las diferentes directivas de reintento y EF, consulte [Resistencia de conexión en EF](/ef/ef6/fundamentals/connection-resiliency/retry-logic).

#### <a name="constructor-rewrites"></a>Reescrituras del constructor

Los ejemplos de código anteriores muestran las reescrituras del constructor predeterminado que requiere la aplicación para usar enrutamiento dependiente de datos con Entity Framework. La siguiente tabla generaliza este método para otros constructores.

| Constructor actual | Constructor reescrito para datos | Constructor base | Notas |
| --- | --- | --- | --- |
| MyContext() |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |La conexión debe ser una función de la asignación de particiones y la clave de enrutamiento dependiente de datos. Se debe omitir la creación de conexión automática por parte de EF y usar en su lugar el mapa de particiones para negociar la conexión. |
| MyContext(string) |ElasticScaleContext(ShardMap, TKey) |DbContext(DbConnection, bool) |La conexión es una función del mapa de particiones y la clave de enrutamiento dependiente de datos. Una cadena de nombre de base de datos o de conexión fija no funciona, ya que omite la validación por parte del mapa de particiones. |
| MyContext(DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |La conexión se crea para el mapa de particiones determinado y la clave de particionamiento con el modelo proporcionado. El modelo compilado se transfiere al constructor base. |
| MyContext(DbConnection, bool) |ElasticScaleContext(ShardMap, TKey, bool) |DbContext(DbConnection, bool) |La conexión se tiene que deducir del mapa de particiones y la clave. No se puede especificar como entrada (a menos que la entrada ya use el mapa de particiones y la clave). Se transfiere el valor booleano. |
| MyContext(string, DbCompiledModel) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel) |DbContext(DbConnection, DbCompiledModel, bool) |La conexión se tiene que deducir del mapa de particiones y la clave. No se puede especificar como entrada (a menos que la entrada use el mapa de particiones y la clave). Se transfiere el modelo compilado. |
| MyContext(ObjectContext, bool) |ElasticScaleContext(ShardMap, TKey, ObjectContext, bool) |DbContext(ObjectContext, bool) |El nuevo constructor debe asegurarse de que las conexiones de ObjectContext pasado como entrada se vuelve a enrutar a una conexión administrada por el escalado elástico. No es el objetivo de este documento dar una explicación detallada de ObjectContext. |
| MyContext(DbConnection, DbCompiledModel, bool) |ElasticScaleContext(ShardMap, TKey, DbCompiledModel, bool) |DbContext(DbConnection, DbCompiledModel, bool); |La conexión se tiene que deducir del mapa de particiones y la clave. La conexión no se puede especificar como entrada (a menos que la entrada ya use el mapa de particiones y la clave). El modelo y los valores booleanos se transfieren al constructor de clase base. |

## <a name="shard-schema-deployment-through-ef-migrations"></a>Implementación del esquema de partición mediante migraciones de EF

La administración automática de esquemas es una ventaja que ofrece Entity Framework. En el contexto de aplicaciones con herramientas de bases de datos elásticas, desea conservar esta capacidad para aprovisionar automáticamente el esquema a las particiones recién creadas cuando se agregan bases de datos a la aplicación particionada. El principal caso de uso es aumentar la capacidad en la capa de datos para aplicaciones particionadas con EF. Al basarse en la funcionalidad de EF para la administración de esquemas se reducen las tareas de administración de base de datos con una aplicación particionada que esté integrada en EF.

La implementación del esquema a través de migraciones de EF funciona mejor en **conexiones sin abrir**. Esto difiere del escenario de enrutamiento dependiente de datos que se basa en la conexión abierta que proporciona la API de cliente de bases de datos elásticas. Otra diferencia es el requisito de coherencia: aunque es conveniente garantizar la coherencia de todas las conexiones de enrutamiento dependiente de datos como protección ante la manipulación del mapa de particiones simultánea, no hay ningún problema con la implementación del esquema inicial en una nueva base de datos que aún no se haya registrado en el mapa de particiones y no se haya asignado para que contenga shardlets. Por lo tanto, puede confiar en las conexiones de base de datos normales para estos escenarios, en lugar de en el enrutamiento dependiente de datos.  

Esto conduce a un método en el que la implementación de esquemas mediante migraciones de EF está estrechamente vinculado al registro de la nueva base de datos como partición del mapa de particiones de la aplicación. Se basa en los requisitos previos siguientes:

* Ya se ha creado la base de datos.
* La base de datos está vacía, no contiene ningún esquema de usuario ni hay datos de usuario.
* Aún no se puede tener acceso a la base de datos mediante las API de cliente de bases de datos elásticas para enrutamiento dependiente de datos.

Una vez satisfechos estos requisitos previos, puede crear una **SqlConnection** normal sin abrir para iniciar migraciones de EF para la implementación del esquema. En el ejemplo de código siguiente se muestra este método.

```csharp
// Enter a new shard - i.e. an empty database - to the shard map, allocate a first tenant to it  
// and kick off EF initialization of the database to deploy schema

public void RegisterNewShard(string server, string database, string connStr, int key)
{

    Shard shard = this.ShardMap.CreateShard(new ShardLocation(server, database));

    SqlConnectionStringBuilder connStrBldr = new SqlConnectionStringBuilder(connStr);
    connStrBldr.DataSource = server;
    connStrBldr.InitialCatalog = database;

    // Go into a DbContext to trigger migrations and schema deployment for the new shard.
    // This requires an un-opened connection.
    using (var db = new ElasticScaleContext<int>(connStrBldr.ConnectionString))
    {
        // Run a query to engage EF migrations
        (from b in db.Blogs
            select b).Count();
    }

    // Register the mapping of the tenant to the shard in the shard map.
    // After this step, data-dependent routing on the shard map can be used

    this.ShardMap.CreatePointMapping(key, shard);
}
```

Este ejemplo muestra el método **RegisterNewShard** que registra la partición en el mapa de particiones, implementa el esquema mediante migraciones de EF y almacena la asignación de una clave de particionamiento en la partición. Se basa en un constructor de la subclase **DbContext** (**ElasticScaleContext** en el ejemplo) que toma una cadena de conexión SQL como entrada. El código de este constructor es sencillo, como se muestra en el ejemplo siguiente:

```csharp
// C'tor to deploy schema and migrations to a new shard
protected internal ElasticScaleContext(string connectionString)
    : base(SetInitializerForConnection(connectionString))
{
}

// Only static methods are allowed in calls into base class c'tors
private static string SetInitializerForConnection(string connectionString)
{
    // You want existence checks so that the schema can get deployed
    Database.SetInitializer<ElasticScaleContext<T>>(
new CreateDatabaseIfNotExists<ElasticScaleContext<T>>());

    return connectionString;
}
```

Se podría haber usado la versión del constructor heredado de la clase base. Pero el código debe garantizar que el inicializador predeterminado para EF se usa al conectarse. De ahí el breve desvío del método estático antes de llamar al constructor de clase base con la cadena de conexión. Tenga en cuenta que el registro de particiones debe ejecutarse en un dominio de aplicación diferente o procesarse para garantizar que los valores de configuración del inicializador para EF no entren en conflicto.

## <a name="limitations"></a>Limitaciones

Los métodos descritos en este documento implican un par de limitaciones:

* Las aplicaciones de EF que usen **LocalDb** deben migrar en primer lugar a una base de datos de SQL Server normal antes de usar la biblioteca de cliente de Base de datos elástica. El escalado horizontal de una aplicación mediante particionamiento con Escalado elástico no es posible con **LocalDb**. Tenga en cuenta que los desarrolladores pueden seguir usando **LocalDb**.
* Los cambios efectuados en la aplicación que implican cambios en el esquema de base de datos deben pasar por migraciones de EF en todas las particiones. El código de ejemplo de este documento no muestra cómo hacerlo. Considere el uso de Update-Database con un parámetro ConnectionString para iterar en todas las particiones; o extraiga el script T-SQL para la migración pendiente usando Update-Database con la opción –Script y aplique el script T-SQL en sus particiones.  
* Dada una solicitud, se supone que todo el procesamiento de la base de datos está contenido en una sola partición que se identifica con la clave de particionamiento especificada por la solicitud. Sin embargo, esta suposición no siempre es cierta. Por ejemplo, cuando no se puede disponer de una clave de particionamiento. Para solucionar este problema, la biblioteca cliente proporciona la clase **MultiShardQuery** que implementa una abstracción de conexión para realizar consultas en varias particiones. No es el objetivo de este documento tratar sobre el uso de **MultiShardQuery** junto con EF.

## <a name="conclusion"></a>Conclusión

Con los pasos descritos en este documento, las aplicaciones de EF pueden utilizar la funcionalidad de enrutamiento dependiente de datos de la biblioteca cliente de bases de datos elásticas mediante la refactorización de constructores de las subclases **DbContext** que se usan en la aplicación de EF. Esto limita los cambios necesarios en los lugares donde ya existen clases **DbContext** . Además, las aplicaciones de EF pueden seguir aprovechando la implementación automática de esquemas mediante la combinación de los pasos que invocan las migraciones de EF necesarias con el registro de nuevas particiones y asignaciones en el mapa de particiones.

[!INCLUDE [elastic-scale-include](../../../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-use-entity-framework-applications-visual-studio/sample.png