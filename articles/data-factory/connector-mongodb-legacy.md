---
title: Copia de datos de MongoDB (heredado)
description: Aprenda a copiar datos desde Mongo DB en almacenes de datos receptores compatibles mediante una actividad de copia de una canalización heredada de Azure Data Factory.
author: jianleishen
ms.author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019; seo-dt-2019
ms.date: 08/12/2019
ms.openlocfilehash: 4882949abd21e782d98dff8de2ba7be1b66ecdfb
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482190"
---
# <a name="copy-data-from-mongodb-using-azure-data-factory-legacy"></a>Copia de datos desde MongoDB mediante Azure Data Factory (heredado)

> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](v1/data-factory-on-premises-mongodb-connector.md)
> * [Versión actual](connector-mongodb.md)
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se explica el uso de la actividad de copia de Azure Data Factory para copiar datos desde una base de datos MongoDB. El documento se basa en el artículo de [introducción a la actividad de copia](copy-activity-overview.md) que describe información general de la actividad de copia.

>[!IMPORTANT]
>ADF publica un nuevo conector de MongoDB, que proporciona una mejor compatibilidad nativa con MongoDB en comparación con esta implementación basada en ODBC; para obtener más información, vea el artículo [Conector de MongoDB](connector-mongodb.md). Este conector de MongoDB heredado sigue recibiendo soporte tal cual para garantizar la compatibilidad con versiones anteriores; pero, para cualquier carga de trabajo nueva, use el nuevo conector.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Puede copiar datos desde la base de datos MongoDB en cualquier almacén de datos receptor compatible. Consulte la tabla de [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver una lista de almacenes de datos que la actividad de copia admite como orígenes o receptores.

En concreto, este conector MongoDB admite las siguientes funcionalidades:

- **Versiones 2.4, 2.6, 3.0, 3.2, 3.4 y 3.6** de MongoDB.
- Copiar datos mediante la autenticación **Básica** o **Anónima**.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [data-factory-v2-integration-runtime-requirements](includes/data-factory-v2-integration-runtime-requirements.md)]

El entorno Integration Runtime proporciona un controlador de MongoDB integrado, por lo tanto, no es necesario instalar manualmente los controladores cuando se copian datos desde MongoDB.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)]

