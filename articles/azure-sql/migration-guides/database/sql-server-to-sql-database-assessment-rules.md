---
title: Reglas de evaluación para la migración de SQL Server a Azure SQL Database
description: Reglas de evaluación para identificar los problemas con la instancia de origen de SQL Server que deben solucionarse antes de migrar a Azure SQL Database.
ms.service: sql-database
ms.subservice: migration-guide
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: rajeshsetlem
ms.author: rsetlem
ms.reviewer: mathoma, cawrites
ms.date: 12/15/2020
ms.openlocfilehash: 7046ac7852585a51174c774a0ef2f1150a72ae1f
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111591969"
---
# <a name="assessment-rules-for-sql-server-to-azure-sql-database-migration"></a>Reglas de evaluación para la migración de SQL Server a Azure SQL Database
[!INCLUDE[appliesto--sqldb](../../includes/appliesto-sqldb.md)]

Las herramientas de migración validan la instancia de origen de SQL Server mediante la ejecución de una serie de reglas de evaluación para identificar los problemas que deben solucionarse antes de migrar la base de datos de SQL Server a Azure SQL Database. 

En este artículo se proporciona una lista de las reglas que se usan para evaluar la viabilidad de la migración de la base de datos de SQL Server a Azure SQL Database. 


## <a name="bulk-insert"></a>Inserción masiva<a id="BulkInsert"></a>

**Título: No se admite BULK INSERT con orígenes de datos de blobs que no son de Azure en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**   
Azure SQL Database no puede acceder a recursos compartidos de archivos ni a carpetas de Windows. Vea en la sección "Objetos afectados" los usos específicos de las instrucciones BULK INSERT que no hacen referencia a un BLOB de Azure. Los objetos con "BULK INSERT" en los que el origen no sea Azure Blob Storage no funcionarán después de migrar a Azure SQL Database. 


**Recomendación**   
Deberá convertir las instrucciones BULK INSERT que usan archivos locales o recursos compartidos de archivos para que usen archivos de Azure Blob Storage en su lugar, al realizar la migración a Azure SQL Database. También puede migrar a SQL Server en la máquina virtual de Azure.

## <a name="compute-clause"></a>Cláusula Compute<a id="ComputeClause"></a>

**Título: Se ha descontinuado y quitado la cláusula COMPUTE.**    
**Categoría**: Advertencia   

**Descripción**   
La cláusula COMPUTE genera totales que aparecen como columnas de resumen adicionales al final del conjunto de resultados. Sin embargo, esta cláusula ya no se admite en Azure SQL Database. 


**Recomendación**   
En su lugar, se debe volver a escribir el módulo T-SQL con el operador ROLLUP. En el código siguiente se muestra cómo se puede reemplazar COMPUTE por ROLLUP: 

