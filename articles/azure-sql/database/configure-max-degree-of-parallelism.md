---
title: Configuración del grado máximo de paralelismo (MAXDOP)
titleSuffix: Azure SQL Database
description: Obtenga información sobre el grado máximo de paralelismo (MAXDOP).
ms.date: 04/12/2021
services: sql-database
dev_langs:
- TSQL
ms.service: sql-database
ms.subservice: performance
ms.custom: ''
ms.devlang: tsql
ms.topic: conceptual
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: ''
ms.openlocfilehash: 774114a27e5bcb23bc3cdddc08f5d42b3c43bb36
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108132046"
---
# <a name="configure-the-max-degree-of-parallelism-maxdop-in-azure-sql-database"></a>Configuración del grado máximo de paralelismo (MAXDOP) en Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

  En este artículo se describe el valor de configuración del **grado máximo de paralelismo (MAXDOP)** en Azure SQL Database. 

> [!NOTE]
> **Este contenido está orientado a Azure SQL Database.** Azure SQL Database se basa en la versión estable más reciente del motor de base de datos de Microsoft SQL Server, por lo que gran parte del contenido es similar aunque las opciones de solución de problemas y configuración pueden diferir. Para más información sobre MAXDOP en SQL Server, consulte [Establecer la opción de configuración del servidor Grado máximo de paralelismo](/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option).

## <a name="overview"></a>Información general
  MAXDOP controla el paralelismo dentro de la consulta en el motor de base de datos. Los valores MAXDOP más altos suelen dar lugar a más subprocesos paralelos por consulta y a una ejecución de consultas más rápida. 

  En Azure SQL Database, el valor predeterminado de MAXDOP para cada nueva base de datos única y base de datos de grupo elástico es de 8. Este valor predeterminado evita el uso de recursos innecesario, a la vez que permite al motor de base de datos ejecutar consultas con mayor rapidez mediante subprocesos paralelos. Por lo general, no es necesario hacer ajustes adicionales en el valor de MAXDOP en las cargas de trabajo de Azure SQL Database, a pesar de que puede otorgar ventajas como ejercicio de ajuste de rendimiento avanzado.

