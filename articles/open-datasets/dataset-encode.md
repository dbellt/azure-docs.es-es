---
title: ENCODE
titleSuffix: Azure Open Datasets
description: Aprenda a usar el conjunto de datos de ENCODE en Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: 921f04b4f874fd83403a613823df0f2bd600feb8
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039195"
---
# <a name="encode-encyclopedia-of-dna-elements"></a>ENCODE: Encyclopedia of DNA Elements

El [consorcio Encyclopedia of DNA Elements (ENCODE)](https://www.encodeproject.org/help/project-overview/) es un proyecto en curso de colaboración internacional de grupos de investigación financiado por el Instituto Nacional de Investigación del Genoma Humano (NHGRI). El objetivo de ENCODE es crear una lista completa de los elementos funcionales del genoma humano, incluidos los que actúan a nivel de proteínas y del ARN, así como los elementos reguladores que controlan las células y las circunstancias en las que un gen está activo.

Los investigadores de ENCODE utilizan una gran variedad de análisis y métodos para identificar los elementos funcionales. La detección y anotación de los elementos de los genes se consigue, principalmente, mediante la secuenciación de una diversa gama de orígenes de ARN, la genómica comparativa, métodos bioinformáticos integradores y la labor organizativa humana. Los elementos reguladores se investigan normalmente mediante análisis de hipersensibilidad del ADN, la metilación del ADN y la inmunoprecipitación (IP) de las proteínas que interactúan con el ADN y el ARN, es decir, histonas modificadas, factores de transcripción, reguladores de la cromatina y proteínas de unión al ARN, todo ello seguido de la secuenciación.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

## <a name="data-source"></a>Origen de datos

Este conjunto de datos es un reflejo del almacén de datos de https://www.encodeproject.org/.

## <a name="data-volumes-and-update-frequency"></a>Volúmenes de datos y frecuencia de actualización

Este conjunto de datos incluye, aproximadamente, 756 TB de datos y se actualiza mensualmente durante la primera semana de cada mes.

## <a name="storage-location"></a>Ubicación de almacenamiento

Este conjunto de datos se almacena en las regiones Oeste de EE. UU. 2 y Centro-oeste de EE. UU. de Azure. Por afinidad, se recomienda asignar recursos de proceso de Oeste de EE. UU. 2 o Centro-oeste de EE. UU.

## <a name="data-access"></a>Acceso a datos

Oeste de EE. UU. 2: https://datasetencode.blob.core.windows.net/dataset

Centro-oeste de EE. UU.: https://datasetencode-secondary.blob.core.windows.net/dataset

[Token de SAS](/azure/storage/common/storage-sas-overview): ?sv=2019-10-10&si=prod&sr=c&sig=9qSQZo4ggrCNpybBExU8SypuUZV33igI11xw0P7rB3c%3D

## <a name="use-terms"></a>Términos de uso

Los usuarios de datos externos pueden descargar, analizar y publicar libremente los resultados basados en cualquier dato de ENCODE sin restricciones, independientemente del tipo o tamaño, y no se incluye ningún período de gracia para los productores de datos de ENCODE, ya sea como miembros individuales o como parte del consorcio. Se recomienda a los investigadores que usen datos de ENCODE no publicados que se pongan en contacto con los productores de los datos para estudiar posibles publicaciones. El consorcio seguirá publicando los resultados de sus propios análisis en publicaciones independientes.

ENCODE solicita que los investigadores que utilicen conjuntos de datos de este proyecto (publicados o no) en presentaciones y publicaciones citen el consorcio ENCODE de todas las maneras que se indican en [https://www.encodeproject.org/help/citing-encode/](https://www.encodeproject.org/help/citing-encode/).

## <a name="contact"></a>Contacto

Si tiene alguna pregunta, preocupación o comentario, envíe un correo electrónico a nuestro departamento de soporte técnico en encode-help@lists.stanford.edu.

## <a name="next-steps"></a>Pasos siguientes

Consulte el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).