```sql 
USE AdventureWorks 
GO;  

SELECT SalesOrderID, UnitPrice, UnitPriceDiscount 
FROM Sales.SalesOrderDetail 
ORDER BY SalesOrderID COMPUTE SUM(UnitPrice), SUM(UnitPriceDiscount) BY SalesOrderID GO; 

SELECT SalesOrderID, UnitPrice, UnitPriceDiscount,SUM(UnitPrice) as UnitPrice , 
SUM(UnitPriceDiscount) as UnitPriceDiscount 
FROM Sales.SalesOrderDetail 
GROUP BY SalesOrderID, UnitPrice, UnitPriceDiscount WITH ROLLUP; 
```

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server ](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="change-data-capture-cdc"></a>Captura de datos modificados (CDC)<a id="CDC"></a>

**Título: La captura de datos modificados (CDC) no se admite en Azure SQL Database**   
**Categoría**: Problema   


**Descripción**   
La captura de datos modificados (CDC) no se admite en Azure SQL Database. Evalúe si en su lugar se puede utilizar Change Tracking.  También puede migrar a Azure SQL Managed Instance o SQL Server en Azure Virtual Machines. 


**Recomendación**   
La captura de datos modificados (CDC) no se admite en Azure SQL Database. Evalúe si Change Tracking se puede usar en su lugar o considere la posibilidad de migrar a Azure SQL Managed Instance.

Más información: [Habilitación de Change Tracking de Azure SQL](https://social.technet.microsoft.com/wiki/contents/articles/2976.azure-sql-how-to-enable-change-tracking.aspx)

## <a name="clr-assemblies"></a>Ensamblados CLR<a id="ClrAssemblies"></a>

**Título: Los ensamblados CLR de SQL no se admiten en Azure SQL Database**   
**Categoría**: Problema   


**Descripción**   
Azure SQL Database no admite ensamblados CLR de SQL. 


**Recomendación**   
Actualmente, no hay manera de conseguirlo en Azure SQL Database. Las soluciones alternativas recomendadas requerirán cambios en el código de la aplicación y la base de datos para usar solo los ensamblados admitidos por Azure SQL Database. También puede migrar a Azure SQL Managed Instance o SQL Server en la máquina virtual de Azure.

Más información: [Diferencias de Transact-SQL no admitidas en SQL Database](../../database/transact-sql-tsql-differences-sql-server.md#transact-sql-syntax-not-supported-in-azure-sql-database)

## <a name="cryptographic-provider"></a>Proveedor de servicios criptográficos<a id="CryptographicProvider"></a>

**Título: Se ha encontrado un uso de CREATE CRYPTOGRAPHIC PROVIDER o ALTER CRYPTOGRAPHIC PROVIDER, que no se admite en Azure SQL Database**   
**Categoría**: Problema   

**Descripción**   
Azure SQL Database no es compatible con las instrucciones CRYPTOGRAPHIC PROVIDER porque no puede acceder a los archivos. Consulte en la sección de objetos afectados los usos específicos de las instrucciones CRYPTOGRAPHIC PROVIDER. Los objetos con `CREATE CRYPTOGRAPHIC PROVIDER` o `ALTER CRYPTOGRAPHIC PROVIDER` no funcionarán correctamente después de migrar a Azure SQL Database.  


**Recomendación**   
Revise los objetos con `CREATE CRYPTOGRAPHIC PROVIDER` o `ALTER CRYPTOGRAPHIC PROVIDER`. En los objetos necesarios, quite los usos de estas características. También puede migrar a SQL Server en la máquina virtual de Azure.

## <a name="cross-database-references"></a>Referencias entre bases de datos<a id="CrossDataseReferences"></a>

**Título: Las consultas entre bases de datos no se admiten en Azure SQL Database**   
**Categoría**: Problema   

**Descripción**   
Las bases de datos de este servidor usan consultas entre bases de datos, que no se admiten en Azure SQL Database. 


**Recomendación**   
Azure SQL Database no admite consultas entre bases de datos. Se recomiendan las siguientes acciones: 
- Migre las bases de datos dependientes a Azure SQL Database y use la funcionalidad de consulta de Base de datos elástica (actualmente en versión preliminar) para realizar consultas entre bases de datos de Azure SQL. 
- Mueva los conjuntos de datos dependientes de otras bases de datos a la base de datos que se está migrando. 
- Migre a Azure SQL Managed Instance.
- Migre a SQL Server en la máquina virtual de Azure. 

Más información: [Comprobación de la consulta de Base de datos elástica de Azure SQL Database (versión preliminar)](../../database/elastic-query-overview.md)

## <a name="database-compatibility"></a>Compatibilidad de bases de datos<a id="DbCompatLevelLowerThan100"></a>

**Título: Azure SQL Database no admite niveles de compatibilidad por debajo de 100.**    
**Categoría**: Advertencia   

**Descripción**   
El nivel de compatibilidad de la base de datos es una preciada herramienta que sirve para modernizar las bases de datos, ya que permite actualizar el motor de base de datos de SQL Server mientras se conserva el estado funcional de las aplicaciones conectadas porque se mantiene el mismo nivel de compatibilidad de la base de datos previo a la actualización. Azure SQL Database no admite niveles de compatibilidad por debajo de 100. 


**Recomendación**   
Evalúe si la funcionalidad de la aplicación está intacta cuando el nivel de compatibilidad de la base de datos se actualiza a 100 en Azure SQL Managed Instance. También puede migrar a SQL Server en la máquina virtual de Azure.

## <a name="database-mail"></a>Correo electrónico de base de datos<a id="DatabaseMail"></a>

**Título: No se admite el Correo electrónico de base de datos en Azure SQL Database.**    
**Categoría**: Advertencia   

**Descripción**   
Este servidor usa la característica Correo electrónico de base de datos, que no se admite en Azure SQL Database.


**Recomendación**   
Considere la posibilidad de migrar a Azure SQL Managed Instance que admite el Correo electrónico de base de datos.  Como alternativa, considere la posibilidad de usar Azure Functions y Sendgrid para llevar a cabo la funcionalidad de correo en Azure SQL Database.

Más información: [Enviar correo electrónico desde Azure SQL Database mediante un script de Azure Functions](https://github.com/microsoft/DataMigrationTeam/tree/master/IP%20and%20Scripts/AF%20SendMail)


## <a name="database-principal-alias"></a>Alias de entidad de seguridad de base de datos<a id="DatabasePrincipalAlias"></a>

**Título: SYS.DATABASE_PRINCIPAL_ALIASES se ha descontinuado y quitado.**    
**Categoría**: Problema   

**Descripción**   
SYS.DATABASE_PRINCIPAL_ALIASES se ha descontinuado y quitado de Azure SQL Database.  


**Recomendación**   
Utilice roles en lugar de alias.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)


## <a name="disable_def_cnst_chk-option"></a>Opción DISABLE_DEF_CNST_CHK<a id="DisableDefCNSTCHK"></a>

**Título: Se descontinúa y quita la opción SET DISABLE_DEF_CNST_CHK.**    
**Categoría**: Problema   

**Descripción**   
Se descontinúa y quita la opción SET DISABLE_DEF_CNST_CHK de Azure SQL Database.  


Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="fastfirstrow-hint"></a>Sugerencia FASTFIRSTROW<a id="FastFirstRowHint"></a>

**Título: La sugerencia de consulta FASTFIRSTROW se ha descontinuado y quitado.**    
**Categoría**: Advertencia   

**Descripción**   
La sugerencia de consulta FASTFIRSTROW se ha descontinuado y quitado de Azure SQL Database.  


**Recomendación**   
En lugar de la sugerencia de consulta FASTFIRSTROW, use OPTION (FAST n).

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="filestream"></a>FileStream<a id="FileStream"></a>

**Título: Filestream no se admite en Azure SQL Database**   
**Categoría**: Problema   

**Descripción**   
La característica FileStream, que permite almacenar datos no estructurados, como documentos de texto, imágenes y vídeos, en el sistema de archivos NTFS, no se admite en Azure SQL Database. 


**Recomendación**   
Cargue los archivos no estructurados en Azure Blob Storage y almacene los metadatos relacionados con estos archivos (nombre, tipo, ubicación de dirección URL, clave de almacenamiento, etc.) en Azure SQL Database. Es posible que tenga que volver a diseñar la aplicación para habilitar la transmisión de blobs desde la instancia de Azure SQL Database y hacia ella. También puede migrar a SQL Server en la máquina virtual de Azure.

Más información: [Blog de la transmisión de blobs hacia la instancia de Azure SQL y desde ella](https://azure.microsoft.com/blog/streaming-blobs-to-and-from-sql-azure/)


## <a name="linked-server"></a>Servidor vinculado<a id="LinkedServer"></a>

**Título: La funcionalidad de servidor vinculado no se admite en Azure SQL Database**   
**Categoría**: Problema   

**Descripción**   
Los servidores vinculados permiten al Motor de base de datos de SQL Server ejecutar comandos en orígenes de datos OLE DB fuera de la instancia de SQL Server. 


**Recomendación**   
Azure SQL Database no admite la funcionalidad de servidor vinculado. Se recomiendan las siguientes acciones para eliminar la necesidad de servidores vinculados: 
- Identifique los conjuntos de datos dependientes de instancias remotas de SQL Server y considere la posibilidad de moverlos a la base de datos que se está migrando. 
- Migre las bases de datos dependientes a Azure y use la funcionalidad de consulta de Base de datos elástica (actualmente en versión preliminar) para realizar consultas entre bases de datos de Azure SQL Database. 

Más información: [Comprobación de la consulta elástica de Azure SQL Database (versión preliminar)](../../database/elastic-query-overview.md) 

## <a name="ms-dtc"></a>MS DTC<a id="MSDTCTransactSQL"></a>

**Título: BEGIN DISTRIBUTED TRANSACTION no se admite en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**   
La transacción distribuida iniciada por Transact SQL BEGIN DISTRIBUTED TRANSACTION y administrada por el Coordinador de transacciones distribuidas de Microsoft (MS DTC) no se admite en Azure SQL Database.  


**Recomendación**   
Revise la sección de objetos afectados en Azure Migrate para ver todos los objetos que usan BEGIN DISTRUBUTED TRANSACTION. Considere la posibilidad de migrar las bases de datos de participantes a Azure SQL Managed Instance donde se admiten las transacciones distribuidas entre varias instancias (actualmente en versión preliminar). También puede migrar a SQL Server en la máquina virtual de Azure.

Más información: [Transacciones entre varios servidores de Azure SQL Managed Instance ](../../database/elastic-transactions-overview.md#transactions-across-multiple-servers-for-azure-sql-managed-instance)


## <a name="openrowset-bulk"></a>OPENROWSET (masivo)<a id="OpenRowsetWithNonBlobDataSourceBulk"></a>

**Título: OpenRowSet no se admite en operaciones masivas con orígenes de datos que no son de Azure Blob Storage en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**: OPENROWSET admite operaciones masivas a través de un proveedor integrado BULK que permite que se lean los datos de un archivo y se devuelvan como un conjunto de filas. No se admite OPENROWSET con orígenes de datos que no son Azure Blob Storage en Azure SQL Database.


**Recomendación**   
Azure SQL Database no puede acceder a los recursos compartidos de archivos ni a las carpetas de Windows, por lo que los archivos se deben importar desde Azure Blob Storage. Por lo tanto, solo se admite el tipo de blob DATASOURCE en la función OPENROWSET. También puede migrar a SQL Server en la máquina virtual de Azure.

Más información: [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](../../database/transact-sql-tsql-differences-sql-server.md#transact-sql-syntax-not-supported-in-azure-sql-database)


## <a name="openrowset-provider"></a>OPENROWSET (proveedor)<a id="OpenRowsetWithSQLAndNonSQLProvider"></a>

**Título: No se admite OpenRowSet con proveedor de SQL o que no es de SQL en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**   
OpenRowSet con proveedor de SQL o que no es SQL es una alternativa para acceder a las tablas de un servidor vinculado y es un método ad hoc de una sola vez para conectarse y acceder a datos remotos utilizando OLE DB. No se admite OpenRowSet con proveedor de SQL o que no es de SQL en Azure SQL Database.


**Recomendación**   
Azure SQL Database admite OPENROWSET solo para importar desde Azure Blob Storage. También puede migrar a SQL Server en la máquina virtual de Azure.

Más información: [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](../../database/transact-sql-tsql-differences-sql-server.md#transact-sql-syntax-not-supported-in-azure-sql-database)


## <a name="non-ansi-left-outer-join"></a>Combinación externa izquierda no ANSI<a id="NonANSILeftOuterJoinSyntax"></a>

**Título: La combinación externa izquierda de estilo no ANSI se ha descontinuado y quitado.**    
**Categoría**: Advertencia   

**Descripción**   
La combinación externa izquierda de estilo no ANSI se ha descontinuado y quitado de Azure SQL Database. 


**Recomendación**   
Utilice la sintaxis de unión de ANSI.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)


## <a name="non-ansi-right-outer-join"></a>Combinación externa derecha no ANSI<a id="NonANSIRightOuterJoinSyntax"></a>

**Título: La combinación externa derecha de estilo no ANSI se ha descontinuado y quitado.**    
**Categoría**: Advertencia   

**Descripción**   
La combinación externa derecha de estilo no ANSI se ha descontinuado y quitado de Azure SQL Database. 


**Recomendación**   
Utilice la sintaxis de unión de ANSI.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="next-column"></a>Columna Next<a id="NextColumn"></a>

**Título: Las tablas y columnas denominadas NEXT darán lugar a un error en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**   
Se detectaron tablas o columnas denominadas NEXT. Las secuencias, que se introdujeron en Microsoft SQL Server, usan la función NEXT VALUE FOR del estándar ANSI. Si una tabla o columna se denomina NEXT y la columna tiene el alias VALUE, y si se omite la palabra clave AS del estándar ANSI, la instrucción resultante puede producir un error.  


**Recomendación**   
Reescriba las instrucciones para incluir la palabra clave AS del estándar ANSI cuando asigne un alias a una tabla o columna. Por ejemplo, cuando un columna se denomina NEXT y tiene el alias VALUE, la consulta `SELECT NEXT VALUE FROM TABLE` produce un error y debe reescribirse como SELECT NEXT AS VALUE FROM TABLE. Igualmente, cuando una tabla se denomina NEXT y tiene el alias VALUE, la consulta `SELECT Col1 FROM NEXT VALUE` produce un error y debe reescribirse como `SELECT Col1 FROM NEXT AS VALUE`.

## <a name="raiserror"></a>RAISERROR<a id="RAISERROR"></a>

**Título: Las llamadas a RAISERROR de estilo heredado se deben reemplazar por equivalentes modernos.**    
**Categoría**: Advertencia   

**Descripción**   
Las llamadas a RAISERROR, como el ejemplo siguiente, se denominan de estilo heredado porque no incluyen las comas y los paréntesis. `RAISERROR 50001 'this is a test'`. Este método de llamada a RAISERROR se ha descontinuado y quitado de Azure SQL Database. 


**Recomendación**   
Vuelva a escribir la instrucción con la sintaxis de RAISERROR actual o evalúe si es viable el enfoque moderno de `BEGIN TRY { }  END TRY BEGIN CATCH {  THROW; } END CATCH`.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="server-audits"></a>Auditorías de servidor<a id="ServerAudits"></a>

**Título: Uso de las características de auditoría de Azure SQL Database para reemplazar las auditorías de servidor**   
**Categoría**: Advertencia   

**Descripción**   
Las auditorías de servidor no se admite en Azure SQL Database.


**Recomendación**   
Considere la posibilidad de usar las características de auditoría de Azure SQL Database para reemplazar las auditorías de servidor.  Azure SQL admite las auditorías y las características están más enriquecidas que en SQL Server. Azure SQL Database puede auditar varias acciones y eventos de base de datos, entre los que se incluyen el acceso a datos, los cambios de esquema (DDL), los cambios de datos (DML), las cuentas, roles y permisos (DCL) y las excepciones de seguridad. La auditoría de SQL Database aumenta la capacidad de la organización de obtener información sobre los eventos y los cambios que se producen en la base de datos, incluidas las actualizaciones y las consultas efectuadas en los datos. También puede migrar a Azure SQL Managed Instance o SQL Server en la máquina virtual de Azure.

Más información: [Auditoría de Azure SQL Database ](../../database/auditing-overview.md)

## <a name="server-credentials"></a>Credenciales de servidor<a id="ServerCredentials"></a>

**Título: No se admite la credencial con ámbito de servidor en Azure SQL Database**   
**Categoría**: Advertencia   

**Descripción**   
Una credencial es un registro que contiene la información de autenticación (credenciales) necesaria para conectarse a un recurso situado fuera de SQL Server. Azure SQL Database admite las credenciales de base de datos, pero no las creadas en el ámbito de SQL Server.   


**Recomendación**   
Azure SQL Database admite las credenciales con ámbito de base de datos. Convierta las credenciales con ámbito de servidor en credenciales con ámbito de base de datos. También puede migrar a Azure SQL Managed Instance o SQL Server en la máquina virtual de Azure.

Más información: [Creación de credencial con ámbito de base de datos](/sql/t-sql/statements/create-database-scoped-credential-transact-sql)

## <a name="service-broker"></a>Service Broker<a id="ServiceBroker"></a>

**Título: La característica Service Broker no se admite en Azure SQL Database**   
**Categoría**: Problema   

**Descripción**   
SQL Server Service Broker proporciona compatibilidad nativa de aplicaciones de mensajería y de puesta en cola en el Motor de base de datos de SQL Server. La característica Service Broker no se admite en Azure SQL Database.


**Recomendación**   
La característica Service Broker no se admite en Azure SQL Database. Considere la posibilidad de migrar a Azure SQL Managed Instance que admite Service Broker dentro de la misma instancia. También puede migrar a SQL Server en la máquina virtual de Azure. 

## <a name="server-scoped-triggers"></a>Desencadenadores de ámbito de servidor<a id="ServerScopedTriggers"></a>

**Título: No se admiten desencadenadores con ámbito de servidor en Azure SQL Database**   
**Categoría**: Advertencia   

**Descripción**   
Un desencadenador es un tipo especial de procedimiento almacenado que se ejecuta en respuesta a una determinada acción en una tabla, como la inserción, la eliminación o la actualización de datos. No se admiten desencadenadores con ámbito de servidor en Azure SQL Database. Azure SQL Database no admite las siguientes opciones para los desencadenadores: FOR LOGON, ENCRYPTION, WITH APPEND, NOT FOR REPLICATION, EXTERNAL NAME (no hay compatibilidad con el método externo), ALL SERVER (desencadenador DDL), desencadenar en un evento LOGON (desencadenador de inicio de sesión), Azure SQL Database no admite desencadenadores CLR.


**Recomendación**   
Use desencadenadores de nivel de base de datos en su lugar. También puede migrar a Azure SQL Managed Instance o SQL Server en la máquina virtual de Azure.

Más información: [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](../../database/transact-sql-tsql-differences-sql-server.md#transact-sql-syntax-not-supported-in-azure-sql-database)


## <a name="sql-agent-jobs"></a>Trabajos del Agente SQL<a id="AgentJobs"></a>

**Título: Los trabajos del Agente SQL Server no están disponibles en Azure SQL Database**   
**Categoría**: Advertencia   

**Descripción**   
El Agente SQL Server es un servicio de Microsoft Windows que ejecuta tareas administrativas programadas, denominadas trabajos, en SQL Server. Los trabajos del Agente SQL Server no están disponibles en Azure SQL Database. 


**Recomendación** Use trabajos elásticos (versión preliminar), que son el reemplazo de los trabajos de Agente SQL Server en Azure SQL Database. Los trabajos de Base de datos elástica para Azure SQL Database permite ejecutar de forma confiable scripts de T-SQL que abarcan varias bases de datos al tiempo que realizan reintentos automáticos y ofrecen garantías de finalización futura. También puede considerar la posibilidad de migrar a Azure SQL Managed Instance o SQL Server en Azure Virtual Machines.

Más información: [Introducción a trabajos de Base de datos elástica (versión preliminar) ](../../database/elastic-jobs-overview.md)

## <a name="sql-database-size"></a>Tamaño de SQL Database<a id="SQLDBDatabaseSize"></a>

**Título: Azure SQL Database no admite un tamaño de base de datos superior a 100 TB.**    
**Categoría**: Problema   

**Descripción**   
El tamaño de la base de datos es mayor que el tamaño máximo admitido de 100 TB. 


**Recomendación**   
Evalúe si los datos se pueden archivar o comprimir en varias bases de datos. También puede migrar a SQL Server en la máquina virtual de Azure. 

Más información: [Límites de recursos de núcleo virtual](../../database/resource-limits-vcore-single-databases.md) 

## <a name="sql-mail"></a>SQL Mail<a id="SqlMail"></a>

**Título: SQL Mail se ha descontinuado.**    
**Categoría**: Advertencia   

**Descripción**   
SQL Mail se ha descontinuado y quitado de Azure SQL Database.


**Recomendación**   
Considere la posibilidad de migrar a Azure SQL Managed Instance o SQL Server en Azure Virtual Machines y usar el Correo electrónico de base de datos.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="systemprocedures110"></a>SystemProcedures110<a id="SystemProcedures110"></a>

**Título: Se han detectado instrucciones que hacen referencia a procedimientos almacenados del sistema que se han quitado y no están disponibles en Azure SQL Database.**    
**Categoría**: Advertencia   

**Descripción**   
No se pueden usar en Azure SQL Database los siguientes procedimientos almacenados extendidos y del sistema no admitidos: `sp_dboption`, `sp_addserver`, `sp_dropalias`, `sp_activedirectory_obj`, `sp_activedirectory_scp` y `sp_activedirectory_start`.


**Recomendación**    
Quite las referencias a los procedimientos del sistema no admitidos que se han quitado de Azure SQL Database.

Más información: [Funcionalidad del motor de base de datos descontinuada en SQL Server](/previous-versions/sql/2014/database-engine/discontinued-database-engine-functionality-in-sql-server-2016#Denali)

## <a name="trace-flags"></a>Marcas de seguimiento<a id="TraceFlags"></a>

**Título: Azure SQL Database no admite marcas de seguimiento**   
**Categoría**: Advertencia   

**Descripción**   
Las marcas de seguimiento se utilizan para establecer temporalmente características específicas del servidor o para desactivar un comportamiento determinado. Las marcas de seguimiento se suelen utilizar para diagnosticar problemas de rendimiento o para depurar procedimientos almacenados o sistemas complejos. Azure SQL Database no admite marcas de seguimiento. 


**Recomendación**   
Revise la sección de objetos afectados en Azure Migrate para ver todas las marcas de seguimiento que no se admiten en Azure SQL Database y evaluar si se pueden quitar. También puede migrar a Azure SQL Managed Instance que admite un número limitado de marcas de seguimiento globales o SQL Server en una máquina virtual de Azure.

Más información: [Resolución de diferencias de Transact-SQL durante la migración a SQL Database](../../database/transact-sql-tsql-differences-sql-server.md#transact-sql-syntax-not-supported-in-azure-sql-database)


## <a name="windows-authentication"></a>Autenticación de Windows<a id="WindowsAuthentication"></a>

**Título: Los usuarios de base de datos asignados con la autenticación de Windows (seguridad integrada) no se admiten en Azure SQL Database.**    
**Categoría**: Advertencia   

**Descripción**   
Azure SQL Database admite dos tipos de autenticación 
- Autenticación de SQL: usa un nombre de usuario y una contraseña. 
- Autenticación de Azure Active Directory: usa las identidades administradas por Azure Active Directory y es compatible con dominios administrados e integrados. 

Los usuarios de base de datos asignados con la autenticación de Windows (seguridad integrada) no se admiten en Azure SQL Database. 



**Recomendación**   
Federe la instancia local de Active Directory con Azure Active Directory. La identidad de Windows se puede reemplazar por las identidades de Azure Active Directory equivalentes. También puede migrar a SQL Server en la máquina virtual de Azure.

Más información: [Funcionalidades de seguridad de SQL Database](../../database/security-overview.md#authentication)

## <a name="xp_cmdshell"></a>XP_cmdshell<a id="XpCmdshell"></a>

**Título: no se admite xp_cmdshell en Azure SQL Database.**    
**Categoría**: Problema   

**Descripción**   
xp_cmdshell, que genera un shell de comandos de Windows y pasa una cadena para su ejecución, no se admite en Azure SQL Database. 


**Recomendación**   
Revise la sección de objetos afectados en Azure Migrate para ver todos los objetos que usan xp_cmdshell y evaluar si se puede quitar la referencia a xp_cmdshell o al objeto afectado. Considere también la posibilidad de explorar Azure Automation que ofrece un servicio de automatización y configuración basado en la nube. También puede migrar a SQL Server en la máquina virtual de Azure. 

## <a name="next-steps"></a>Pasos siguientes

Para empezar a migrar la instancia de SQL Server a Azure SQL Database, consulte la [guía de migración de SQL Server a SQL Database](sql-server-to-sql-database-guide.md).

- Para obtener una matriz de los servicios y las herramientas de Microsoft y de otros fabricantes que están disponibles para ayudarlo en diversos escenarios de migración de datos y bases de datos, además de las tareas especializadas, consulte [Servicios y herramientas de migración de datos](../../../dms/dms-tools-matrix.md).

- Para más información sobre SQL Database, consulte:
   - [Información general sobre Azure SQL Database](../../database/sql-database-paas-overview.md)
   - [Calculadora del costo total de propiedad de Azure](https://azure.microsoft.com/pricing/tco/calculator/) 

- Para más información sobre el marco y el ciclo de adopción de las migraciones en la nube, consulte:
   -  [Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Procedimientos recomendados para la gestión de los costos y los ajustes de tamaño de las cargas de trabajo migradas a Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 


- Para evaluar el nivel de acceso de la aplicación, consulte [Data Access Migration Toolkit (versión preliminar)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- Para más información sobre cómo realizar pruebas A/B en la capa de acceso a datos, consulte [Información general del Asistente para experimentación con bases de datos](/sql/dea/database-experimentation-assistant-overview).
