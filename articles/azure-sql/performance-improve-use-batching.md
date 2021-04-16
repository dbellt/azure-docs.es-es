---
title: Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones
description: El tema proporciona evidencia de que el procesamiento por lotes de las operaciones de base de datos mejora en gran medida la velocidad y la escalabilidad de las aplicaciones de Azure SQL Database e Instancia administrada de Azure SQL. Aunque estas técnicas de procesamiento por lotes funcionan para cualquier base de datos SQL, el artículo se centra en Azure.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: genemi
ms.date: 01/25/2019
ms.openlocfilehash: 7f45e7d1515f0d6fc4467b36d95242ef8697c75d
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105641395"
---
# <a name="how-to-use-batching-to-improve-azure-sql-database-and-azure-sql-managed-instance-application-performance"></a>Uso del procesamiento por lotes para mejorar el rendimiento de las aplicaciones de Azure SQL Database e Instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

El procesamiento de operaciones por lotes para Azure SQL Database e Instancia administrada de Azure SQL mejora notablemente el rendimiento y la escalabilidad de las aplicaciones. Para comprender las ventajas, la primera parte de este artículo trata algunos resultados de pruebas de ejemplo que comparan solicitudes por lotes y secuenciales a una base de datos de Azure SQL Database o Instancia administrada de Azure SQL. El resto del artículo muestra las técnicas, los escenarios y las consideraciones para ayudarlo a usar el procesamiento por lotes correctamente en las aplicaciones de Azure.

## <a name="why-is-batching-important-for-azure-sql-database-and-azure-sql-managed-instance"></a>Importancia del procesamiento por lotes para Azure SQL Database y Azure SQL Managed Instance

El procesamiento por lotes de las llamadas a un servicio remoto es una estrategia conocida para aumentar el rendimiento y la escalabilidad. Existen costos fijos de procesamiento para cualquier interacción con un servicio remoto, como la serialización, la transferencia de red y la deserialización. El empaquetado de muchas transacciones diferentes en un único lote minimiza estos costos.

En este artículo, deseamos examinar diversas estrategias y escenarios de procesamiento por lotes. Aunque estas estrategias también son importantes para las aplicaciones locales que usan SQL Server, existen varias razones para destacar el uso del procesamiento por lotes para Azure SQL Database e Instancia administrada de Azure SQL:

* Potencialmente la latencia de red al acceder a Azure SQL Database e Instancia administrada de Azure SQL es mayor, sobre todo si accede a Azure SQL Database o Instancia administrada de Azure SQL desde fuera del mismo centro de datos de Microsoft Azure.
* Las características para varios inquilinos de Azure SQL Database e Instancia administrada de Azure SQL suponen que la eficacia de la capa de acceso a datos se correlacione con la escalabilidad general de la base de datos. En respuesta a un uso superior a las cuotas predefinidas, Azure SQL Database e Instancia administrada de Azure SQL pueden reducir el rendimiento o responder con excepciones de limitación. Las eficiencias, como el procesamiento por lotes, le permiten hacer más trabajo antes de alcanzar estos límites.
* Además, el procesamiento por lotes es efectivo para las arquitecturas que usan varias bases de datos (particionamiento). La eficacia de su interacción con cada unidad de base de datos sigue siendo un factor clave para la escalabilidad global.

Una de las ventajas del uso de Azure SQL Database o Azure SQL Managed Instance es que no es necesario administrar los servidores que hospedan la base de datos. Sin embargo, esta infraestructura administrada conlleva también que se deban considerar las optimizaciones de la base de datos de manera diferente. Ya no puede intentar mejorar la infraestructura de hardware o de red de la base de datos, porque Microsoft Azure controla esos entornos. El principal aspecto que puede controlar es cómo la aplicación interactúa con Azure SQL Database e Instancia administrada de Azure SQL. El procesamiento por lotes es una de estas optimizaciones.

En la primera parte del artículo, se examinan diversas técnicas de procesamiento por lotes para aplicaciones .NET que usan Azure SQL Database o Instancia administrada de Azure SQL. En las dos últimas secciones, se explican las instrucciones y los escenarios de procesamiento por lotes.

## <a name="batching-strategies"></a>Estrategias de procesamiento por lotes

### <a name="note-about-timing-results-in-this-article"></a>Nota sobre los tiempos resultantes en este artículo

> [!NOTE]
> Los resultados no sirven para pruebas comparativas, sino que están diseñados para mostrar el **rendimiento relativo**. Los tiempos se basan en un promedio de un mínimo de 10 series de pruebas. Las operaciones son inserciones en una tabla vacía. Estas pruebas se midieron antes de V12 y no se corresponden necesariamente con el rendimiento que podría observarse en una base de datos V12 con los nuevos [niveles de servicio de DTU](database/service-tiers-dtu.md) o los [niveles de servicio de núcleos virtuales](database/service-tiers-vcore.md). La ventaja relativa de la técnica de procesamiento por lotes debería ser semejante.

### <a name="transactions"></a>Transacciones

Parece extraño comenzar una revisión del procesamiento por lotes hablando de transacciones. Pero el uso de transacciones del lado cliente surte un sutil efecto de procesamiento por lotes del lado servidor que mejora el rendimiento. Además, las transacciones se pueden agregar con unas pocas líneas de código, lo que proporciona una forma rápida de mejorar el rendimiento de las operaciones secuenciales.

Observe el siguiente código C# que contiene una secuencia de operaciones de inserción y actualización en una tabla sencilla.

```csharp
List<string> dbOperations = new List<string>();
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 1");
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 2");
dbOperations.Add("update MyTable set mytext = 'updated text' where id = 3");
dbOperations.Add("insert MyTable values ('new value',1)");
dbOperations.Add("insert MyTable values ('new value',2)");
dbOperations.Add("insert MyTable values ('new value',3)");
```

