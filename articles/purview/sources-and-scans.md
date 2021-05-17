---
title: Tipos de archivo y orígenes de datos admitidos
description: En este artículo se proporcionan detalles conceptuales sobre los orígenes de datos y los tipos de archivo admitidos en Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/24/2020
ms.custom: references_regions
ms.openlocfilehash: 15ade6fca3885bfabba7a23e2c3d8e561a9e6a0c
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109738471"
---
# <a name="supported-data-sources-and-file-types-in-azure-purview"></a>Tipos de archivo y orígenes de datos admitidos en Azure Purview

En este artículo se describen los orígenes de datos, los tipos de archivo y los conceptos de examen admitidos en Purview.

## <a name="supported-data-sources"></a>Orígenes de datos admitidos

Purview admite todos los orígenes de datos enumerados [aquí](purview-connector-overview.md).

## <a name="file-types-supported-for-scanning"></a>Tipos de archivo admitidos para examen

Los siguientes tipos de archivo se admiten para los exámenes y para la extracción y clasificación de esquemas, si procede:

- Formatos de archivo estructurados admitidos por extensión: AVRO, ORC, PARQUET, CSV, JSON, PSV, SSV, TSV, TXT, XML, GZIP
- Formatos de archivo de documento admitidos por extensión: DOC, DOCM, DOCX, DOT, ODP, ODS, ODT, PDF, POT, PPS, PPSX, PPT, PPTM, PPTX, XLC, XLS, XLSB, XLSM, XLSX, XLT
- Purview también admite extensiones de archivo personalizadas y analizadores personalizados.
 
> [!Note]
> Cada archivo Gzip debe estar asignado a un solo archivo CSV. Los archivos Gzip están sujetos a reglas de clasificación personalizadas y del sistema. Actualmente no se admite el examen de un archivo Gzip asignado a varios archivos o cualquier otro tipo de archivo que no sea CSV. Además, el analizador de Purview admite el examen de tipos de archivo PARQUET y AVRO comprimidos con Snappy para la extracción y clasificación de esquemas.

> [!Note]
> El analizador de Purview no admite tipos de datos complejos en los tipos de archivo AVRO, ORC y PARQUET para la extracción de esquemas.   

## <a name="sampling-within-a-file"></a>Muestreo en un archivo

En la terminología de Purview:
- Examen L1: extrae información básica y metadatos como el nombre de archivo, el tamaño y el nombre completo
- Examen L2: extrae el esquema de los tipos de archivo estructurados y las tablas de base de datos
- Examen L3: extrae el esquema cuando proceda y somete el archivo muestreado a reglas de clasificación personalizadas y del sistema.

En todos los formatos de archivo estructurados, el examen de Purview muestrea los archivos de la siguiente manera:

- En el caso de los tipos de archivo estructurados, muestrea 128 filas de cada columna o 1 MB, lo que sea menor.
- En el caso de los formatos de archivo de documento, muestrea 20 MB de cada archivo.
    - Si un archivo de documento es mayor de 20 MB, no estará sujeto a un examen profundo (sujeto a clasificación). En tal caso, Purview captura solo metadatos básicos como el nombre de archivo y el nombre completo.

## <a name="resource-set-file-sampling"></a>Muestreo de archivos del conjunto de recursos

En Purview, una carpeta o grupo de archivos de partición se detecta como un *conjunto de recursos* si coincide con una directiva de conjuntos de recursos del sistema o con una directiva de conjuntos de recursos definida por el cliente. Si se detecta un conjunto de recursos, Purview muestreará cada carpeta que contenga. Consulte [aquí](concept-resource-sets.md) más información sobre los conjuntos de recursos.

Muestreo de archivos para conjuntos de recursos por tipos de archivo:

- **Archivos delimitados (CSV, PSV, SSV, TSV)** : se muestrea uno de cada 100 archivos (examen L3) de una carpeta o grupo de archivos de partición que se considere un grupo de recursos.
- **Tipos de archivo de Data Lake (Parquet, Avro, Orc)** : se muestrea uno de cada 18446744073709551615 archivos (longitud máxima) (examen L3) de una carpeta o grupo de archivos de partición que se considere *grupo de recursos*.
- **Otros tipos de archivo estructurados (JSON, XML, TXT)** : se muestrea uno de cada 100 archivos (examen L3) de una carpeta o grupo de archivos de partición que se considere un grupo de recursos.
- **Objetos SQL y entidades de CosmosDB**: cada archivo se somete a un examen L3.
- **Tipos de archivo de documento**: cada archivo se somete a un examen L3. Los patrones de conjuntos de recursos no se aplican a estos tipos de archivo.

## <a name="classification"></a>clasificación

Las 105 reglas de clasificación del sistema se aplican a los formatos de archivo estructurados. Solo las reglas de clasificación de MCE se aplican a los tipos de archivo de documento (no los patrones de expresiones regulares nativos del examen de datos, detección basada en filtros de Bloom). Para más información sobre las clasificaciones admitidas, consulte [Clasificaciones admitidas en Azure Purview](supported-classifications.md).

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: ejecución de Starter Kit y examen de datos](tutorial-scan-data.md)
- [Administración de orígenes de datos en Azure Purview (versión preliminar)](manage-data-sources.md)