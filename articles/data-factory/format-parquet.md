---
title: Formato Parquet en Azure Data Factory
description: En este tema se describe cómo tratar con el formato Parquet en Azure Data Factory.
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 09/27/2020
ms.author: jianleishen
ms.openlocfilehash: 279df09cddf2bb8b2bcb5a936666e302987cac27
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109486618"
---
# <a name="parquet-format-in-azure-data-factory"></a>Formato Parquet en Azure Data Factory
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Siga este artículo cuando quiera **analizar los archivos Parquet o escribir los datos en formato Parquet**. 

El formato Parquet se admite para los conectores siguientes: [Amazon S3](connector-amazon-simple-storage-service.md), [Azure Blob](connector-azure-blob-storage.md), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md), [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md), [Azure File Storage](connector-azure-file-storage.md), [File System](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [HTTP](connector-http.md) y [SFTP](connector-sftp.md).

## <a name="dataset-properties"></a>Propiedades del conjunto de datos

Si desea ver una lista completa de las secciones y propiedades disponibles para definir conjuntos de datos, consulte el artículo sobre [conjuntos de datos](concepts-datasets-linked-services.md). En esta sección se proporciona una lista de las propiedades que admite el conjunto de datos de Parquet.

| Propiedad         | Descripción                                                  | Obligatorio |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | La propiedad type del conjunto de datos debe establecerse en **Parquet**. | Sí      |
| ubicación         | Configuración de ubicación de los archivos. Cada conector basado en archivos tiene su propio tipo de ubicación y propiedades compatibles en `location`. **Vea los detalles en el artículo de conectores -> sección de propiedades del conjunto de datos**. | Sí      |
| compressionCodec | El códec de compresión que se usará al escribir en archivos Parquet. Al realizar la lectura desde archivos Parquet, las instancias de Data Factory determinan automáticamente el códec de compresión basado en los metadatos del archivo.<br>Los tipos admitidos son "**none**","**gzip**","**snappy**" (predeterminado) e "**lzo**". Tenga en cuenta que la actividad de copia no es compatible actualmente con LZO cuando hay archivos Parquet de lectura y escritura. | No       |

> [!NOTE]
> No se admiten espacios en blanco en el nombre de columna de los archivos Parquet.

A continuación encontrará un ejemplo de un conjunto de datos de Parquet en Azure Blob Storage:

```json
{
    "name": "ParquetDataset",
    "properties": {
        "type": "Parquet",
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
            "compressionCodec": "snappy"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propiedades de la actividad de copia

Si desea ver una lista completa de las secciones y propiedades disponibles para definir actividades, consulte el artículo sobre [canalizaciones](concepts-pipelines-activities.md). En esta sección se proporciona una lista de las propiedades que admiten el receptor y el origen Parquet.

### <a name="parquet-as-source"></a>Parquet como origen

En la sección ***\*source\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del origen de la actividad de copia debe establecerse en **ParquetSource**. | Sí      |
| storeSettings | Un grupo de propiedades sobre cómo leer datos de un almacén de datos. Cada conector basado en archivos tiene su propia configuración de lectura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

### <a name="parquet-as-sink"></a>Parquet como receptor

En la sección ***\*sink\**** de la actividad de copia se admiten las siguientes propiedades.

| Propiedad      | Descripción                                                  | Obligatorio |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | La propiedad type del receptor de la actividad de copia debe establecerse en **ParquetSink**. | Sí      |
| formatSettings | Un grupo de propiedades. Consulte la tabla **Configuración de escritura de Parquet** a continuación. |    No      |
| storeSettings | Un grupo de propiedades sobre cómo escribir datos en un almacén de datos. Cada conector basado en archivos tiene su propia configuración de escritura admitida en `storeSettings`. **Vea los detalles en el artículo de conectores -> sección de propiedades de la actividad de copia**. | No       |

**Configuración de escritura de Parquet** compatible en `formatSettings`:

| Propiedad      | Descripción                                                  | Obligatorio                                              |
| ------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| type          | La propiedad type de formatSettings debe establecerse en **ParquetWriteSettings**. | Sí                                                   |
| maxRowsPerFile | Al escribir datos en una carpeta, puede optar por escribir en varios archivos y especificar el número máximo de filas por archivo.  | No |
| fileNamePrefix | Se aplica cuando se configura `maxRowsPerFile`.<br> Especifique el prefijo de nombre de archivo al escribir datos en varios archivos, lo que da como resultado este patrón: `<fileNamePrefix>_00000.<fileExtension>`. Si no se especifica, el prefijo de nombre de archivo se generará automáticamente. Esta propiedad no se aplica cuando el origen es un almacén basado en archivos o un [almacén de datos habilitado para la opción de partición](copy-activity-performance-features.md).  | No |

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

En los flujos de datos de asignación, puede leer y escribir en formato Parquet en los siguientes almacenes de datos: [Azure Blob Storage](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage Gen1](connector-azure-data-lake-store.md#mapping-data-flow-properties) y [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#mapping-data-flow-properties).

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que un origen Parquet admite. Puede editar estas propiedades en la pestaña **Source options** (Opciones del origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `parquet`. | sí | `parquet` | format |
| Rutas de acceso comodín | Se procesarán todos los archivos que coincidan con la ruta de acceso comodín. Reemplaza a la carpeta y la ruta de acceso del archivo establecidas en el conjunto de datos. | no | String[] | wildcardPaths |
| Ruta de acceso raíz de la partición | En el caso de datos de archivos con particiones, puede especificar una ruta de acceso raíz de la partición para leer las carpetas con particiones como columnas. | no | String | partitionRootPath |
| Lista de archivos | Si el origen apunta a un archivo de texto que enumera los archivos que se van a procesar. | no | `true` o `false` | fileList |
| Columna para almacenar el nombre de archivo | Se crea una nueva columna con el nombre y la ruta de acceso del archivo de origen. | no | String | rowUrlColumn |
| Después de finalizar | Se eliminan o mueven los archivos después del procesamiento. La ruta de acceso del archivo comienza en la raíz del contenedor. | no | Borrar: `true` o `false` <br> Mover: `[<from>, <to>]` | purgeFiles <br> moveFiles |
| Filtrar por última modificación | Elija si desea filtrar los archivos en función de cuándo se modificaron por última vez. | no | Timestamp | modifiedAfter <br> modifiedBefore |
| No permitir que se encuentren archivos | Si es true, no se devuelve un error si no se encuentra ningún archivo. | no | `true` o `false` | ignoreNoFilesFound |

### <a name="source-example"></a>Ejemplo de origen

En la imagen siguiente se presenta un ejemplo de una configuración de origen Parquet en flujos de datos de asignación.

![Origen Parquet](media/data-flow/parquet-source.png)

El script de flujo de datos asociado es:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    rowUrlColumn: 'fileName',
    format: 'parquet') ~> ParquetSource
```

### <a name="sink-properties"></a>Propiedades del receptor

En la tabla siguiente se enumeran las propiedades que un receptor Parquet admite. Puede editar estas propiedades en la pestaña **Configuración**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `parquet`. | sí | `parquet` | format |
| Borrar la carpeta | Si la carpeta de destino se borra antes de escribir. | no | `true` o `false` | truncate |
| Opción de nombre de archivo | El formato de nombre de los datos escritos. De forma predeterminada, un archivo por partición en formato `part-#####-tid-<guid>`. | no | Patrón: String <br> Por partición: String[] <br> Como datos de columna: String <br> Salida en un solo archivo: `['<fileName>']` | filePattern <br> partitionFileNames <br> rowUrlColumn <br> partitionFileNames |

### <a name="sink-example"></a>Ejemplo de receptor

En la imagen siguiente se presenta un ejemplo de una configuración de receptor Parquet en flujos de datos de asignación.

![Receptor Parquet](media/data-flow/parquet-sink.png)

El script de flujo de datos asociado es:

```
ParquetSource sink(
    format: 'parquet',
    filePattern:'output[n].parquet',
    truncate: true,
    allowSchemaDrift: true,
    validateSchema: false,
    skipDuplicateMapInputs: true,
    skipDuplicateMapOutputs: true) ~> ParquetSink
```

## <a name="data-type-support"></a>Compatibilidad con tipos de datos

Actualmente, los tipos de datos complejos de Parquet (por ejemplo, MAP, LIST, STRUCT) solo se admiten en los flujos de datos, no en la actividad de copia. Para usar tipos complejos en flujos de datos, no importe el esquema de archivo en el conjunto de datos y deje el esquema en blanco en el conjunto de datos. A continuación, en la transformación de origen, importe la proyección.

## <a name="using-self-hosted-integration-runtime"></a>Uso del entorno de ejecución de integración autohospedado

> [!IMPORTANT]
> En el caso de las copias autorizadas por el entorno de ejecución de integración (IR) autohospedado (por ejemplo, entre almacenes de datos locales y almacenes de datos en la nube), si no va a copiar archivos ORC **tal y como están**, tendrá que instalar **JRE (Java Runtime Environment) 8 de 64 bits u OpenJDK** y el **paquete Microsoft Visual C++ 2010 Redistributable** en la máquina de IR. Consulte el párrafo siguiente para más información.

En el caso de las copias que se ejecutan en el IR autohospedado con la serialización o deserialización de archivos Parquet, para encontrar el entorno de ejecución de Java, ADF consulta primero el Registro *`(SOFTWARE\JavaSoft\Java Runtime Environment\{Current Version}\JavaHome)`* de JRE; si no lo encuentra, comprueba la variable del sistema *`JAVA_HOME`* de OpenJDK.

- **Para usar JRE**: el IR de 64 bits necesita JRE de 64 bits. Puede encontrarlo [aquí](https://go.microsoft.com/fwlink/?LinkId=808605).
- **Para usar OpenJDK**: se admite desde la versión 3.13 de IR. Empaquete jvm.dll con todos los demás ensamblados de OpenJDK necesarios en la máquina del IR autohospedado y establezca la variable de entorno del sistema JAVA_HOME en el valor que corresponda.
- **Para instalar el paquete Visual C++ 2010 Redistributable**: el paquete Visual C++ 2010 Redistributable no se instala con las instalaciones de IR autohospedadas. Puede encontrarlo [aquí](https://www.microsoft.com/download/details.aspx?id=26999).

> [!TIP]
> Si copia datos desde o hacia Parquet mediante Integration Runtime autohospedado y recibe un error que indica que "Se produjo un error al invocar Java, mensaje: **Espacio en el montón java.lang.OutOfMemoryError:Java**", puede agregar una variable de entorno `_JAVA_OPTIONS` en la máquina que hospeda IR autohospedado para ajustar el tamaño del montón mínimo y máximo para JVM a fin de facilitar dicha copia y, a continuación, volver a ejecutar la canalización.

![Establecimiento del tamaño del montón JVM en IR autohospedado](./media/supported-file-formats-and-compression-codecs/set-jvm-heap-size-on-selfhosted-ir.png)

Ejemplo: establecimiento de la variable `_JAVA_OPTIONS` con el valor `-Xms256m -Xmx16g`. La marca `Xms` especifica el grupo de asignación de memoria inicial para una máquina virtual Java (JVM), mientras que `Xmx` especifica el grupo de asignación de memoria máxima. Esto significa que JVM se iniciará con la cantidad de memoria `Xms` y podrá utilizar `Xmx` como máximo. De forma predeterminada, ADF usa 64 MB como mínimo y 1 GB como máximo.

## <a name="next-steps"></a>Pasos siguientes

- [Información general de la actividad de copia](copy-activity-overview.md)
- [Asignación de Data Flow](concepts-data-flow-overview.md)
- [Actividad de búsqueda](control-flow-lookup-activity.md)
- [Actividad GetMetadata](control-flow-get-metadata-activity.md)
