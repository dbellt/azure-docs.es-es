---
title: GATK Resource Bundle
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos Lote de recursos de GATK en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: a895654027caa9b56a3a21d217ac9d105b1d6130
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039093"
---
# <a name="gatk-resource-bundle"></a>GATK Resource Bundle

El [lote de recursos de GATK](https://gatk.broadinstitute.org/hc/articles/360035890811-Resource-bundle) es una colección de archivos estándar para trabajar con datos de resecuenciación humana con el kit GATK.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

Este conjunto de datos es un reflejo del almacén de datos de https://gatk.broadinstitute.org/hc/articles/360035890811-Resource-bundle.

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

1. datasetgatkbestpractices: 542 GB
1. datasetgatklegacybundles: 61 GB
1. datasetgatktestdata: 2 TB
1. datasetpublicbroadref: 477 GB
1. datasetbroadpublic: 3 TB

Los conjuntos de datos se actualizan mensualmente durante la primera semana de cada mes.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en las regiones Oeste de EE. UU. 2 y Centro-oeste de EE. UU. de Azure. Por afinidad, se recomienda asignar recursos de proceso de estas dos regiones.

## <a name="data-access"></a>Acceso a datos

1. datasetgatkbestpractices

    Oeste de EE. UU. 2: https://datasetgatkbestpractices.blob.core.windows.net/dataset
    
    Centro-oeste de EE. UU.: https://datasetgatkbestpractices-secondary.blob.core.windows.net/dataset
    
    [Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2020-04-08&si=prod&sr=c&sig=6SaDfKtXAIfdpO%2BkvNA%2FsTNmNij%2Byh%2F%2F%2Bf98WAUqs7I%3D

2. datasetgatklegacybundles

    Oeste de EE. UU. 2: https://datasetgatklegacybundles.blob.core.windows.net/dataset
    
    Centro-oeste de EE. UU.: https://datasetgatklegacybundles-secondary.blob.core.windows.net/dataset
    
    [Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2020-04-08&si=prod&sr=c&sig=xBfxOPBqHKUCszzwbNCBYF0k9osTQjKnZbEjXCW7gU0%3D

3. datasetgatktestdata

    Oeste de EE. UU. 2: https://datasetgatktestdata.blob.core.windows.net/dataset
    
    Centro-oeste de EE. UU.: https://datasetgatktestdata-secondary.blob.core.windows.net/dataset
    
    [Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2020-04-08&si=prod&sr=c&sig=fzLts1Q2vKjuvR7g50vE4HteEHBxTcJbNvf%2FZCeDMO4%3D

4. datasetpublicbroadref
    
    Oeste de EE. UU. 2: https://datasetpublicbroadref.blob.core.windows.net/dataset
    
    Centro-oeste de EE. UU.: https://datasetpublicbroadref-secondary.blob.core.windows.net/dataset
    
    [Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2020-04-08&si=prod&sr=c&sig=DQxmjB4D1lAfOW9AxIWbXwZx6ksbwjlNkixw597JnvQ%3D

5. datasetbroadpublic

    Oeste de EE. UU. 2: https://datasetpublicbroadpublic.blob.core.windows.net/dataset
    
    Centro-oeste de EE. UU.: https://datasetpublicbroadpublic-secondary.blob.core.windows.net/dataset
    
    [Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2020-04-08&si=prod&sr=c&sig=u%2Bg2Ab7WKZEGiAkwlj6nKiEeZ5wdoJb10Az7uUwis%2Fg%3D

## <a name="use-terms"></a>Términos de uso

Visite el [sitio oficial del lote de recursos de GATK](https://gatk.broadinstitute.org/hc/articles/360035890811-Resource-bundle).

## <a name="contact"></a>Contacto

Visite el [sitio oficial del lote de recursos de GATK](https://gatk.broadinstitute.org/hc/articles/360035890811-Resource-bundle).

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).