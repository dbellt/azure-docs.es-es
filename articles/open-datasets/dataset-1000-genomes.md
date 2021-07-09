---
title: 1000 Genomes
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos 1000 Genomes en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 4418d7866f2476d528290f99d396dcf52c47c3ec
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039176"
---
# <a name="1000-genomes"></a>1000 Genomes

El proyecto 1000 Genomes se llevó a cabo entre 2008 y 2015, y supuso la creación del mayor catálogo público de datos de variantes y genotipos humanos. El conjunto de datos definitivo contiene datos de 2504 personas de 26 poblaciones y 84 millones de variantes identificadas. Si desea obtener más información, consulte el sitio web del proyecto 1000 Genomes y las siguientes publicaciones:

Análisis piloto: A map of human genome variation from population-scale sequencing Nature 467, 1061-1073 (28 de octubre de 2010)

Análisis de fase 1: An integrated map of genetic variation from 1,092 human genomes Nature 491, 56-65 (1 de noviembre de 2012)

Análisis de fase 3: A global reference for human genetic variation Nature 526, 68-74 (1 de octubre de 2015) y An integrated map of structural variation in 2,504 human genomes, Nature 526, 75-81 (1 de octubre de 2015)

Para obtener información sobre los formatos de los datos, consulte http://www.internationalgenome.org/formats.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

Este conjunto de datos es un reflejo de ftp://ftp.1000genomes.ebi.ac.uk/vol1/ftp/

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

Este conjunto de datos contiene unos 815 TB de datos y se actualiza a diario.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en las regiones Oeste de EE. UU. 2 y Centro-oeste de EE. UU. de Azure. Por afinidad, se recomienda asignar recursos de proceso de estas dos regiones.

## <a name="data-access"></a>Acceso a datos

Oeste de EE. UU. 2: https://dataset1000genomes.blob.core.windows.net/dataset

Centro-oeste de EE. UU.: https://dataset1000genomes-secondary.blob.core.windows.net/dataset

[Token de SAS](../storage/common/storage-sas-overview.md): sv=2019-10-10&si=prod&sr=c&sig=9nzcxaQn0NprMPlSh4RhFQHcXedLQIcFgbERiooHEqM%3D

## <a name="use-terms"></a>Términos de uso

Después de las publicaciones definitivas, los datos del proyecto 1000 Genomes están disponibles públicamente sin restricciones para que cualquier persona pueda usarlos conforme a los términos que proporciona el origen del conjunto de datos ([http://www.internationalgenome.org/data](http://www.internationalgenome.org/data)). El uso de los datos debe citarse como se explica en las [preguntas más frecuentes]() del proyecto 1000 Genomes.

## <a name="contact"></a>Contacto

https://www.internationalgenome.org/contact

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).