---
title: Copia de datos desde Cassandra mediante Azure Data Factory
description: Obtenga información sobre cómo copiar datos desde Cassandra a almacenes de datos receptores compatibles a través de una actividad de copia de una canalización de Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 08/12/2019
ms.author: jianleishen
ms.openlocfilehash: 50aa5053cf7eff3a56b9afe37dcded033b6988b9
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109487410"
---
# <a name="copy-data-from-cassandra-using-azure-data-factory"></a>Copia de datos desde Cassandra mediante Azure Data Factory
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-onprem-cassandra-connector.md)
> * [Versión actual](connector-cassandra.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una base de datos Cassandra. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Cassandra es compatible con las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)

Puede copiar datos desde una base de datos Cassandra a cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector Cassandra admite lo siguiente:

- Las **versiones 2.X y 3.x** de Cassandra.
- Copiar datos mediante la autenticación **Básica** o **Anónima**.

>[!NOTE]
>Para la actividad que se ejecuta en Integration Runtime (autohospedado), Cassandra 3.x es compatible a partir de IR 3.7 y versiones posteriores.

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

El entorno Integration Runtime proporciona un controlador de Cassandra integrado, por lo tanto, no es necesario instalar manualmente los controladores cuando se copian datos desde Cassandra.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector Cassandra.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de Cassandra:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **Cassandra** |Sí |
| host |Una o varias direcciones IP o nombres de host de los servidores de Cassandra.<br/>Especifica una lista de direcciones IP o nombres de host separada por comas para conectar con todos los servidores a la vez. |Sí |
| port |Puerto TCP que el servidor de Cassandra utiliza para escuchar las conexiones del cliente. |No (el valor predeterminado es 9042). |
| authenticationType | El tipo de autenticación que se utiliza para conectarse a la base de datos Cassandra.<br/>Los valores permitidos son: **Basic** (básica) y **Anonymous** (anónima). |Sí |
| username |Especifique el nombre de usuario de la cuenta de usuario. |Sí, si el valor de authenticationType es Basic. |
| password |Especifique la contraseña para la cuenta de usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). |Sí, si el valor de authenticationType es Basic. |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usará Azure Integration Runtime. |No |

>[!NOTE]
>Actualmente no se admite la conexión a Cassandra mediante TLS.

**Ejemplo**:

