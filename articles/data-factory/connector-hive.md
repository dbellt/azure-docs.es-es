---
title: Copiar datos de Hive con Azure Data Factory
description: Obtenga información sobre cómo copiar datos de Hive en almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 11/17/2020
ms.author: jianleishen
ms.openlocfilehash: 88f91ca27bad44f0b9d6235d257b141ce277b191
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485070"
---
# <a name="copy-and-transform-data-from-hive-using-azure-data-factory"></a>Copia y transformación de datos de Hive con Azure Data Factory 
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos de Hive. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Hive es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos de Hive en cualquier almacén de datos de receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

Azure Data Factory proporciona un controlador integrado para habilitar la conectividad. Por lo tanto, no es necesario instalar manualmente ningún controlador mediante este conector.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector de Hive.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Hive:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type debe establecerse en: **Hive** | Sí |
| host | Dirección IP o nombre de host del servidor de Hive, separados por ";" para varios hosts (solo cuando serviceDiscoveryMode está habilitado).  | Sí |
| port | Puerto TCP que el servidor de Hive utiliza para escuchar las conexiones del cliente. Si se conecta a Azure HDInsights, especifique el puerto 443. | Sí |
| serverType | Tipo de servidor de Hive. <br/>Los valores permitidos son: **HiveServer1**, **HiveServer2** y **HiveThriftServer** | No |
| thriftTransportProtocol | Protocolo de transporte que se va a usar en la capa de Thrift. <br/>Los valores permitidos son: **Binary**, **SASL** y **HTTP** | No |
| authenticationType | Método de autenticación que se usa para tener acceso al servidor de Hive. <br/>Los valores permitidos son: **Anonymous**, **Username**, **UsernameAndPassword** y **WindowsAzureHDInsightService**. Actualmente no se admite la autenticación de Kerberos. | Sí |
| serviceDiscoveryMode | True para indicar que se usa el servicio de ZooKeeper; false para indicar que no.  | No |
| zooKeeperNameSpace | Espacio de nombres en ZooKeeper en el que se agregan nodos de Hive Server 2.  | No |
| useNativeQuery | Especifica si el controlador usa las consultas nativas de HiveQL o las convierte en un formato equivalente en HiveQL.  | No |
| username | Nombre de usuario que utiliza para acceder al servidor de Hive.  | No |
| password | Contraseña que corresponde al usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). | No |
| httpPath | Dirección URL parcial correspondiente al servidor de Hive.  | No |
| enableSsl | Especifica si las conexiones al servidor se cifran mediante TLS. El valor predeterminado es false.  | No |
| trustedCertPath | Ruta de acceso completa del archivo .pem que contiene certificados de CA de confianza para comprobar el servidor al conectarse a través de TLS. Esta propiedad solo puede establecerse al utilizar TLS en IR autohospedados. El valor predeterminado es el archivo cacerts.pem instalado con el IR.  | No |
| useSystemTrustStore | Especifica si se utiliza un certificado de CA del almacén de confianza del sistema o de un archivo PEM especificado. El valor predeterminado es false.  | No |
| allowHostNameCNMismatch | Especifica si se requiere que el nombre del certificado TLS/SSL emitido por una CA coincida con el nombre de host del servidor al conectarse a través de TLS. El valor predeterminado es false.  | No |
| allowSelfSignedServerCert | Especifica si se permiten los certificados autofirmados del servidor. El valor predeterminado es false.  | No |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usará Azure Integration Runtime. |No |
| storageReference | Referencia al servicio vinculado de la cuenta de almacenamiento que se usa para el almacenamiento provisional de los datos en el flujo de datos de asignación. Solo se necesita cuando se usa el servicio vinculado de Hive en el flujo de datos de asignación. | No |

**Ejemplo**:

```json
{
    "name": "HiveLinkedService",
    "properties": {
        "type": "Hive",
        "typeProperties": {
            "host" : "<cluster>.azurehdinsight.net",
            "port" : "<port>",
            "authenticationType" : "WindowsAzureHDInsightService",
            "username" : "<username>",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        }
    }
}
```

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades compatibles con el conjunto de datos de Hive.