El siguiente código ADO.NET realiza estas operaciones de forma secuencial.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    conn.Open();

    foreach(string commandString in dbOperations)
    {
        SqlCommand cmd = new SqlCommand(commandString, conn);
        cmd.ExecuteNonQuery();
    }
}
```

La mejor manera de optimizar este código es implementar algún tipo de procesamiento por lotes del lado cliente para estas llamadas. Pero hay una forma sencilla de aumentar el rendimiento de este código simplemente encapsulando la secuencia de llamadas en una transacción. El siguiente código es el mismo que usa una transacción.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    conn.Open();
    SqlTransaction transaction = conn.BeginTransaction();

    foreach (string commandString in dbOperations)
    {
        SqlCommand cmd = new SqlCommand(commandString, conn, transaction);
        cmd.ExecuteNonQuery();
    }

    transaction.Commit();
}
```

Realmente, se usan transacciones en ambos ejemplos. En el primer ejemplo, cada llamada individual es una transacción implícita. En el segundo ejemplo, una transacción explícita encapsula todas las llamadas. Según la documentación del [registro de transacciones de escritura previa](/sql/relational-databases/sql-server-transaction-log-architecture-and-management-guide?view=sql-server-ver15&preserve-view=true#WAL), las entradas del registro se vacían en el disco cuando se confirma la transacción. Por lo tanto, al incluir más llamadas en una transacción, la escritura en el registro de transacciones se puede retrasar hasta que se confirma la transacción. En efecto, está habilitando el procesamiento por lotes para las escrituras en el registro de transacciones del servidor.

En la tabla siguiente se muestran algunos resultados de pruebas ad hoc. En las pruebas se realizaron las mismas inserciones secuenciales con y sin transacciones. Para obtener más perspectiva, el primer conjunto de pruebas se ejecutó de forma remota de un equipo portátil a la base de datos de Microsoft Azure. El segundo conjunto de pruebas se ejecutó desde un servicio en la nube y una base de datos que residían en el mismo centro de datos de Microsoft Azure (Oeste de EE. UU.). En la tabla siguiente se muestra la duración en milisegundos de las inserciones secuenciales con y sin transacciones.

**Local a Azure**:

| Operaciones | Ninguna transacción (ms) | Transacción (ms) |
| --- | --- | --- |
| 1 |130 |402 |
| 10 |1208 |1226 |
| 100 |12662 |10395 |
| 1000 |128852 |102917 |

**De Azure a Azure (mismo centro de datos)** :

| Operaciones | Ninguna transacción (ms) | Transacción (ms) |
| --- | --- | --- |
| 1 |21 |26 |
| 10 |220 |56 |
| 100 |2145 |341 |
| 1000 |21479 |2756 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

A partir de los resultados de las pruebas anteriores, encapsular una única operación en una transacción en realidad reduce el rendimiento. Pero a medida que aumente el número de operaciones dentro de una única transacción, la mejora del rendimiento se vuelve más marcada. La diferencia de rendimiento también es más apreciable cuando todas las operaciones se producen dentro del centro de datos de Microsoft Azure. La mayor latencia existente cuando se usa Azure SQL Database o Instancia administrada de Azure SQL desde fuera del centro de datos de Microsoft Azure contrarresta la ganancia de rendimiento por el uso de transacciones.

Aunque el uso de transacciones puede aumentar el rendimiento, siga [respetando las prácticas recomendadas para las transacciones y las conexiones](/previous-versions/sql/sql-server-2008-r2/ms187484(v=sql.105)). Mantenga la transacción lo más corta posible y cierre la conexión con la base de datos una vez finalizado el trabajo. La instrucción using del ejemplo anterior garantiza que la conexión se cierre cuando finalice el bloque de código subsiguiente.

El ejemplo anterior muestra que puede agregar una transacción local a cualquier código ADO.NET con dos líneas. Las transacciones ofrecen una forma rápida de mejorar el rendimiento del código que realiza operaciones secuenciales de inserción, actualización y eliminación. Sin embargo, para lograr el máximo rendimiento, podría cambiar aún más el código para aprovechar las ventajas del procesamiento por lotes del lado cliente, como los parámetros con valores de tabla.

Para obtener más información acerca de las transacciones en ADO.NET, consulte [Transacciones locales](/dotnet/framework/data/adonet/local-transactions).

### <a name="table-valued-parameters"></a>Parámetros con valores de tabla

Los parámetros con valores de tabla admiten tipos de tabla definidos por el usuario como parámetros en instrucciones Transact-SQL, procedimientos almacenados y funciones. Esta técnica de procesamiento por lotes del lado cliente permite enviar varias filas de datos dentro del parámetro con valores de tabla. Para usar parámetros con valores de tabla, primero debe definir un tipo de tabla. La siguiente instrucción Transact-SQL crea un tipo de tabla denominado **MyTableType**.

```sql
    CREATE TYPE MyTableType AS TABLE
    ( mytext TEXT,
      num INT );
```

En el código, se crea un **DataTable** con los mismos nombres y tipos exactos del tipo de tabla. Pase este **DataTable** en un parámetro de una consulta de texto o una llamada de procedimiento almacenado. En el siguiente ejemplo se muestra esta técnica:

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    DataTable table = new DataTable();
    // Add columns and rows. The following is a simple example.
    table.Columns.Add("mytext", typeof(string));
    table.Columns.Add("num", typeof(int));
    for (var i = 0; i < 10; i++)
    {
        table.Rows.Add(DateTime.Now.ToString(), DateTime.Now.Millisecond);
    }

    SqlCommand cmd = new SqlCommand(
        "INSERT INTO MyTable(mytext, num) SELECT mytext, num FROM @TestTvp",
        connection);

    cmd.Parameters.Add(
        new SqlParameter()
        {
            ParameterName = "@TestTvp",
            SqlDbType = SqlDbType.Structured,
            TypeName = "MyTableType",
            Value = table,
        });

    cmd.ExecuteNonQuery();
}
```

En el ejemplo anterior, el objeto **SqlCommand** inserta filas desde un parámetro con valores de tabla, **\@TestTvp**. El objeto **DataTable** creado antes se asigna a este parámetro con el método **SqlCommand.Parameters.Add**. El procesamiento por lotes de las inserciones en una llamada aumenta notablemente el rendimiento en comparación con las inserciones secuenciales.

Para mejorar aún más el ejemplo anterior, use un procedimiento almacenado en lugar de un comando de texto. El siguiente comando Transact-SQL crea un procedimiento almacenado que acepta el parámetro con valores de tabla **SimpleTestTableType** .

```sql
CREATE PROCEDURE [dbo].[sp_InsertRows]
@TestTvp as MyTableType READONLY
AS
BEGIN
INSERT INTO MyTable(mytext, num)
SELECT mytext, num FROM @TestTvp
END
GO
```

Después, cambie la declaración del objeto **SqlCommand** en el ejemplo de código anterior a lo siguiente.

```csharp
SqlCommand cmd = new SqlCommand("sp_InsertRows", connection);
cmd.CommandType = CommandType.StoredProcedure;
```

En la mayoría de los casos, los parámetros con valores de tabla tienen un rendimiento equivalente o mejor que otras técnicas de procesamiento por lotes. Los parámetros con valores de tabla son a menudo preferibles, ya que son más flexibles que otras opciones. Por ejemplo, otras técnicas, como la copia masiva de SQL, solo permiten la inserción de filas nuevas. Sin embargo, con los parámetros con valores de tabla, puede usar lógica en el procedimiento almacenado para determinar qué filas son actualizaciones y cuáles son inserciones. También se puede modificar el tipo de tabla para que contenga una columna "Operación" que indica si la fila especificada se debe insertar, actualizar o eliminar.

En la tabla siguiente se muestran los resultados de pruebas ad hoc para el uso de parámetros con valores de tabla, expresados en milisegundos.

| Operaciones | Local a Azure (ms) | Azure en el mismo centro de datos (ms) |
| --- | --- | --- |
| 1 |124 |32 |
| 10 |131 |25 |
| 100 |338 |51 |
| 1000 |2615 |382 |
| 10000 |23830 |3586 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

El aumento del rendimiento gracias al procesamiento por lotes es evidente de inmediato. En la prueba secuencial anterior, 1000 operaciones tardaban 129 segundos fuera del centro de datos y 21 segundos dentro del centro de datos. Pero con parámetros con valores de tabla, 1000 operaciones solo tardan 2,6 segundos fuera del centro de datos y 0,4 segundos dentro del centro de datos.

Para obtener más información sobre los parámetros con valores de tabla, consulte [Usar parámetros con valores de tabla (motor de base de datos)](/sql/relational-databases/tables/use-table-valued-parameters-database-engine).

### <a name="sql-bulk-copy"></a>Copia masiva de SQL

La copia masiva de SQL es otra forma de insertar grandes cantidades de datos en una base de datos de destino. Las aplicaciones .NET pueden usar la clase **SqlBulkCopy** para realizar operaciones de inserción masiva. **SqlBulkCopy** desempeña una función similar a la herramienta de línea de comandos **Bcp.exe** o la instrucción Transact-SQL **BULK INSERT**. En el ejemplo de código siguiente se muestra cómo realizar una copia masiva de las filas de la tabla de origen **DataTable** en la tabla de destino, MyTable.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    using (SqlBulkCopy bulkCopy = new SqlBulkCopy(connection))
    {
        bulkCopy.DestinationTableName = "MyTable";
        bulkCopy.ColumnMappings.Add("mytext", "mytext");
        bulkCopy.ColumnMappings.Add("num", "num");
        bulkCopy.WriteToServer(table);
    }
}
```

