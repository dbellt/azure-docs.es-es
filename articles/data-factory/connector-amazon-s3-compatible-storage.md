---
title: Copia de datos de Amazon Simple Storage Service (S3) Compatible Storage
description: Aprenda a copiar datos de Amazon S3 Compatible Storage a almacenes de datos de receptor compatibles mediante Azure Data Factory.
ms.author: jianleishen
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 05/11/2021
ms.openlocfilehash: 96b8ac29fadc03ca6e5b374acc269d5e19210b03
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110104491"
---
# <a name="copy-data-from-amazon-s3-compatible-storage-by-using-azure-data-factory"></a>Copia de datos de Amazon S3 Compatible Storage mediante Azure Data Factory


[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se describe cómo copiar datos de Amazon Simple Storage Service (Amazon S3) Compatible Storage. Para información sobre Azure Data Factory, lea el [artículo de introducción](introduction.md).



## <a name="supported-capabilities"></a>Funcionalidades admitidas

Este conector de Amazon S3 Compatible Storage se admite en las actividades siguientes:

- [Actividad de copia](copy-activity-overview.md) con [matriz de origen o receptor compatible](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)

Concretamente, este conector de Amazon S3 Compatible Storage admite la copia de archivos tal cual, o el análisis de los mismos con los [códecs de compresión y los formatos de archivo compatibles](supported-file-formats-and-compression-codecs.md). El conector usa [AWS Signature versión 4](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) para autenticar las solicitudes a S3. Este conector de Amazon S3 Compatible Storage se puede usar para copiar datos de cualquier proveedor de almacenamiento compatible con S3. Especifique la dirección URL del servicio correspondiente en la configuración del servicio vinculado.



## <a name="required-permissions"></a>Permisos necesarios

Para copiar datos de Amazon S3 Compatible Storage, asegúrese de que se han concedido los permisos siguientes para las operaciones de objeto de Amazon S3: `s3:GetObject` y `s3:GetObjectVersion`.

Si usa la interfaz de usuario de Data Factory para crear, se requieren los permisos `s3:ListAllMyBuckets` y `s3:ListBucket`/`s3:GetBucketLocation` adicionales para operaciones como probar la conexión al servicio vinculado y examinar desde la raíz. Si no quiere conceder estos permisos, puede elegir las opciones "Test connection to file path" (Probar conexión con la ruta de acceso del archivo) o "Browse from specified path" (Examinar desde la ruta de acceso especificada) en la interfaz de usuario.

Para obtener la lista completa de los permisos de Amazon S3, consulte [Specifying Permissions in a Policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html) (Especificación de permisos en una directiva) en el sitio de AWS.

## <a name="getting-started"></a>Introducción

[!INCLUDE [data-factory-v2-connector-get-started](includes/data-factory-v2-connector-get-started.md)] 

En las secciones siguientes se proporcionan detalles sobre las propiedades que se usan para definir entidades de Data Factory específicas de Amazon S3 Compatible Storage.

## <a name="linked-service-properties"></a>Propiedades del servicio vinculado

Las siguientes propiedades se admiten en el servicio vinculado Amazon S3 Compatible:

| Propiedad | Descripción | Obligatorio |
|:--- |:--- |:--- |
| type | La propiedad **type** debe establecerse en **AmazonS3Compatible**. | Sí |
| accessKeyId | Id. de la clave de acceso secreta. |Sí |
| secretAccessKey | La propia clave de acceso secreta. Marque este campo como [SecureString](store-credentials-in-key-vault.md) para almacenarlo de forma segura en Data Factory, o bien **para hacer referencia a un secreto almacenado en Azure Key Vault**. |Sí |
| serviceUrl | Especifique el punto de conexión de S3 personalizado `https://<service url>`. | No |
| forcePathStyle | Indica si se va a usar el [acceso de estilo ruta de acceso](https://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html#path-style-access) de S3 en lugar del acceso de estilo hospedado virtual. Los valores permitidos son **false** (valor predeterminado) y **true**.<br> Consulte la documentación de cada almacén de datos para saber si el acceso de estilo ruta de acceso es o no necesario. |No |
| connectVia | El [entorno de ejecución de integración](concepts-integration-runtime.md) que se usará para conectarse al almacén de datos. Se puede usar Azure Integration Runtime o un entorno de ejecución de integración autohospedado (si el almacén de datos está en una red privada). Si no se especifica esta propiedad, el servicio usa el valor predeterminado de Azure Integration Runtime. |No |



**Ejemplo**:

```json
{
    "name": "AmazonS3CompatibleLinkedService",
    "properties": {
        "type": "AmazonS3Compatible",
        "typeProperties": {
            "accessKeyId": "<access key id>",
            "secretAccessKey": {
                "type": "SecureString",
                "value": "<secret access key>"
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

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten en Amazon S3 Compatible en la configuración `location` de un conjunto de datos basado en formato:

| Propiedad   | Descripción                                                  | Obligatorio |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | La propiedad **type** de la sección `location` de un conjunto de datos se debe establecer en **AmazonS3CompatibleLocation**. | Sí      |
| bucketName | Nombre del cubo de S3 Compatible Storage.                                          | Sí      |
| folderPath | Ruta de acceso a la carpeta del cubo especificado. Si quiere usar un carácter comodín para filtrar la carpeta, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| fileName   | Nombre de archivo en el cubo y la ruta de acceso de la carpeta indicados. Si quiere usar un carácter comodín para filtrar archivos, omita este valor y especifíquelo en la configuración del origen de actividad. | No       |
| version | La versión del objeto S3 Compatible Storage, si está habilitado el control de versiones de S3 Compatible Storage. Si no se especifica, se obtendrá la versión más reciente. |No |

**Ejemplo**:

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<Amazon S3 Compatible Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AmazonS3CompatibleLocation",
                "bucketName": "bucketname",
                "folderPath": "folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admite el origen de Amazon S3 Compatible Storage.

### <a name="amazon-s3-compatible-storage-as-a-source-type"></a>Amazon S3 Compatible Storage como tipo de origen

[!INCLUDE [data-factory-v2-file-formats](includes/data-factory-v2-file-formats.md)] 

Las propiedades siguientes se admiten para Amazon S3 Compatible Storage en la configuración `storeSettings` de un origen de la actividad de copia basado en formato:

| Propiedad                 | Descripción                                                  | Obligatorio                                                    |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------------- |
| type                     | La propiedad **type** de la sección `storeSettings` se debe establecer en **AmazonS3CompatibleReadSettings**. | Sí                                                         |
| ***Buscar los archivos que se van a copiar:*** |  |  |
| OPCIÓN 1: ruta de acceso estática<br> | Realice la copia desde el cubo o la ruta de acceso de archivos o carpeta especificadas en el conjunto de datos. Si quiere copiar todos los archivos de un cubo o carpeta, especifique también `wildcardFileName` como `*`. |  |
| OPCIÓN 2: prefijo de S3 Compatible Storage<br>- prefix | Prefijo del nombre de la clave de S3 Compatible Storage en el cubo específico configurado en un conjunto de datos para filtrar archivos de origen de S3 Compatible Storage. Se seleccionan las claves de S3 Compatible Storage cuyos nombres comienzan por `bucket_in_dataset/this_prefix`. Se emplea el filtro del servicio de S3 Compatible Storage, que proporciona un mejor rendimiento que el filtro de comodín.<br/><br/>Al usar el prefijo y elegir copiar en el receptor basado en archivos con la opción de conservar la jerarquía, tenga en cuenta que la subruta de acceso después del último "/" en el prefijo se conserva. Por ejemplo, si tiene el archivo `bucket/folder/subfolder/file.txt` de origen y configura el prefijo como `folder/sub`, la ruta de acceso de archivo conservada es `subfolder/file.txt`. | No |
| OPCIÓN 3: carácter comodín<br>- wildcardFolderPath | Ruta de acceso de carpeta con caracteres comodín en el cubo específico configurado en un conjunto de datos para filtrar las carpetas de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de la carpeta contiene un carácter comodín o este carácter de escape. <br>Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | No                                            |
| OPCIÓN 3: carácter comodín<br>- wildcardFileName | Nombre de archivo con caracteres comodín en el cubo y la ruta de carpeta (o ruta de carpeta con carácter comodín) indicada para filtrar los archivos de origen. <br>Los caracteres comodín permitidos son: `*` (equivale a cero o a varios caracteres) y `?` (equivale a cero o a un único carácter). Use `^` como escape si el nombre de archivo contiene un carácter comodín o este carácter de escape.  Ver más ejemplos en [Ejemplos de filtros de carpetas y archivos](#folder-and-file-filter-examples). | Sí |
| OPCIÓN 4: una lista de archivos<br>- fileListPath | Indica que se copie un conjunto de archivos determinado. Apunte a un archivo de texto que incluya una lista de los archivos que quiere copiar, con un archivo por línea, que sea la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos.<br/>Al usar esta opción, no especifique un nombre de archivo en el conjunto de datos. Ver más ejemplos en [Ejemplos de lista de archivos](#file-list-examples). |No |
| ***Configuración adicional:*** |  | |
| recursive | Indica si los datos se leen de forma recursiva de las subcarpetas o solo de la carpeta especificada. Tenga en cuenta que cuando **recursive** se establece en **true** y el receptor es un almacén basado en archivos, no se crea una carpeta o una subcarpeta vacía en el receptor. <br>Los valores permitidos son: **True** (valor predeterminado) y **False**.<br>Esta propiedad no se aplica al configurar `fileListPath`. |No |
| deleteFilesAfterCompletion | Indica si los archivos binarios se eliminarán del almacén de origen después de moverse correctamente al almacén de destino. Cada archivo se elimina individualmente, de modo que cuando se produzca un error en la actividad de copia, algunos archivos ya se habrán copiado al destino y se habrán eliminado del origen, mientras que otros seguirán aún en el almacén de origen. <br/>Esta propiedad solo es válida en el escenario de copia de archivos binarios. El valor predeterminado es false. |No |
| modifiedDatetimeStart    | Los archivos se filtran en función del atributo Last Modified. <br>Los archivos se seleccionarán si la hora de su última modificación está dentro del intervalo de tiempo entre `modifiedDatetimeStart` y `modifiedDatetimeEnd`. La hora se aplica a una zona horaria UTC en el formato "2018-12-01T05:00:00Z". <br> Las propiedades pueden ser **NULL**, lo que significa que no se aplica ningún filtro de atributo de archivo al conjunto de datos.  Cuando `modifiedDatetimeStart` tiene un valor de fecha y hora, pero `modifiedDatetimeEnd` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea mayor o igual que el valor de fecha y hora.  Cuando `modifiedDatetimeEnd` tiene un valor de fecha y hora, pero `modifiedDatetimeStart` es **NULL**, significa que se seleccionarán los archivos cuyo último atributo modificado sea menor que el valor de fecha y hora.<br/>Esta propiedad no se aplica al configurar `fileListPath`. | No                                            |
| modifiedDatetimeEnd      | Igual que el anterior.                                               | No                                                          |
| enablePartitionDiscovery | En el caso de archivos con particiones, especifique si quiere analizar las particiones de la ruta de acceso del archivo y agregarlas como columnas de origen adicionales.<br/>Los valores permitidos son **false** (valor predeterminado) y **true**. | No                                            |
| partitionRootPath | Cuando esté habilitada la detección de particiones, especifique la ruta de acceso raíz absoluta para poder leer las carpetas con particiones como columnas de datos.<br/><br/>Si no se especifica, de forma predeterminada,<br/>- Cuando se usa la ruta de acceso de archivo en un conjunto de datos o una lista de archivos del origen, la ruta de acceso raíz de la partición es la ruta de acceso configurada en el conjunto de datos.<br/>- Cuando se usa el filtro de carpeta con caracteres comodín, la ruta de acceso raíz de la partición es la subruta antes del primer carácter comodín.<br/>- Cuando se usa un prefijo, la ruta de acceso raíz de la partición es la subruta antes del último "/". <br/><br/>Por ejemplo, supongamos que configura la ruta de acceso en el conjunto de datos como "root/folder/year=2020/month=08/day=27":<br/>- Si especifica la ruta de acceso raíz de la partición como "root/folder/year=2020", la actividad de copia generará dos columnas más, `month` y `day`, con el valor "08" y "27", respectivamente, además de las columnas de los archivos.<br/>- Si no se especifica la ruta de acceso raíz de la partición, no se generará ninguna columna adicional. | No                                            |
| maxConcurrentConnections |Número máximo de conexiones simultáneas establecidas en el almacén de datos durante la ejecución de la actividad. Especifique un valor solo cuando quiera limitar las conexiones simultáneas.| No                                                          |

**Ejemplo**:

```json
"activities":[
    {
        "name": "CopyFromAmazonS3CompatibleStorage",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
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
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "AmazonS3CompatibleReadSettings",
                    "recursive": true,
                    "wildcardFolderPath": "myfolder*A",
                    "wildcardFileName": "*.csv"
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Ejemplos de filtros de carpetas y archivos

Esta sección describe el comportamiento resultante de la ruta de acceso de la carpeta y el nombre de archivo con los filtros de carácter comodín.

| bucket | key | recursive | Resultado de estructura de carpeta de origen y filtro (se recuperan los archivos en negrita)|
|:--- |:--- |:--- |:--- |
| bucket | `Folder*/*` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*` | true | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*.csv` | false | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File5.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |
| bucket | `Folder*/*.csv` | true | bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;AnotherFolderB<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File6.csv |

### <a name="file-list-examples"></a>Ejemplos de lista de archivos

En esta sección se describe el comportamiento resultante de usar una ruta de acceso de la lista de archivos en un origen de la actividad de copia.

Suponga que tiene la siguiente estructura de carpetas de origen y quiere copiar los archivos en negrita:

| Estructura de origen de ejemplo                                      | Contenido de FileListToCopy.txt                             | Configuración de Data Factory                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| bucket<br/>&nbsp;&nbsp;&nbsp;&nbsp;FolderA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Subfolder1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;File4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**File5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Metadatos<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | File1.csv<br>Subfolder1/File3.csv<br>Subfolder1/File5.csv | **En el conjunto de datos:**<br>- Cubo: `bucket`<br>- Ruta de acceso de la carpeta: `FolderA`<br><br>**En el origen de la actividad de copia:**<br>- Ruta de acceso de la lista de archivos: `bucket/Metadata/FileListToCopy.txt` <br><br>La ruta de acceso de la lista de archivos apunta a un archivo de texto en el mismo almacén de datos que incluye una lista de archivos que se quieren copiar, con un archivo por línea, con la ruta de acceso relativa a la ruta de acceso configurada en el conjunto de datos. |


## <a name="lookup-activity-properties"></a>Propiedades de la actividad de búsqueda

Para obtener información detallada sobre las propiedades, consulte [Actividad de búsqueda](control-flow-lookup-activity.md).

## <a name="getmetadata-activity-properties"></a>Propiedades de la actividad GetMetadata

Para información detallada sobre las propiedades, consulte [la actividad GetMetadata](control-flow-get-metadata-activity.md). 

## <a name="delete-activity-properties"></a>Propiedades de la actividad de eliminación

Para información detallada sobre las propiedades, consulte [Actividad de eliminación](delete-activity.md).


## <a name="next-steps"></a>Pasos siguientes
Para ver una lista de los almacenes de datos que la actividad de copia de Azure Data Factory admite como orígenes y receptores, consulte los [almacenes de datos compatibles](copy-activity-overview.md#supported-data-stores-and-formats).
