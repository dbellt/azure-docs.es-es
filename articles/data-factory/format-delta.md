---
title: Formato delta en Azure Data Factory
description: Transformación y movimiento de datos desde un lago delta con el formato delta
author: dcstwh
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/26/2020
ms.author: weetok
ms.openlocfilehash: 6d9d2b0d185750cf8ed8192661f28a2b82d88b78
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222545"
---
# <a name="delta-format-in-azure-data-factory"></a>Formato delta en Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se señala cómo copiar datos con un lago delta, como origen o destino, almacenado en [Azure Data Lake Store Gen2](connector-azure-data-lake-storage.md) o [Azure Blob Storage](connector-azure-blob-storage.md) con el formato delta. Este conector está disponible como [conjunto de datos en línea](data-flow-source.md#inline-datasets) en los flujos de datos de asignación como origen y receptor.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4ALTs]

## <a name="mapping-data-flow-properties"></a>Propiedades de Asignación de instancias de Data Flow

Este conector está disponible como [conjunto de datos en línea](data-flow-source.md#inline-datasets) en los flujos de datos de asignación como origen y receptor.

### <a name="source-properties"></a>Propiedades de origen

En la tabla siguiente se enumeran las propiedades que admite un origen delta. Puede editar estas propiedades en la pestaña **Source options** (Opciones del origen).

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `delta`. | sí | `delta` | format |
| Sistema de archivos | Sistema de archivos o contenedor del lago delta | sí | String | fileSystem |
| Ruta de acceso a la carpeta | Directo del lago delta | sí | String | folderPath |
| Tipo de compresión | Tipo de compresión de la tabla delta | no | `bzip2`<br>`gzip`<br>`deflate`<br>`ZipDeflate`<br>`snappy`<br>`lz4` | compressionType |
| Nivel de compresión | Elija si la compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante debe comprimirse de forma óptima. | obligatorio si se especifica `compressedType` | `Optimal` o `Fastest` | compressionLevel |
| Viaje en el tiempo | Elija si se debe consultar una instantánea anterior de una tabla delta | no | Consulta por marca de tiempo: Timestamp <br> Consulta por versión: Entero | timestampAsOf <br> versionAsOf |
| No permitir que se encuentren archivos | Si es true, no se devuelve un error si no se encuentra ningún archivo. | no | `true` o `false` | ignoreNoFilesFound |

#### <a name="import-schema"></a>Importar esquema

Delta solo está disponible como conjunto de datos insertado y, de forma predeterminada, no tiene un esquema asociado. Para obtener los metadatos de columna, haga clic en el botón **Importar esquema** en la pestaña **Proyección**. Esto le permitirá hacer referencia a los nombres de columna y los tipos de datos especificados por corpus. Para importar el esquema, una [sesión de depuración de flujo de datos](concepts-data-flow-debug-mode.md) debe estar activa y debe tener un archivo de definición de entidad CDM existente al que apuntar.
 

### <a name="delta-source-script-example"></a>Ejemplo de script de origen delta

```
source(output(movieId as integer,
            title as string,
            releaseDate as date,
            rated as boolean,
            screenedOn as timestamp,
            ticketPrice as decimal(10,2)
            ),
    store: 'local',
    format: 'delta',
    versionAsOf: 0,
    allowSchemaDrift: false,
    folderPath: $tempPath + '/delta'
  ) ~> movies
```

### <a name="sink-properties"></a>Propiedades del receptor

En la tabla siguiente se enumeran las propiedades que admite un receptor delta. Puede editar estas propiedades en la pestaña **Configuración**.

| Nombre | Descripción | Obligatorio | Valores permitidos | Propiedad de script de flujo de datos |
| ---- | ----------- | -------- | -------------- | ---------------- |
| Formato | El formato debe ser `delta`. | sí | `delta` | format |
| Sistema de archivos | Sistema de archivos o contenedor del lago delta | sí | String | fileSystem |
| Ruta de acceso a la carpeta | Directo del lago delta | sí | String | folderPath |
| Tipo de compresión | Tipo de compresión de la tabla delta | no | `bzip2`<br>`gzip`<br>`deflate`<br>`ZipDeflate`<br>`snappy`<br>`lz4` | compressionType |
| Nivel de compresión | Elija si la compresión debe completarse tan pronto como sea posible, incluso si el archivo resultante debe comprimirse de forma óptima. | obligatorio si se especifica `compressedType` | `Optimal` o `Fastest` | compressionLevel |
| Vacío | Especifique el umbral de retención en horas para versiones anteriores de la tabla. Un valor de 0 o menos toma el valor predeterminado de 30 días | sí | Entero | vacuum |
| Método de actualización | Especifique qué operaciones de actualización se permiten en el lago delta. En el caso de los métodos que no sean insert, se requiere una transformación Alter row anterior para marcar las filas. | sí | `true` o `false` | deletable <br> insertable <br> updateable <br> merge |
| Escritura optimizada | Logre un mayor rendimiento de la operación de escritura mediante la optimización del orden aleatorio interno en los ejecutores de Spark. Como resultado, es posible que observe menos particiones y archivos de un tamaño mayor. | no | `true` o `false` | optimizedWrite: true |
| Compactación automática | Una vez completada la operación de escritura, Spark ejecutará automáticamente el comando ```OPTIMIZE``` para volver a organizar los datos, lo que dará lugar a más particiones si es necesario, para mejorar el rendimiento de la lectura en el futuro. | no | `true` o `false` |    autoCompact: true |

### <a name="delta-sink-script-example"></a>Ejemplo de script de receptor delta

El script de flujo de datos asociado es:

```
moviesAltered sink(
          input(movieId as integer,
                title as string
            ),
           mapColumn(
                movieId,
                title
            ),
           insertable: true,
           updateable: true,
           deletable: true,
           upsertable: false,
           keys: ['movieId'],
            store: 'local',
           format: 'delta',
           vacuum: 180,
           folderPath: $tempPath + '/delta'
           ) ~> movieDB
```

### <a name="known-limitations"></a>Restricciones conocidas

Al escribir en un receptor delta, existe una limitación conocida en la que el número de filas escritas no se devolverá en el resultado de la supervisión.

## <a name="next-steps"></a>Pasos siguientes

* Cree una [transformación de origen](data-flow-source.md) en flujo de datos de asignación.
* Cree una [transformación de receptor](data-flow-sink.md) en el flujo de datos de asignación.
* Cree una [transformación Alter row](data-flow-alter-row.md) para marcar filas como insert, update, upsert o delete.