Hay algunos casos en que la copia masiva es preferible a los parámetros con valores de tabla. Consulte la comparativa de los parámetros con valores de tabla frente a las operaciones BULK INSERT que encontrará en este artículo sobre [parámetros con valores de tabla](/sql/relational-databases/tables/use-table-valued-parameters-database-engine).

Los resultados de pruebas ad hoc siguientes muestran el rendimiento del procesamiento por lotes con **SqlBulkCopy** en milisegundos.

| Operaciones | Local a Azure (ms) | Azure en el mismo centro de datos (ms) |
| --- | --- | --- |
| 1 |433 |57 |
| 10 |441 |32 |
| 100 |636 |53 |
| 1000 |2535 |341 |
| 10000 |21605 |2737 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

En lotes más pequeños, el uso de parámetros con valores de tabla superó el rendimiento de la clase **SqlBulkCopy** . Sin embargo, el rendimiento con **SqlBulkCopy** fue entre un 12 % y un 31 % mayor que los parámetros con valores de tabla en las pruebas de 1,000 y 10.000 filas. Como los parámetros con valores de tabla, **SqlBulkCopy** es una buena opción para las inserciones por lotes, especialmente cuando se compara con el rendimiento de las operaciones sin lotes.

Para obtener más información sobre la copia masiva en ADO.NET, consulte [Operaciones de copia masiva](/dotnet/framework/data/adonet/sql/bulk-copy-operations-in-sql-server).

### <a name="multiple-row-parameterized-insert-statements"></a>Instrucciones INSERT con parámetros de varias filas

Una alternativa para los lotes pequeños es construir una instrucción INSERT con parámetros grande que inserte varias filas. En el siguiente ejemplo de código se muestra esta técnica.

