---
title: Formato Avro en Azure Data Factory
description: En este tema se describe cómo tratar con el formato Avro en Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/15/2020
ms.author: jianleishen
ms.openlocfilehash: 0efd4ed26f5057df2cc045940553b29c362513ff
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110090233"
---
# <a name="avro-format-in-azure-data-factory"></a>Formato Avro en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Siga este artículo cuando quiera **analizar los archivos Avro o escribir los datos en formato Avro**. 

El formato Avro se admite en los siguientes conectores: [Amazon S3](connector-amazon-simple-storage-service.md), [Amazon S3 Compatible Storage](connector-amazon-s3-compatible-storage.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md), [Oracle Cloud Storage](connector-oracle-cloud-storage.md) y [SFTP](connector-sftp.md).

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Avro.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **Avro**. | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`. **Vea los detalles en el artículo de conectores -> sección de propiedades del conjunto de datos**. | Sí      |
| avroCompressionCodec | El códec de compresión que se usará al escribir en archivos Avro. Al leer desde archivos Avro, Data Factory determina automáticamente el códec de compresión según los metadatos del archivo.<br>Los tipos admitidos son "**none**" (valor predeterminado), "**deflate**", "**snappy**". Tenga en cuenta que la actividad de copia no es compatible actualmente con Snappy cuando hay archivos Avro de lectura y escritura. | No       |

> [!NOTE]
> No se admiten espacios en blanco en el nombre de columna de los archivos Avro.

A continuación se muestra un ejemplo de un conjunto de datos de Avro en Azure Blob Storage:

```json
{
    "name": "AvroDataset",
    "properties": {
        "type": "Avro",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "avroCompressionCodec": "snappy"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen de Avro.

### <a name="avro-as-source"></a>Avro como origen

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **AvroSource**. | Sí      |
| storeSettings | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

### <a name="avro-as-sink"></a>Avro como receptor

En la sección ***\*sink\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **AvroSink**. | Sí      |
| formatSettings          | Un grupo de propiedades. Consulte la tabla **Configuración de escritura de Avro** a continuación.| No      |
| storeSettings | Un grupo de propiedades sobre cómo escribir datos en un almacén de datos. Cada conector basado en archivos tiene su propia configuración de escritura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

**Configuración de escritura de Avro** compatible en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | La propiedad type de formatSettings debe establecerse en **AvroWriteSettings**. | Sí                                                   |
| maxRowsPerFile | Al escribir datos en una carpeta, puede optar por escribir en varios archivos y especificar el número máximo de filas por archivo.  | No |
| fileNamePrefix | Se aplica cuando se configura `maxRowsPerFile`.<br> Especifique el prefijo de nombre de archivo al escribir datos en varios archivos, lo que da como resultado este patrón: `<fileNamePrefix>_00000.<fileExtension>`. Si no se especifica, el prefijo de nombre de archivo se generará automáticamente. Esta propiedad no se aplica cuando el origen es un almacén basado en archivos o un [almacén de datos habilitado para la opción de partición](copy-activity-performance-features.md).  | No |

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

En los flujos de datos de asignación, puede leer y escribir en formato Avro en los siguientes almacenes de datos: [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties).

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que admite un origen Avro. Puede editar estas propiedades en la pestaña **Source options** (Opciones de origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Rutas de acceso comodín | Se procesarán todos los archivos que coincidan con la ruta de acceso comodín. Reemplaza a la carpeta y la ruta de acceso del archivo establecidas en el conjunto de datos. | no | String[] | wildcardPaths |
| Ruta de acceso raíz de la partición | En el caso de datos de archivos con particiones, puede especificar una ruta de acceso raíz de la partición para leer las carpetas con particiones como columnas. | no | String | partitionRootPath |
| Lista de archivos | Si el origen apunta a un archivo de texto que enumera los archivos que se van a procesar. | no | `true` o `false` | fileList |
| Columna para almacenar el nombre de archivo | Se crea una nueva columna con el nombre y la ruta de acceso del archivo de origen. | no | String | rowUrlColumn |
| Después de finalizar | Se eliminan o mueven los archivos después del procesamiento. La ruta de acceso del archivo comienza en la raíz del contenedor. | no | Borrar: `true` o `false` <br> Mover: `['<from>', '<to>']` | purgeFiles <br> moveFiles |
| Filtrar por última modificación | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | no | Timestamp | modifiedAfter <br> modifiedBefore |
| No permitir que se encuentren archivos | Si es true, no se devuelve un error si no se encuentra ningún archivo. | no | `true` o `false` | ignoreNoFilesFound |

### <a name="sink-properties"></a>Propiedades del receptor

En la tabla siguiente se enumeran las propiedades que admite un receptor Avro. Puede editar estas propiedades en la pestaña **Configuración**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Borrar la carpeta | Si la carpeta de destino se borra antes de escribir. | no | `true` o `false` | truncate |
| Opción de nombre de archivo | El formato de nombre de los datos escritos. De forma predeterminada, un archivo por partición en formato `part-#####-tid-<guid>`. | no | Patrón: String <br> Por partición: String[] <br> Como datos de columna: String <br> Salida en un solo archivo: `['<fileName>']`  | filePattern <br> partitionFileNames <br> rowUrlColumn <br> partitionFileNames |
| Entrecomillar todo | Incluye todos los valores entre comillas. | no | `true` o `false` | quoteAll |

## <a name="data-type-support"></a>Compatibilidad con tipos de datos

### <a name="copy-activity"></a>Actividad de copia
No se admiten [tipos de datos complejos](https://avro.apache.org/docs/current/spec.html#schema_complex) de Avro (registros, enumeraciones, matrices, asignaciones, uniones y fijos) en actividad de copia.

### <a name="data-flows"></a>Flujos de datos
Al trabajar con archivos Avro en flujos de datos, puede leer y escribir tipos de datos complejos, pero asegúrese de borrar primero el esquema físico del conjunto de datos. En flujos de datos, puede establecer la proyección lógica y derivar columnas que sean estructuras complejas, y después asignar automáticamente esos campos a un archivo Avro.

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
