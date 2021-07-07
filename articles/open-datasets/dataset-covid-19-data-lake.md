---
title: Lago de datos de COVID-19
titleSuffix: Azure Open Datasets
description: Aprenda a usar el lago de datos de COVID-19 de Azure Open Datasets.
ms.service: open-datasets
ms.topic: sample
author: peterclu
ms.author: peterlu
ms.date: 04/16/2021
ms.openlocfilehash: e43ea946511efaa313204ca26bf7e05f1b3f923a
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112039111"
---
# <a name="covid-19-data-lake"></a>Lago de datos de COVID-19

El lago de datos de COVID-19 contiene conjuntos de datos relacionados con la COVID-19 procedentes de varios orígenes. Abarca los datos de seguimiento de resultados de pruebas y pacientes, la directiva de distanciamiento social, la capacidad hospitalaria, la movilidad, etc.

[!INCLUDE [Open Dataset usage notice](../../includes/open-datasets-usage-note.md)]

El lago de datos COVID-19 está hospedado en Azure Data Lake Storage en la región Este de EE. UU. Para cada conjunto de datos, se pueden encontrar versiones modificadas en los formatos csv, json, json-lines y parquet. Los datos sin procesar también están disponibles como ingeridos.

Para simplificar la combinación, se agregan códigos de subdivisión conforme a la norma ISO 3166 donde no existen. Los nombres de columna se han vuelto a formatear en minúsculas con separadores de subrayado. Los conjuntos de datos se actualizan a diario, con copias históricas de archivos modificados y sin procesar también disponibles.

## <a name="datasets"></a>Conjuntos de datos

| Conjuntos de datos                                                                 | Descripción                                                                                                                                                                                                                                             |
|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Datos de COVID-19 de Bing](dataset-bing-covid-19.md)                                                       | Los datos de COVID-19 de Bing incluyen los casos confirmados, los fallecimientos y los casos recuperados de todas las regiones. Se actualizan a diario.                                                                                                                                                      |
| [Proyecto de seguimiento de COVID](dataset-covid-tracking.md)                                                | El conjunto de datos de COVID Tracking Project ofrece las últimas cifras sobre las pruebas, los casos confirmados, las hospitalizaciones y los pronósticos de los pacientes de cada estado y territorio de EE. UU.                                                                                     |
| [Casos de Covid-19 del Centro Europeo para la Prevención y el Control de Enfermedades (ECDC)](dataset-ecdc-covid-cases.md) | Datos públicos más recientes sobre la distribución geográfica de los casos de COVID-19 en todo el mundo proporcionados por el Centro Europeo para la Prevención y el Control de las Enfermedades (ECDC). Cada fila o entrada contiene el número de casos nuevos notificados cada día por países o regiones. |
| [Oxford COVID-19 Government Response Tracker](dataset-oxford-covid-government-response-tracker.md)                              | El conjunto de datos de seguimiento de la respuesta gubernamental a la Covid-19 de Oxford (OxCGRT) contiene información sistemática sobre qué gobiernos han tomado qué medidas y cuándo.                                                                                              |

---

## <a name="next-steps"></a>Pasos siguientes

Vea el resto de los conjuntos de datos en el [catálogo de Open Datasets](dataset-catalog.md).