```json
{
    "name": "CassandraLinkedService",
    "properties": {
        "type": "Cassandra",
        "typeProperties": {
            "host": "<host>",
            "authenticationType": "Basic",
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Cassandra.

Para copiar datos desde Cassandra, establezca la propiedad type del conjunto de datos en **CassandraTable**. Se admiten las siguientes propiedades:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **CassandraTable** | Sí |
| keyspace |Nombre del espacio de claves o esquema de la base de datos de Cassandra. |Sí (si no hay especificada ninguna "consulta" para "CassandraSource") |
| tableName |Nombre de la tabla de la base de datos de Cassandra. |Sí (si no hay especificada ninguna "consulta" para "CassandraSource") |

**Ejemplo**:

```json
{
    "name": "CassandraDataset",
    "properties": {
        "type": "CassandraTable",
        "typeProperties": {
            "keySpace": "<keyspace name>",
            "tableName": "<table name>"
        },
        "schema": [],
        "linkedServiceName": {
            "referenceName": "<Cassandra linked service name>",
            "type": "LinkedServiceReference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia


Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen Cassandra.

### <a name="cassandra-as-source"></a>Cassandra como origen

Para copiar datos desde Cassandra, establezca el tipo de origen de la actividad de copia en **CassandraSource**. Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **CassandraSource** | Sí |
| Query |Utilice la consulta personalizada para leer los datos. Consulta SQL-92 o consulta CQL. Vea la [CQL reference](https://docs.datastax.com/en/cql/3.1/cql/cql_reference/cqlReferenceTOC.html)(referencia de CQL). <br/><br/>Cuando utilice una consulta SQL, especifique **nombre de espacio de claves.nombre de tabla** para representar la tabla que quiere consultar. |No (si "tableName" y "keyspace" en el conjunto de datos están especificados) |
| consistencyLevel |El nivel de coherencia establece el número de réplicas que deben responder a una solicitud de lectura antes de que se devuelvan datos a la aplicación cliente. Cassandra comprueba el número de réplicas especificado para que los datos satisfagan la solicitud de lectura. Para más información, consulte [Configuring data consistency](https://docs.datastax.com/en/cassandra/2.1/cassandra/dml/dml_config_consistency_c.html) (Configuración de la coherencia de datos).<br/><br/>Los valores permitidos son: **ONE**, **TWO**, **THREE**, **QUORUM**, **ALL**, **LOCAL_QUORUM**, **EACH_QUORUM** y **LOCAL_ONE**. |No (el valor predeterminado es `ONE`) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromCassandra",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Cassandra input dataset name>",
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
                "type": "CassandraSource",
                "query": "select id, firstname, lastname from mykeyspace.mytable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

## <a name="data-type-mapping-for-cassandra"></a>Asignación de tipos de Cassandra

Al copiar datos desde Cassandra, se utilizan las siguientes asignaciones de tipos de datos de Cassandra en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de Cassandra | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| ASCII |String |
| bigint |Int64 |
| BLOB |Byte[] |
| BOOLEAN |Boolean |
| DECIMAL |Decimal |
| DOUBLE |Double |
| FLOAT |Single |
| INET |String |
| INT |Int32 |
| TEXT |String |
| timestamp |DateTime |
| TIMEUUID |Guid |
| UUID |Guid |
| VARCHAR |String |
| VARINT |Decimal |

> [!NOTE]
> Par más información sobre tipos de colecciones (map, set, list, etc.), consulte la sección [Uso de colecciones con tablas virtuales](#work-with-collections-using-virtual-table) .
>
> No se admiten tipos definidos por el usuario.
>
> La longitud de la columna binaria y la columna de cadena no puede ser superior a 4000.
>

## <a name="work-with-collections-using-virtual-table"></a>Uso de colecciones con tablas virtuales

Azure Data Factory utiliza un controlador ODBC integrado para conectarse a una base de datos de Cassandra y copiar datos de dicha base de datos. En el caso de los tipos de colección (como map, set y list), el controlador volverá a normalizar los datos en las tablas virtuales correspondientes. En concreto, si una tabla contiene columnas de colecciones, el controlador generará las siguientes tablas virtuales:

* Una **tabla base**, que contiene los mismos datos que la tabla real, salvo las columnas de colecciones. La tabla base utiliza el mismo nombre que la tabla real a la que representa.
* Una **tabla virtual** para cada columna de la colección, que ampliará los datos anidados. Para asignar un nombre a las tablas virtuales que representan colecciones, se utiliza el nombre de la tabla real, un separador "*vt*" y el nombre de la columna.

Las tablas virtuales hacen referencia a los datos de la tabla real, lo que permite al controlador obtener acceso a los datos no normalizados. Consulte la sección de ejemplo para más información. Para acceder al contenido de las colecciones de Cassandra, puede crear consultas y combinar las tablas virtuales.

### <a name="example"></a>Ejemplo

Por ejemplo, el siguiente "ExampleTable" es una tabla de una base de datos de Cassandra que contiene una columna de clave principal de enteros denominada “pk_int”, una columna de texto denominada "value", una columna List, una columna Map y una columna Set (denominada "StringSet").

| pk_int | Value | List | Map | StringSet |
| --- | --- | --- | --- | --- |
| 1 |"valor de ejemplo 1" |["1", "2", "3"] |{"S1": "a", "S2": "b"} |{"A", "B", "C"} |
| 3 |"valor de ejemplo 3" |["100", "101", "102", "105"] |{"S1": "t"} |{"A", "E"} |

El controlador generará varias tablas virtuales que representan a esta tabla. Las columnas de clave externa de las tablas virtuales hacen referencia a las columnas de clave principal de la tabla real e indican qué fila de la tabla real se corresponde con la fila de la tabla virtual.

La primera tabla virtual es la tabla base y se denomina "ExampleTable", tal y como se muestra en la siguiente tabla: 

| pk_int | Value |
| --- | --- |
| 1 |"valor de ejemplo 1" |
| 3 |"valor de ejemplo 3" |

La tabla base contiene los mismos datos que la tabla de base de datos original a excepción de las colecciones, que no aparecen en esta tabla, sino que se amplían en otras tablas virtuales.

Las tablas siguientes representan las tablas virtuales que normalizan de nuevo los datos de las columnas List, Map y StringSet. Las columnas cuyos nombres terminan en "_index" o "_key" indican la posición de los datos en la columna List o Map original. Las columnas cuyos nombres terminan en "_value" contienen los datos ampliados de la colección.

**Tabla "ExampleTable_vt_List":**

| pk_int | List_index | List_value |
| --- | --- | --- |
| 1 |0 |1 |
| 1 |1 |2 |
| 1 |2 |3 |
| 3 |0 |100 |
| 3 |1 |101 |
| 3 |2 |102 |
| 3 |3 |103 |

**Tabla "ExampleTable_vt_Map":**

| pk_int | Map_key | Map_value |
| --- | --- | --- |
| 1 |S1 |Un |
| 1 |S2 |b |
| 3 |S1 |t |

**Tabla "ExampleTable_vt_StringSet":**

| pk_int | StringSet_value |
| --- | --- |
| 1 |A |
| 1 |B |
| 1 |C |
| 3 |Un |
| 3 |E |

## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