Las secciones siguientes proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas del conector MongoDB.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades son compatibles con el servicio vinculado de MongoDB:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type |La propiedad type debe establecerse en: **MongoDB** |Sí |
| server |Dirección IP o nombre de host del servidor de MongoDB. |Sí |
| port |Puerto TCP que el servidor de MongoDB utiliza para escuchar las conexiones del cliente. |No (el valor predeterminado es 27017) |
| databaseName |Nombre de la base de datos de MongoDB a la que desea acceder. |Sí |
| authenticationType | Tipo de autenticación usado para conectarse a la base de datos MongoDB.<br/>Los valores permitidos son: **Basic** (básica) y **Anonymous** (anónima). |Sí |
| username |Cuenta de usuario para tener acceso a MongoDB. |Sí (si se usa la autenticación básica). |
| password |Contraseña del usuario. Marque este campo como SecureString para almacenarlo de forma segura en Data Factory o [para hacer referencia a un secreto almacenado en Azure Key Vault](store-credentials-in-key-vault.md). |Sí (si se usa la autenticación básica). |
| authSource |Nombre de la base de datos de MongoDB que desea usar para comprobar las credenciales de autenticación. |No. Para la autenticación básica, el valor predeterminado se utiliza la cuenta de administrador y la base de datos especificada mediante la propiedad databaseName. |
| enableSsl | Especifica si las conexiones al servidor se cifran mediante TLS. El valor predeterminado es false.  | No |
| allowSelfSignedServerCert | Especifica si se permiten los certificados autofirmados del servidor. El valor predeterminado es false.  | No |
| connectVia | El entorno [Integration Runtime](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Obtenga más información en la sección [Requisitos previos](#prerequisites). Si no se especifica, se usará Azure Integration Runtime. |No |

**Ejemplo**:

```json
{
    "name": "MongoDBLinkedService",
    "properties": {
        "type": "MongoDb",
        "typeProperties": {
            "server": "<server name>",
            "databaseName": "<database name>",
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

Para ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte [Conjuntos de datos y servicios vinculados](concepts-datasets-linked-services.md). Las siguientes propiedades son compatibles con el conjunto de datos de MongoDB:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del conjunto de datos debe establecerse en: **MongoDbCollection** | Sí |
| collectionName |Nombre de la colección en la base de datos de MongoDB. |Sí |

**Ejemplo**:

```json
{
    "name": "MongoDbDataset",
    "properties": {
        "type": "MongoDbCollection",
        "linkedServiceName": {
            "referenceName": "<MongoDB linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "collectionName": "<Collection name>"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen MongoDB.

### <a name="mongodb-as-source"></a>MongoDB como origen

Se admiten las siguientes propiedades en la sección **source** de la actividad de copia:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad type del origen de la actividad de copia debe establecerse en: **MongoDbSource** | Sí |
| Query |Utilice la consulta SQL-92 personalizada para leer los datos. Por ejemplo: select * from MyTable. |No (si se especifica "collectionName" en el conjunto de datos) |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromMongoDB",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<MongoDB input dataset name>",
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
                "type": "MongoDbSource",
                "query": "SELECT * FROM MyTable"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

> [!TIP]
> Cuando se especifica la consulta SQL, preste atención a la diferencia del formato de fecha y hora. Por ejemplo: `SELECT * FROM Account WHERE LastModifiedDate >= '2018-06-01' AND LastModifiedDate < '2018-06-02'` o para usar el parámetro `SELECT * FROM Account WHERE LastModifiedDate >= '@{formatDateTime(pipeline().parameters.StartTime,'yyyy-MM-dd HH:mm:ss')}' AND LastModifiedDate < '@{formatDateTime(pipeline().parameters.EndTime,'yyyy-MM-dd HH:mm:ss')}'`

## <a name="schema-by-data-factory"></a>Esquema de Data Factory

El servicio de Azure Data Factory deduce el esquema de una colección de MongoDB mediante el **uso de los últimos 100 documentos de la colección**. Si estos 100 documentos no contienen el esquema completo, se pueden omitir algunas columnas durante la operación de copia.

## <a name="data-type-mapping-for-mongodb"></a>Asignación de tipos para MongoDB

Al copiar datos desde MongoDB, se utilizan las siguientes asignaciones de tipos de datos de MongoDB en los tipos de datos provisionales de Azure Data Factory. Consulte el artículo sobre [asignaciones de tipos de datos y esquema](copy-activity-schema-and-type-mapping.md) para información sobre cómo la actividad de copia asigna el tipo de datos y el esquema de origen al receptor.

| Tipo de datos de MongoDB | Tipo de datos provisionales de Data Factory |
|:--- |:--- |
| Binary |Byte[] |
| Boolean |Boolean |
| Date |DateTime |
| NumberDouble |Double |
| NumberInt |Int32 |
| NumberLong |Int64 |
| ObjectID |String |
| String |String |
| UUID |Guid |
| Object |Renormalizado en columnas acopladas con "_" como separador anidado |

> [!NOTE]
> Para obtener más información sobre la compatibilidad con matrices con tablas virtuales, consulte la sección [Compatibilidad para tipos complejos que usan tablas virtuales](#support-for-complex-types-using-virtual-tables) que aparece más adelante.
>
> Actualmente, los siguientes tipos de datos de MongoDB no se admiten: DBPointer, JavaScript, clave máx./mín., expresión regular, símbolo, marca de tiempo, sin definir.

## <a name="support-for-complex-types-using-virtual-tables"></a>Compatibilidad para tipos complejos que usan tablas virtuales

Azure Data Factory utiliza un controlador ODBC integrado para conectarse a una base de datos de MongoDB y copiar datos de dicha base de datos. Para los tipos complejos como matrices u objetos con diferentes tipos en los documentos, el controlador volverá a normalizar los datos en las tablas virtuales correspondientes. En concreto, si una tabla contiene estas columnas, el controlador generará las siguientes tablas virtuales:

* Una **tabla base**, que contiene los mismos datos que la tabla real, salvo las columnas de tipo complejo. La tabla base utiliza el mismo nombre que la tabla real a la que representa.
* Una **tabla virtual** para cada columna de tipo complejo, que ampliará los datos anidados. Para asignar un nombre a las tablas virtuales, se utiliza el nombre de la tabla real, un separador "_" y el nombre de la matriz u objeto.

Las tablas virtuales hacen referencia a los datos de la tabla real, lo que permite al controlador obtener acceso a los datos no normalizados. Para acceder al contenido de las matrices de MongoDB, puede crear consultas y combinar las tablas virtuales.

### <a name="example"></a>Ejemplo

Por ejemplo, la tabla "ExampleTable" que aparece a continuación es una tabla de MongoDB que tiene una columna con una matriz de objetos en cada celda: Facturas y una columna con una matriz de tipos escalares, Clasificaciones.

| _id | Nombre del cliente | Facturas | Nivel de servicio | Clasificaciones |
| --- | --- | --- | --- | --- |
| 1111 |ABC |[{invoice_id:"123", item:"toaster", price:"456", discount:"0.2"}, {invoice_id:"124", item:"oven", price: "1235", discount: "0.2"}] |Plata |[5,6] |
| 2222 |XYZ |[{invoice_id:"135", item:"fridge", price: "12543", discount: "0.0"}] |Oro |[1,2] |

El controlador generará varias tablas virtuales que representan a esta tabla. La primera tabla virtual es la tabla base y se denomina “ExampleTable”, tal y como se muestra en el ejemplo. La tabla base contiene todos los datos de la tabla original, pero los datos de las matrices se han omitido y se ampliarán en las tablas virtuales.

| _id | Nombre del cliente | Nivel de servicio |
| --- | --- | --- |
| 1111 |ABC |Plata |
| 2222 |XYZ |Oro |

Las siguientes tablas muestran las tablas virtuales que representan las matrices originales en el ejemplo. Estas tablas contienen lo siguiente:

* Una referencia a la columna de clave principal original correspondiente a la fila de la matriz original (a través del identificador de la columna)
* Una indicación de la posición de los datos dentro de la matriz original
* Los datos ampliados para cada elemento de la matriz

**Tabla "ExampleTable_Invoices":**

| _id | ExampleTable_Invoices_dim1_idx | invoice_id | item | price | Descuento |
| --- | --- | --- | --- | --- | --- |
| 1111 |0 |123 |tostadora |456 |0,2 |
| 1111 |1 |124 |horno |1235 |0,2 |
| 2222 |0 |135 |frigorífico |12543 |0,0 |

**Tabla "ExampleTable_Ratings":**

| _id | ExampleTable_Ratings_dim1_idx | ExampleTable_Ratings |
| --- | --- | --- |
| 1111 |0 |5 |
| 1111 |1 |6 |
| 2222 |0 |1 |
| 2222 |1 |2 |

## <a name="next-steps"></a>Pasos siguientes
Consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats) para ver la lista de almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores.