```csharp
using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
{
    connection.Open();

    string insertCommand = "INSERT INTO [MyTable] ( mytext, num ) " +
        "VALUES (@p1, @p2), (@p3, @p4), (@p5, @p6), (@p7, @p8), (@p9, @p10)";

    SqlCommand cmd = new SqlCommand(insertCommand, connection);

    for (int i = 1; i <= 10; i += 2)
    {
        cmd.Parameters.Add(new SqlParameter("@p" + i.ToString(), "test"));
        cmd.Parameters.Add(new SqlParameter("@p" + (i+1).ToString(), i));
    }

    cmd.ExecuteNonQuery();
}
```

Este ejemplo está pensado para demostrar el concepto básico. En un escenario más realista, se recorrerían las entidades necesarias para construir la cadena de consulta y los parámetros del comando simultáneamente. Está limitado a un total de 2100 parámetros de consulta, lo que restringe el número total de filas que pueden procesarse de esta manera.

Los resultados de pruebas ad hoc siguientes muestran el rendimiento de este tipo de instrucción de inserción en milisegundos.

| Operaciones | Parámetros con valores de tabla (ms) | Instrucción INSERT única (ms) |
| --- | --- | --- |
| 1 |32 |20 |
| 10 |30 |25 |
| 100 |33 |51 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

Este enfoque puede ser algo más rápido para los lotes de menos de 100 filas. Aunque la mejora es pequeña, esta técnica es otra opción que podría funcionar bien en su escenario de aplicaciones específico.

### <a name="dataadapter"></a>DataAdapter

La clase **DataAdapter** le permite modificar un objeto **DataSet** y después enviar los cambios como operaciones INSERT, UPDATE y DELETE. Si está usando la clase **DataAdapter** de esta manera, es importante tener en cuenta que se realizan llamadas independientes para cada operación distinta. Para mejorar el rendimiento, use la propiedad **UpdateBatchSize** con el número de operaciones que deben procesarse por lotes a la vez. Para obtener más información, consulte [Realizar operaciones por lotes mediante DataAdapters](/dotnet/framework/data/adonet/performing-batch-operations-using-dataadapters).

### <a name="entity-framework"></a>Entity Framework

