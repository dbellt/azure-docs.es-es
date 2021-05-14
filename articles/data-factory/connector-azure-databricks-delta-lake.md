---
title: Copia de datos en Azure Databricks Delta Lake como origen y destino
description: Aprenda a copiar datos en Azure Databricks Delta Lake como origen o destino mediante una actividad de copia en una canalización de Azure Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/29/2021
ms.openlocfilehash: b1aa174645288f5f3024779a0e5b9e8bdbb57452
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109480462"
---
# <a name="copy-data-to-and-from-azure-databricks-delta-lake-by-using-azure-data-factory"></a>Copia de datos con Azure Databricks Delta Lake como origen o destino mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se resume el uso de la actividad de copia de Azure Data Factory para copiar datos con Azure Databricks Delta Lake como origen o destino. Este se compila en función del artículo [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Databricks Delta Lake es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con tabla de [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

En general, Azure Data Factory es compatible con Delta Lake con las siguientes funcionalidades para responder a sus necesidades.

- La actividad de copia admite el conector de Azure Databricks Delta Lake para copiar datos desde cualquier almacén de datos de origen compatible a la tabla de Azure Databricks Delta Lake, y desde la tabla de Delta Lake a cualquier almacén de datos receptor compatible. Aprovecha el clúster de Databricks para realizar el movimiento de datos. Consulte los detalles en la [sección Requisitos previos](#prerequisites).
- El [flujo de datos de asignación](concepts-data-flow-overview.md) admite el [formato Delta](format-delta.md) genérico en Azure Storage como origen y receptor para leer y escribir archivos delta para las operaciones de ETL sin código y se ejecuta en una instancia administrada de Azure Integration Runtime.
- Las [actividades de Databricks](transform-data-databricks-notebook.md) admiten la orquestación de la carga de trabajo de ETL centrada en código, o de aprendizaje automático basada en Delta Lake.

## <a name="prerequisites"></a>Requisitos previos

Para usar este conector de Azure Databricks Delta Lake, debe configurar un clúster en Azure Databricks.

- Para copiar datos a Delta Lake, la actividad de copia invoca un clúster de Azure Databricks para leer los datos de una instancia de Azure Storage, que es el origen original o el área de almacenamiento provisional en la que Data Factory escribe primero los datos de origen a través de la copia preconfigurada integrada. Obtenga más información sobre [Delta Lake como receptor](#delta-lake-as-sink).
- Del mismo modo, para copiar datos desde Delta Lake, la actividad de copia invoca un clúster de Azure Databricks para escribir los datos en una instancia de Azure Storage, que es el receptor original o el área de almacenamiento provisional desde la que Data Factory sigue escribiendo los datos de a un receptor final a través de la copia preconfigurada integrada. Obtenga más información sobre [Delta Lake como origen](#delta-lake-as-source).

El clúster de Databricks debe tener acceso a una cuenta de blob de Azure o Azure Data Lake Storage Gen2, al sistema de archivos o contenedor de almacenamiento usado como origen, receptor o almacenamiento provisional y al sistema de archivos o contenedor en el que quiere escribir las tablas de Delta Lake.

- Para usar **Azure Data Lake Storage Gen2**, puede configurar una **entidad de servicio** en el clúster de Databricks como parte de la configuración de Apache Spark. Siga los pasos que se indican en [Acceso directo con la entidad de servicio](/azure/databricks/data/data-sources/azure/azure-datalake-gen2#--access-directly-with-service-principal-and-oauth-20).

- Para usar **Azure Blob Storage**, puede configurar una **clave de acceso de cuenta de almacenamiento** o **token de SAS** en el clúster de Databricks como parte de la configuración de Apache Spark. Siga los pasos descritos en [Acceso a Azure Blob Storage mediante la API de RDD](/azure/databricks/data/data-sources/azure/azure-storage#access-azure-blob-storage-using-the-rdd-api).

Durante la ejecución de la actividad de copia, si el clúster que configuró se ha finalizado, Data Factory lo inicia automáticamente. Si crea una canalización mediante la interfaz de usuario de creación de Data Factory, en el caso de las operaciones como la vista previa de los datos, deberá tener un clúster activo, ya que Data Factory no iniciará el clúster automáticamente.

#### <a name="specify-the-cluster-configuration"></a>Especificación de la configuración de clúster

1. En el menú desplegable **Modo del clúster**, seleccione **Estándar**.

2. En la lista desplegable **Versión de Databricks Runtime**, seleccione una versión de Databricks Runtime.

3. Active la [Optimización automática](/azure/databricks/delta/optimizations/auto-optimize) al agregar las siguientes propiedades a la [configuración de Spark](/azure/databricks/clusters/configure#spark-config):

   ```
   spark.databricks.delta.optimizeWrite.enabled true
   spark.databricks.delta.autoCompact.enabled true
   ```

4. Configure el clúster en función de sus necesidades de escalado e integración.

Para obtener detalles sobre la configuración del clúster, consulte [Configuración de los clústeres](/azure/databricks/clusters/configure).

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que definen entidades de Data Factory específicas de un conector de Azure Databricks Delta Lake.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con un servicio vinculado de Azure Databricks Delta Lake.

| Propiedad    | Descripción                                                  | Obligatorio |
| :---------- | :----------------------------------------------------------- | :------- |
| type        | Propiedad type, que debe establecerse en **AzureDatabricksDeltaLake**. | Sí      |
| dominio      | Especifique la dirección URL del área de trabajo de Azure Databricks; por ejemplo, `https://adb-xxxxxxxxx.xx.azuredatabricks.net`. |          |
| clusterId   | Especifique el id. de clúster de un clúster existente. Debe tratarse de un clúster interactivo que ya se haya creado. <br>Encontrará el identificador del clúster interactivo en el área de trabajo de Databricks -> Clusters -> Interactive Cluster Name -> Configuration -> Tags (Clústeres -> Nombre del clúster interactivo -> Configuración -> Etiquetas). [Obtenga más información](/azure/databricks/clusters/configure#cluster-tags). |          |
| accessToken | El token de acceso es necesario para que Data Factory se autentique en Azure Databricks. El token de acceso debe generarse a partir del área de trabajo de Databricks. [Aquí](/azure/databricks/dev-tools/api/latest/authentication#generate-token) encontrará más pasos detallados para encontrar el token de acceso. |          |
| connectVia  | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se utiliza para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos se encuentra en una red privada). Si no se especifica, se usará Azure Integration Runtime. | No       |

**Ejemplo**:

```json
{
    "name": "AzureDatabricksDeltaLakeLinkedService",
    "properties": {
        "type": "AzureDatabricksDeltaLake",
        "typeProperties": {
            "domain": "https://adb-xxxxxxxxx.xx.azuredatabricks.net",
            "clusterId": "<cluster id>",
            "accessToken": {
                "type": "SecureString", 
                "value": "<access token>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

Las siguientes propiedades son compatibles con el conjunto de datos de Azure Databricks Delta Lake.

| Propiedad  | Descripción                                                  | Obligatorio                    |
| :-------- | :----------------------------------------------------------- | :-------------------------- |
| type      | Propiedad type del conjunto de datos, que debe establecerse en **AzureDatabricksDeltaLakeDataset**. | Sí                         |
| database | Nombre de la base de datos. |No para el origen, sí para el receptor  |
| table | Nombre de la tabla de Delta. |No para el origen, sí para el receptor  |

**Ejemplo**:

```json
{
    "name": "AzureDatabricksDeltaLakeDataset",
    "properties": {
        "type": "AzureDatabricksDeltaLakeDataset",
        "typeProperties": {
            "database": "<database name>",
            "table": "<delta table name>"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "linkedServiceName": {
            "referenceName": "<name of linked service>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el receptor y el origen de Azure Databricks Delta Lake.

### <a name="delta-lake-as-source"></a>Delta Lake como origen

Para copiar datos de Azure Databricks Delta Lake, se admiten las siguientes propiedades en la sección de **origen** de la actividad de copia.

| Propiedad                     | Descripción                                                  | Obligatorio |
| :--------------------------- | :----------------------------------------------------------- | :------- |
| type                         | Propiedad type del origen de la actividad de copia, que debe establecerse en: **AzureDatabricksDeltaLakeSource**. | Sí      |
| Query          | Especifique la consulta SQL para leer los datos. Para el control de viaje en el tiempo, use el siguiente patrón:<br>- `SELECT * FROM events TIMESTAMP AS OF timestamp_expression`<br>- `SELECT * FROM events VERSION AS OF version` | No       |
| exportSettings | Configuración avanzada utilizada para recuperar datos de la tabla de Delta. | No       |
| ***En`exportSettings`:*** |  |  |
| type | Propiedad type del comando de exportación, establecida en **AzureDatabricksDeltaLakeExportCommand**. | Yes |
| dateFormat | Da formato de cadena con formato de fecha a un elemento de tipo date. Los formatos de fecha personalizados siguen los formatos del [patrón de datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Si no se especifica, el valor predeterminado es `yyyy-MM-dd`. | No |
| timestampFormat | Da formato de cadena con formato de marca de tiempo a un elemento de tipo timestamp. Los formatos de fecha personalizados siguen los formatos del [patrón de datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Si no se especifica, el valor predeterminado es `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`. | No |

#### <a name="direct-copy-from-delta-lake"></a>Copia directa desde Delta Lake

Si el almacén de datos y el formulario del receptor cumplen los criterios descritos en esta sección, puede usar la actividad de copia para copiar directamente de la tabla de Azure Databricks Delta al receptor. Data Factory comprueba la configuración y produce un error en la ejecución de la actividad de copia si no se cumplen los siguientes criterios:

- El **servicio vinculado del receptor** es [Azure Blob Storage](connector-azure-blob-storage.md) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md). Las credenciales de la cuenta deben estar configuradas previamente en la configuración de clúster de Azure Databricks. Obtenga más información sobre los [Requisitos previos](#prerequisites).

- El **formato de datos de receptor** es **Parquet**, **texto delimitado** o **JSON** con estas configuraciones, y apunta a una carpeta en lugar de un archivo.

    - Para el formato **Parquet**, el códec de compresión es **none**, **snappy**, o **gzip**.
    - Para el formato de **texto delimitado**:
        - `rowDelimiter` es cualquier carácter individual.
        - `compression` puede ser **none**, **bzip2** o **gzip**.
        - No se admite `encodingName` con el valor UTF-7.
    - En el caso del formato **Avro**, el códec de compresión es **none**, **deflate** o **snappy**.

- En el origen de la actividad Copy, `additionalColumns` no se especifica.
- Si se copian datos en texto delimitado, en el receptor de la actividad de copia, `fileExtension` debe ser ".csv".
- La conversión de tipos no está habilitada en la asignación de la actividad de copia.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
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
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

#### <a name="staged-copy-from-delta-lake"></a>Copia almacenada provisionalmente desde Delta Lake

Cuando el formato o el almacén de datos del receptor no coincide con los criterios de copia directa, como se menciona en la última sección, habilite la copia preconfigurada integrada con una instancia intermedia de Azure Storage. La característica de copia almacenada provisionalmente también proporciona un mejor rendimiento. Data Factory exporta datos de Azure Databricks Delta Lake al almacenamiento provisional, copia los datos en el receptor y, por último, limpia los datos temporales del almacenamiento provisional. Consulte [Copia almacenada provisionalmente](copy-activity-performance-features.md#staged-copy) para obtener más información sobre cómo copiar datos mediante el almacenamiento provisional.

Para usar esta característica, cree un [servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) que haga referencia a la cuenta de almacenamiento como almacenamiento provisional temporal. Luego especifique las propiedades `enableStaging` y `stagingSettings` en la actividad de copia.

>[!NOTE]
>Las credenciales de la cuenta de almacenamiento provisional deben estar configuradas previamente en la configuración de clúster de Azure Databricks. Obtenga más información sobre los [Requisitos previos](#prerequisites).

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delta lake input dataset name>",
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
                "type": "AzureDatabricksDeltaLakeSource",
                "sqlReaderQuery": "SELECT * FROM events TIMESTAMP AS OF timestamp_expression"
            },
            "sink": {
                "type": "<sink type>"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingStorage",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

### <a name="delta-lake-as-sink"></a>Delta Lake como receptor

Para copiar datos a Azure Databricks Delta Lake, se admiten las siguientes propiedades en la sección de **receptor** de la actividad de copia.

| Propiedad      | Descripción                                                  | Obligatorio |
| :------------ | :----------------------------------------------------------- | :------- |
| type          | Propiedad type del receptor de la actividad de copia, establecida en **AzureDatabricksDeltaLakeSink**. | Sí      |
| preCopyScript | Especifica una consulta SQL para que la ejecute la actividad de copia antes de escribir datos en la tabla de Databricks Delta en cada ejecución. Puede usar esta propiedad para limpiar los datos cargados previamente, o agregar una instrucción TRUNCATE TABLE o VACUUM. | No       |
| importSettings | Configuración avanzada usada para escribir datos en la tabla de Delta. | No |
| ***En`importSettings`:*** |                                                              |  |
| type | Propiedad type del comando de importación, establecida en **AzureDatabricksDeltaLakeImportCommand**. | Yes |
| dateFormat | Da formato de tipo date con formato de fecha a una cadena. Los formatos de fecha personalizados siguen los formatos del [patrón de datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Si no se especifica, el valor predeterminado es `yyyy-MM-dd`. | No |
| timestampFormat | Da formato de tipo timestamp con formato de marca de tiempo a una cadena. Los formatos de fecha personalizados siguen los formatos del [patrón de datetime](https://spark.apache.org/docs/latest/sql-ref-datetime-pattern.html). Si no se especifica, el valor predeterminado es `yyyy-MM-dd'T'HH:mm:ss[.SSS][XXX]`. | No |

#### <a name="direct-copy-to-delta-lake"></a>Copia directa a Delta Lake

Si el almacén de datos y el formulario de origen cumplen los criterios descritos en esta sección, puede usar la actividad de copia para copiar directamente desde el origen a Azure Databricks Delta Lake. Azure Data Factory comprueba la configuración y devuelve un error en la ejecución de la actividad de copia si no se cumplen los siguientes criterios:

- El **servicio vinculado de origen** es [Azure Blob Storage](connector-azure-blob-storage.md) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md). Las credenciales de la cuenta deben estar configuradas previamente en la configuración de clúster de Azure Databricks. Obtenga más información sobre los [Requisitos previos](#prerequisites).

- El **formato de datos de origen** es **Parquet**, **texto delimitado** o **JSON** con estas configuraciones, y apunta a una carpeta en lugar de un archivo.

    - Para el formato **Parquet**, el códec de compresión es **none**, **snappy**, o **gzip**.
    - Para el formato de **texto delimitado**:
        - `rowDelimiter` es el valor predeterminado, o cualquier carácter individual.
        - `compression` puede ser **none**, **bzip2** o **gzip**.
        - No se admite `encodingName` con el valor UTF-7.
    - En el caso del formato **Avro**, el códec de compresión es **none**, **deflate** o **snappy**.

- En el origen de la actividad de copia: 

    - `wildcardFileName` solo contiene caracteres `*` comodín, pero ningún `?` y no se especifica el valor de `wildcardFolderName`.
    - `prefix`, `modifiedDateTimeStart`, `modifiedDateTimeEnd` y `enablePartitionDiscovery` no se especifican.
    - `additionalColumns` no se especifica.

- La conversión de tipos no está habilitada en la asignación de la actividad de copia.

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            }
        }
    }
]
```

#### <a name="staged-copy-to-delta-lake"></a>Copia almacenada provisionalmente a Delta Lake

Cuando el formato o el almacén de datos del origen no coincide con los criterios de copia directa, como se menciona en la última sección, habilite la copia preconfigurada integrada con una instancia intermedia de Azure Storage. La característica de copia almacenada provisionalmente también proporciona un mejor rendimiento. Data Factory convierte automáticamente los datos para que cumplan los requisitos de formato de datos en el almacenamiento provisional y, a continuación, los carga en Delta Lake desde allí. Por último, limpia los datos temporales del almacenamiento. Consulte [Copia almacenada provisionalmente](copy-activity-performance-features.md#staged-copy) para obtener más información sobre cómo copiar datos con el almacenamiento provisional.

Para usar esta característica, cree un [servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties) o [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) que haga referencia a la cuenta de almacenamiento como almacenamiento provisional temporal. Luego especifique las propiedades `enableStaging` y `stagingSettings` en la actividad de copia.

>[!NOTE]
>Las credenciales de la cuenta de almacenamiento provisional deben estar configuradas previamente en la configuración de clúster de Azure Databricks. Obtenga más información sobre los [Requisitos previos](#prerequisites).

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToDeltaLake",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Delta lake output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDatabricksDeltaLakeSink"
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": {
                    "referenceName": "MyStagingBlob",
                    "type": "LinkedServiceReference"
                },
                "path": "mystagingpath"
            }
        }
    }
]
```

## <a name="monitoring"></a>Supervisión

Azure Data Factory proporciona la misma [experiencia de supervisión de la actividad de copia](copy-activity-monitoring.md) que otros conectores. Además, dado que la carga de datos con Delta Lake como origen o destino se está ejecutando en el clúster de Azure Databricks, puede [ver los registros de clúster detallados](/azure/databricks/clusters/clusters-manage#--view-cluster-logs) y [supervisar el rendimiento](/azure/databricks/clusters/clusters-manage#--monitor-performance).

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para más información sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes

Consulte los [formatos y almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de los almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores.
