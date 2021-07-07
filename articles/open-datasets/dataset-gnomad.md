---
title: Genome Aggregation Database (gnomAD)
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de Genome Aggregation Database (gnomAD) en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 52fc504135e77ef9e32cd1b83f1186d7ddb18fcd
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039080"
---
# <a name="genome-aggregation-database-gnomad"></a>Genome Aggregation Database (gnomAD)

[Genome Aggregation Database (gnomAD)](https://gnomad.broadinstitute.org/) es un recurso desarrollado por una coalición internacional de investigadores con el objetivo de agregar y armonizar los datos de la secuenciación del exoma y del genoma procedentes de una gran variedad de proyectos de secuenciación a gran escala, y de poner los datos resumidos a disposición de la amplia comunidad científica.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

Este conjunto de datos se hospeda en colaboración con el Broad Institute y el catálogo de datos completo de gnomAD se puede ver en https://gnomad.broadinstitute.org/downloads.

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

Este conjunto contiene unos 30 TB de datos y se actualiza con cada versión de gnomAD.

## <a name="storage-location"></a>Ubicación de almacenamiento

La cuenta de almacenamiento que hospeda este conjunto de datos se encuentra en la región Este de EE. UU. de Azure. Se recomienda asignar recursos de proceso de la misma región por afinidad.

## <a name="data-access"></a>Acceso a datos

Cuenta de almacenamiento: https://azureopendatastorage.blob.core.windows.net/gnomad

Los datos están disponibles públicamente sin restricciones y se recomienda la herramienta azcopy para realizar operaciones masivas. Por ejemplo, para ver los archivos VCF de la versión 3.0 de gnomAD:

```powershell
$ azcopy ls https://azureopendatastorage.blob.core.windows.net/gnomad/release/3.0/vcf/genomes
```

Para descargar todos los archivos VCF de forma recursiva:

```powershell
$ azcopy cp --recursive=true https://azureopendatastorage.blob.core.windows.net/gnomad/release/3.0/vcf/genomes .
```

El [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) también es una herramienta útil para examinar la lista de archivos de la versión de gnomAD.

## <a name="use-terms"></a>Términos de uso

Los datos están disponibles sin restricciones. Para obtener más información y detalles de cita, consulte la [página de información de gnomAD](https://gnomad.broadinstitute.org/about).

## <a name="contact"></a>Contacto

Si tiene alguna pregunta o comentario sobre este conjunto de datos, póngase en contacto con el [equipo de gnomAD](https://gnomad.broadinstitute.org/contact).

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).