[Entity Framework Core](/ef/efcore-and-ef6/#saving-data) admite ahora el procesamiento por lotes.

### <a name="xml"></a>XML

Para ofrecer una imagen completa, creemos que es importante hablar de XML como estrategia de procesamiento por lotes. Sin embargo, el uso de XML no supone ninguna ventaja respecto a otros métodos, pero sí varias desventajas. El enfoque es similar a los parámetros con valores de tabla, excepto en que se pasa una cadena o un archivo XML a un procedimiento almacenado en lugar de una tabla definida por el usuario. El procedimiento almacenado analiza los comandos en el procedimiento almacenado.

Existen varias desventajas con este enfoque:

* El trabajo con XML puede resultar tedioso y propenso a errores.
* Analizar el XML en la base de datos puede consumir bastantes recursos de la CPU.
* En la mayoría de los casos, este método es más lento que los parámetros con valores de tabla.

Por estas razones, no se recomienda el uso de XML para las consultas por lotes.

## <a name="batching-considerations"></a>Consideraciones sobre el procesamiento por lotes

Las secciones siguientes proporcionan más instrucciones para el uso del procesamiento por lotes en las aplicaciones de Azure SQL Database e Instancia administrada de Azure SQL.

### <a name="tradeoffs"></a>Compromisos

Según la arquitectura, el procesamiento por lotes puede suponer un compromiso entre el rendimiento y la resistencia. Por ejemplo, piense en un escenario en que su rol deja de funcionar inesperadamente. Si pierde una fila de datos, el efecto es menor que si pierde un lote grande de filas sin enviar. Existe un riesgo mayor cuando almacena filas en búfer antes de enviarlas a la base de datos en un período de tiempo especificado.

Debido a este compromiso, evalúe el tipo de operaciones que procese por lotes. Intensifique el procesamiento por lotes (con lotes y períodos de tiempo mayores) con aquellos datos que sean menos críticos.

### <a name="batch-size"></a>Tamaño de lote

En nuestras pruebas, normalmente dividir los lotes grandes en fragmentos menores no supuso ninguna ventaja. De hecho, esta subdivisión produjo a menudo un rendimiento más lento que el envío de un único lote grande. Por ejemplo, considere un escenario en el que desea insertar 1000 filas. La siguiente tabla muestra cuánto tiempo se tarda usando parámetros con valores de tabla para insertar 1000 filas cuando se dividen en lotes más pequeños.

| Tamaño de lote | Iteraciones | Parámetros con valores de tabla (ms) |
| --- | --- | --- |
| 1000 |1 |347 |
| 500 |2 |355 |
| 100 |10 |465 |
| 50 |20 |630 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

Es obvio que el mejor rendimiento para 1000 filas es enviarlas todas a la vez. En otras pruebas (no mostradas aquí), hubo una pequeña mejora de rendimiento al dividir un lote de 10 000 filas en dos lotes de 5000. Pero el esquema de tabla para estas pruebas es relativamente simple, por lo que debería realizar pruebas con sus datos y tamaños de lote específicos para verificar estos hallazgos.

Otro factor que se debe considerar es que, si el lote total es demasiado grande, es posible que Azure SQL Database o Instancia administrada de Azure SQL imponga limitaciones y no confirme el lote. Para obtener resultados óptimos, pruebe su escenario concreto para determinar si existe un tamaño de lote ideal. Haga que el tamaño de lote sea configurable en tiempo de ejecución para permitir ajustes rápidos en función del rendimiento o la presencia de errores.

Por último, sopese el tamaño del lote y los riesgos asociados con el procesamiento por lotes. Si se producen errores transitorios o de rol, considere las consecuencias de reintentar la operación o de perder los datos en el lote.

### <a name="parallel-processing"></a>Procesamiento paralelo

¿Qué pasa si adopta el enfoque de reducir el tamaño del lote pero usa varios subprocesos para ejecutar el trabajo? Una vez más, nuestras pruebas mostraron que varios lotes multiproceso más pequeños presentaban normalmente un rendimiento peor que un único lote más grande. La siguiente prueba intenta insertar 1000 filas en uno o varios lotes paralelos. Esta prueba muestra cómo el uso de más lotes simultáneos realmente reduce el rendimiento.

| Tamaño de lote [iteraciones] | Dos subprocesos (ms) | Cuatro subprocesos (ms) | Seis subprocesos (ms) |
| --- | --- | --- | --- |
| 1000 [1] |277 |315 |266 |
| 500 [2] |548 |278 |256 |
| 250 [4] |405 |329 |265 |
| 100 [10] |488 |439 |391 |

> [!NOTE]
> Los resultados no sirven para pruebas comparativas. Consulte la [nota sobre los tiempos resultantes en este artículo](#note-about-timing-results-in-this-article).

Hay varias razones posibles para la degradación del rendimiento debido al paralelismo:

* Se realizan varias llamadas de red simultáneas en lugar de una.
* Varias operaciones en una sola tabla pueden conllevar contención y bloqueo.
* Hay sobrecargas asociadas con el subprocesamiento múltiple.
* El gasto que supone abrir varias conexiones sobrepasa las ventajas del procesamiento en paralelo.

Si elige como destino diferentes tablas o bases de datos, es posible que observe alguna mejora de rendimiento con esta estrategia. Un escenario de particionamiento de base de datos o de federaciones sería adecuado para este enfoque. El particionamiento usa varias bases de datos y enruta distintos datos a cada base de datos. Si cada lote pequeño va a una base de datos diferente, puede ser más eficaz realizar las operaciones en paralelo. Sin embargo, la mejora de rendimiento no es lo bastante importante como para usarla como base para tomar la decisión de emplear el particionamiento de base de datos en la solución.

En algunos diseños, la ejecución en paralelo de lotes más pequeños podría mejorar el rendimiento de las solicitudes en un sistema con mucha carga. En este caso, aunque es más rápido procesar un único lote más grande, es posible que el procesamiento de varios lotes en paralelo sea más eficaz.

Si usa la ejecución en paralelo, considere la posibilidad de controlar el número máximo de subprocesos de trabajo. Un número menor podría causar menos contención y un tiempo de ejecución más rápido. Además, tenga en cuenta la carga adicional sobre la base de datos de destino tanto en conexiones como en transacciones.

### <a name="related-performance-factors"></a>Factores de rendimiento relacionados

Las instrucciones típicas sobre el rendimiento de la base de datos también afectan al procesamiento por lotes. Por ejemplo, el rendimiento de inserción se reduce para aquellas tablas que tengan una clave principal grande o varios índices no agrupados.

Si los parámetros con valores de tabla usan un procedimiento almacenado, puede emplear el comando **SET NOCOUNT ON** al principio del procedimiento. Esta instrucción suprime la devolución del recuento de las filas afectadas en el procedimiento. Sin embargo, en nuestras pruebas, el uso de **SET NOCOUNT ON** no tiene ningún efecto sobre el rendimiento o lo disminuye. El procedimiento almacenado de la prueba era simple, con un solo comando **INSERT** desde el parámetro con valores de tabla. Es posible que los procedimientos almacenados más complejos se beneficien de esta instrucción. Pero no dé por supuesto que agregar **SET NOCOUNT ON** al procedimiento almacenado mejora automáticamente el rendimiento. Para entender el efecto, pruebe el procedimiento almacenado con la instrucción **SET NOCOUNT ON** y sin ella.

## <a name="batching-scenarios"></a>Escenarios de procesamiento por lotes

En las secciones siguientes, se describe cómo usar parámetros con valores de tabla en tres escenarios de aplicaciones. El primer escenario muestra cómo el almacenamiento en búfer y el procesamiento por lotes funcionan juntos. El segundo escenario mejora el rendimiento al realizar operaciones maestro/detalle en una sola llamada a procedimiento almacenado. El último escenario muestra cómo usar parámetros con valores de tabla en una operación "UPSERT".

### <a name="buffering"></a>de respuesta

Aunque hay algunos escenarios que son candidatos obvios para el procesamiento por lotes, muchos otros podrían beneficiarse del procesamiento por lotes difiriendo el procesamiento. Sin embargo, el procesamiento diferido también plantea un mayor riesgo de que los datos se pierdan si se produce un error inesperado. Es importante comprender este riesgo y tener en cuenta las consecuencias.

Por ejemplo, piense en una aplicación web que registra el historial de navegación de cada usuario. Con cada solicitud de página, la aplicación podría llamar a una base de datos para registrar la vista de página del usuario. Pero se pueden conseguir mayor rendimiento y escalabilidad si se almacenan las actividades de navegación de los usuarios en el búfer y después se envían estos datos a la base de datos en lotes. Puede desencadenar la actualización de la base de datos según el tiempo transcurrido o el tamaño de búfer. Por ejemplo, una regla podría especificar que se debería procesar el lote después de 20 segundos o cuando el búfer alcance los 1000 elementos.

El siguiente ejemplo de código usa [extensiones reactivas - Rx](/previous-versions/dotnet/reactive-extensions/hh242985(v=vs.103)) para procesar los eventos almacenados en búfer generados por una clase de supervisión. Cuando el búfer se llena o se alcanza el tiempo de espera, se envía el lote de datos de usuarios a la base de datos con un parámetro con valores de tabla.

La siguiente clase NavHistoryData modela los detalles de navegación de los usuarios. Contiene información básica como el identificador de usuario, la dirección URL visitada y el tiempo de acceso.

```csharp
public class NavHistoryData
{
    public NavHistoryData(int userId, string url, DateTime accessTime)
    { UserId = userId; URL = url; AccessTime = accessTime; }
    public int UserId { get; set; }
    public string URL { get; set; }
    public DateTime AccessTime { get; set; }
}
```

La clase NavHistoryDataMonitor se encarga de almacenar los datos de navegación de los usuarios en búfer en la base de datos. Contiene un método, RecordUserNavigationEntry, que responde generando un evento **OnAdded** . El código siguiente muestra la lógica del constructor que usa Rx para crear una colección observable basada en el evento. Después, se suscribe a esta colección observable con el método Buffer. La sobrecarga especifica que el búfer se debe enviar cada 20 segundos o 1000 entradas.

```csharp
public NavHistoryDataMonitor()
{
    var observableData =
        Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

    observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);
}
```

El controlador convierte todos los elementos almacenados en búfer en un tipo con valores de tabla y después pasa este tipo a un procedimiento almacenado que procesa el lote. El código siguiente muestra la definición completa de las clases NavHistoryDataEventArgs y NavHistoryDataMonitor.

```csharp
public class NavHistoryDataEventArgs : System.EventArgs
{
    public NavHistoryDataEventArgs(NavHistoryData data) { Data = data; }
    public NavHistoryData Data { get; set; }
}

public class NavHistoryDataMonitor
{
    public event EventHandler<NavHistoryDataEventArgs> OnAdded;

    public NavHistoryDataMonitor()
    {
        var observableData =
            Observable.FromEventPattern<NavHistoryDataEventArgs>(this, "OnAdded");

        observableData.Buffer(TimeSpan.FromSeconds(20), 1000).Subscribe(Handler);
    }
```

El controlador convierte todos los elementos almacenados en búfer en un tipo con valores de tabla y después pasa este tipo a un procedimiento almacenado que procesa el lote. El código siguiente muestra la definición completa de las clases NavHistoryDataEventArgs y NavHistoryDataMonitor.

```csharp
    public class NavHistoryDataEventArgs : System.EventArgs
    {
        if (OnAdded != null)
            OnAdded(this, new NavHistoryDataEventArgs(data));
    }

    protected void Handler(IList<EventPattern<NavHistoryDataEventArgs>> items)
    {
        DataTable navHistoryBatch = new DataTable("NavigationHistoryBatch");
        navHistoryBatch.Columns.Add("UserId", typeof(int));
        navHistoryBatch.Columns.Add("URL", typeof(string));
        navHistoryBatch.Columns.Add("AccessTime", typeof(DateTime));
        foreach (EventPattern<NavHistoryDataEventArgs> item in items)
        {
            NavHistoryData data = item.EventArgs.Data;
            navHistoryBatch.Rows.Add(data.UserId, data.URL, data.AccessTime);
        }

        using (SqlConnection connection = new SqlConnection(CloudConfigurationManager.GetSetting("Sql.ConnectionString")))
        {
            connection.Open();

            SqlCommand cmd = new SqlCommand("sp_RecordUserNavigation", connection);
            cmd.CommandType = CommandType.StoredProcedure;

            cmd.Parameters.Add(
                new SqlParameter()
                {
                    ParameterName = "@NavHistoryBatch",
                    SqlDbType = SqlDbType.Structured,
                    TypeName = "NavigationHistoryTableType",
                    Value = navHistoryBatch,
                });

            cmd.ExecuteNonQuery();
        }
    }
}
```

Para usar esta clase de almacenamiento en búfer, la aplicación crea un objeto NavHistoryDataMonitor estático. Cada vez que un usuario accede a una página, la aplicación llama al método NavHistoryDataMonitor.RecordUserNavigationEntry. La lógica de almacenamiento en búfer procede a enviar estas entradas a la base de datos en lotes.

### <a name="master-detail"></a>Maestro/detalle

Los parámetros con valores de tabla son útiles en escenarios INSERT sencillos. Sin embargo, puede ser más difícil procesar por lotes aquellas inserciones para más de una tabla. El escenario de "maestro/detalle" es un buen ejemplo. La tabla maestra identifica la entidad principal. Una o varias tablas de detalle almacenan más datos sobre la entidad. En este escenario, las relaciones de clave externa aplican la relación de los detalles con una entidad maestra única. Considere una versión simplificada de una tabla PurchaseOrder y su tabla OrderDetail asociada. El siguiente código Transact-SQL crea la tabla PurchaseOrder con cuatro columnas: OrderID, OrderDate, CustomerID y Status.

```sql
CREATE TABLE [dbo].[PurchaseOrder](
[OrderID] [int] IDENTITY(1,1) NOT NULL,
[OrderDate] [datetime] NOT NULL,
[CustomerID] [int] NOT NULL,
[Status] [nvarchar](50) NOT NULL,
CONSTRAINT [PrimaryKey_PurchaseOrder]
PRIMARY KEY CLUSTERED ( [OrderID] ASC ))
```

Cada pedido contiene una o más compras de productos. Esta información se captura en la tabla PurchaseOrderDetail. El siguiente código Transact-SQL crea la tabla PurchaseOrderDetail con cinco columnas: OrderID, OrderDetailID, ProductID, UnitPrice y OrderQty.

```sql
CREATE TABLE [dbo].[PurchaseOrderDetail](
[OrderID] [int] NOT NULL,
[OrderDetailID] [int] IDENTITY(1,1) NOT NULL,
[ProductID] [int] NOT NULL,
[UnitPrice] [money] NULL,
[OrderQty] [smallint] NULL,
CONSTRAINT [PrimaryKey_PurchaseOrderDetail] PRIMARY KEY CLUSTERED
( [OrderID] ASC, [OrderDetailID] ASC ))
```

La columna OrderID de la tabla PurchaseOrderDetail debe hacer referencia a un pedido de la tabla PurchaseOrder. La siguiente definición de una clave externa aplica esta restricción.

```sql
ALTER TABLE [dbo].[PurchaseOrderDetail]  WITH CHECK ADD
CONSTRAINT [FK_OrderID_PurchaseOrder] FOREIGN KEY([OrderID])
REFERENCES [dbo].[PurchaseOrder] ([OrderID])
```

Para poder usar parámetros con valores de tabla, debe tener un tipo de tabla definido por el usuario para cada tabla de destino.

```sql
CREATE TYPE PurchaseOrderTableType AS TABLE
( OrderID INT,
    OrderDate DATETIME,
    CustomerID INT,
    Status NVARCHAR(50) );
GO

CREATE TYPE PurchaseOrderDetailTableType AS TABLE
( OrderID INT,
    ProductID INT,
    UnitPrice MONEY,
    OrderQty SMALLINT );
GO
```

Después, defina un procedimiento almacenado que acepte tablas de estos tipos. Este procedimiento permite que una aplicación procese un conjunto de pedidos y detalles de pedido por lotes localmente en una sola llamada. El siguiente código Transact-SQL proporciona la declaración completa del procedimiento almacenado para este ejemplo de pedido de compra.

```sql
CREATE PROCEDURE sp_InsertOrdersBatch (
@orders as PurchaseOrderTableType READONLY,
@details as PurchaseOrderDetailTableType READONLY )
AS
SET NOCOUNT ON;

-- Table that connects the order identifiers in the @orders
-- table with the actual order identifiers in the PurchaseOrder table
DECLARE @IdentityLink AS TABLE (
SubmittedKey int,
ActualKey int,
RowNumber int identity(1,1)
);

-- Add new orders to the PurchaseOrder table, storing the actual
-- order identifiers in the @IdentityLink table
INSERT INTO PurchaseOrder ([OrderDate], [CustomerID], [Status])
OUTPUT inserted.OrderID INTO @IdentityLink (ActualKey)
SELECT [OrderDate], [CustomerID], [Status] FROM @orders ORDER BY OrderID;

-- Match the passed-in order identifiers with the actual identifiers
-- and complete the @IdentityLink table for use with inserting the details
WITH OrderedRows As (
SELECT OrderID, ROW_NUMBER () OVER (ORDER BY OrderID) As RowNumber
FROM @orders
)
UPDATE @IdentityLink SET SubmittedKey = M.OrderID
FROM @IdentityLink L JOIN OrderedRows M ON L.RowNumber = M.RowNumber;

-- Insert the order details into the PurchaseOrderDetail table,
-- using the actual order identifiers of the master table, PurchaseOrder
INSERT INTO PurchaseOrderDetail (
[OrderID],
[ProductID],
[UnitPrice],
[OrderQty] )
SELECT L.ActualKey, D.ProductID, D.UnitPrice, D.OrderQty
FROM @details D
JOIN @IdentityLink L ON L.SubmittedKey = D.OrderID;
GO
```

En este ejemplo, la tabla @IdentityLink definida localmente almacena los valores de OrderID reales de las filas recién insertadas. Estos identificadores de pedidos son diferentes de los valores de OrderID temporales de los parámetros con valores de tabla @orders y @details. Por este motivo, la tabla @IdentityLink conecta después los valores de OrderID del parámetro @orders a los valores de OrderID reales para las nuevas filas de la tabla PurchaseOrder. Después de este paso, la tabla @IdentityLink puede facilitar la inserción de los detalles del pedido con el OrderID real que cumple la restricción de clave externa.

Este procedimiento almacenado puede usarse desde el código o desde otras llamadas Transact-SQL. Consulte la sección Parámetros con valores de tabla de este artículo para obtener un ejemplo de código. El siguiente código Transact-SQL muestra cómo llamar a sp_InsertOrdersBatch.

```sql
declare @orders as PurchaseOrderTableType
declare @details as PurchaseOrderDetailTableType

INSERT @orders
([OrderID], [OrderDate], [CustomerID], [Status])
VALUES(1, '1/1/2013', 1125, 'Complete'),
(2, '1/13/2013', 348, 'Processing'),
(3, '1/12/2013', 2504, 'Shipped')

INSERT @details
([OrderID], [ProductID], [UnitPrice], [OrderQty])
VALUES(1, 10, $11.50, 1),
(1, 12, $1.58, 1),
(2, 23, $2.57, 2),
(3, 4, $10.00, 1)

exec sp_InsertOrdersBatch @orders, @details
```

Esta solución permite que cada lote use un conjunto de valores de OrderID que empiezan en 1. Estos valores temporales de OrderID describen las relaciones en el lote, pero los valores de OrderID reales se determinan en el momento de la operación de inserción. Puede ejecutar las mismas instrucciones en el ejemplo anterior repetidamente y generar pedidos únicos en la base de datos. Por este motivo, podría agregar más lógica de base de datos o código que evite la duplicación de pedidos cuando se usa esta técnica de procesamiento por lotes.

Este ejemplo demuestra que las operaciones de base de datos más complejas, como las operaciones maestro/detalle, se pueden procesar por lotes con parámetros con valores de tabla.

### <a name="upsert"></a>UPSERT

Otro escenario de procesamiento por lotes supone la actualización de filas existentes y la inserción de nuevas filas de forma simultánea. Esta operación se conoce a veces como operación "UPSERT" (actualización + inserción). En lugar de realizar llamadas independientes para insertar (INSERT) y actualizar (UPDATE), la instrucción MERGE es más adecuada para esta tarea. La instrucción MERGE puede realizar ambas operaciones en una sola llamada.

Los parámetros con valores de tabla pueden usarse con la instrucción MERGE para realizar actualizaciones e inserciones. Por ejemplo, piense en una tabla Employee simplificada que contiene las siguientes columnas: EmployeeID, FirstName, LastName y SocialSecurityNumber:

```sql
CREATE TABLE [dbo].[Employee](
[EmployeeID] [int] IDENTITY(1,1) NOT NULL,
[FirstName] [nvarchar](50) NOT NULL,
[LastName] [nvarchar](50) NOT NULL,
[SocialSecurityNumber] [nvarchar](50) NOT NULL,
CONSTRAINT [PrimaryKey_Employee] PRIMARY KEY CLUSTERED
([EmployeeID] ASC ))
```

En este ejemplo, puede aprovechar el hecho de que SocialSecurityNumber (número del seguro social) sea único para realizar una operación MERGE de varios empleados. En primer lugar, cree el tipo de tabla definido por el usuario:

```sql
CREATE TYPE EmployeeTableType AS TABLE
( Employee_ID INT,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    SocialSecurityNumber NVARCHAR(50) );
GO
```

Después, cree un procedimiento almacenado o escriba código que use la instrucción MERGE para realizar la actualización y la inserción. En el ejemplo siguiente, se usa la instrucción MERGE en un parámetro con valores de tabla, @employees, del tipo EmployeeTableType. El contenido de la tabla @employees no se muestra aquí.

```sql
MERGE Employee AS target
USING (SELECT [FirstName], [LastName], [SocialSecurityNumber] FROM @employees)
AS source ([FirstName], [LastName], [SocialSecurityNumber])
ON (target.[SocialSecurityNumber] = source.[SocialSecurityNumber])
WHEN MATCHED THEN
UPDATE SET
target.FirstName = source.FirstName,
target.LastName = source.LastName
WHEN NOT MATCHED THEN
    INSERT ([FirstName], [LastName], [SocialSecurityNumber])
    VALUES (source.[FirstName], source.[LastName], source.[SocialSecurityNumber]);
```

Para obtener más información, consulte la documentación y los ejemplos de la instrucción MERGE. Aunque se podría realizar el mismo trabajo en una llamada a procedimiento almacenado de varios pasos con operaciones INSERT y UPDATE separadas, la instrucción MERGE es más eficaz. Además, el código de la base de datos puede construir llamadas Transact-SQL que usen la instrucción MERGE directamente sin necesidad de realizar dos llamadas de base de datos para INSERT y UPDATE.

## <a name="recommendation-summary"></a>Resumen de recomendaciones

En la lista siguiente, se resumen las recomendaciones sobre el procesamiento por lotes tratadas en este artículo:

* Use el almacenamiento en búfer y el procesamiento por lotes para aumentar el rendimiento y la escalabilidad de las aplicaciones de Azure SQL Database e Instancia administrada de Azure SQL.
* Comprenda los compromisos entre el procesamiento por lotes o el almacenamiento en búfer y la resistencia. Durante un error de rol, el riesgo de perder un lote sin procesar de datos esenciales para la empresa podría sobrepasar las ventajas de rendimiento del procesamiento por lotes.
* Intente mantener todas las llamadas a la base de datos dentro de un único centro de datos para reducir la latencia.
* Si elige una técnica de procesamiento con un solo lote, los parámetros con valores de tabla ofrecen el mejor rendimiento y flexibilidad.
* Para lograr el rendimiento de inserción de mayor velocidad, siga estas instrucciones generales y pruebe su escenario:
  * Para < 100 filas, use un único comando INSERT con parámetros.
  * Para < 1000 filas, use parámetros con valores de tabla.
  * Para > = 1000 filas, use SqlBulkCopy.
* Para las operaciones de actualización y eliminación, use parámetros con valores de tabla con lógica de procedimiento almacenado que determine la operación correcta en cada fila en el parámetro de la tabla.
* Instrucciones para el tamaño de lote:
  * Use los tamaños de lote más grandes que tengan sentido para los requisitos de la aplicación y de la empresa.
  * Equilibre la ganancia en rendimiento de los lotes grandes con el riesgo de los errores temporales o catastróficos. ¿Cuál es la consecuencia de los reintentos o la pérdida de datos en el lote?
  * Pruebe el tamaño de lote más grande para verificar que Azure SQL Database o Instancia administrada de Azure SQL no lo rechace.
  * Cree parámetros de configuración que controlen el procesamiento por lotes, como el tamaño del lote o el período de tiempo de almacenamiento en búfer. Estas configuraciones proporcionan flexibilidad. Puede cambiar el comportamiento de procesamiento por lotes en producción sin volver a implementar el servicio en la nube.
* Evite la ejecución en paralelo de lotes que operan en una sola tabla en una base de datos. Si decide dividir un único lote entre varios subprocesos de trabajo, ejecute pruebas para determinar el número ideal de subprocesos. Después de traspasar un umbral no especificado, el aumento del número de subprocesos hará que el rendimiento disminuya en lugar de mejorarlo.
* Considere la posibilidad de almacenar en búfer por tamaño y hora como una manera de implementar el procesamiento por lotes para más escenarios.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en cómo el diseño de base de datos y las técnicas de codificado relacionadas con el procesamiento por lotes pueden mejorar el rendimiento y la escalabilidad de las aplicaciones. Sin embargo, esto es solamente un factor en la estrategia global. Para conocer más formas de mejorar el rendimiento y la escalabilidad, consulte [Guía de rendimiento de la base de datos](database/performance-guidance.md) y [Consideraciones de precio y rendimiento para un grupo elástico](database/elastic-pool-overview.md).