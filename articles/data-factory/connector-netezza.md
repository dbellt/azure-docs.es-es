---
title: Copia de datos de Netezza con Azure Data Factory
description: Obtenga información sobre cómo copiar datos de Netezza en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 05/28/2020
ms.author: jianleishen
ms.openlocfilehash: d7e61b6db0570f1cc7c0425a87bad8f1ee97da3f
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485772"
---
# <a name="copy-data-from-netezza-by-using-azure-data-factory"></a>Copia de datos de Netezza con Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Netezza. El artículo se basa en [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

>[!TIP]
>Para información sobre el escenario de migración de datos de Netezza a Azure, consulte [Uso de Azure Data Factory para migrar datos de un servidor de Netezza local a Azure](data-migration-guidance-netezza-azure-sqldw.md).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Netezza es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)


Puede copiar datos de Netezza en cualquier almacén de datos de receptor admitido. Para obtener una lista de almacenes de datos que la actividad de copia admite como orígenes y receptores, consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).

El conector de Netezza admite la copia en paralelo desde el origen. Consulte la sección [Copia en paralelo desde Netezza](#parallel-copy-from-netezza) para obtener más detalles.

Azure Data Factory proporciona un controlador integrado para permitir la conectividad. No es necesario instalar manualmente uno para usar este conector.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Introducción

Puede crear una canalización que use una actividad de copia mediante el SDK de .NET, el SDK de Python, Azure PowerShell, la API de REST o una plantilla de Azure Resource Manager. Consulte el [tutorial de la actividad de copia](quickstart-create-data-factory-dot-net.md) para obtener instrucciones paso a paso sobre cómo crear una canalización que tenga una actividad de copia.

En las secciones siguientes se proporcionan detalles sobre las propiedades que puede usar para definir entidades de Data Factory específicas del conector de Netezza.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Netezza:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **Netezza**. | Sí |
| connectionString | Cadena de conexión de ODBC para conectarse a Netezza. <br/>También puede colocar la contraseña en Azure Key Vault y extraer la configuración de `pwd` de la cadena de conexión. Consulte los siguientes ejemplos y el artículo [Almacenamiento de credenciales en Azure Key Vault](store-credentials-in-key-vault.md) con información detallada. | Sí |
| connectVia | Instancia de [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

Una cadena de conexión típica es `Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>`. En la tabla siguiente se describen más propiedades que puede establecer:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| SecurityLevel | Nivel de seguridad que usa el controlador para la conexión con el almacén de datos. El controlador admite conexiones SSL con autenticación unidireccional mediante la versión 3 de SSL. <br>Ejemplo: `SecurityLevel=preferredSecured`. Los valores admitidos son:<br/>- **Solo sin protección** (**onlyUnSecured**): el controlador no usa SSL.<br/>- **Sin protección preferido (preferredUnSecured) (valor predeterminado)** : si el servidor proporciona una opción, el controlador no usa SSL. <br/>- **Con protección preferido (preferredSecured)** : si el servidor proporciona una opción, el controlador usa SSL. <br/>- **Solo con protección (onlySecured)** : el controlador no se conectará a menos que haya una conexión SSL disponible. | No |
| CaCertFile | La ruta de acceso completa al certificado SSL que usa el servidor. Ejemplo: `CaCertFile=<cert path>;`| Sí, si se ha habilitado SSL |

**Ejemplo**

```json
{
    "name": "NetezzaLinkedService",
    "properties": {
        "type": "Netezza",
        "typeProperties": {
            "connectionString": "Server=<server>;Port=<port>;Database=<database>;UID=<user name>;PWD=<password>"
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
    "name": "NetezzaLinkedService",
    "properties": {
        "type": "Netezza",
        "typeProperties": {
            "connectionString": "Server=<server>;Port=<port>;Database=<database>;UID=<user name>;",
            "pwd": { 
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

En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Netezza.

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos](concepts-datasets-linked-services.md).

Para copiar datos desde Netezza, establezca la propiedad **type** del conjunto de datos en **NetezzaTable**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **NetezzaTable** | Sí |
| esquema | Nombre del esquema. |No (si se especifica "query" en el origen de la actividad)  |
| table | Nombre de la tabla. |No (si se especifica "query" en el origen de la actividad)  |
| tableName | Nombre de la tabla con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Use `schema` y `table` para la carga de trabajo nueva. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "NetezzaDataset",
    "properties": {
        "type": "NetezzaTable",
        "linkedServiceName": {
            "referenceName": "<Netezza linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {}
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

En esta sección se proporciona una lista de las propiedades que admite el origen de Netezza.

Para ver una lista completa de las secciones y propiedades que hay disponibles para definir actividades, consulte [Canalizaciones](concepts-pipelines-activities.md).

### <a name="netezza-as-source"></a>Netezza como origen

>[!TIP]
>Para cargar datos desde Netezza de manera eficaz con la creación de particiones de datos, obtenga más información en la sección [Copia en paralelo desde Netezza](#parallel-copy-from-netezza).

Para copiar datos desde Netezza, establezca el tipo **source** en la actividad de copia en **NetezzaSource**. La sección **source** de la actividad de copia admite las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia debe establecerse en **NetezzaSource**. | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Ejemplo: `"SELECT * FROM MyTable"` | No (si se especifica "tableName" en el conjunto de datos) |
| partitionOptions | Especifica los opciones de creación de particiones de datos que se usan para cargar datos desde Netezza. <br>Los valores permitidos son los siguientes: **None** (valor predeterminado), **DataSlice** y **DynamicRange**.<br>Cuando se habilita una opción de partición (es decir, no `None`), el grado de paralelismo para cargar simultáneamente datos de una base de datos Netezza se controla mediante la configuración [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) en la actividad de copia. | No |
| partitionSettings | Especifique el grupo de configuración para la creación de particiones de datos. <br>Se aplica cuando la opción de partición no es `None`. | No |
| partitionColumnName | Especifique el nombre de la columna de origen **in integer type** que usará la creación de particiones por rangos para la copia en paralelo. Si no se especifica, se detectará automáticamente la clave principal de la tabla y se usará como columna de partición. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionColumnName` en la cláusula WHERE. Consulte un ejemplo en la sección [Copia en paralelo desde Netezza](#parallel-copy-from-netezza). | No |
| partitionUpperBound | El valor máximo de la columna de partición para copiar datos. <br>Se aplica cuando la opción de partición es `DynamicRange`. Si usa la consulta para recuperar datos de origen, enlace `?AdfRangePartitionUpbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Netezza](#parallel-copy-from-netezza) para ver un ejemplo. | No |
| partitionLowerBound | El valor mínimo de la columna de partición para copiar datos. <br>Se aplica si la opción de partición es `DynamicRange`. Si usa una consulta para recuperar datos de origen, enlace `?AdfRangePartitionLowbound` en la cláusula WHERE. Consulte la sección [Copia en paralelo desde Netezza](#parallel-copy-from-netezza) para ver un ejemplo. | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromNetezza",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Netezza input dataset name>",
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
                "type": "NetezzaSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="parallel-copy-from-netezza"></a>Copia en paralelo desde Netezza

El conector de Netezza de Data Factory proporciona la creación de particiones de datos integrados para copiar datos de Netezza en paralelo. Puede encontrar las opciones de creación de particiones de datos en la pestaña **Origen** de la actividad de copia.

![Captura de pantalla de las opciones de partición](./media/connector-netezza/connector-netezza-partition-options.png)

Al habilitar la copia con particiones, Data Factory ejecuta consultas en paralelo en el origen de Netezza para cargar los datos mediante particiones. El grado en paralelo se controla mediante el valor [`parallelCopies`](copy-activity-performance-features.md#parallel-copy) de la actividad de copia. Por ejemplo, si establece `parallelCopies` como cuatro, Data Factory genera y ejecuta al mismo tiempo cuatro consultas de acuerdo con la configuración y la opción de partición que ha especificado, y cada consulta recupera una porción de datos de la base de datos de Netezza.

Se le sugiere que habilite la copia en paralelo con la creación de particiones de datos, especialmente si carga grandes cantidades de datos de la base de datos de Netezza. Estas son algunas configuraciones sugeridas para diferentes escenarios. Cuando se copian datos en un almacén de datos basado en archivos, se recomienda escribir en una carpeta como varios archivos (solo especifique el nombre de la carpeta), en cuyo caso el rendimiento es mejor que escribir en un único archivo.

| Escenario                                                     | Configuración sugerida                                           |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Carga completa de una tabla grande.                                   | **Opción de partición**: Segmento de datos. <br><br/>Durante la ejecución, Data Factory particiona automáticamente los datos basándose en los [segmentos de datos integrados de Netezza](https://www.ibm.com/support/knowledgecenter/en/SSULQD_7.2.1/com.ibm.nz.adm.doc/c_sysadm_data_slices_parts_disks.html) y copia los datos por particiones. |
| Cargue grandes cantidades de datos mediante una consulta personalizada.                 | **Opción de partición**: Segmento de datos.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE mod(datasliceid, ?AdfPartitionCount) = ?AdfDataSliceCondition AND <your_additional_where_clause>`.<br>Durante la ejecución, Data Factory reemplaza `?AdfPartitionCount` (por el número de copia en paralelo establecido en la actividad de copia) y `?AdfDataSliceCondition` por la lógica de partición del segmento de datos, y lo envía a Netezza. |
| Carga de grandes cantidades de datos mediante una consulta personalizada, con una columna de enteros con valor distribuido uniformemente para la creación de particiones por rangos. | **Opciones de partición**: partición por rangos dinámica.<br>**Consulta**: `SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>`.<br>**Columna de partición**: especifique la columna usada para crear la partición de datos. Puede crear particiones en la columna con un tipo de datos entero.<br>**Límite de partición superior** y **límite de partición inferior**: especifique si quiere filtrar en la columna de partición para recuperar solo los datos entre el intervalo inferior y el superior.<br><br>Durante la ejecución, Data Factory reemplaza `?AdfRangePartitionColumnName`, `?AdfRangePartitionUpbound` y `?AdfRangePartitionLowbound` por el nombre real de la columna y los intervalos de valor de cada partición y se los envía a Netezza. <br>Por ejemplo, si establece la columna de partición "ID" con un límite inferior de 1 y un límite superior de 80, con la copia en paralelo establecida en 4, Data Factory recupera los datos mediante 4 particiones. Los identificadores están comprendidos entre [1, 20], [21, 40], [41, 60] y [61, 80] respectivamente. |

**Ejemplo: consulta con partición de segmento de datos**

```json
"source": {
    "type": "NetezzaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE mod(datasliceid, ?AdfPartitionCount) = ?AdfDataSliceCondition AND <your_additional_where_clause>",
    "partitionOption": "DataSlice"
}
```

**Ejemplo: consulta con partición por rangos dinámica**

```json
"source": {
    "type": "NetezzaSource",
    "query": "SELECT * FROM <TABLENAME> WHERE ?AdfRangePartitionColumnName <= ?AdfRangePartitionUpbound AND ?AdfRangePartitionColumnName >= ?AdfRangePartitionLowbound AND <your_additional_where_clause>",
    "partitionOption": "DynamicRange",
    "partitionSettings": {
        "partitionColumnName": "<dynamic_range_partition_column_name>",
        "partitionUpperBound": "<upper_value_of_partition_column>",
        "partitionLowerBound": "<lower_value_of_partition_column>"
    }
}
```

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes

Para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes y receptores en Azure Data Factory, consulte los [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