> [!Note]
>   En septiembre de 2020, basándose en años de telemetría en el servicio Azure SQL Database, MAXDOP 8 se convirtió en el [valor predeterminado para las nuevas bases de datos](https://techcommunity.microsoft.com/t5/azure-sql/changing-default-maxdop-in-azure-sql-database-and-azure-sql/ba-p/1538528) por ser el valor óptimo para la mayor variedad de cargas de trabajo de los clientes. Este valor predeterminado ayudó a evitar problemas de rendimiento debido a un paralelismo excesivo. Previamente, el valor predeterminado para las nuevas bases de datos era MAXDOP 0. MAXDOP no se modificó automáticamente para las bases de datos existentes creadas antes de septiembre de 2020.

  En general, si el motor de base de datos decide ejecutar una consulta mediante paralelismo, el tiempo de ejecución es más rápido. Sin embargo, el exceso de paralelismo puede consumir recursos adicionales del procesador sin mejorar el rendimiento de las consultas. A escala, el exceso de paralelismo puede afectar negativamente al rendimiento de las consultas para todas las consultas que se ejecutan en la misma instancia del motor de base de datos. Tradicionalmente, establecer un límite superior para el paralelismo ha sido un ejercicio de optimización del rendimiento común en las cargas de trabajo de SQL Server.

  En la tabla siguiente se describe el comportamiento del motor de base de datos al ejecutar consultas con valores MAXDOP distintos:

| MAXDOP | Comportamiento | 
|--|--|
| = 1 | El motor de base de datos usa un único subproceso en serie para ejecutar consultas. No se usan subprocesos paralelos. | 
| > 1 | El motor de base de datos establece el número de [programadores](/sql/relational-databases/thread-and-task-architecture-guide#sql-server-task-scheduling) adicionales que usarán los subprocesos paralelos en el valor MAXDOP, o el número total de procesadores lógicos, el que sea menor. |
| = 0 | El motor de base de datos establece el número de [programadores](/sql/relational-databases/thread-and-task-architecture-guide#sql-server-task-scheduling) adicionales que usarán los subprocesos paralelos en el número total de procesadores lógicos o en 64, el que sea menor. | 
| | |

> [!Note]
> Cada consulta se ejecuta con al menos un programador y un subproceso de trabajo en dicho programador.
>
> Una consulta que se ejecuta con paralelismo usa programadores adicionales y subprocesos paralelos adicionales. Dado que varios subprocesos paralelos se pueden ejecutar en el mismo programador, el número total de subprocesos usados para ejecutar una consulta puede ser mayor que el valor MAXDOP especificado o que el número total de procesadores lógicos. Para más información, consulte [Programación de tareas paralelas](/sql/relational-databases/thread-and-task-architecture-guide#scheduling-parallel-tasks).

##  <a name="considerations"></a><a name="Considerations"></a> Consideraciones  

-   En Azure SQL Database, puede cambiar el valor predeterminado de MAXDOP:
    -   En el nivel de consulta, mediante la [sugerencia de consulta](/sql/t-sql/queries/hints-transact-sql-query) **MAXDOP**.     
    -   En el nivel de base de datos, con la [configuración con ámbito de base de datos](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) **MAXDOP**.

-   Las consideraciones y [recomendaciones](/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option#Guidelines) para MAXDOP de SQL Server de siempre son aplicables a Azure SQL Database. 

-   Las operaciones de índice que crean o vuelven a generar un índice, o que eliminan un índice clúster, pueden consumir recursos de forma intensiva. Puede omitir el valor MAXDOP en la base de datos para operaciones de índice si especifica la opción de índice MAXDOP en la instrucción `CREATE INDEX` o `ALTER INDEX`. El valor de MAXDOP se aplica a la instrucción en tiempo de ejecución y no se almacena en los metadatos del índice. Para obtener más información, vea [Configurar operaciones de índice en paralelo](/sql/relational-databases/indexes/configure-parallel-index-operations).  
  
-   Además de las operaciones de consultas e índices, la opción de configuración con ámbito de la base de datos para MAXDOP también controla el paralelismo de otras instrucciones que pueden usar la ejecución en paralelo, como DBCC CHECKTABLE, DBCC CHECKDB y DBCC CHECKFILEGROUP. 

##  <a name="recommendations"></a><a name="Recommendations"></a> Recomendaciones  

  El cambio de MAXDOP para la base de datos puede afectar de manera significativa al rendimiento de las consultas y la utilización de los recursos, tanto de manera positiva como negativa. Sin embargo, no hay ningún valor de MAXDOP único que sea óptimo para todas las cargas de trabajo. Las [recomendaciones](/sql/database-engine/configure-windows/configure-the-max-degree-of-parallelism-server-configuration-option#Guidelines) para establecer MAXDOP están matizadas y dependen de muchos factores. 

  Algunos picos de cargas de trabajo simultáneas pueden funcionar mejor con un valor MAXDOP diferente que otros. Un valor MAXDOP configurado correctamente debe reducir el riesgo de incidentes de rendimiento y disponibilidad, y en algunos casos puede reducir los costos al poder evitar el uso de recursos innecesarios y, por tanto, reducir verticalmente a un objetivo de servicio inferior.

### <a name="excessive-parallelism"></a>Paralelismo excesivo

  Un valor MAXDOP más alto suele reducir la duración de las consultas con un uso intensivo de la CPU. Sin embargo, un paralelismo excesivo puede empeorar el rendimiento de otras cargas de trabajo simultáneas al privar a otras consultas de recursos de CPU y subprocesos de trabajo. En casos extremos, el paralelismo excesivo puede usar todos los recursos de la base de datos o del grupo elástico, lo que provoca tiempos de espera para las consultas, errores e interrupciones de la aplicación. 

> [!Tip]
> Se recomienda que los clientes eviten configurar un MAXDOP de 0, incluso si no parece que se produzcan incidencias en la actualidad.

  Un paralelismo excesivo se vuelve más problemático cuando hay más solicitudes simultáneas de las que pueden admitir el CPU y los recursos de subproceso que proporciona el objetivo de servicio. Evite el uso de MAXDOP 0 para reducir el riesgo de posibles problemas futuros debido a un paralelismo excesivo si una base de datos se escala verticalmente, o si las futuras generaciones de hardware en Azure SQL Database proporcionan más núcleos para el mismo objetivo de servicio de base de datos.

### <a name="modifying-maxdop"></a>Modificación de MAXDOP 

  Si determina que una configuración de MAXDOP diferente del valor predeterminado es óptima para la carga de trabajo de Azure SQL Database, puede usar la instrucción T-SQL `ALTER DATABASE SCOPED CONFIGURATION`. Para obtener ejemplos, vea la sección [Ejemplos de uso de Transact-SQL](#examples) más adelante. Para cambiar MAXDOP a un valor no predeterminado para cada nueva base de datos que cree, agregue este paso al proceso de implementación de la base de datos.

  Si el valor MAXDOP no predeterminado solo beneficia a un pequeño subconjunto de consultas en la carga de trabajo, puede invalidar el valor MAXDOP en el nivel de consulta al agregar la sugerencia OPTION (MAXDOP). Para obtener ejemplos, vea la sección [Ejemplos de uso de Transact-SQL](#examples) más adelante. 

  Pruebe a fondo sus cambios de configuración de MAXDOP con pruebas de carga que incluyan cargas de consultas simultáneas realistas. 

  MAXDOP para las réplicas principal y secundaria se puede configurar de forma independiente si las configuraciones de MAXDOP diferentes resultan óptimas para las cargas de trabajo de lectura-escritura y de solo lectura. Esto se aplica al [escalado horizontal de lectura](read-scale-out.md) de Azure SQL Database, la [replicación geográfica](active-geo-replication-overview.md) y las [réplicas secundarias de hiperescala](service-tier-hyperscale.md). De forma predeterminada, todas las réplicas secundarias heredan la configuración de MAXDOP de la réplica principal.

## <a name="security"></a><a name="Security"></a> Seguridad  
  
###  <a name="permissions"></a><a name="Permissions"></a> Permisos  
  La instrucción `ALTER DATABASE SCOPED CONFIGURATION` se debe ejecutar como administrador del servidor, como miembro del rol de la base de datos `db_owner` o como un usuario al que se le ha concedido el permiso `ALTER ANY DATABASE SCOPED CONFIGURATION`.
 
## <a name="examples"></a>Ejemplos   

  En estos ejemplos se usa la base de datos de ejemplo **AdventureWorksLT** más reciente cuando se elige la opción `SAMPLE` para una nueva base de datos única de Azure SQL Database.

### <a name="powershell"></a>PowerShell

#### <a name="maxdop-database-scoped-configuration"></a>Configuración con ámbito de la base de datos de MAXDOP   

  En este ejemplo se muestra cómo usar la instrucción [ALTER DATABASE SCOPED CONFIGURATION](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) para establecer la configuración `MAXDOP` en `2`. Esta configuración surte efecto inmediatamente para las nuevas consultas. El cmdlet [Invoke-SqlCmd](/powershell/module/sqlserver/invoke-sqlcmd) de PowerShell ejecuta las consultas T-SQL para establecer y devolver la configuración con ámbito de la base de datos MAXDOP. 

```powershell
$dbName = "sample" 
$serverName = <server name here>
$serveradminLogin = <login here>
$serveradminPassword = <password here>
$desiredMAXDOP = 8

$params = @{
    'database' = $dbName
    'serverInstance' =  $serverName
    'username' = $serveradminLogin
    'password' = $serveradminPassword
    'outputSqlErrors' = $true
    'query' = 'ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = ' + $desiredMAXDOP + ';
     SELECT [value] FROM sys.database_scoped_configurations WHERE [name] = ''MAXDOP'';'
  }
  Invoke-SqlCmd @params
```

Este ejemplo se usa con las bases de datos de Azure SQL Database con [réplicas de escalado horizontal de lectura habilitadas](read-scale-out.md), [replicación geográfica](active-geo-replication-overview.md) y [réplicas secundarias de hiperescala de Azure SQL Database](service-tier-hyperscale.md). Por ejemplo, la réplica principal se establece en un valor MAXDOP predeterminado diferente al de la réplica secundaria, con lo que se prevé que puede haber diferencias entre una carga de trabajo de lectura-escritura y una de solo lectura.

```powershell
$dbName = "sample" 
$serverName = <server name here>
$serveradminLogin = <login here>
$serveradminPassword = <password here>
$desiredMAXDOP_primary = 8
$desiredMAXDOP_secondary_readonly = 1
 
$params = @{
    'database' = $dbName
    'serverInstance' =  $serverName
    'username' = $serveradminLogin
    'password' = $serveradminPassword
    'outputSqlErrors' = $true
    'query' = 'ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = ' + $desiredMAXDOP + ';
    ALTER DATABASE SCOPED CONFIGURATION FOR SECONDARY SET MAXDOP = ' + $desiredMAXDOP_secondary_readonly + ';
    SELECT [value], value_for_secondary FROM sys.database_scoped_configurations WHERE [name] = ''MAXDOP'';'
  }
  Invoke-SqlCmd @params
```

### <a name="transact-sql"></a>Transact-SQL
  
  Puede usar el [editor de consultas de Azure Portal](connect-query-portal.md), [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) o [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio) para ejecutar consultas de T-SQL en la instancia de Azure SQL Database.

1.  Abra una nueva ventana de consulta.

2.  Conéctese a la base de datos en la que quiere cambiar el valor de MAXDOP. No se puede cambiar la configuración de ámbito de base de datos en la base de datos maestra.
  
3.  Copie y pegue el ejemplo siguiente en la ventana de consulta y seleccione **Ejecutar**. 

#### <a name="maxdop-database-scoped-configuration"></a>Configuración con ámbito de la base de datos de MAXDOP

  Este ejemplo muestra cómo determinar la configuración actual con ámbito de la base de datos MAXDOP utilizando la vista del catálogo del sistema [sys.database_scoped_configurations](/sql/relational-databases/system-catalog-views/sys-database-scoped-configurations-transact-sql).

```sql
SELECT [value] FROM sys.database_scoped_configurations WHERE [name] = 'MAXDOP';
```

  En este ejemplo se muestra cómo usar la instrucción [ALTER DATABASE SCOPED CONFIGURATION](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql) para establecer la configuración `MAXDOP` en `8`. La configuración surte efecto inmediatamente.  
  
```sql  
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 8;
```  

Este ejemplo se usa con las bases de datos de Azure SQL Database con [réplicas de escalado horizontal de lectura habilitadas](read-scale-out.md), [replicación geográfica](active-geo-replication-overview.md) y réplicas secundarias de [hiperescala](service-tier-hyperscale.md). Por ejemplo, la réplica principal está establecida en un valor MAXDOP diferente al de la réplica secundaria, con lo que se prevé que puede haber diferencias entre las carga de trabajo de lectura-escritura y las de solo lectura. Todas las instrucciones se ejecutan en la réplica principal. La columna `value_for_secondary` de `sys.database_scoped_configurations` contiene los valores de la réplica secundaria.

```sql
ALTER DATABASE SCOPED CONFIGURATION SET MAXDOP = 8;
ALTER DATABASE SCOPED CONFIGURATION FOR SECONDARY SET MAXDOP = 1;
SELECT [value], value_for_secondary FROM sys.database_scoped_configurations WHERE [name] = 'MAXDOP';
```

#### <a name="maxdop-query-hint"></a>MAXDOP, sugerencia de consulta

  En este ejemplo se muestra cómo ejecutar una consulta con la sugerencia de consulta para forzar `max degree of parallelism` a `2`.  

```sql 
SELECT ProductID, OrderQty, SUM(LineTotal) AS Total  
FROM SalesLT.SalesOrderDetail  
WHERE UnitPrice < 5  
GROUP BY ProductID, OrderQty  
ORDER BY ProductID, OrderQty  
OPTION (MAXDOP 2);    
GO
```
#### <a name="maxdop-index-option"></a>MAXDOP, opción de índice

  Este ejemplo muestra cómo recompilar un índice utilizando la opción de índice para forzar `max degree of parallelism` a `12`.  

```sql 
ALTER INDEX ALL ON SalesLT.SalesOrderDetail 
REBUILD WITH 
   (     MAXDOP = 12
       , SORT_IN_TEMPDB = ON
       , ONLINE = ON);
```

## <a name="see-also"></a>Consulte también  
* [ALTER DATABASE SCOPED CONFIGURATION &#40;Transact-SQL&#41;](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)        
* [sys.database_scoped_configurations (Transact-SQL)](/sql/relational-databases/system-catalog-views/sys-database-scoped-configurations-transact-sql)
* [Configurar operaciones de índice en paralelo](/sql/relational-databases/indexes/configure-parallel-index-operations)    
* [Sugerencias de consulta &#40;Transact-SQL&#41;](/sql/t-sql/queries/hints-transact-sql-query)     
* [Establecer opciones de índice](/sql/relational-databases/indexes/set-index-options)     
* [Descripción y resolución de problemas de bloqueo en Azure SQL Database](understand-resolve-blocking.md)

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión y optimización del rendimiento](/sql/relational-databases/performance/monitor-and-tune-for-performance)