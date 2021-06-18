---
title: Transacciones distribuidas en bases de datos en la nube (versión preliminar)
description: Información general sobre las transacciones de Base de datos elástica con Azure SQL Database y Azure SQL Managed Instance.
services: sql-database
ms.service: sql-database
ms.subservice: scale-out
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: scoriani
ms.author: scoriani
ms.reviewer: mathoma
ms.date: 03/12/2019
ms.openlocfilehash: c5586c425de4ca33565df39b1839bcab075faee5
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110690104"
---
# <a name="distributed-transactions-across-cloud-databases-preview"></a>Transacciones distribuidas en bases de datos en la nube (versión preliminar)
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Las transacciones de base de datos elástica de Azure SQL Database y Azure SQL Managed Instance permiten ejecutar transacciones que abarcan varias bases de datos. Las transacciones de base de datos elástica están disponibles para aplicaciones .NET con ADO.NET y se integran con la conocida experiencia de programación en la que se emplean las clases [System.Transaction](/dotnet/api/system.transactions). Para obtener la biblioteca, vea [.NET Framework 4.6.1 (instalador web)](https://www.microsoft.com/download/details.aspx?id=49981).
Además, las transacciones distribuidas de Managed Instance están disponibles en [Transact-SQL](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql).

De forma local, este escenario requiere normalmente la ejecución del Coordinador de transacciones distribuidas de Microsoft (MSDTC). Puesto que MSDTC no está disponible en Azure para aplicaciones de plataforma como servicio, la capacidad de coordinar transacciones distribuidas ahora se integra directamente en SQL Database o en Managed Instance. Las aplicaciones pueden conectarse a cualquier base de datos para iniciar transacciones distribuidas, y una de las bases de datos o de los servidores coordina de forma transparente la transacción distribuida, como se muestra en la ilustración siguiente.

En este documento, los términos "transacciones distribuidas" y "transacciones de base de datos elástica" se consideran sinónimos y se usan indistintamente.

  ![Transacciones distribuidas con Azure SQL Database mediante transacciones de base de datos elástica ][1]

## <a name="common-scenarios"></a>Escenarios frecuentes

Las transacciones de base de datos elástica permiten a las aplicaciones realizar cambios atómicos en los datos almacenados en varias bases de datos diferentes. La versión preliminar se centra en las experiencias de desarrollo del lado cliente en C# y. NET. La experiencia del servidor (código escrito en procedimientos almacenados o scripts del servidor) mediante [Transact-SQL](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql) solo está disponible para Managed Instance.
> [!IMPORTANT]
> En la versión preliminar, de momento no se admite la ejecución de transacciones de base de datos elástica entre Azure SQL Database y Azure SQL Managed Instance. La transacción de base de datos elástica solo puede abarcar un conjunto de instancias de SQL Database o de Managed Instance.

Las transacciones de base de datos elástica están orientadas a los siguientes escenarios:

* Aplicaciones de varias bases de datos en Azure: Con este escenario, los datos se particionan verticalmente en varias bases de datos de SQL Database o Managed Instance, de forma que diferentes clases de datos residen en diferentes bases de datos. Algunas operaciones requieren cambios en los datos que se mantienen en dos o más bases de datos. La aplicación emplea transacciones de base de datos elástica para coordinar los cambios entre las bases de datos y garantizar la atomicidad.
* Aplicaciones de base de datos particionada en Azure: Con este escenario, la capa de datos usa la [biblioteca cliente de Base de datos elástica](elastic-database-client-library.md) o el particionamiento automático para particionar horizontalmente los datos en muchas bases de datos de SQL Database o Managed Instance. Un caso destacado de uso es cuando existe la necesidad de realizar cambios atómicos en una aplicación particionada multiempresa cuando los cambios abarcan a los inquilinos. Piense por ejemplo en una transferencia desde un inquilino a otro, donde cada uno reside en una base de datos diferente. Un segundo caso es el particionamiento específico para satisfacer las necesidades de capacidad de un inquilino de gran tamaño, lo que a su vez supone normalmente que algunas operaciones atómicas deban extenderse entre varias bases de datos usadas para el mismo inquilino. Un tercer caso es el de las actualizaciones atómicas para hacer referencia a los datos que se replican entre bases de datos. Ahora se pueden coordinar operaciones de transacciones atómicas a lo largo de estas líneas entre varias bases de datos mediante la versión preliminar.
  Las transacciones de base de datos elástica usan la confirmación en dos fases para garantizar la atomicidad de las transacciones entre bases de datos. Esto resulta adecuado para transacciones que suponen menos de 100 bases de datos a la vez en una única transacción. Aunque estos límites no se aplican, se supone que las tasas de rendimiento y éxito de las transacciones de base de datos elástica se verán afectadas cuando se excedan estos límites.

## <a name="installation-and-migration"></a>Instalación y migración

Las capacidades de las transacciones de base de datos elástica se proporcionan por medio de actualizaciones de las bibliotecas .NET System.Data.dll y System.Transactions.dll. Los archivos DLL garantizan que la confirmación en dos fases se usa cuando es necesario para asegurar la atomicidad. Para empezar a desarrollar aplicaciones mediante transacciones de base de datos elástica, instale [.NET Framework 4.6.1](https://www.microsoft.com/download/details.aspx?id=49981) o versión posterior. Cuando se ejecutan en una versión anterior de .NET Framework, las transacciones no podrán promoverse a una transacción distribuida y se producirá una excepción.

Tras la instalación, puede usar las API de transacciones distribuidas de System.Transactions con conexiones a SQL Database y Managed Instance. Si tiene aplicaciones de MSDTC existentes que usan estas API, recompílelas para .NET 4.6 después de instalar Framework 4.6.1. Si los proyectos se destinan a .NET 4.6, usan automáticamente los archivos DLL actualizados de la nueva versión de Framework, y las llamadas API de transacciones distribuidas en combinación con las conexiones a SQL Database o Managed Instance ahora se realizan correctamente.

Recuerde que las transacciones de base de datos elástica no precisan la instalación de MSDTC. Por el contrario, se administran directamente mediante y dentro del servicio. Con ello se simplifican enormemente los escenarios de nube, ya que no es necesario implementar MSDTC para usar transacciones distribuidas con SQL Database o Managed Instance. En la sección 4 se explica con más detalle cómo implementar transacciones de base de datos elástica y  la versión de .NET Framework necesaria junto con sus aplicaciones de nube en Azure.

## <a name="net-installation-for-azure-cloud-services"></a>Instalación de .NET para Azure Cloud Services

Azure proporciona varias ofertas para hospedar aplicaciones. NET. Hay disponible una comparación de las diferentes ofertas en [Comparación de Azure App Service, Cloud Services y Virtual Machines](/azure/architecture/guide/technology-choices/compute-decision-tree). Si el SO invitado de la oferta es inferior a .NET 4.6.1, que es el que se requiere para las transacciones elásticas, debe actualizar el SO invitado a 4.6.1.

Para Azure App Service, no se admiten las actualizaciones del SO invitado en estos momentos. En el caso de Azure Virtual Machines, solo tiene que iniciar sesión en la máquina virtual y ejecutar el instalador para la última versión de .NET Framework. Para Azure Cloud Services, hay que incluir la instalación de una versión más reciente de .NET en las tareas de inicio de la implementación. Los conceptos y los pasos se documentan en [Instalación de .NET en un rol de servicio en la nube](../../cloud-services/cloud-services-dotnet-install-dotnet.md).  

Tenga en cuenta que el instalador de .NET 4.6.1 puede requerir más espacio de almacenamiento temporal durante el proceso de arranque en los Servicios en la nube de Azure que el instalador de .NET 4.6. Para garantizar una instalación correcta, debe aumentar el almacenamiento temporal para el servicio en la nube de Azure en el archivo ServiceDefinition.csdef en la sección LocalResources y en la configuración del entorno de la tarea de inicio, como se muestra en el ejemplo siguiente:

```xml
    <LocalResources>
    ...
        <LocalStorage name="TEMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
        <LocalStorage name="TMP" sizeInMB="5000" cleanOnRoleRecycle="false" />
    </LocalResources>
    <Startup>
        <Task commandLine="install.cmd" executionContext="elevated" taskType="simple">
            <Environment>
        ...
                <Variable name="TEMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TEMP']/@path" />
                </Variable>
                <Variable name="TMP">
                    <RoleInstanceValue xpath="/RoleEnvironment/CurrentInstance/LocalResources/LocalResource[@name='TMP']/@path" />
                </Variable>
            </Environment>
        </Task>
    </Startup>
```

## <a name="net-development-experience"></a>Experiencia de desarrollo de .NET

### <a name="multi-database-applications"></a>Aplicaciones de varias bases de datos

En el ejemplo de código siguiente se usa la experiencia de programación conocida con System.Transactions de .NET. La clase TransactionScope establece una transacción de ambiente en. NET. (Una "transacción de ambiente" es aquella que reside en el subproceso actual). Todas las conexiones abiertas dentro de TransactionScope intervienen en la transacción. Si intervienen bases de datos diferentes, la transacción se eleva automáticamente a transacción distribuida. El resultado de la transacción se controla mediante la configuración del ámbito como completo para indicar una confirmación.

```csharp
    using (var scope = new TransactionScope())
    {
        using (var conn1 = new SqlConnection(connStrDb1))
        {
            conn1.Open();
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = new SqlConnection(connStrDb2))
        {
            conn2.Open();
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T2 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }
```

### <a name="sharded-database-applications"></a>Aplicaciones de base de datos particionada

Las transacciones de base de datos elástica de SQL Database y Managed Instance también admiten la coordinación de transacciones distribuidas en las que se usa el método OpenConnectionForKey de la biblioteca cliente de base de datos elástica a fin de abrir conexiones para una capa de datos escalada horizontalmente. Tenga en cuenta los casos en lo que deba garantizar la coherencia de las transacciones para los cambios entre varios valores diferentes de clave de particionamiento. Las conexiones a las particiones que hospedan los diferentes valores de clave de particionamiento se interrumpen mediante OpenConnectionForKey. Por lo general, las conexiones pueden ser a particiones diferentes de forma que, para garantizar las transacciones, se necesita una transacción distribuida.
En el ejemplo de código siguiente se muestra este método. Se supone que una variable llamada shardmap se usa para representar un mapa de particiones de la biblioteca de cliente de base de datos elástica:

```csharp
    using (var scope = new TransactionScope())
    {
        using (var conn1 = shardmap.OpenConnectionForKey(tenantId1, credentialsStr))
        {
            SqlCommand cmd1 = conn1.CreateCommand();
            cmd1.CommandText = string.Format("insert into T1 values(1)");
            cmd1.ExecuteNonQuery();
        }

        using (var conn2 = shardmap.OpenConnectionForKey(tenantId2, credentialsStr))
        {
            var cmd2 = conn2.CreateCommand();
            cmd2.CommandText = string.Format("insert into T1 values(2)");
            cmd2.ExecuteNonQuery();
        }

        scope.Complete();
    }
```

## <a name="transact-sql-development-experience"></a>Experiencia de desarrollo de Transact-SQL

Las transacciones distribuidas del servidor que usan Transact-SQL solo están disponibles para Azure SQL Managed Instance. La transacción distribuida solo se puede ejecutar entre instancias de Managed Instance que pertenezcan al mismo [Grupo de confianza del servidor](../managed-instance/server-trust-group-overview.md). En este caso, las instancias de Managed Instance deben usar un [servidor vinculado](/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine#TsqlProcedure) para hacerse referencia entre sí.

En el siguiente código de Transact-SQL de ejemplo se usa [BEGIN DISTRIBUTED TRANSACTION](/sql/t-sql/language-elements/begin-distributed-transaction-transact-sql) para iniciar la transacción distribuida.

```Transact-SQL

    -- Configure the Linked Server
    -- Add one Azure SQL Managed Instance as Linked Server
    EXEC sp_addlinkedserver
        @server='RemoteServer', -- Linked server name
        @srvproduct='',
        @provider='sqlncli', -- SQL Server Native Client
        @datasrc='managed-instance-server.46e7afd5bc81.database.windows.net' -- Managed Instance endpoint

    -- Add credentials and options to this Linked Server
    EXEC sp_addlinkedsrvlogin
        @rmtsrvname = 'RemoteServer', -- Linked server name
        @useself = 'false',
        @rmtuser = '<login_name>',         -- login
        @rmtpassword = '<secure_password>' -- password

    USE AdventureWorks2012;
    GO
    SET XACT_ABORT ON;
    GO
    BEGIN DISTRIBUTED TRANSACTION;
    -- Delete candidate from local instance.
    DELETE AdventureWorks2012.HumanResources.JobCandidate
        WHERE JobCandidateID = 13;
    -- Delete candidate from remote instance.
    DELETE RemoteServer.AdventureWorks2012.HumanResources.JobCandidate
        WHERE JobCandidateID = 13;
    COMMIT TRANSACTION;
    GO
```

## <a name="combining-net-and-transact-sql-development-experience"></a>Combinación de la experiencia de desarrollo de .NET y Transact-SQL

Las aplicaciones .NET que usan las clases System.Transaction pueden combinar la clase TransactionScope con la instrucción de Transact-SQL BEGIN DISTRIBUTED TRANSACTION. En TransactionScope, la transacción interna que ejecuta BEGIN DISTRIBUTED TRANSACTION se promueve explícitamente a transacción distribuida. Además, cuando se abre el segundo SqlConnecton en TransactionScope, se promociona implícitamente a transacción distribuida. Una vez iniciada la transacción distribuida, todas las solicitudes posteriores de transacciones, tanto si proceden de .NET como de Transact-SQL, se unen a la transacción distribuida primaria. Como consecuencia, todos los ámbitos de transacción anidados iniciados por la instrucción BEGIN terminan en la misma transacción y las instrucciones COMMIT/ROLLBACK tienen el efecto siguiente en el resultado general:
 * La instrucción COMMIT no tiene ningún efecto en el ámbito de la transacción iniciado por la instrucción BEGIN, es decir, no se confirman resultados antes de que se invoque al método Complete() en el objeto TransactionScope. Si el objeto TransactionScope se destruye antes de completarse, se revierten todos los cambios realizados dentro del ámbito.
 * La instrucción ROLLBACK hace que se revierta TransactionScope completamente. Cualquier intento de dar de alta nuevas transacciones en TransactionScope produce un error posteriormente, así como el intento de invocar a Complete() en el objeto TransactionScope.

Este es un ejemplo en el que la transacción se promueve explícitamente a transacción distribuida con Transact-SQL.

```csharp
    using (TransactionScope s = new TransactionScope())
    {
        using (SqlConnection conn = new SqlConnection(DB0_ConnectionString)
        {
            conn.Open();
        
            // Transaction is here promoted to distributed by BEGIN statement
            //
            Helper.ExecuteNonQueryOnOpenConnection(conn, "BEGIN DISTRIBUTED TRAN");
            // ...
        }
     
        using (SqlConnection conn2 = new SqlConnection(DB1_ConnectionString)
        {
            conn2.Open();
            // ...
        }
        
        s.Complete();
    }
```

En el ejemplo siguiente se muestra una transacción que se promueve implícitamente a transacción distribuida una vez que se inicia el segundo SqlConnecton en TransactionScope.

```csharp
    using (TransactionScope s = new TransactionScope())
    {
        using (SqlConnection conn = new SqlConnection(DB0_ConnectionString)
        {
            conn.Open();
            // ...
        }
        
        using (SqlConnection conn = new SqlConnection(DB1_ConnectionString)
        {
            // Because this is second SqlConnection within TransactionScope transaction is here implicitly promoted distributed.
            //
            conn.Open(); 
            Helper.ExecuteNonQueryOnOpenConnection(conn, "BEGIN DISTRIBUTED TRAN");
            Helper.ExecuteNonQueryOnOpenConnection(conn, lsQuery);
            // ...
        }
        
        s.Complete();
    }
```

## <a name="transactions-across-multiple-servers-for-azure-sql-database"></a>Transacciones en varios servidores de Azure SQL Database

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]
> [!IMPORTANT]
> El módulo de Azure Resource Manager para PowerShell todavía es compatible con Azure SQL Database, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. Para estos cmdlets, consulte [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos.

Se admiten transacciones de la base de datos elástica entre diferentes servidores en Azure SQL Database. Cuando las transacciones cruzan los límites del servidor, los servidores implicados en primer lugar deben involucrarse en una relación de comunicación mutua. Una vez que se ha establecido la relación de comunicación, cualquier base de datos ubicada en uno de los dos servidores puede participar en transacciones elásticas con bases de datos desde el otro servidor. En el caso de las transacciones distribuidas en más de dos servidores, debe existir una relación de comunicación para cualquier par de servidores.

Use los siguientes cmdlets de PowerShell para administrar las relaciones de comunicación entre los servidores para las transacciones de la base de datos elástica:

* **New-AzSqlServerCommunicationLink**: este cmdlet se usa para crear una relación de comunicación nueva entre dos servidores en Azure SQL Database. La relación es simétrica, lo que significa que ambos servidores pueden iniciar transacciones con el otro servidor.
* **Get-AzSqlServerCommunicationLink**: este cmdlet se usa para recuperar una relación de comunicación existente y sus propiedades.
* **Remove-AzSqlServerCommunicationLink**: este cmdlet se usa para eliminar una relación de comunicación existente.

## <a name="transactions-across-multiple-servers-for-azure-sql-managed-instance"></a>Transacciones en varios servidores de Azure SQL Managed Instance

Se admiten transacciones distribuidas en diferentes servidores de Azure SQL Managed Instance. Cuando las transacciones cruzan los límites de Managed Instance, las instancias participantes primero deben involucrarse en una relación de comunicación y seguridad mutua. Esto se consigue mediante la creación de un [Grupo de confianza del servidor](../managed-instance/server-trust-group-overview.md), que se puede hacer en Azure Portal. Si las instancias de Managed Instance no están en la misma red virtual, es necesario configurar el [Emparejamiento de redes virtuales](../../virtual-network/virtual-network-peering-overview.md) y las reglas de entrada y salida del grupo de seguridad de red deben permitir los puertos 5024 y 11000-12000 en todas las redes virtuales que participan.

  ![Grupos de confianza del servidor en Azure Portal][3]

En el diagrama siguiente se muestra el Grupo de confianza del servidor con instancias de Managed Instance que pueden ejecutar transacciones distribuidas con .NET o Transact-SQL.

  ![Transacciones distribuidas con Azure SQL Managed Instance con transacciones elásticas][2]

## <a name="monitoring-transaction-status"></a>Supervisión del estado de la transacción

Use vistas de administración dinámica (DMV) para supervisar el estado y el progreso de las transacciones en curso de base de datos elástica. Todas las DMV relacionadas con las transacciones son pertinentes para las transacciones distribuidas en SQL Database y Managed Instance. Puede encontrar la lista correspondiente de DMV aquí: [Funciones y vistas de administración dinámica relacionadas con transacciones (Transact-SQL)](/sql/relational-databases/system-dynamic-management-views/transaction-related-dynamic-management-views-and-functions-transact-sql).

Estas DMV son especialmente útiles:

* **sys.dm\_tran\_active\_transactions**: enumera las transacciones actualmente activas y su estado. La columna UOW (unidad de trabajo) puede identificar las distintas transacciones secundarias que pertenecen a la misma transacción distribuida. Todas las transacciones de la misma transacción distribuida llevan el mismo valor de UOW. Para obtener más información, vea la [documentación de DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-active-transactions-transact-sql).
* **sys.dm\_tran\_database\_transactions**: proporciona información adicional sobre las transacciones, como la colocación de la transacción en el registro. Para obtener más información, vea la [documentación de DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-database-transactions-transact-sql).
* **sys.dm\_tran\_locks**: ofrece información sobre los bloqueos actuales de las transacciones. Para obtener más información, vea la [documentación de DMV](/sql/relational-databases/system-dynamic-management-views/sys-dm-tran-locks-transact-sql).

## <a name="limitations"></a>Limitaciones

Las limitaciones siguientes se aplican actualmente a transacciones de base de datos elástica en SQL Database:

* Se admiten únicamente transacciones entre bases de datos en SQL Database. Otros proveedores de recursos y bases de datos de [X/Open XA](https://en.wikipedia.org/wiki/X/Open_XA) externos a SQL Database no podrán participar en transacciones de base de datos elástica. Esto significa que dichas transacciones no pueden extenderse entre bases de datos locales de SQL Server y Azure SQL Database. Para las transacciones distribuidas en el entorno local, siga usando MSDTC.
* Solo se admiten transacciones coordinadas por el cliente desde una aplicación .NET. Está prevista la compatibilidad en el lado servidor con T-SQL, por ejemplo, INICIAR TRANSACCIÓN DISTRIBUIDA, pero aún no se encuentra disponible.
* No se admiten las transacciones por los servicios WCF. Por ejemplo, tiene un método de servicio de WCF que se ejecuta una transacción. Si se encierra la llamada dentro de un ámbito de transacción, se producirá un error como [System.ServiceModel.ProtocolException](/dotnet/api/system.servicemodel.protocolexception).

Las siguientes limitaciones se aplican actualmente a las transacciones distribuidas de Managed Instance:

* Se admiten únicamente transacciones en bases de datos de Managed Instance. Otros proveedores de recursos de [X/Open XA](https://en.wikipedia.org/wiki/X/Open_XA) y bases de datos externos a Azure SQL Managed Instance no pueden participar en transacciones distribuidas. Esto significa que las transacciones distribuidas no pueden extenderse entre Azure SQL Managed Instance y SQL Server local. Para las transacciones distribuidas en el entorno local, siga usando MSDTC.
* No se admiten las transacciones por los servicios WCF. Por ejemplo, tiene un método de servicio de WCF que se ejecuta una transacción. Si se encierra la llamada dentro de un ámbito de transacción, se producirá un error como [System.ServiceModel.ProtocolException](/dotnet/api/system.servicemodel.protocolexception).
* Azure SQL Managed Instance debe formar parte de un [Grupo de confianza del servidor](../managed-instance/server-trust-group-overview.md) para poder participar en transacciones distribuidas.
* Las limitaciones de los [Grupos de confianza del servidor](../managed-instance/server-trust-group-overview.md) afectan a las transacciones distribuidas.
* Las instancias de Managed Instance que participan en las transacciones distribuidas deben disponer de conectividad mediante puntos de conexión privados (mediante la dirección IP privada de la red virtual en la que se implementan) y deben hacerse referencia mutuamente mediante FQDN privados. Las aplicaciones cliente pueden usar transacciones distribuidas en puntos de conexión privados. Además, en aquellos casos en los que Transact-SQL aproveche los servidores vinculados que hacen referencia a puntos de conexión privados, las aplicaciones cliente también pueden usar las transacciones distribuidas en puntos de conexión públicos. Dicha limitación se explica en el siguiente diagrama.
  ![Limitación de conectividad del punto de conexión privado][4]
## <a name="next-steps"></a>Pasos siguientes

* Si tiene preguntas, póngase en contacto con nosotros en la [página de preguntas de Microsoft Q&A sobre SQL Database](/answers/topics/azure-sql-database.html).
* Para solicitar características, agréguelas al [foro de comentarios de SQL Database](https://feedback.azure.com/forums/217321-sql-database/) o al [foro de Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance).



<!--Image references-->
[1]: ./media/elastic-transactions-overview/distributed-transactions.png
[2]: ./media/elastic-transactions-overview/sql-mi-distributed-transactions.png
[3]: ./media/elastic-transactions-overview/server-trust-groups-azure-portal.png
[4]: ./media/elastic-transactions-overview/managed-instance-distributed-transactions-private-endpoint-limitations.png
