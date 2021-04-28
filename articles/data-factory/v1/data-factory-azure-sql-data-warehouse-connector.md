---
title: Copia de datos en y desde Azure Synapse Analytics
description: Información sobre cómo copiar datos en y desde Azure Synapse Analytics con Azure Data Factory
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/10/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 35aa0110106f121adb9373f83ff29d299ce049fc
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108135984"
---
# <a name="copy-data-to-and-from-azure-synapse-analytics-using-azure-data-factory"></a>Copia de datos en y desde Azure Synapse Analytics con Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-azure-sql-data-warehouse-connector.md)
> * [Versión 2 (versión actual)](../connector-azure-sql-data-warehouse.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Conector de Azure Synapse Analytics en V2](../connector-azure-sql-data-warehouse.md).

En este artículo se explica el uso de la actividad de copia en Azure Data Factory para mover datos hacia Azure Synapse Analytics y desde este servicio. Se basa en la información general que ofrece el artículo [Movimiento de datos con la actividad de copia](data-factory-data-movement-activities.md).

> [!TIP]
> Para obtener el mejor rendimiento posible, use PolyBase para cargar datos en Azure Synapse Analytics. Encontrará más detalles en la sección [Usar PolyBase para cargar datos en SQL Data Warehouses](#use-polybase-to-load-data-into-azure-synapse-analytics). Para un tutorial con un caso de uso, vea [Carga de datos en Azure SQL Data Warehouse mediante Azure Data Factory](data-factory-load-sql-data-warehouse.md).

## <a name="supported-scenarios"></a>Escenarios admitidos
Puede copiar datos **desde Azure Synapse Analytics** a los siguientes almacenes de datos:

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Puede copiar datos de los siguientes almacenes de datos **a Azure Synapse Analytics**:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!TIP]
> Al copiar datos desde SQL Server o Azure SQL Database en Azure Synapse Analytics, si la tabla no se encuentra en el almacén de destino, Data Factory puede crearla automáticamente en Azure Synapse Analytics mediante el esquema de la tabla del almacén de datos de origen. Vea [Creación automática de tablas](#auto-table-creation) para obtener más información.

## <a name="supported-authentication-type"></a>Tipos de autenticación que se admiten
El conector de Azure Synapse Analytics admite la autenticación básica.

## <a name="getting-started"></a>Introducción
Puede crear una canalización con actividad de copia que mueva datos a una instancia de Azure Synapse Analytics o desde allí mediante diferentes herramientas o API.

La manera más sencilla de crear una canalización que copie datos en Azure Synapse Analytics o desde ese servicio es usar el Asistente para copia de datos. Consulte [Tutorial: Carga de datos en Synapse Analytics con Data Factory](../load-azure-sql-data-warehouse.md) para una rápida visita guiada sobre la creación de una canalización mediante el asistente para copia de datos.

Puede usar las siguientes herramientas para crear una canalización: **Visual Studio**, **Azure PowerShell**, una **plantilla de Azure Resource Manager**, la **API de .NET** y **API REST**. Consulte el [tutorial de actividad de copia](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) para obtener instrucciones paso a paso para crear una canalización con una actividad de copia.

Tanto si usa las herramientas como las API, realice los pasos siguientes para crear una canalización que mueva datos de un almacén de datos de origen a un almacén de datos receptor:

1. Crear una **factoría de datos**. Una factoría de datos puede contener una o más canalizaciones. 
2. Cree **servicios vinculados** para vincular almacenes de datos de entrada y salida a la factoría de datos. Por ejemplo, si va a copiar datos desde una cuenta de Azure Blob Storage en una instancia de Azure Synapse Analytics, vinculará los dos servicios, con la cuenta de Azure Storage y la instancia de Azure Synapse Analytics vinculados a la factoría de datos. Para información sobre las propiedades de los servicios vinculados que son específicas de Azure Synapse Analytics, consulte la sección [Propiedades del servicio vinculado](#linked-service-properties). 
3. Cree **conjuntos de datos** con el fin de representar los datos de entrada y salida para la operación de copia. En el ejemplo mencionado en el último paso, se crea un conjunto de datos para especificar el contenedor de blobs y la carpeta que contiene los datos de entrada. Además, se crea otro conjunto de datos para especificar la tabla en la instancia de Azure Synapse Analytics que contiene los datos copiados del almacenamiento de blobs. Para información sobre las propiedades del conjunto de datos que son específicas de Azure Synapse Analytics, consulte la sección [Propiedades del conjunto de datos](#dataset-properties).
4. Cree una **canalización** con una actividad de copia que tome como entrada un conjunto de datos y un conjunto de datos como salida. En el ejemplo que se ha mencionado anteriormente, se usa BlobSource como origen y SqlSink como receptor para la actividad de copia. De igual forma, si va a copiar desde Azure Synapse Analytics en Azure Blob Storage, usará SqlDWSource y BlobSink en la actividad de copia. Para información sobre las propiedades de la actividad de copia específicas de Azure Synapse Analytics, consulte la sección [Propiedades de la actividad de copia](#copy-activity-properties). Para obtener más información sobre cómo usar un almacén de datos como origen o receptor, haga clic en el vínculo de la sección anterior para el almacén de datos.

Cuando se usa el Asistente, se crean automáticamente definiciones de JSON para estas entidades de Data Factory (servicios vinculados, conjuntos de datos y la canalización). Al usar herramientas o API (excepto la API de .NET), se definen estas entidades de Data Factory con el formato JSON. Para ejemplos con definiciones de JSON para las entidades de Data Factory que se utilizan para copiar datos en Azure Synapse Analytics o desde ese servicio, consulte la sección [Ejemplos de JSON](#json-examples-for-copying-data-to-and-from-azure-synapse-analytics) de este artículo.

En las secciones siguientes se proporcionan detalles sobre las propiedades JSON que se usan para definir las entidades de Data Factory específicas de Azure Synapse Analytics:

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado
En la tabla siguiente se proporciona la descripción de los elementos JSON específicos del servicio vinculado Azure Synapse Analytics.

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| type |La propiedad type debe establecerse en: **AzureSqlDW** |Sí |
| connectionString |Especifique la información necesaria para conectarse a la instancia de Azure Synapse Analytics para la propiedad connectionString. Solo se admite la autenticación básica. |Sí |

> [!IMPORTANT]
> Configure el [firewall de Azure SQL Database](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure) y el servidor de bases de datos para [permitir que los servicios de Azure tengan acceso al servidor](/previous-versions/azure/ee621782(v=azure.100)#ConnectingFromAzure). Además, si va a copiar datos en Azure Synapse Analytics desde fuera de Azure, incluidos orígenes de datos locales con puerta de enlace de la factoría de datos, configure el intervalo de direcciones IP adecuado para a máquina que envía datos a Azure Synapse Analytics.

## <a name="dataset-properties"></a>Propiedades del conjunto de datos
Para una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, vea el artículo [Creación de conjuntos de datos](data-factory-create-datasets.md). Las secciones como structure, availability y policy del código JSON del conjunto de datos son similares para todos los tipos de conjunto de datos (SQL Azure, blob de Azure, tabla de Azure, etc.).

La sección typeProperties es diferente en cada tipo de conjunto de datos y proporciona información acerca de la ubicación de los datos en el almacén de datos. La sección **typeProperties** del conjunto de datos de tipo **AzureSqlDWTable** tiene las propiedades siguientes:

| Propiedad | Descripción | Obligatorio |
| --- | --- | --- |
| tableName |Nombre de la tabla o vista en la base de datos de Azure Synapse Analytics a la que hace referencia el servicio vinculado. |Sí |

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia
Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo [Creación de canalizaciones](data-factory-create-pipelines.md). Las propiedades (como nombre, descripción, tablas de entrada y salida, y directivas) están disponibles para todos los tipos de actividades.

> [!NOTE]
> La actividad de copia toma solo una entrada y genera una única salida.

Por otra parte, las propiedades disponibles en la sección typeProperties de la actividad varían con cada tipo de actividad. Para la actividad de copia, varían en función de los tipos de orígenes y receptores.

### <a name="sqldwsource"></a>SqlDWSource
Si el origen es de tipo **SqlDWSource**, estarán disponibles las propiedades siguientes en la sección **typeProperties**:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlReaderQuery |Utilice la consulta personalizada para leer los datos. |Cadena de consulta SQL. Por ejemplo: select * from MyTable. |No |
| sqlReaderStoredProcedureName |Nombre del procedimiento almacenado que lee datos de la tabla de origen. |Nombre del procedimiento almacenado. La última instrucción SQL debe ser una instrucción SELECT del procedimiento almacenado. |No |
| storedProcedureParameters |Parámetros del procedimiento almacenado. |Pares nombre-valor. Los nombres y las mayúsculas y minúsculas de los parámetros deben coincidir con las mismas características de los parámetros de procedimiento almacenado. |No |

Si se especifica **sqlReaderQuery** para SqlDWSource, la actividad de copia ejecuta esta consulta en el origen Azure Synapse Analytics para obtener los datos.

Como alternativa, puede indicar un procedimiento almacenado mediante la especificación de **sqlReaderStoredProcedureName** y **storedProcedureParameters** (si el procedimiento almacenado toma parámetros).

Si no especifica sqlReaderQuery ni sqlReaderStoredProcedureName, las columnas definidas en la sección sobre la estructura del conjunto de datos JSON se usan para crear una consulta y ejecutarla en Azure Synapse Analytics. Ejemplo: `select column1, column2 from mytable`. Si la definición del conjunto de datos no tiene la estructura, se seleccionan todas las columnas de la tabla.

#### <a name="sqldwsource-example"></a>Ejemplo de SqlDWSource

```JSON
"source": {
    "type": "SqlDWSource",
    "sqlReaderStoredProcedureName": "CopyTestSrcStoredProcedureWithParameters",
    "storedProcedureParameters": {
        "stringData": { "value": "str3" },
        "identifier": { "value": "$$Text.Format('{0:yyyy}', SliceStart)", "type": "Int"}
    }
}
```
**Definición del procedimiento almacenado:**

```SQL
CREATE PROCEDURE CopyTestSrcStoredProcedureWithParameters
(
    @stringData varchar(20),
    @identifier int
)
AS
SET NOCOUNT ON;
BEGIN
    select *
    from dbo.UnitTestSrcTable
    where dbo.UnitTestSrcTable.stringData != stringData
    and dbo.UnitTestSrcTable.identifier != identifier
END
GO
```

### <a name="sqldwsink"></a>SqlDWSink
**SqlDWSink** admite las siguientes propiedades:

| Propiedad | Descripción | Valores permitidos | Obligatorio |
| --- | --- | --- | --- |
| sqlWriterCleanupScript |Especifique una consulta para que se ejecute la actividad de copia de tal forma que se limpien los datos de un segmento específico. Consulte la [sección sobre repetibilidad](#repeatability-during-copy)para más información. |Una instrucción de consulta. |No |
| allowPolyBase |Indica si se usa PolyBase (si procede) en lugar del mecanismo BULKINSERT. <br/><br/> **El uso de PolyBase es el método recomendado para cargar datos en Azure Synapse Analytics.** Consulte la sección [Uso de PolyBase para cargar datos en Azure Synapse Analytics](#use-polybase-to-load-data-into-azure-synapse-analytics) para restricciones y más información. |True <br/>False (valor predeterminado) |No |
| polyBaseSettings |Un grupo de propiedades que se pueden especificar si el valor de la propiedad **allowPolybase** está establecido en **true**. |&nbsp; |No |
| rejectValue |Especifica el número o porcentaje de filas que se pueden rechazar antes de que se produzca un error en la consulta. <br/><br/>Más información sobre las opciones de rechazo de PolyBase en la sección **Argumentos** del tema [CREATE EXTERNAL TABLE (Transact-SQL)](/sql/t-sql/statements/create-external-table-transact-sql). |0 (predeterminado), 1, 2, … |No |
| rejectType |Especifica si se indica la opción rejectValue como un valor literal o un porcentaje. |Valor (predeterminado), Porcentaje |No |
| rejectSampleValue |Determina el número de filas que se van a recuperar antes de que PolyBase vuelva a calcular el porcentaje de filas rechazadas. |1, 2, … |Sí, si el valor de **rejectType** es **percentage** |
| useTypeDefault |Especifica cómo administrar valores que faltan en archivos de texto delimitado cuando PolyBase recupera datos del archivo de texto.<br/><br/>Más información sobre esta propiedad en la sección de argumentos de [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql). |True, False (predeterminada) |No |
| writeBatchSize |Inserta datos en la tabla SQL cuando el tamaño del búfer alcance writeBatchSize |Entero (número de filas) |No (valor predeterminado: 10000) |
| writeBatchTimeout |Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera. |timespan<br/><br/> Ejemplo: "00:30:00" (30 minutos). |No |

#### <a name="sqldwsink-example"></a>Ejemplo de SqlDWSink

```JSON
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true
}
```

## <a name="use-polybase-to-load-data-into-azure-synapse-analytics"></a>Uso de PolyBase para cargar datos en Azure Synapse Analytics
Usar **[PolyBase](/sql/relational-databases/polybase/polybase-guide)** es una manera eficaz de cargar grandes cantidades de datos en Azure Synapse Analytics con un alto rendimiento. Puede ver una gran mejora en el rendimiento mediante el uso de PolyBase en lugar del mecanismo BULKINSERT predeterminado. Vea la [copia del número de referencia de rendimiento](data-factory-copy-activity-performance.md#performance-reference) con una comparación detallada. Para un tutorial con un caso de uso, vea [Carga de datos en Azure SQL Data Warehouse mediante Azure Data Factory](data-factory-load-sql-data-warehouse.md).

* Si los datos de origen están en **Azure Blob o Azure Data Lake Store**, y el formato es compatible con PolyBase, puede realizar las copias directamente en Azure Synapse Analytics mediante PolyBase. Consulte **[Copias directas con PolyBase](#direct-copy-using-polybase)** para detalles.
* Si el formato y el almacenamiento de datos de origen no es compatible originalmente con PolyBase, puede usar en su lugar la característica **[Copias almacenadas provisionalmente con PolyBase](#staged-copy-using-polybase)** . También proporciona un mejor rendimiento al convertir automáticamente los datos en formato compatible con PolyBase y almacenarlos en Azure Blob Storage. Después, carga los datos en Azure Synapse Analytics.

En la propiedad `allowPolyBase`, elija **true** como se muestra en el ejemplo siguiente de Azure Data Factory para usar PolyBase con el fin de copiar datos en Azure Synapse Analytics. Cuando se establece allowPolyBase en true, se puede especificar las propiedades específicas de PolyBase mediante el grupo de propiedades `polyBaseSettings`. Consulte la sección [SqlDWSink](#sqldwsink) para más información acerca de las propiedades que puede utilizar con polyBaseSettings.

```JSON
"sink": {
    "type": "SqlDWSink",
    "allowPolyBase": true,
    "polyBaseSettings":
    {
        "rejectType": "percentage",
        "rejectValue": 10.0,
        "rejectSampleValue": 100,
        "useTypeDefault": true
    }
}
```

### <a name="direct-copy-using-polybase"></a>Copias directas con PolyBase
PolyBase de Azure Synapse Analytics admite directamente Azure Blob y Azure Data Lake Store (con la entidad de servicio) como origen y con requisitos de formato de archivo específicos. Si el origen de datos cumple los criterios descritos en esta sección, puede realizar las copias directamente desde el almacén de datos de origen en Azure Synapse Analytics mediante PolyBase. De lo contrario, puede usar [Copias almacenadas provisionalmente con PolyBase](#staged-copy-using-polybase).

> [!TIP]
> Para copiar datos desde Data Lake Store en Azure Synapse Analytics de forma más eficiente, obtenga más información en [Azure Data Factory hace incluso más fácil y cómodo el descubrimiento de información de datos cuando se usa Data Lake Store con Azure Synapse Analytics](/archive/blogs/azuredatalake/azure-data-factory-makes-it-even-easier-and-convenient-to-uncover-insights-from-data-when-using-data-lake-store-with-sql-data-warehouse).

Si no se cumplen los requisitos, Azure Data Factory comprobará la configuración y volverá automáticamente al mecanismo BULKINSERT para realizar el movimiento de datos.

1. El **servicio vinculado de origen** es del tipo: **AzureStorage** o **AzureDataLakeStore con autenticación de la entidad de servicio**.
2. El **conjunto de datos de entrada** es del tipo: **AzureBlob** o **AzureDataLakeStore**, y el tipo de formato de las propiedades `type` es **OrcFormat**, **ParquetFormat** o **TextFormat** con las siguientes configuraciones:

   1. `rowDelimiter` debe ser **\n**.
   2. `nullValue` se establece en **una cadena vacía** ("") o `treatEmptyAsNull` se establece en **true**.
   3. `encodingName` se establece en **utf-8**, que es el valor **predeterminado**.
   4. `escapeChar`, `quoteChar`, `firstRowAsHeader` y `skipLineCount` no se especifican.
   5. `compression` puede ser **no compression**, **GZip** o **Deflate**.

      ```JSON
      "typeProperties": {
       "folderPath": "<blobpath>",
       "format": {
           "type": "TextFormat",
           "columnDelimiter": "<any delimiter>",
           "rowDelimiter": "\n",
           "nullValue": "",
           "encodingName": "utf-8"
       },
       "compression": {
           "type": "GZip",
           "level": "Optimal"
       }
      },
      ```

3. No hay ningún valor `skipHeaderLineCount` en **BlobSource** o **AzureDataLakeStore** para la actividad de copia en la canalización.
4. No hay ningún valor `sliceIdentifierColumnName` en **SqlDWSink** para la actividad de copia en la canalización. (PolyBase garantiza que todos los datos se actualizan o que no se actualiza ninguno en una única ejecución. Para lograr la **capacidad de repetición**, se puede usar `sqlWriterCleanupScript`).
5. No se usa `columnMapping` en la actividad de copia asociada.

### <a name="staged-copy-using-polybase"></a>Copias almacenadas provisionalmente con PolyBase
Si los datos de origen no cumplen los criterios especificados en la sección anterior, puede habilitar la copia de datos a través de un almacenamiento provisional de Azure Blob Storage (no puede ser Premium Storage). En este caso, Azure Data Factory realiza transformaciones automáticamente en los datos para que cumplan los requisitos de formato de los datos de PolyBase y lo usa para cargar datos en Azure Synapse Analytics; por último, borra los datos temporales de Blob Storage. Consulte [Copias almacenadas provisionalmente](data-factory-copy-activity-performance.md#staged-copy) para más información sobre cómo funciona en general la copia de datos por medio de un blob de Azure de almacenamiento provisional.

> [!NOTE]
> Al copiar datos de un almacén de datos local en Azure Synapse Analytics mediante PolyBase y el almacenamiento provisional, si la versión de Data Management Gateway es anterior a la 2.4, se necesita JRE (Java Runtime Environment) en la máquina de la puerta de enlace que se usa para transformar los datos de origen en un formato correcto. Se recomienda actualizar la puerta de enlace a la versión más reciente para evitar esta dependencia.
>

Para usar esta característica, cree un [servicio vinculado de Azure Storage](data-factory-azure-blob-connector.md#azure-storage-linked-service) que haga referencia a la cuenta de Azure Storage que tenga el almacenamiento de blobs provisional y, después, especifique las propiedades `enableStaging` y `stagingSettings` de la actividad de copia como se muestra en el siguiente código:

```json
"activities":[
{
    "name": "Sample copy activity from SQL Server to Azure Synapse Analytics via PolyBase",
    "type": "Copy",
    "inputs": [{ "name": "OnpremisesSQLServerInput" }],
    "outputs": [{ "name": "AzureSQLDWOutput" }],
    "typeProperties": {
        "source": {
            "type": "SqlSource",
        },
        "sink": {
            "type": "SqlDwSink",
            "allowPolyBase": true
        },
        "enableStaging": true,
        "stagingSettings": {
            "linkedServiceName": "MyStagingBlob"
        }
    }
}
]
```

## <a name="best-practices-when-using-polybase"></a>Procedimientos recomendados al usar PolyBase
En las secciones siguientes se proporcionan procedimientos recomendados adicionales a los que se mencionan en [Procedimientos recomendados para Azure Synapse Analytics](../../synapse-analytics/sql/best-practices-dedicated-sql-pool.md).

### <a name="required-database-permission"></a>Permiso de base de datos necesario
El uso de PolyBase requiere que el usuario que se usa para cargar datos en Azure Synapse Analytics tenga [permiso "CONTROL"](/sql/relational-databases/security/permissions-database-engine) en la base de datos de destino. Una manera de conseguirlo es agregar ese usuario como miembro del rol "db_owner". Obtenga información sobre cómo hacerlo siguiendo [esta sección](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-manage-security.md#authorization).

### <a name="row-size-and-data-type-limitation"></a>Limitación del tipo de datos y del tamaño de fila
Las cargas de PolyBase se limitan a la carga de filas más pequeñas que **1 MB** y no pueden cargar en VARCHR(MAX), NVARCHAR(MAX) ni VARBINARY(MAX). O haga clic [aquí](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-service-capacity-limits.md#loads).

Si tiene datos de origen con filas mayores de 1 MB, puede dividir las tablas de origen verticalmente en varias más pequeñas, donde el tamaño de fila mayor de cada una de ellas no supere el límite. Las tablas más pequeñas se pueden cargar con PolyBase y combinar en Azure Synapse Analytics.

### <a name="azure-synapse-analytics-resource-class"></a>Clase de recurso de Azure Synapse Analytics
Para obtener el mejor rendimiento posible, considere la posibilidad de asignar una clase de recurso mayor al usuario que se usa para cargar daos en Azure Synapse Analytics mediante PolyBase. Infórmese acerca de cómo hacerlo siguiendo las instrucciones en [Cambio de ejemplo de clase de recursos de usuario](../../synapse-analytics/sql-data-warehouse/resource-classes-for-workload-management.md).

### <a name="tablename-in-azure-synapse-analytics"></a>tableName en Azure Synapse Analytics
En la tabla siguiente se proporcionan ejemplos sobre cómo especificar la propiedad **tableName** en el conjunto de datos JSON para diversas combinaciones de nombre de tabla y esquema.

| Esquema de base de datos | Nombre de la tabla | Propiedad JSON tableName |
| --- | --- | --- |
| dbo |MyTable |MyTable o dbo.MyTable o [dbo].[MyTable] |
| dbo1 |MyTable |dbo1.MyTable o [dbo1].[MyTable] |
| dbo |My.Table |[My.Table] o [dbo].[My.Table] |
| dbo1 |My.Table |[dbo1].[My.Table] |

Si ve el siguiente error, podría deberse a un problema con el valor especificado para la propiedad tableName. Consulte en la tabla la forma correcta de especificar los valores para la propiedad tableName de JSON.

```
Type=System.Data.SqlClient.SqlException,Message=Invalid object name 'stg.Account_test'.,Source=.Net SqlClient Data Provider
```

### <a name="columns-with-default-values"></a>Columnas con valores predeterminados
Actualmente, la característica PolyBase en Data Factory solo acepta el mismo número de columnas que la tabla de destino. Supongamos que tiene una tabla con cuatro columnas y una de ellas está definida con un valor predeterminado. Los datos de entrada deberían seguir conteniendo cuatro columnas. Si se proporciona un conjunto de datos de entrada de tres columnas, se producirá un error parecido al siguiente mensaje:

```
All columns of the table must be specified in the INSERT BULK statement.
```
El valor NULL es una forma especial de valor predeterminado. Si la columna admite valores NULL, los datos de entrada (en el blob) para esa columna pueden estar vacíos (no pueden faltar en el conjunto de datos de entrada). PolyBase insertará valores NULL para ellos en Azure Synapse Analytics.

## <a name="auto-table-creation"></a>Creación automáticamente de tablas
Si usa el Asistente para copia para copiar datos desde SQL Server o Azure SQL Database en Azure Synapse Analytics y la tabla que corresponde a la tabla de origen no existe en el almacén de destino, Data Factory puede crear la tabla automáticamente en el almacenamiento de datos mediante el esquema de la tabla de origen.

Data Factory crea la tabla en el almacén de destino con el mismo nombre de tabla del almacén de datos de origen. Los tipos de datos de columnas se eligen en función de la siguiente asignación de tipo. Si es necesario, realiza conversiones de tipos para corregir las incompatibilidades entre los almacenes de origen y de destino. También se utiliza la distribución Round Robin.

| Tipo de columna de SQL Database de origen | Tipo de columna de Azure Synapse Analytics de destino (límite de tamaño) |
| --- | --- |
| Int | Int |
| BigInt | BigInt |
| SmallInt | SmallInt |
| TinyInt | TinyInt |
| bit | bit |
| Decimal | Decimal |
| Numeric | Decimal |
| Float | Float |
| Money | Money |
| Real | Real |
| SmallMoney | SmallMoney |
| Binary | Binary |
| Varbinary | Varbinary (hasta 8000) |
| Date | Date |
| DateTime | DateTime |
| DateTime2 | DateTime2 |
| Time | Time |
| DateTimeOffset | DateTimeOffset |
| SmallDateTime | SmallDateTime |
| Texto | Varchar (hasta 8000) |
| NText | NVarChar (hasta 4000) |
| Imagen | VarBinary (hasta 8000) |
| UniqueIdentifier | UniqueIdentifier |
| Char | Char |
| NChar | NChar |
| VarChar | VarChar (hasta 8000) |
| NVarChar | NVarChar (hasta 4000) |
| Xml | Varchar (hasta 8000) |

[!INCLUDE [data-factory-type-repeatability-for-sql-sources](../../../includes/data-factory-type-repeatability-for-sql-sources.md)]

## <a name="type-mapping-for-azure-synapse-analytics"></a>Asignación de tipos para Azure Synapse Analytics
Como se mencionó en el artículo sobre [actividades del movimiento de datos](data-factory-data-movement-activities.md) , la actividad de copia realiza conversiones automáticas de los tipos de origen a los tipos de receptor con el siguiente enfoque de dos pasos:

1. Conversión de tipos de origen nativos al tipo .NET
2. Conversión de tipo .NET al tipo del receptor nativo

Al migrar datos a Azure Synapse Analytics y desde allí, se usarán las siguientes asignaciones del tipo SQL al tipo .NET, y viceversa.

La asignación es igual que la asignación de [tipo de datos de SQL Server para ADO.NET](/dotnet/framework/data/adonet/sql-server-data-type-mappings).

| Tipo de motor de base de datos SQL Server | Tipo de .NET Framework |
| --- | --- |
| bigint |Int64 |
| binary |Byte[] |
| bit |Boolean |
| char |String, Char[] |
| date |DateTime |
| Datetime |DateTime |
| datetime2 |DateTime |
| Datetimeoffset |DateTimeOffset |
| Decimal |Decimal |
| FILESTREAM attribute (varbinary(max)) |Byte[] |
| Float |Double |
| imagen |Byte[] |
| int |Int32 |
| money |Decimal |
| NCHAR |String, Char[] |
| ntext |String, Char[] |
| NUMERIC |Decimal |
| NVARCHAR |String, Char[] |
| real |Single |
| rowversion |Byte[] |
| smalldatetime |DateTime |
| SMALLINT |Int16 |
| SMALLMONEY |Decimal |
| sql_variant |Object * |
| text |String, Char[] |
| time |TimeSpan |
| timestamp |Byte[] |
| TINYINT |Byte |
| UNIQUEIDENTIFIER |Guid |
| varbinary |Byte[] |
| varchar |String, Char[] |
| Xml |Xml |

También puede asignar columnas del conjunto de datos de origen a las del conjunto de datos receptor en la definición de actividad de copia. Para más información, consulte [Mapping dataset columns in Azure Data Factory](data-factory-map-columns.md) (Asignación de columnas de conjunto de datos de Azure Data Factory).

## <a name="json-examples-for-copying-data-to-and-from-azure-synapse-analytics"></a>Ejemplos de JSON para copiar datos a Azure Synapse Analytics y desde allí
En los siguientes ejemplos, se proporcionan definiciones JSON de ejemplo que puede usar para crear una canalización mediante [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) o [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Muestran cómo copiar datos entre Azure Blob Storage y Azure Synapse Analytics. Sin embargo, los datos se pueden copiar **directamente** de cualquiera de los orígenes a cualquiera de los receptores indicados [aquí](data-factory-data-movement-activities.md#supported-data-stores-and-formats) mediante la actividad de copia en Azure Data Factory.

### <a name="example-copy-data-from-azure-synapse-analytics-to-azure-blob"></a>Ejemplo: Copiar datos desde Azure Synapse Analytics en Azure blob.
El ejemplo define las siguientes entidades de Data Factory:

1. Un servicio vinculado de tipo [AzureSqlDW](#linked-service-properties).
2. Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
3. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureSqlDWTable](#dataset-properties).
4. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [SqlDWSource](#copy-activity-properties) y [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

El ejemplo copia los datos que pertenecen a una serie temporal (por horas, días, etc.) desde una tabla de una base de datos de Azure Synapse Analytics a un blob cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado Azure Synapse Analytics:**

```JSON
{
  "name": "AzureSqlDWLinkedService",
  "properties": {
    "type": "AzureSqlDW",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Servicio vinculado de Azure Blob Storage:**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Conjunto de datos de entrada de Azure Synapse Analytics:**

En el ejemplo se supone que ha creado una tabla "MyTable" en Azure Synapse Analytics y que contiene una columna denominada "timestampcolumn" para los datos de series temporales.

Si se establece "external": "true", se informa al servicio Data Factory de que el conjunto de datos es externo a la factoría de datos y que no lo genera ninguna actividad de la factoría de datos.

```JSON
{
  "name": "AzureSqlDWInput",
  "properties": {
    "type": "AzureSqlDWTable",
    "linkedServiceName": "AzureSqlDWLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Conjunto de datos de salida de blob de Azure:**

Los datos se escriben en un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta para el blob se evalúa dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa las partes year, month, day y hours de la hora de inicio.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Actividad de copia en una canalización con SqlDWSource y BlobSink:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **SqlDWSource** y el tipo **sink**, en **BlobSink**. La consulta SQL especificada para la propiedad **SqlReaderQuery** selecciona los datos de la última hora que se van a copiar.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "AzureSQLDWtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSqlDWInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlDWSource",
            "sqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd HH:mm}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd HH:mm}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
> [!NOTE]
> En el ejemplo, **sqlReaderQuery** se especifica para SqlDWSource. La actividad de copia ejecuta esta consulta en el origen Azure Synapse Analytics para obtener los datos.
>
> Como alternativa, puede indicar un procedimiento almacenado mediante la especificación de **sqlReaderStoredProcedureName** y **storedProcedureParameters** (si el procedimiento almacenado toma parámetros).
>
> Si no especifica sqlReaderQuery ni sqlReaderStoredProcedureName, las columnas definidas en la sección sobre la estructura del conjunto de datos JSON se usan para crear una consulta (seleccione column1, column2 en mytable) y ejecutarla en Azure Synapse Analytics. Si la definición del conjunto de datos no tiene la estructura, se seleccionan todas las columnas de la tabla.
>
>

### <a name="example-copy-data-from-azure-blob-to-azure-synapse-analytics"></a>Ejemplo: Copiar datos desde Azure Blob en Azure Synapse Analytics.
El ejemplo define las siguientes entidades de Data Factory:

1. Un servicio vinculado de tipo [AzureSqlDW](#linked-service-properties).
2. Un servicio vinculado de tipo [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties)
3. Un [conjunto de datos](data-factory-create-datasets.md) de entrada de tipo [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
4. Un [conjunto de datos](data-factory-create-datasets.md) de salida de tipo [AzureSqlDWTable](#dataset-properties).
5. Una [canalización](data-factory-create-pipelines.md) con la actividad de copia que usa [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) y [SqlDWSink](#copy-activity-properties).

En el ejemplo, los datos de la serie temporal (por horas, días, etc.) se copian de un blob de Azure en una tabla de Azure Synapse Analytics cada hora. Las propiedades JSON usadas en estos ejemplos se describen en las secciones que aparecen después de los ejemplos.

**Servicio vinculado Azure Synapse Analytics:**

```JSON
{
  "name": "AzureSqlDWLinkedService",
  "properties": {
    "type": "AzureSqlDW",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```
**Servicio vinculado de Azure Blob Storage:**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```
**Conjunto de datos de entrada de blob de Azure:**

Los datos se seleccionan de un nuevo blob cada hora (frecuencia: hora, intervalo: 1). La ruta de acceso de la carpeta y el nombre de archivo para el blob se evalúan dinámicamente según la hora de inicio del segmento que se está procesando. La ruta de acceso de la carpeta usa la parte year, month y day de la hora de inicio y el nombre de archivo, la parte hour. El valor "external": "true" informa al servicio Data Factory que esta tabla es externa a la factoría de datos y no la produce una actividad de la factoría de datos.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": ",",
        "rowDelimiter": "\n"
      }
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```
**Conjunto de datos de salida de Azure Synapse Analytics:**

En el ejemplo, los datos se copian en una tabla denominada "MyTable" en Azure Synapse Analytics. Cree la tabla en Azure Synapse Analytics con el mismo número de columnas que espera que contenga el archivo CSV del blob. Se agregan nuevas filas a la tabla cada hora.

```JSON
{
  "name": "AzureSqlDWOutput",
  "properties": {
    "type": "AzureSqlDWTable",
    "linkedServiceName": "AzureSqlDWLinkedService",
    "typeProperties": {
      "tableName": "MyOutputTable"
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```
**Actividad de copia en una canalización con BlobSource y SqlDWSink:**

La canalización contiene una actividad de copia que está configurada para usar los conjuntos de datos de entrada y de salida y está programada para ejecutarse cada hora. En la definición de JSON de canalización, el tipo **source** se establece en **BlobSource** y el tipo **sink**, en **SqlDWSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[
      {
        "name": "AzureBlobtoSQLDW",
        "description": "Copy Activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureBlobInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureSqlDWOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "BlobSource",
            "blobColumnSeparators": ","
          },
          "sink": {
            "type": "SqlDWSink",
            "allowPolyBase": true
          }
        },
        "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```
Para ver un tutorial, consulte los artículos [Carga de 1 TB en Azure Synapse Analytics en menos de 15 minutos con Azure Data Factory](data-factory-load-sql-data-warehouse.md) y [Carga de datos con Data Factory](../load-azure-sql-data-warehouse.md) en la documentación de Azure Synapse Analytics.

## <a name="performance-and-tuning"></a>Rendimiento y optimización
Consulte [Guía de optimización y rendimiento de la actividad de copia](data-factory-copy-activity-performance.md) para más información sobre los factores clave que afectan al rendimiento del movimiento de datos (actividad de copia) en Azure Data Factory y las diversas formas de optimizarlo.