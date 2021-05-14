---
title: Copia de datos de Amazon Redshift
description: Obtenga información sobre cómo copiar datos desde Amazon Redshift a almacenes de datos de receptor compatibles mediante Azure Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 12/09/2020
ms.openlocfilehash: c1dd0144395d1eed533300808d8d872742f0d710
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482244"
---
# <a name="copy-data-from-amazon-redshift-using-azure-data-factory"></a>Copia de datos de Amazon Redshift mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-amazon-redshift-connector.md)
> * [Versión actual](connector-amazon-redshift.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se resume el uso de la actividad de copia de Azure Data Factory para copiar datos de Amazon Redshift. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Amazon Redshift es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Amazon Redshift en cualquier almacén de datos de receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector de Amazon Redshift admite la recuperación de datos de Redshift mediante una consulta o la compatibilidad integrada con Redshift UNLOAD.

> [!TIP]
> Para obtener el mejor rendimiento al copiar grandes cantidades de datos desde Redshift, considere usar Redshift UNLOAD integrado a través de Amazon S3. Vea la sección [Uso de UNLOAD para copiar datos de Amazon Redshift](#use-unload-to-copy-data-from-amazon-redshift) para más detalles.

## <a name="prerequisites"></a>Prerrequisitos

* Si va a copiar datos a un local almacén de datos local mediante [Integration Runtime autohospedado](create-self-hosted-integration-runtime.md), conceda a Integration Runtime (use la dirección IP de la máquina) el acceso al clúster de Amazon Redshift. Consulte [Authorize access to the cluster](https://docs.aws.amazon.com/redshift/latest/gsg/rs-gsg-authorize-cluster-access.html) (Autorización para acceder al clúster) para obtener instrucciones.
* Si va a copiar datos a un almacén de datos de Azure, consulte [Azure Data Center IP Ranges](https://www.microsoft.com/download/details.aspx?id=41653) (Intervalos de direcciones IP de Azure Data Center) para ver los intervalos de direcciones IP de Compute y los intervalos SQL que se utilizan en los centros de datos de Azure.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Amazon Redshift.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado Amazon Redshift:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **AmazonRedshift** | Sí |
| server |Dirección IP o nombre de host del servidor de Amazon Redshift. |Sí |
| port |El número del puerto TCP que el servidor de Amazon Redshift utiliza para escuchar las conexiones del cliente. |No, el valor predeterminado es 5439 |
| database |Nombre de la base de datos de Amazon Redshift. |Sí |
| username |Nombre del usuario que tiene acceso a la base de datos. |Sí |
| password |Contraseña para la cuenta de usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). |Sí |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Puede usar los entornos Integration Runtime (autohospedado) (si el almacén de datos se encuentra en una red privada) o Azure Integration Runtime. Si no se especifica, se usará Azure Integration Runtime. |No |

**Ejemplo**:

```json
{
    "name": "AmazonRedshiftLinkedService",
    "properties":
    {
        "type": "AmazonRedshift",
        "typeProperties":
        {
            "server": "<server name>",
            "database": "<database name>",
            "username": "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que el conjunto de datos de Amazon Redshift admite.

Para copiar datos de Amazon Redshift, se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **AmazonRedshiftTable** | Sí |
| esquema | Nombre del esquema. |No (si se especifica "query" en el origen de la actividad)  |
| table | Nombre de la tabla. |No (si se especifica "query" en el origen de la actividad)  |
| tableName | Nombre de la tabla con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Use `schema` y `table` para la carga de trabajo nueva. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "AmazonRedshiftDataset",
    "properties":
    {
        "type": "AmazonRedshiftTable",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Amazon Redshift linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

Si estaba usando un conjunto de datos de tipo `RelationalTable`, todavía se admite tal cual, aunque se aconseja usar el nuevo en el futuro.

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que el origen Amazon Redshift admite.

### <a name="amazon-redshift-as-source"></a>Amazon Redshift como origen

Para copiar datos desde Amazon Redshift, establezca el tipo de origen de la actividad de copia en **AmazonRedshiftSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **AmazonRedshiftSource** | Sí |
| Query |Utilice la consulta personalizada para leer los datos. Por ejemplo: select * from MyTable. |No (si se especifica "tableName" en el conjunto de datos) |
| redshiftUnloadSettings | Grupo de propiedades al usar Amazon Redshift UNLOAD. | No |
| s3LinkedServiceName | Hace referencia a Amazon S3 que se usa como almacenamiento provisional especificando para ello un nombre de servicio vinculado de tipo "AmazonS3". | Sí, se utiliza UNLOAD |
| bucketName | Indique el depósito S3 para almacenar los datos provisionales. Si no se proporciona, el servicio Data Factory lo genera automáticamente.  | Sí, se utiliza UNLOAD |

**Ejemplo: Origen Amazon Redshift de la actividad de copia mediante UNLOAD**

```json
"source": {
    "type": "AmazonRedshiftSource",
    "query": "<SQL query>",
    "redshiftUnloadSettings": {
        "s3LinkedServiceName": {
            "referenceName": "<Amazon S3 linked service>",
            "type": "LinkedServiceReference"
        },
        "bucketName": "bucketForUnload"
    }
}
```

Obtenga más información en la sección siguiente sobre cómo usar UNLOAD para copiar datos desde Amazon Redshift de forma eficaz.

## <a name="use-unload-to-copy-data-from-amazon-redshift"></a>Uso de UNLOAD para copiar datos de Amazon Redshift

[UNLOAD](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html) es un mecanismo proporcionado por Amazon Redshift, que puede descargar los resultados de una consulta a uno o varios archivos en Amazon Simple Storage Service (Amazon S3). Es la manera que Amazon recomienda para copiar el conjunto de datos grande de Redshift.

**Ejemplo: copiar datos de Amazon Redshift en Azure Synapse Analytics con UNLOAD, copia almacenada provisionalmente y PolyBase**

En este caso de uso de ejemplo, la actividad de copia descarga los datos de Amazon Redshift en Amazon S3 según se configura en "redshiftUnloadSettings", luego copia los datos de Amazon S3 en Azure Blob, según se especifica en "stagingSettings", y al final se usa PolyBase para cargar los datos en Azure Synapse Analytics. Todo el formato provisional es controlado como corresponde por la actividad de copia.

![Flujo de trabajo de copia de Redshift a Azure Synapse Analytics](media/copy-data-from-amazon-redshift/redshift-to-sql-dw-copy-workflow.png)

```json
"activities":[
    {
        "name": "CopyFromAmazonRedshiftToSQLDW",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "AmazonRedshiftDataset",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "AzureSQLDWDataset",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "AmazonRedshiftSource",
                "query": "select * from MyTable",
                "redshiftUnloadSettings": {
                    "s3LinkedServiceName": {
                        "referenceName": "AmazonS3LinkedService",
                        "type": "LinkedServiceReference"
                    },
                    "bucketName": "bucketForUnload"
                }
            },
            "sink": {
                "type": "SqlDWSink",
                "allowPolyBase": true
            },
            "enableStaging": true,
            "stagingSettings": {
                "linkedServiceName": "AzureStorageLinkedService",
                "path": "adfstagingcopydata"
            },
            "dataIntegrationUnits": 32
        }
    }
]
```

## <a name="data-type-mapping-for-amazon-redshift"></a>Asignación de tipos de datos para Amazon Redshift

Al copiar datos desde Amazon Redshift, se utilizan las siguientes asignaciones de tipos de datos de Amazon Redshift en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de Amazon Redshift | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| bigint |Int64 |
| BOOLEAN |String |
| CHAR |String |
| DATE |DateTime |
| DECIMAL |Decimal |
| DOUBLE PRECISION |Double |
| INTEGER |Int32 |
| real |Single |
| SMALLINT |Int16 |
| TEXT |String |
| timestamp |DateTime |
| VARCHAR |String |

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
