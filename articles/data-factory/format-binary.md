---
title: Formato binario en Azure Data Factory
description: En este tema se describe cómo tratar con el formato binario en Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/29/2020
ms.author: jianleishen
ms.openlocfilehash: 816251e7a27c1b7693ac5fc42ed912135e7a9460
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486636"
---
# <a name="binary-format-in-azure-data-factory"></a>Formato binario en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

El formato binario se admite para los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md).

Puede utilizar un conjunto de datos binarios en la [actividad de copia](copy-activity-overview.md), la [actividad GetMetadata](control-flow-get-metadata-activity.md) o la [actividad de eliminación](delete-activity.md). Cuando se usa un conjunto de datos binario, ADF no analiza el contenido del archivo, sino que lo trata tal cual. 

>[!NOTE]
>Al utilizar un conjunto de archivos binario en la actividad de copia, solo puede copiar de un conjunto de datos binario a un conjunto de datos binario.

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos binarios.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **binario**. | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`. **Vea los detalles en el artículo de conectores -> sección de propiedades del conjunto de datos**. | Sí      |
| compression | Grupo de propiedades para configurar la compresión de archivo. Configure esta sección si desea realizar la compresión o descompresión durante la ejecución de la actividad. | No |
| type | El códec de compresión usado para leer y escribir archivos binarios. <br>Los valores permitidos son **bzip2**, **gzip**, **deflate**, **ZipDeflate**, **Tar** o **TarGzip**. <br>**Tenga en cuenta** que, cuando se utiliza la actividad de copia para descomprimir archivos **ZipDeflate**/**TarGzip**/**Tar** y escribir en el almacén de datos receptor basado en archivos, los archivos se extraen de manera predeterminada en la carpeta: `<path specified in dataset>/<folder named as source compressed file>/`. Use `preserveZipFileNameAsFolder`/`preserveCompressionFileNameAsFolder` en el [origen de la actividad de copia](#binary-as-source) para controlar si se debe conservar el nombre de los archivos comprimidos como una estructura de carpetas.| No       |
| level | La razón de compresión. Se aplica cuando se utiliza el conjunto de datos en el receptor de la actividad de copia.<br>Los valores permitidos son **Optimal** o **Fastest**.<br>- **Fastest:** la operación de compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante no se comprime de forma óptima.<br>- **Optimal**: la operación de compresión se debe comprimir óptimamente, incluso si tarda más tiempo en completarse. Para más información, consulte el tema [Nivel de compresión](/dotnet/api/system.io.compression.compressionlevel) . | No       |

A continuación se muestra un ejemplo de un conjunto de datos binarios en Azure Blob Storage:

```json
{
    "name": "BinaryDataset",
    "properties": {
        "type": "Binary",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "compression": {
                "type": "ZipDeflate"
            }
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen binarios.

>[!NOTE]
>Al utilizar un conjunto de archivos binario en la actividad de copia, solo puede copiar de un conjunto de datos binario a un conjunto de datos binario.

### <a name="binary-as-source"></a>Binario como origen

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **BinarySource**. | Sí      |
| formatSettings | Un grupo de propiedades. Eche un vistazo a la tabla **Configuración de lectura de JSON** de abajo. | No       |
| storeSettings | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

**Configuración de lectura binaria** compatible en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | El tipo de formatSettings debe establecerse en **BinaryReadSettings**. | Sí      |
| compressionProperties | Un grupo de propiedades sobre cómo descomprimir datos para un códec de compresión determinado. | No       |
| preserveZipFileNameAsFolder<br>(*en `compressionProperties`->`type` como `ZipDeflateReadSettings`* ) | Se aplica cuando el conjunto de datos de entrada se configura con compresión **ZipDeflate**. Indica si se debe conservar el nombre del archivo ZIP de origen como estructura de carpetas durante la copia.<br>- Cuando se establece en **true (valor predeterminado)** , Data Factory escribe archivos descomprimidos en `<path specified in dataset>/<folder named as source zip file>/`.<br>- Cuando se establece en **false**, Data Factory escribe los archivos descomprimidos directamente en `<path specified in dataset>`. Asegúrese de que no tenga nombres de archivo duplicados en distintos archivos ZIP de origen para evitar comportamientos acelerados o inesperados.  | No |
| preserveCompressionFileNameAsFolder<br>(*en `compressionProperties`->`type` como `TarGZipReadSettings` o `TarReadSettings`* ) | Se aplica cuando el conjunto de datos de entrada está configurado con la compresión **TarGzip**/**Tar**. Indica si se debe conservar el nombre del archivo de origen comprimido como estructura de carpetas durante la copia.<br>- Cuando se establece en **true (valor predeterminado)** , Data Factory escribe los archivos descomprimidos en `<path specified in dataset>/<folder named as source compressed file>/`. <br>- Cuando se establece en **false**, Data Factory escribe los archivos descomprimidos directamente en `<path specified in dataset>`. Asegúrese de que no haya nombres de archivo duplicados en distintos archivos de origen para evitar comportamientos acelerados o inesperados. | No |

```json
"activities": [
    {
        "name": "CopyFromBinary",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "BinarySource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true,
                    "deleteFilesAfterCompletion": true
                },
                "formatSettings": {
                    "type": "BinaryReadSettings",
                    "compressionProperties": {
                        "type": "ZipDeflateReadSettings",
                        "preserveZipFileNameAsFolder": false
                    }
                }
            },
            ...
        }
        ...
    }
]
```

### <a name="binary-as-sink"></a>Binario como receptor

En la sección ***\*sink\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en: **BinarySink**. | Sí      |
| storeSettings | Un grupo de propiedades sobre cómo escribir datos en un almacén de datos. Cada conector basado en archivos tiene su propia configuración de escritura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
- [Actividad de eliminación](delete-activity.md)