Para copiar datos de Hive, establezca la propiedad type del conjunto de datos en **HiveObject**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **HiveObject** | Sí |
| esquema | Nombre del esquema. |No (si se especifica "query" en el origen de la actividad)  |
| table | Nombre de la tabla. |No (si se especifica "query" en el origen de la actividad)  |
| tableName | Nombre de la tabla que incluye la parte del esquema. Esta propiedad permite la compatibilidad con versiones anteriores. Para la nueva carga de trabajo use `schema` y `table`. | No (si se especifica "query" en el origen de la actividad) |

**Ejemplo**

```json
{
    "name": "HiveDataset",
    "properties": {
        "type": "HiveObject",
        "typeProperties": {},
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Hive linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades compatibles con el origen de Hive.

### <a name="hivesource-as-source"></a>HiveSource como origen

Para copiar datos de Hive, establezca el tipo de origen de la actividad de copia en **HiveSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **HiveSource** | Sí |
| Query | Use la consulta SQL personalizada para leer los datos. Por ejemplo: `"SELECT * FROM MyTable"`. | No (si se especifica "tableName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromHive",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Hive input dataset name>",
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
                "type": "HiveSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

El conector de Hive se admite como un origen de [conjunto de datos en línea](data-flow-source.md#inline-datasets) en los flujos de datos de asignación. Lea mediante una consulta o directamente desde una tabla de Hive en HDInsight. Los datos de Hive se almacenan provisionalmente en una cuenta de almacenamiento como archivos de Parquet antes de transformarse como parte de un flujo de datos. 

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se indican las propiedades que admite un origen de Hive. Puede editar estas propiedades en la pestaña **Source options** (Opciones del origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Tienda | Debe ser `hive` | sí |  `hive` | store | 
| Formato | Si está leyendo desde una tabla o una consulta | sí | `table` o `query` | format |
| Nombre del esquema | Si está leyendo desde una tabla, el esquema de la tabla de origen |  sí, si el formato es `table` | String | schemaName |
| Nombre de la tabla | Si está leyendo desde una tabla, el nombre de la tabla |   sí, si el formato es `table` | String | tableName |
| Consultar | Si el formato es `query`, la consulta de origen en el servicio vinculado de Hive | sí, si el formato es `query` | String | Query |
| Almacenado provisionalmente | La tabla de Hive siempre va a estar almacenada provisionalmente. | sí | `true` | staged |
| Contenedor de almacenamiento | Contenedor de almacenamiento que se usa para almacenar provisionalmente los datos antes de leer desde Hive o escribir en Hive. El clúster de Hive debe tener acceso a este contenedor. | sí | String | storageContainer |
| Base de datos provisional | El esquema o la base de datos a los que tiene acceso la cuenta de usuario especificada en el servicio vinculado. Se usa para crear tablas externas durante el almacenamiento provisional y luego se elimina | no | `true` o `false` | stagingDatabaseName |
| Scripts de SQL previos | Código SQL que se va a ejecutar en la tabla de Hive antes de leer los datos | no | String | preSQLs |

#### <a name="source-example"></a>Ejemplo de origen

A continuación se muestra un ejemplo de una configuración de origen de Hive:

![Ejemplo de origen de Hive](media/data-flow/hive-source.png "[Ejemplo de origen de Hive")

Esta configuración se traduce en el siguiente script de flujo de datos:

```
source(
    allowSchemaDrift: true,
    validateSchema: false,
    ignoreNoFilesFound: false,
    format: 'table',
    store: 'hive',
    schemaName: 'default',
    tableName: 'hivesampletable',
    staged: true,
    storageContainer: 'khive',
    storageFolderPath: '',
    stagingDatabaseName: 'default') ~> hivesource
```
### <a name="known-limitations"></a>Restricciones conocidas

* No se admite la lectura de tipos complejos como matrices, asignaciones, structs y uniones. 
* El conector de Hive solo admite tablas de Hive en Azure HDInsight de la versión 4.0 o posterior (Apache Hive 3.1.0)

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).


## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
