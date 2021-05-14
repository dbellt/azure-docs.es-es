---
title: Copia y transformación de datos en Azure Database for MySQL
description: Aprenda a copiar y transformar datos en Azure Database for MySQL mediante Azure Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/10/2021
ms.openlocfilehash: 69ed45d50c98321c68e55aa15304e6d439fe58ad
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109480264"
---
# <a name="copy-and-transform-data-in-azure-database-for-mysql-by-using-azure-data-factory"></a>Copia y transformación de datos en Azure Database for MySQL mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia en Azure Data Factory para copiar datos desde Azure Database for MySQL y hacia allí, y el uso de Data Flow para transformarlos en Azure Database for MySQL. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).

Ese conector se usa especialmente para el [servicio Azure Database for MySQL](../mysql/overview.md). Para copiar datos desde una base de datos MySQL genérica ubicada en el entorno local o en la nube, use el [conector MySQL](connector-mysql.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Database for MySQL es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas para el conector de Azure Database for MySQL.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Database for MySQL:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AzureMySql** | Sí |
| connectionString | Especifique la información necesaria para conectarse a la instancia de Azure Database for MySQL. <br/> También puede colocar la contraseña en Azure Key Vault y extraer la configuración de `password` de la cadena de conexión. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. | Sí |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar los entornos Integration Runtime (autohospedado) (si el almacén de datos se encuentra en una red privada) o Azure Integration Runtime. Si no se especifica, se usará Azure Integration Runtime. |No |

Una cadena de conexión típica es `Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;PWD=<password>`. Más propiedades que puede establecer para su caso:

| Propiedad | Descripción | Opciones | Obligatorio |
|:--- |:--- |:--- |:--- |
| SSLMode | Esta opción especifica si el controlador utiliza cifrado TLS y comprobación durante la conexión a MySQL. Por ejemplo, `SSLMode=<0/1/2/3/4>`| DISABLED (0) / PREFERRED (1) **(valor predeterminado)** / REQUIRED (2) / VERIFY_CA (3) / VERIFY_IDENTITY (4) | No |
| UseSystemTrustStore | Esta opción concreta si se usa un certificado de entidad de certificación del almacén de confianza del sistema o de un archivo PEM especificado. Por ejemplo, `UseSystemTrustStore=<0/1>;`| Habilitado (1) / Deshabilitado (0) **(valor predeterminado)** | No |

**Ejemplo**:

```json
{
    "name": "AzureDatabaseForMySQLLinkedService",
    "properties": {
        "type": "AzureMySql",
        "typeProperties": {
            "connectionString": "Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;PWD=<password>"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Ejemplo: Almacenamiento de la contraseña en Azure Key Vault**

```json
{
    "name": "AzureDatabaseForMySQLLinkedService",
    "properties": {
        "type": "AzureMySql",
        "typeProperties": {
            "connectionString": "Server=<server>.mysql.database.azure.com;Port=<port>;Database=<database>;UID=<username>;",
            "password": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<Azure Key Vault linked service name>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<secretName>" 
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Azure Database for MySQL.

Para copiar datos desde Azure Database for MySQL, establezca la propiedad type del conjunto de datos en **AzureMySqlTable**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **AzureMySqlTable** | Sí |
| tableName | Nombre de la tabla de la base de datos MySQL. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "AzureMySQLDataset",
    "properties": {
        "type": "AzureMySqlTable",
        "linkedServiceName": {
            "referenceName": "<Azure MySQL linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "tableName": "<table name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite Azure Database for MySQL como origen y receptor.

### <a name="azure-database-for-mysql-as-source"></a>Azure Database for MySQL como origen

Para copiar datos desde Azure Database for MySQL, se admiten las siguientes propiedades en la sección de **origen** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **AzureMySqlSource** | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM MyTable"`. | No (si se especifica "tableName" en el conjunto de datos) |
| queryCommandTimeout | El tiempo de espera antes de que se agote el tiempo de espera de la solicitud de consulta. El valor predeterminado es 120 minutos (02:00:00). | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromAzureDatabaseForMySQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Azure MySQL input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AzureMySqlSource",
                "query": "<custom query e.g. SELECT * FROM MyTable>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="azure-database-for-mysql-as-sink"></a>Azure Database for MySQL como receptor

Para copiar datos en Azure Database for MySQL, se admiten las siguientes propiedades en la sección de **receptor** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del receptor de la actividad de copia debe establecerse en: **AzureMySqlSink** | Sí |
| preCopyScript | Especifique una consulta SQL para que la actividad de copia se ejecute antes de escribir datos en Azure Database for MySQL en cada ejecución. Puede usar esta propiedad para limpiar los datos cargados previamente. | No |
| writeBatchSize | Inserta datos en la tabla de Azure Database for MySQL cuando el tamaño de búfer alcanza el valor de writeBatchSize.<br>El valor permitido es un entero que representa el número de filas. | No (el valor predeterminado es 10 000) |
| writeBatchTimeout | Tiempo de espera para que la operación de inserción por lotes se complete antes de que se agote el tiempo de espera.<br>Los valores permitidos son intervalos de tiempo. Un ejemplo es 00:30:00 (30 minutos). | No (el valor predeterminado es 00:00:30) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToAzureDatabaseForMySQL",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure MySQL output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureMySqlSink",
                "preCopyScript": "<custom SQL script>",
                "writeBatchSize": 100000
            }
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Al transformar datos en el flujo de datos de asignación, puede leer tablas y escribir en ellas desde Azure Database for MySQL. Para más información, vea la [transformación de origen](data-flow-source.md) y la [transformación de receptor](data-flow-sink.md) en los flujos de datos de asignación. Puede optar por usar un conjunto de datos de Azure Database for MySQL o un [conjunto de datos insertado](data-flow-source.md#inline-datasets) como tipo de origen y receptor.

### <a name="source-transformation"></a>Transformación de origen

En la tabla siguiente se enumeran las propiedades compatibles con el origen de Azure Database for MySQL. Puede editar estas propiedades en la pestaña **Source options** (Opciones del origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Tabla | Si selecciona Tabla como entrada, el flujo de datos captura todos los datos de la tabla especificada en el conjunto de datos. | No | - |*(solo para conjunto de datos en línea)*<br>tableName |
| Consultar | Si selecciona Consultar como entrada, especifique una consulta SQL para capturar datos del origen, lo que invalida cualquier tabla que especifique en el conjunto de datos. El uso de consultas es una excelente manera de reducir las filas para pruebas o búsquedas.<br><br>La cláusula **Ordenar por** no se admite, pero puede establecer una instrucción SELECT FROM completa. También puede usar las funciones de tabla definidas por el usuario. **select * from udfGetData()** es un UDF in SQL que devuelve una tabla que puede utilizar en el flujo de datos.<br>Ejemplo de consulta: `select * from mytable where customerId > 1000 and customerId < 2000` o `select * from "MyTable"`.| No | String | Query |
| Tamaño de lote | Especifique un tamaño de lote para fragmentar datos de gran tamaño en lotes. | No | Entero | batchSize |
| Nivel de aislamiento | Elija uno de los siguientes niveles de aislamiento:<br>- Read Committed<br>- Read Uncommitted (predeterminado)<br>- Repeatable Read<br>- Serializable<br>- None (ignorar el nivel de aislamiento) | No | <small>READ_COMMITTED<br/>READ_UNCOMMITTED<br/>REPEATABLE_READ<br/>SERIALIZABLE<br/>NONE</small> |isolationLevel |

#### <a name="azure-database-for-mysql-source-script-example"></a>Ejemplo de script de origen de Azure Database for MySQL

Cuando se usa Azure Database for MySQL como tipo de origen, el script de flujo de datos asociado es:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    isolationLevel: 'READ_UNCOMMITTED',
    query: 'select * from mytable',
    format: 'query') ~> AzureMySQLSource
```

### <a name="sink-transformation"></a>Transformación de receptor

En la tabla siguiente se enumeran las propiedades compatibles con el receptor de Azure Database for MySQL. Puede editar estas propiedades en la pestaña **Opciones del receptor**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Método de actualización | Especifique qué operaciones se permiten en el destino de la base de datos. El valor predeterminado es permitir solamente las inserciones.<br>Para actualizar, upsert o eliminar filas, se requiere una [transformación de alteración de fila](data-flow-alter-row.md) a fin de etiquetar filas para esas acciones. | Sí | `true` o `false` | deletable <br/>insertable <br/>updateable <br/>upsertable |
| Columnas de clave | En el caso de las actualizaciones, upserts y eliminaciones, se deben establecer columnas de clave para determinar la fila que se va a modificar.<br>El nombre de columna que elija como clave se usará como parte de las operaciones posteriores de actualización, upsert y eliminación. Por lo tanto, debe seleccionar una columna que exista en la asignación del receptor. | No | Array | claves |
| Omitir escritura de columnas de clave | Si no quiere escribir el valor en la columna de clave, seleccione "Skip writing key columns" (Omitir escritura de columnas de clave). | No | `true` o `false` | skipKeyWrites |
| Acción Table |determina si se deben volver a crear o quitar todas las filas de la tabla de destino antes de escribir.<br>- **Ninguno**: no se realizará ninguna acción en la tabla.<br>- **Volver a crear**: se quitará la tabla y se volverá a crear. Obligatorio si se crea una nueva tabla dinámicamente.<br>- **Truncar**: se quitarán todas las filas de la tabla de destino. | No | `true` o `false` | recreate<br/>truncate |
| Tamaño de lote | Especifique el número de filas que se escriben en cada lote. Los tamaños de lote más grandes mejoran la compresión y la optimización de memoria, pero se arriesgan a obtener excepciones de memoria al almacenar datos en caché. | No | Entero | batchSize |
| Scripts SQL anteriores y posteriores | Especifique scripts de SQL de varias líneas que se ejecutarán antes (preprocesamiento) y después (procesamiento posterior) de que los datos se escriban en la base de datos del receptor. | No | String | preSQLs<br>postSQLs |

#### <a name="azure-database-for-mysql-sink-script-example"></a>Ejemplo de script de receptor de Azure Database for MySQL

Cuando se usa Azure Database for MySQL como tipo de receptor, el script de flujo de datos asociado es:

```
IncomingStream sink(allowSchemaDrift: true,
    validateSchema: false,
    deletable:false,
    insertable:true,
    updateable:true,
    upsertable:true,
    keys:['keyColumn'],
    format: 'table',
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> AzureMySQLSink
```

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="data-type-mapping-for-azure-database-for-mysql"></a>Asignación de tipos de datos para Azure Database for MySQL

Al copiar datos desde Azure Database for MySQL, se utilizan las siguientes asignaciones de tipos de datos de MySQL en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de Azure Database for MySQL | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| `bigint` |`Int64` |
| `bigint unsigned` |`Decimal` |
| `bit` |`Boolean` |
| `bit(M), M>1`|`Byte[]`|
| `blob` |`Byte[]` |
| `bool` |`Int16` |
| `char` |`String` |
| `date` |`Datetime` |
| `datetime` |`Datetime` |
| `decimal` |`Decimal, String` |
| `double` |`Double` |
| `double precision` |`Double` |
| `enum` |`String` |
| `float` |`Single` |
| `int` |`Int32` |
| `int unsigned` |`Int64`|
| `integer` |`Int32` |
| `integer unsigned` |`Int64` |
| `long varbinary` |`Byte[]` |
| `long varchar` |`String` |
| `longblob` |`Byte[]` |
| `longtext` |`String` |
| `mediumblob` |`Byte[]` |
| `mediumint` |`Int32` |
| `mediumint unsigned` |`Int64` |
| `mediumtext` |`String` |
| `numeric` |`Decimal` |
| `real` |`Double` |
| `set` |`String` |
| `smallint` |`Int16` |
| `smallint unsigned` |`Int32` |
| `text` |`String` |
| `time` |`TimeSpan` |
| `timestamp` |`Datetime` |
| `tinyblob` |`Byte[]` |
| `tinyint` |`Int16` |
| `tinyint unsigned` |`Int16` |
| `tinytext` |`String` |
| `varchar` |`String` |
| `year` |`Int32` |

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
