---
title: Copiar datos de Impala mediante Azure Data Factory
description: Aprenda a copiar datos de Impala en almacenes de datos receptores compatibles mediante una actividad de copia en una canalización de factoría de datos.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: jianleishen
ms.openlocfilehash: 4d8ff83bc7ecc9bc98a5962badfd0056fae23fb9
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109484962"
---
# <a name="copy-data-from-impala-by-using-azure-data-factory"></a>Copiar datos de Impala mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Impala. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que presenta información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Impala es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Impala en cualquier almacén de datos receptor admitido. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Data Factory proporciona un controlador integrado para permitir la conectividad. Por lo tanto, no es necesario instalar manualmente uno para usar este conector.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="get-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Impala.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Impala:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe estar establecida en **Impala**. | Sí |
| host | La dirección IP o el nombre de host del servidor de Impala (es decir, 192.168.222.160).  | Sí |
| port | Puerto TCP que el servidor de Impala usa para escuchar las conexiones del cliente. El valor predeterminado es 21050.  | No |
| authenticationType | Tipo de autenticación que se debe usar. <br/>Los valores permitidos son: **Anonymous**, **SASLUsername** y **UsernameAndPassword**. | Sí |
| username | Nombre de usuario que se usa para acceder al servidor de Impala. El valor predeterminado es anonymous cuando se usa SASLUsername.  | No |
| password | La contraseña que se corresponde con el nombre de usuario cuando se usa UsernameAndPassword. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | No |
| enableSsl | Especifica si las conexiones al servidor se cifran mediante TLS. El valor predeterminado es **false**.  | No |
| trustedCertPath | La ruta de acceso completa del archivo .pem que contiene certificados de entidad de certificación de confianza usados para comprobar el servidor cuando se conecta a través de TLS. Esta propiedad solo puede establecerse cuando se usa TLS en el entorno de ejecución de integración autohospedado. El valor predeterminado es el archivo cacerts.pem instalado con Integration Runtime.  | No |
| useSystemTrustStore | Especifica si se utiliza un certificado de CA del almacén de confianza del sistema o de un archivo PEM especificado. El valor predeterminado es **false**.  | No |
| allowHostNameCNMismatch | Especifica si se requiere que el nombre del certificado TLS/SSL emitido por una entidad de certificación coincida con el nombre de host del servidor al conectarse a través de TLS. El valor predeterminado es **false**.  | No |
| allowSelfSignedServerCert | Especifica si se permiten los certificados autofirmados del servidor. El valor predeterminado es **false**.  | No |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usará Azure Integration Runtime. |No |

**Ejemplo**:

```json
{
    "name": "ImpalaLinkedService",
    "properties": {
        "type": "Impala",
        "typeProperties": {
            "host" : "<host>",
            "port" : "<port>",
            "authenticationType" : "UsernameAndPassword",
            "username" : "<username>",
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de Impala.

Para copiar datos de Impala, establezca la propiedad type del conjunto de datos en **ImpalaObject**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **ImpalaObject** | Sí |
| esquema | Nombre del esquema. |No (si se especifica "query" en el origen de la actividad)  |
| table | Nombre de la tabla. |No (si se especifica "query" en el origen de la actividad)  |
| tableName | Nombre de la tabla con el esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Use `schema` y `table` para la carga de trabajo nueva. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "ImpalaDataset",
    "properties": {
        "type": "ImpalaObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Impala linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el tipo de origen de Impala.

### <a name="impala-as-a-source-type"></a>Impala como tipo de origen

Para copiar datos de Impala, establezca el tipo de origen de la actividad de copia en **ImpalaSource**. En la sección **source** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en **ImpalaSource**. | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Un ejemplo es `"SELECT * FROM MyTable"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromImpala",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Impala input dataset name>",
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
                "type": "ImpalaSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes
Para ver la lista de almacenes de datos que la actividad de copia de Data Factory admite como orígenes y receptores consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).
