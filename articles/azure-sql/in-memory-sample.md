---
title: Ejemplo en memoria
description: Pruebe las tecnologías en memoria de Azure SQL Database con el ejemplo de almacén de columnas y OLTP.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: sample
author: srinia
ms.author: srinia
ms.reviewer: mathoma
ms.date: 12/18/2018
ms.openlocfilehash: d0070c08daccd3762977c03f645b145e8b22fa48
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111592311"
---
# <a name="in-memory-sample"></a>Ejemplo en memoria
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

Las tecnologías en memoria de Azure SQL Database permiten mejorar el rendimiento de la aplicación y reducen el costo de la base de datos. Mediante el uso de las tecnologías en memoria de Azure SQL Database, puede lograr mejoras de rendimiento con diversas cargas de trabajo.

En este artículo se muestran dos ejemplos que ilustran el uso de OLTP en memoria y de los índices del almacén de columnas en Azure SQL Database.

Para más información, consulte:

- [Información general y escenarios de uso de OLTP en memoria](/sql/relational-databases/in-memory-oltp/overview-and-usage-scenarios) (incluye referencias a información y casos prácticos de clientes para familiarizarse)
- [Documentación de In-Memory OLTP](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)
- [Descripción de los índices de almacén de columnas](/sql/relational-databases/indexes/columnstore-indexes-overview)
- Procesamiento analítico y transaccional híbrido (HTAP), también conocido como [análisis operativo en tiempo real](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

<a id="install_oltp_manuallink" name="install_oltp_manuallink"></a>

&nbsp;

## <a name="1-install-the-in-memory-oltp-sample"></a>1. Instalación del ejemplo de In-Memory OLTP.

La base de datos de ejemplo AdventureWorksLT se puede crear con unos pocos clics en el [Azure Portal](https://portal.azure.com/). Los pasos descritos en esta sección explican cómo puede enriquecer la base de datos AdventureWorksLT con objetos de OLTP en memoria y muestran las ventajas de rendimiento.

Si desea ver una demostración más simple, pero más atractiva visualmente, del rendimiento de OLTP en memoria, consulte:

- Versión: [in-memory-oltp-demo-v1.0](https://github.com/Microsoft/sql-server-samples/releases/tag/in-memory-oltp-demo-v1.0)
- Código fuente: [in-memory-oltp-demo-source-code](https://github.com/microsoft/sql-server-samples/tree/master/samples/features/in-memory-database)

### <a name="installation-steps"></a>Pasos de instalación

1. En [Azure Portal](https://portal.azure.com/), cree una base de datos de nivel Premium o Crítico para la empresa en un servidor. Establezca el **Origen** en la base de datos de ejemplo de AdventureWorksLT. Para obtener instrucciones detalladas, consulte [Creación de la primera base de datos de Azure SQL Database](database/single-database-create-quickstart.md).

2. Conéctese a la base de datos con SQL Server Management Studio [(SSMS.exe)](/sql/ssms/download-sql-server-management-studio-ssms).

3. Copie el [script Transact-SQL de In-Memory OLTP](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_oltp_sample.sql) en el Portapapeles. El script T-SQL crea los objetos en memoria necesarios en la base de datos de ejemplo AdventureWorksLT que se creó en el paso 1.

4. Pegue el script T-SQL en SSMS.exe y, luego, ejecútelo. Es crucial la instrucción CREATE TABLE de la cláusula `MEMORY_OPTIMIZED = ON`. Por ejemplo:

```sql
CREATE TABLE [SalesLT].[SalesOrderHeader_inmem](
    [SalesOrderID] int IDENTITY NOT NULL PRIMARY KEY NONCLUSTERED ...,
    ...
) WITH (MEMORY_OPTIMIZED = ON);
```

### <a name="error-40536"></a>Error 40536

Si recibe el error 40536 cuando ejecuta el script T-SQL, ejecute el siguiente script de T-SQL para comprobar que la base de datos admite In-Memory:

```sql
SELECT DatabasePropertyEx(DB_Name(), 'IsXTPSupported');
```

Un resultado de **0** significa que no se admite en memoria, mientras que un resultado de **1** significa que se admite. Para diagnosticar el problema, asegúrese de que la base de datos se encuentra en el nivel de servicio Premium.

### <a name="about-the-created-memory-optimized-items"></a>Acerca de los elementos creados optimizados para memoria

**Tablas**: el ejemplo contiene las siguientes tablas optimizadas para memoria:

- SalesLT.Product_inmem
- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem
- Demo.DemoSalesOrderHeaderSeed
- Demo.DemoSalesOrderDetailSeed

Puede inspeccionar las tablas optimizadas para memoria a través del **Explorador de objetos** en SSMS. Haga clic con el botón derecho en **Tablas** > **Filtro** > **Configuración de filtro** > **Con optimización para memoria**. El valor es igual a 1.

O bien puede consultar las vistas de catálogo como:

```sql
SELECT is_memory_optimized, name, type_desc, durability_desc
    FROM sys.tables
    WHERE is_memory_optimized = 1;
```

**Procedimientos almacenados compilados de forma nativa**: puede inspeccionar SalesLT.usp_InsertSalesOrder_inmem mediante una consulta de la vista de catálogo:

```sql
SELECT uses_native_compilation, OBJECT_NAME(object_id), definition
    FROM sys.sql_modules
    WHERE uses_native_compilation = 1;
```

&nbsp;

### <a name="run-the-sample-oltp-workload"></a>Ejecución de la carga de trabajo de OLTP de ejemplo

La única diferencia entre los dos *procedimientos almacenados* siguientes es que el primer procedimiento usa las versiones de las tablas optimizadas para memoria, mientras que el segundo procedimiento usa las tablas en disco habituales:

- SalesLT **.** usp_InsertSalesOrder **_inmem**
- SalesLT **.** usp_InsertSalesOrder **_ondisk**

En esta sección verá cómo usar la práctica utilidad **ostress.exe** para ejecutar los dos procedimientos almacenados a niveles de esfuerzo. Puede comparar el tiempo que tardan en completarse las dos ejecuciones de esfuerzo.

Cuando se ejecuta ostress.exe, le recomendamos pasar los valores de parámetros diseñados para estos dos casos:

- Ejecute un gran número de conexiones simultáneas, mediante el uso de -n100.
- Haga que cada conexión se repita en bucle cientos de veces, mediante el uso de -r500.

Pero es posible que quiera comenzar con valores mucho más pequeños, como -n10 y -r50, para garantizar que todo está funcionando.

### <a name="script-for-ostressexe"></a>Script para ostress.exe

En esta sección se muestra el script de T-SQL que se inserta en la línea de comandos de ostress.exe. El script usa elementos creados por el script de T-SQL que instaló antes.

El siguiente script inserta un pedido de ventas de ejemplo con cinco elementos de línea en las siguientes *tablas* optimizadas para memoria:

- SalesLT.SalesOrderHeader_inmem
- SalesLT.SalesOrderDetail_inmem

```sql
DECLARE
    @i int = 0,
    @od SalesLT.SalesOrderDetailType_inmem,
    @SalesOrderID int,
    @DueDate datetime2 = sysdatetime(),
    @CustomerID int = rand() * 8000,
    @BillToAddressID int = rand() * 10000,
    @ShipToAddressID int = rand() * 10000;

INSERT INTO @od
    SELECT OrderQty, ProductID
    FROM Demo.DemoSalesOrderDetailSeed
    WHERE OrderID= cast((rand()*60) as int);

WHILE (@i < 20)
begin;
    EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT,
        @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od;
    SET @i = @i + 1;
end
```

Para hacer que la versión *_ondisk* del script T-SQL anterior sirva para ostress.exe, hay que reemplazar ambas repeticiones de la subcadena *_inmem* por *_ondisk*. Estos reemplazos afectan a los nombres de tablas y procedimientos almacenados.

#### <a name="install-rml-utilities-and-ostress"></a>Instalación de `ostress` y de utilidades de RML

Lo ideal sería planear la ejecución de ostress.exe en una máquina virtual de Azure (VM). Se crearía una [VM de Azure](https://azure.microsoft.com/documentation/services/virtual-machines/) en la misma región geográfica de Azure en que reside la base de datos AdventureWorksLT. Pero puede ejecutar si lo desea ostress.exe en el equipo portátil.

En la VM, o en cualquier host que elija, instale las utilidades de Replay Markup Language (RML). Las utilidades incluyen ostress.exe.

Para más información, consulte:

- La explicación de ostress.exe en [Base de datos de ejemplo para OLTP en memoria](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- [Base de datos de ejemplo para OLTP en memoria](/sql/relational-databases/in-memory-oltp/sample-database-for-in-memory-oltp).
- El [blog para instalar ostress.exe](https://techcommunity.microsoft.com/t5/sql-server-support/cumulative-update-2-to-the-rml-utilities-for-microsoft-sql/ba-p/317910).

<!--
dn511655.aspx is for SQL 2014,
[Extensions to AdventureWorks to Demonstrate In-Memory OLTP]
(https://msdn.microsoft.com/library/dn511655&#x28;v=sql.120&#x29;.aspx)

whereas for SQL 2016+
[Sample Database for In-Memory OLTP]
(https://msdn.microsoft.com/library/mt465764.aspx)
-->

#### <a name="run-the-_inmem-stress-workload-first"></a>Ejecute de la carga de trabajo de esfuerzo *_inmem* en primer lugar

Puede usar una ventana *del símbolo del sistema de RML* para ejecutar la línea de comandos de ostress.exe. Los parámetros de línea de comandos dirigen `ostress` para:

- Ejecutar 100 conexiones simultáneamente (-n100).
- Hacer que cada conexión ejecute el script de T-SQL 50 veces (-r50).

```
ostress.exe -n100 -r50 -S<servername>.database.windows.net -U<login> -P<password> -d<database> -q -Q"DECLARE @i int = 0, @od SalesLT.SalesOrderDetailType_inmem, @SalesOrderID int, @DueDate datetime2 = sysdatetime(), @CustomerID int = rand() * 8000, @BillToAddressID int = rand() * 10000, @ShipToAddressID int = rand()* 10000; INSERT INTO @od SELECT OrderQty, ProductID FROM Demo.DemoSalesOrderDetailSeed WHERE OrderID= cast((rand()*60) as int); WHILE (@i < 20) begin; EXECUTE SalesLT.usp_InsertSalesOrder_inmem @SalesOrderID OUTPUT, @DueDate, @CustomerID, @BillToAddressID, @ShipToAddressID, @od; set @i += 1; end"
```

Para ejecutar la línea de comandos ostress.exe anterior:

1. Restablezca el contenido de los datos de la base de datos mediante la ejecución del siguiente comando en SSMS para eliminar todos los datos que se insertaron en las ejecuciones anteriores:

    ``` tsql
    EXECUTE Demo.usp_DemoReset;
    ```

2. Copie el texto de la línea de comandos ostress.exe anterior en el Portapapeles.

3. Reemplace el `<placeholders>` de los parámetros -S -U -P -d por los valores reales correctos.

4. Ejecute la línea de comandos modificada en una ventana del símbolo del sistema de RML.

#### <a name="result-is-a-duration"></a>El resultado es una duración

Cuando finaliza `ostress.exe`, escribe la duración de la ejecución como la última línea de la salida en la ventana de símbolo del sistema de RML. Por ejemplo, una ejecución de prueba más corta duró aproximadamente 1,5 minutos:

`11/12/15 00:35:00.873 [0x000030A8] OSTRESS exiting normally, elapsed time: 00:01:31.867`

#### <a name="reset-edit-for-_ondisk-then-rerun"></a>Restablezca y edite *_ondisk* y, después, vuelva a ejecutarlo

Una vez que tenga el resultado de la ejecución de *_inmem*, realice los pasos siguientes para la ejecución de *_ondisk*:

1. Restablezca la base de datos mediante la ejecución del siguiente comando en SSMS para eliminar todos los datos que insertó la ejecución anterior:

   ```sql
   EXECUTE Demo.usp_DemoReset;
   ```

2. Edite la línea de comandos de ostress.exe para reemplazar todos los *_inmem* con *_ondisk*.

3. Ejecute ostress.exe por segunda vez y capture el resultado de la duración.

4. Vuelva a restablecer la base de datos (para la eliminación responsable de lo que puede constituir una gran cantidad de datos de prueba).

#### <a name="expected-comparison-results"></a>Resultados de la comparación esperados

Las pruebas en memoria demostraron tener un rendimiento **9 veces** mejor en esta carga de trabajo simplista, con `ostress` ejecutándose en una VM de Azure ubicada en la misma región de Azure que la base de datos.

<a id="install_analytics_manuallink" name="install_analytics_manuallink"></a>

&nbsp;

## <a name="2-install-the-in-memory-analytics-sample"></a>2. Instalación del ejemplo de In-Memory Analytics.

En esta sección, compara los resultados de optimización de infraestructura y de estadísticas cuando usa un índice del almacén de columnas en lugar de un índice de árbol b tradicional.

Para realizar análisis en tiempo real en una carga de trabajo de OLTP, suele ser mejor usar un índice del almacén de columnas no agrupado. Para obtener más información, consulte [Guía de índices de almacén de columnas](/sql/relational-databases/indexes/columnstore-indexes-overview).

### <a name="prepare-the-columnstore-analytics-test"></a>Preparación de la prueba de análisis del almacén de columnas

1. Use el Portal de Azure para crear una nueva base de datos AdventureWorksLT a partir del ejemplo.
   - Use ese nombre exacto.
   - Elija un nivel de servicio Premium.

2. Copie [sql_in-memory_analytics_sample](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/sql_in-memory_analytics_sample.sql) en el Portapapeles.
   - El script T-SQL crea los objetos en memoria necesarios en la base de datos de ejemplo AdventureWorksLT que se creó en el paso 1.
   - El script crea la tabla de dimensiones y dos tablas de hechos. Las tablas de hechos se rellenan con 3,5 millones de filas cada una.
   - El script podría tardar 15 minutos en completarse.

3. Pegue el script T-SQL en SSMS.exe y, luego, ejecútelo. La palabra clave **COLUMNSTORE** es crucial en la instrucción **CREATE INDEX**, como en:<br/>`CREATE NONCLUSTERED COLUMNSTORE INDEX ...;`

4. Establezca AdventureWorksLT en un nivel de compatibilidad 130:<br/>`ALTER DATABASE AdventureworksLT SET compatibility_level = 130;`

    El nivel 130 no está directamente relacionado con las características de In-Memory. Pero el nivel 130 suele ofrecer un rendimiento de consultas más rápido que el nivel 120.

#### <a name="key-tables-and-columnstore-indexes"></a>Tablas e índices de almacén de columnas clave

- dbo.FactResellerSalesXL_CCI es una tabla con un índice de almacén de columnas agrupado que tiene una compresión avanzada a nivel de *datos*.

- dbo.FactResellerSalesXL_PageCompressed es una tabla con un índice agrupado equivalente normal, que se comprime solo a nivel de *página*.

#### <a name="key-queries-to-compare-the-columnstore-index"></a>Consultas cruciales para comparar el índice de almacén de columnas

Aquí hay [varios tipos de consultas de T-SQL que se pueden ejecutar](https://raw.githubusercontent.com/microsoft/sql-server-samples/master/samples/features/in-memory-database/in-memory-oltp/t-sql-scripts/clustered_columnstore_sample_queries.sql) para ver las mejoras de rendimiento. En el paso 2 del script T-SQL, preste atención a estas dos consultas. Difieren solo en una línea:

- `FROM FactResellerSalesXL_PageCompressed a`
- `FROM FactResellerSalesXL_CCI a`

Un índice de almacén de columnas en clúster se encuentra en la tabla FactResellerSalesXL\_CCI.

El siguiente fragmento de script de T-SQL imprime las estadísticas de IO y TIME para la consulta de cada tabla.

```sql
/*********************************************************************
Step 2 -- Overview
-- Page Compressed BTree table v/s Columnstore table performance differences
-- Enable actual Query Plan in order to see Plan differences when Executing
*/
-- Ensure Database is in 130 compatibility mode
ALTER DATABASE AdventureworksLT SET compatibility_level = 130
GO

-- Execute a typical query that joins the Fact Table with dimension tables
-- Note this query will run on the Page Compressed table, Note down the time
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO

SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_PageCompressed a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO
SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO


-- This is the same Prior query on a table with a clustered columnstore index CCI
-- The comparison numbers are even more dramatic the larger the table is (this is an 11 million row table only)
SET STATISTICS IO ON
SET STATISTICS TIME ON
GO
SELECT c.Year
    ,e.ProductCategoryKey
    ,FirstName + ' ' + LastName AS FullName
    ,count(SalesOrderNumber) AS NumSales
    ,sum(SalesAmount) AS TotalSalesAmt
    ,Avg(SalesAmount) AS AvgSalesAmt
    ,count(DISTINCT SalesOrderNumber) AS NumOrders
    ,count(DISTINCT a.CustomerKey) AS CountCustomers
FROM FactResellerSalesXL_CCI a
INNER JOIN DimProduct b ON b.ProductKey = a.ProductKey
INNER JOIN DimCustomer d ON d.CustomerKey = a.CustomerKey
Inner JOIN DimProductSubCategory e on e.ProductSubcategoryKey = b.ProductSubcategoryKey
INNER JOIN DimDate c ON c.DateKey = a.OrderDateKey
GROUP BY e.ProductCategoryKey,c.Year,d.CustomerKey,d.FirstName,d.LastName
GO

SET STATISTICS IO OFF
SET STATISTICS TIME OFF
GO
```

En una base de datos con el plan de tarifa P2, puede esperar un rendimiento nueve veces superior, aproximadamente, para esta consulta al usar el índice de almacén de columnas en clúster, en comparación con el índice tradicional. Con P15, puede esperar, aproximadamente, un rendimiento 57 veces superior gracias al uso de índice de almacén de columnas.

## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido 1: Tecnologías de OLTP en memoria para acelerar el rendimiento de Transact-SQL](/sql/relational-databases/in-memory-oltp/survey-of-initial-areas-in-in-memory-oltp)

- [Uso de OLTP en memoria para mejorar el rendimiento de las aplicaciones en Azure SQL](in-memory-oltp-configure.md)

- [Supervisión del almacenamiento de OLTP en memoria](in-memory-oltp-monitor-space.md)

## <a name="additional-resources"></a>Recursos adicionales

### <a name="deeper-information"></a>Información más detallada

- [Más información sobre cómo Quorum duplica cargas de trabajo clave de las bases de datos a la vez que reduce las DTU en un 70 % con OLTP en memoria en Azure SQL Database](https://customers.microsoft.com/story/quorum-doubles-key-databases-workload-while-lowering-dtu-with-sql-database)

- [In-Memory OLTP in Azure SQL Database Blog Post](https://azure.microsoft.com/blog/in-memory-oltp-in-azure-sql-database/) (OLTP en memoria en la entrada de Blog de base de datos de SQL de Azure)

- [Más información sobre OLTP en memoria](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)

- [Más información sobre los índices de almacén de columnas](/sql/relational-databases/indexes/columnstore-indexes-overview)

- [Más información sobre los análisis operativos en tiempo real](/sql/relational-databases/indexes/get-started-with-columnstore-for-real-time-operational-analytics)

- Consulte [Common Workload Patterns and Migration Considerations](/previous-versions/dn673538(v=msdn.10)) (Patrones de cargas de trabajo comunes y consideraciones de migración), que describe los patrones de carga de trabajo donde In-Memory OLTP normalmente proporciona importantes mejoras de rendimiento.

#### <a name="application-design"></a>Diseño de aplicación

- [In-Memory OLTP (optimización In-Memory)](/sql/relational-databases/in-memory-oltp/in-memory-oltp-in-memory-optimization)

- [Uso de OLTP en memoria para mejorar el rendimiento de las aplicaciones en Azure SQL](in-memory-oltp-configure.md)

#### <a name="tools"></a>Herramientas

- [Azure Portal](https://portal.azure.com/)

- [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms)

- [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt)