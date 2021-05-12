---
title: Actividad de obtención de metadatos en Azure Data Factory
description: Obtenga información sobre cómo usar la actividad de obtención de metadatos en una canalización de Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/25/2021
ms.author: jianleishen
ms.openlocfilehash: e4715802f7b284bd9e6ce9a41b44c3750c33c260
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109482766"
---
# <a name="get-metadata-activity-in-azure-data-factory"></a>Actividad de obtención de metadatos en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

La actividad de obtención de metadatos se puede usar para recuperar metadatos de cualquier dato en Azure Data Factory. Puede usar la salida de la actividad de obtención de metadatos en expresiones condicionales para realizar la validación o consumir los metadatos en actividades posteriores.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

La actividad de obtención de metadatos toma un conjunto de datos como entrada y genera información de metadatos como salida. Actualmente, se admiten los siguientes conectores y los metadatos recuperables correspondientes. El tamaño máximo de los metadatos devueltos es de **4 MB**.

### <a name="supported-connectors"></a>Conectores compatibles

**Almacenamiento de archivos**

| Conector/Metadatos | itemName<br>(archivo/carpeta) | itemType<br>(archivo/carpeta) | tamaño<br>(archivo) | created<br>(archivo/carpeta) | lastModified<sup>1</sup><br>(archivo/carpeta) |childItems<br>(carpeta) |contentMD5<br>(archivo) | structure<sup>2</sup><br/>(archivo) | columnCount<sup>2</sup><br>(archivo) | exists<sup>3</sup><br>(archivo/carpeta) |
|:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |:--- |
| [Amazon S3](connector-amazon-simple-storage-service.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Google Cloud Storage](connector-google-cloud-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Almacenamiento de blobs de Azure](connector-azure-blob-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | √ | √ | √ | √/√ |
| [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md) | √/√ | √/√ | √ | x/x | √/√ | √ | √ | √ | √ | √/√ |
| [Archivos de Azure](connector-azure-file-storage.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [Sistema de archivos](connector-file-system.md) | √/√ | √/√ | √ | √/√ | √/√ | √ | x | √ | √ | √/√ |
| [SFTP](connector-sftp.md) | √/√ | √/√ | √ | x/x | √/√ | √ | x | √ | √ | √/√ |
| [FTP](connector-ftp.md) | √/√ | √/√ | √ | x/x | x/x | √ | x | √ | √ | √/√ |

<sup>1</sup> Metadatos `lastModified`:
- En el caso de Amazon S3 y Google Cloud Storage, `lastModified` se aplica al cubo y a la clave, pero no a la carpeta virtual, y `exists` se aplica al cubo y a la clave, pero no al prefijo ni a la carpeta virtual. 
- En el caso de Azure Blob Storage, `lastModified` se aplica al contenedor y al blob, pero no a la carpeta virtual.

<sup>2</sup> Los metadatos `structure` y `columnCount` no se admiten cuando se obtienen metadatos de archivos binarios, JSON o XML.

<sup>3</sup> Metadatos `exists`: para Amazon S3 y Google Cloud Storage, `exists` se aplica al depósito y a la clave, pero no al prefijo o a la carpeta virtual.

Tenga en cuenta lo siguiente:

- Cuando use la actividad Obtener metadatos en una carpeta, asegúrese de tener el permiso LIST/EXECUTE en la carpeta en cuestión.
- No se admite el filtro de carácter comodín en carpetas o archivos para la actividad de obtención de metadatos.
- Filtro `modifiedDatetimeStart` y `modifiedDatetimeEnd` establecido en el conector:

    - Estas dos propiedades se usan para filtrar los elementos secundarios al obtener los metadatos de una carpeta. No se aplica cuando se obtienen de un archivo.
    - Cuando se usa este tipo de filtro, `childItems` en la salida incluye solo los archivos que se modifican dentro del rango especificado, pero no las carpetas.
    - Para aplicar este filtro, la actividad GetMetadata enumerará todos los archivos de la carpeta especificada y comprobará la hora modificada. Evite apuntar a una carpeta con un gran número de archivos aunque el recuento de archivos completos esperado sea pequeño. 

**Base de datos relacional**

| Conector/Metadatos | structure | columnCount | exists |
|:--- |:--- |:--- |:--- |
| [Azure SQL Database](connector-azure-sql-database.md) | √ | √ | √ |
| [Instancia administrada de Azure SQL](../azure-sql/managed-instance/sql-managed-instance-paas-overview.md) | √ | √ | √ |
| [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) | √ | √ | √ |
| [SQL Server](connector-sql-server.md) | √ | √ | √ |

### <a name="metadata-options"></a>Opciones de metadatos

Puede especificar los siguientes tipos de metadatos en la lista de campos de la actividad de obtención de metadatos para recuperar la información correspondiente:

| Tipo de metadatos | Descripción |
|:--- |:--- |
| itemName | Nombre del archivo o la carpeta. |
| itemType | Tipo del archivo o la carpeta. El valor devuelto es `File` o `Folder`. |
| tamaño | Tamaño del archivo, en bytes. Se aplica solo a los archivos. |
| created | Fecha y hora de creación del archivo o la carpeta. |
| lastModified | Fecha y hora de la última modificación del archivo o la carpeta. |
| childItems | Lista de subcarpetas y archivos dentro de la carpeta determinada. Se aplica solo a las carpetas. El valor devuelto es una lista del nombre y el tipo de cada elemento secundario. |
| contentMD5 | MD5 del archivo. Se aplica solo a los archivos. |
| structure | Estructura de datos del archivo o la tabla de base de datos relacional. El valor devuelto es una lista de nombres de columna y tipos de columna. |
| columnCount | Número de columnas en el archivo o la tabla relacional. |
| exists| Si un archivo, carpeta o tabla existe. Si se especifica `exists` en la lista de campos de obtención de metadatos, la actividad no producirá un error, aunque el archivo, la carpeta o la tabla no existan. En su lugar, se devuelve `exists: false` en la salida. |

> [!TIP]
> Si desea validar que existe un archivo, una carpeta o una tabla, especifique `exists` en la lista de campos de la actividad de obtención de metadatos. A continuación, puede comprobar el resultado `exists: true/false` en la salida de la actividad. Si `exists` no se especifica en la lista de campos, se producirá un error en la actividad de obtención de metadatos si no se encuentra el objeto.

> [!NOTE]
> Si obtiene metadatos de almacenes de archivos y configura `modifiedDatetimeStart` o `modifiedDatetimeEnd`, el elemento `childItems` de la salida solo incluye los archivos de la ruta especificada que tengan una hora de última modificación que se encuentre dentro del intervalo especificado. No se incluyen los elementos de las subcarpetas.

> [!NOTE]
> Para que la lista de campos de **estructura** proporcione la estructura de datos real de los conjuntos de datos con formato de texto delimitado y Excel, debe habilitar la propiedad `First Row as Header`, que solo es compatible con estos orígenes de datos.

## <a name="syntax"></a>Sintaxis

**Actividad de obtención de metadatos**

```json
{
    "name":"MyActivity",
    "type":"GetMetadata",
    "dependsOn":[

    ],
    "policy":{
        "timeout":"7.00:00:00",
        "retry":0,
        "retryIntervalInSeconds":30,
        "secureOutput":false,
        "secureInput":false
    },
    "userProperties":[

    ],
    "typeProperties":{
        "dataset":{
            "referenceName":"MyDataset",
            "type":"DatasetReference"
        },
        "fieldList":[
            "size",
            "lastModified",
            "structure"
        ],
        "storeSettings":{
            "type":"AzureBlobStorageReadSettings"
        },
        "formatSettings":{
            "type":"JsonReadSettings"
        }
    }
}
```

**Dataset**

```json
{
    "name":"MyDataset",
    "properties":{
        "linkedServiceName":{
            "referenceName":"AzureStorageLinkedService",
            "type":"LinkedServiceReference"
        },
        "annotations":[

        ],
        "type":"Json",
        "typeProperties":{
            "location":{
                "type":"AzureBlobStorageLocation",
                "fileName":"file.json",
                "folderPath":"folder",
                "container":"container"
            }
        }
    }
}
```

## <a name="type-properties"></a>Propiedades de tipo

Actualmente, la actividad de obtención de metadatos puede capturar los siguientes tipos de información de metadatos:

Propiedad | Descripción | Obligatorio
-------- | ----------- | --------
fieldList | Los tipos de información de metadatos necesarios. Para más información sobre los metadatos admitidos, consulte la sección [Opciones de metadatos](#metadata-options) de este artículo. | Sí 
dataset | Conjunto de datos de referencia cuyos metadatos debe recuperar la actividad de obtención de metadatos. Consulte la sección[Capacidades](#supported-capabilities) para obtener información sobre los conectores admitidos. Consulte los temas específicos del conector para obtener información detallada sobre la sintaxis del conjunto de datos. | Sí
formatSettings | Se aplica cuando se usa el tipo de formato dataset. | No
storeSettings | Se aplica cuando se usa el tipo de formato dataset. | No

## <a name="sample-output"></a>Salida de ejemplo

Los resultados de la obtención de metadatos se muestran en la salida de la actividad. A continuación se muestran dos ejemplos que muestran amplias opciones de metadatos. Para utilizar los resultados en una actividad posterior, utilice este patrón: `@{activity('MyGetMetadataActivity').output.itemName}`.

### <a name="get-a-files-metadata"></a>Obtener los metadatos de un archivo

```json
{
  "exists": true,
  "itemName": "test.csv",
  "itemType": "File",
  "size": 104857600,
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "contentMD5": "cMauY+Kz5zDm3eWa9VpoyQ==",
  "structure": [
    {
        "name": "id",
        "type": "Int64"
    },
    {
        "name": "name",
        "type": "String"
    }
  ],
  "columnCount": 2
}
```

### <a name="get-a-folders-metadata"></a>Obtener los metadatos de una carpeta

```json
{
  "exists": true,
  "itemName": "testFolder",
  "itemType": "Folder",
  "lastModified": "2017-02-23T06:17:09Z",
  "created": "2017-02-23T06:17:09Z",
  "childItems": [
    {
      "name": "test.avro",
      "type": "File"
    },
    {
      "name": "folder hello",
      "type": "Folder"
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes
Consulte otras actividades de flujo de control compatibles con Data Factory:

- [Actividad de ejecución de canalización](control-flow-execute-pipeline-activity.md)
- [Actividad ForEach](control-flow-for-each-activity.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad web](control-flow-web-activity.md)
