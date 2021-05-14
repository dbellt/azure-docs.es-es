---
title: Copia de datos a o desde Azure Data Explorer
description: Obtenga información sobre cómo copiar con Azure Data Explorer como origen o destino mediante una actividad de copia de una canalización de Azure Data Factory.
ms.author: orspodek
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 03/24/2020
ms.openlocfilehash: 606d10694b6806b62871ddf24afd259d7bc224bc
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482982"
---
# <a name="copy-data-to-or-from-azure-data-explorer-by-using-azure-data-factory"></a>Copia de datos con Azure Data Explorer como origen o destino mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe el uso de la actividad de copia de Azure Data Factory para copiar datos con [Azure Data Explorer](/azure/data-explorer/data-explorer-overview) como origen o destino. Se basa en el artículo [Actividad de copia en Azure Data Factory](copy-activity-overview.md), en el que se ofrece información general acerca de la actividad de copia.

>[!TIP]
>Para la integración de Azure Data Explorer y Azure Data Factory en general, obtenga más información en [Integración de Azure Data Explorer con Azure Data Factory](/azure/data-explorer/data-factory-integration).

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Azure Data Explorer es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde cualquier almacén de datos de origen compatible a Azure Data Explorer. Además, puede copiar datos desde Azure Data Explorer a cualquier almacén de datos de receptor compatible. Consulte la tabla de [almacenes de datos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de los almacenes de datos que la actividad de copia admite como orígenes o receptores.

>[!NOTE]
>La copia de datos con Azure Data Explorer como origen o destino a través de un almacén de datos local mediante un entorno de ejecución de integración autohospedado se admite en la versión 3.14 y versiones posteriores.

El conector de Azure Data Explorer permite hacer lo siguiente:

* Copiar datos mediante la autenticación de tokens de aplicaciones de Azure Active Directory (Azure AD) con una **entidad de servicio**.
* Como origen, recupere los datos mediante una consulta KQL (Kusto).
* Como receptor, anexe datos a una tabla de destino.

## <a name="getting-started"></a>Introducción

>[!TIP]
>Para ver un tutorial sobre el uso del conector de Azure Data Explorer, consulte [Copia de datos con Azure Data Explorer como origen o destino mediante Azure Data Factory](/azure/data-explorer/data-factory-load-data) y [Copia en bloque desde una base de datos a Azure Data Explorer](/azure/data-explorer/data-factory-template).

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Azure Data Explorer.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

El conector de Azure Data Explorer admite los siguientes tipos de autenticación. Consulte las secciones correspondientes para más información:

- [Autenticación de entidad de servicio](#service-principal-authentication)
- [Identidades administradas para la autenticación de recursos de Azure](#managed-identity)

### <a name="service-principal-authentication"></a>Autenticación de entidad de servicio

Para usar la autenticación de la entidad de servicio, siga estos pasos para obtener una entidad de servicio y para conceder permisos:

1. Registre una entidad de aplicación en Azure Active Directory como se indica en [Registro de la aplicación con un inquilino de Azure AD](../storage/common/storage-auth-aad-app.md#register-your-application-with-an-azure-ad-tenant). Anote los siguientes valores; los usará para definir el servicio vinculado:

    - Identificador de aplicación
    - Clave de la aplicación
    - Id. de inquilino

2. Conceda a la entidad de servicio los permisos correctos en Azure Data Explorer. Consulte [Administración de permisos de base de datos de Azure Data Explorer](/azure/data-explorer/manage-database-permissions) para obtener información detallada sobre los roles y los permisos, así como información sobre la administración de permisos. En general, debe:

    - **Como origen**, conceder al menos el rol **Visor de base de datos** a la base de datos.
    - **Como receptor**, conceder al menos el rol **Agente de ingesta de base de datos** a la base de datos.

>[!NOTE]
>Cuando utiliza la interfaz de usuario de Data Factory para crear, de forma predeterminada, se usa la cuenta de usuario de inicio de sesión para enumerar los clústeres, bases de datos y tablas de Azure Data Explorer. Para mostrar los objetos mediante la entidad de servicio, haga clic en la lista desplegable situada junto al botón Actualizar o escriba manualmente el nombre si no tiene permiso para estas operaciones.

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Data Explorer:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** se debe establecer en **AzureDataExplorer**. | Sí |
| endpoint | Dirección URL del punto de conexión del clúster de Azure Data Explorer, con el formato como `https://<clusterName>.<regionName>.kusto.windows.net`. | Sí |
| database | Nombre de la base de datos. | Sí |
| tenant | Especifique la información del inquilino (nombre de dominio o identificador de inquilino) en el que reside la aplicación. Esto se conoce como "Id. de autoridad" en la [cadena de conexión de Kusto](/azure/kusto/api/connection-strings/kusto#application-authentication-properties). Para recuperarlo, mantenga el puntero del mouse en la esquina superior derecha de Azure Portal. | Sí |
| servicePrincipalId | Especifique el id. de cliente de la aplicación. Esto se conoce como "Id. del cliente de aplicación de AAD" en la [cadena de conexión de Kusto](/azure/kusto/api/connection-strings/kusto#application-authentication-properties). | Sí |
| servicePrincipalKey | Especifique la clave de la aplicación. Esto se conoce como "Clave de aplicación de AAD" en la [cadena de conexión de Kusto](/azure/kusto/api/connection-strings/kusto#application-authentication-properties). Marque este campo como **SecureString** para almacenarlo de forma segura en Data Factory, o bien [haga referencia a datos seguros almacenados en Azure Key Vault](store-credentials-in-key-vault.md). | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si el almacén de datos está en una red privada, se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado. Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo: uso de la autenticación de claves de entidad de servicio**

```json
{
    "name": "AzureDataExplorerLinkedService",
    "properties": {
        "type": "AzureDataExplorer",
        "typeProperties": {
            "endpoint": "https://<clusterName>.<regionName>.kusto.windows.net ",
            "database": "<database name>",
            "tenant": "<tenant name/id e.g. microsoft.onmicrosoft.com>",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": {
                "type": "SecureString",
                "value": "<service principal key>"
            }
        }
    }
}
```

### <a name="managed-identities-for-azure-resources-authentication"></a><a name="managed-identity"></a> Identidades administradas para la autenticación de recursos de Azure

Para usar las identidades administradas para la autenticación de recursos de Azure, siga estos pasos para conceder los permisos:

1. [Recupere la información de la identidad administrada de Data Factory](data-factory-service-identity.md#retrieve-managed-identity) mediante la copia del valor de **Id. del objeto de identidad administrada** que se ha generado junto con la factoría.

2. Conceda a la identidad administrada los permisos correctos en Azure Data Explorer. Consulte [Administración de permisos de base de datos de Azure Data Explorer](/azure/data-explorer/manage-database-permissions) para obtener información detallada sobre los roles y los permisos, así como información sobre la administración de permisos. En general, debe:

    - **Como origen**, conceder al menos el rol **Visor de base de datos** a la base de datos.
    - **Como receptor**, conceder al menos el rol **Agente de ingesta de base de datos** a la base de datos.

>[!NOTE]
>Cuando utiliza la interfaz de usuario de Data Factory para crear, se usa la cuenta de usuario de inicio de sesión para enumerar los clústeres, bases de datos y tablas de Azure Data Explorer. Escriba manualmente el nombre si no tiene permiso para realizar estas operaciones.

Las siguientes propiedades son compatibles con el servicio vinculado de Azure Data Explorer:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** se debe establecer en **AzureDataExplorer**. | Sí |
| endpoint | Dirección URL del punto de conexión del clúster de Azure Data Explorer, con el formato como `https://<clusterName>.<regionName>.kusto.windows.net`. | Sí |
| database | Nombre de la base de datos. | Sí |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Si el almacén de datos está en una red privada, se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado. Si no se especifica, se usa el valor predeterminado de Azure Integration Runtime. |No |

**Ejemplo: Uso de la autenticación de identidad administrada**

```json
{
    "name": "AzureDataExplorerLinkedService",
    "properties": {
        "type": "AzureDataExplorer",
        "typeProperties": {
            "endpoint": "https://<clusterName>.<regionName>.kusto.windows.net ",
            "database": "<database name>",
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si quiere ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos en Azure Data Factory](concepts-datasets-linked-services.md). En esta sección se enumeran las propiedades compatibles con el conjunto de datos de Azure Data Explorer.

Para copiar datos a Azure Data Explorer, establezca la propiedad type del conjunto de datos en **AzureDataExplorerTable**.

Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AzureDataExplorerTable**. | Sí |
| table | El nombre de la tabla a la que hace referencia el servicio vinculado. | Sí para el receptor, no para el origen |

**Ejemplo de propiedades del conjunto de datos:**

```json
{
   "name": "AzureDataExplorerDataset",
    "properties": {
        "type": "AzureDataExplorerTable",
        "typeProperties": {
            "table": "<table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Azure Data Explorer linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Para ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte [Canalizaciones y actividades en Azure Data Factory](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con los orígenes y los receptores de Azure Data Explorer.

### <a name="azure-data-explorer-as-source"></a>Azure Data Explorer como origen

Para copiar datos desde Azure Data Explorer, establezca la propiedad **type** del origen de la actividad de copia en **AzureDataExplorerSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del origen de la actividad de copia debe establecerse en: **AzureDataExplorerSource** | Sí |
| Query | Una solicitud de solo lectura dada en un [formato KQL](/azure/kusto/query/). Use la consulta KQL personalizada como referencia. | Sí |
| queryTimeout | El tiempo de espera antes de que se agote el tiempo de espera de la solicitud de consulta. El valor predeterminado es de 10 minutos (00:10:00); el valor máximo permitido es de 1 hora (01: 00:00). | No |
| noTruncation | Indica si se debe truncar el conjunto de resultados devuelto. De forma predeterminada, el resultado se trunca pasados los 500 000 registros o los 64 megabytes (MB). Se recomienda encarecidamente el truncamiento para garantizar el comportamiento correcto de la actividad. |No |

>[!NOTE]
>De forma predeterminada, el origen de Azure Data Explorer tiene un límite de tamaño de 500 000 registros o 64 MB. Para recuperar todos los registros sin que se produzca truncamiento, puede especificar `set notruncation;` al principio de la consulta. Para más información, consulte [Límites de la consulta](/azure/kusto/concepts/querylimits).

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromAzureDataExplorer",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "AzureDataExplorerSource",
                "query": "TestTable1 | take 10",
                "queryTimeout": "00:10:00"
            },
            "sink": {
                "type": "<sink type>"
            }
        },
        "inputs": [
            {
                "referenceName": "<Azure Data Explorer input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

### <a name="azure-data-explorer-as-sink"></a>Azure Data Explorer como receptor

Para copiar datos en Azure Data Explorer, establezca la propiedad type del receptor de la actividad de copia en **AzureDataExplorerSink**. Se admiten las siguientes propiedades en la sección **sink** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** del receptor de la actividad de copia debe establecerse en: **AzureDataExplorerSink**. | Sí |
| ingestionMappingName | Nombre de una [asignación](/azure/kusto/management/mappings#csv-mapping) creada previamente en una tabla de Kusto. Para asignar las columnas del origen a Azure Data Explorer (lo que se aplica a todos los [almacenes y formatos de origen admitidos](copy-activity-overview.md#supported-data-stores-and-formats), incluidos los formatos CSV, JSON o Avro), puede usar la [asignación de columnas](copy-activity-schema-and-type-mapping.md) de la actividad de copia (implícitamente por nombre o explícitamente según la configuración) o las asignaciones de Azure Data Explorer. | No |
| additionalProperties | Un contenedor de propiedades que se puede usar para especificar cualquiera de las propiedades de ingesta que aún no ha establecido el receptor de Azure Data Explorer. En concreto, puede ser útil para especificar etiquetas de ingesta. Más información en la [documentación sobre ingesta de datos de Azure Data Explore](/azure/data-explorer/ingestion-properties). | No |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyToAzureDataExplorer",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "AzureDataExplorerSink",
                "ingestionMappingName": "<optional Azure Data Explorer mapping name>",
                "additionalProperties": {<additional settings for data ingestion>}
            }
        },
        "inputs": [
            {
                "referenceName": "<input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Azure Data Explorer output dataset name>",
                "type": "DatasetReference"
            }
        ]
    }
]
```

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para más información sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes

* Para ver una lista de los almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores, consulte los [almacenes de datos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).

* Obtenga más información sobre la [copia de datos de Azure Data Factory a Azure Data Explorer](/azure/data-explorer/data-factory-load-data).