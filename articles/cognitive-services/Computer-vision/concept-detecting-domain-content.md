---
title: 'Contenido específico del dominio: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Obtenga información acerca de cómo especificar un dominio de categorización de imágenes para devolver información más detallada sobre una imagen.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 02/08/2019
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: b0a046441466efeef559544814f5471b5802b1d4
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109714843"
---
# <a name="detect-domain-specific-content"></a>Detectar contenido específico del dominio

Además de la categorización de alto nivel y el etiquetado, Computer Vision también admite un análisis más refinado y específico del dominio con modelos que se hayan entrenado en datos especializados.

Hay dos maneras de utilizar los modelos específicos del dominio: solos (análisis con ámbito) o como una mejora de la característica de categorización.

### <a name="scoped-analysis"></a>Análisis con ámbito

Puede analizar una imagen usando solo el modelo específico de dominio elegido mediante una llamada a la API [Models/\<model\>/Analyze](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f21b).

El siguiente es una muestra de respuesta JSON devuelta por la API **models/celebrities/analyze** para la imagen especificada:

![Satya Nadella en pie, sonriendo](./images/satya.jpeg)

```json
{
  "result": {
    "celebrities": [{
      "faceRectangle": {
        "top": 391,
        "left": 318,
        "width": 184,
        "height": 184
      },
      "name": "Satya Nadella",
      "confidence": 0.99999856948852539
    }]
  },
  "requestId": "8217262a-1a90-4498-a242-68376a4b956b",
  "metadata": {
    "width": 800,
    "height": 1200,
    "format": "Jpeg"
  }
}
```

### <a name="enhanced-categorization-analysis"></a>Análisis de categorización mejorada

También puede usar modelos específicos del dominio para complementar los análisis de imágenes generales. Esto forma parte de la [categorización de alto nivel](concept-categorizing-images.md) mediante la especificación de los modelos específicos del dominio en la llamada al parámetro *details* de la llamada a la API [Analyze](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f21b).

En este caso, se llama primero al clasificador de la taxonomía de las 86 categorías. Si alguna de las categorías detectadas tiene un modelo específico de dominio coincidente, la imagen se pasa por ese modelo y se agregan los resultados.

La siguiente respuesta JSON muestra el modo en que el análisis específico del dominio puede incluirse como el nodo `detail` en un análisis de categorización más amplio.

```json
"categories":[
  {
    "name":"abstract_",
    "score":0.00390625
  },
  {
    "name":"people_",
    "score":0.83984375,
    "detail":{
      "celebrities":[
        {
          "name":"Satya Nadella",
          "faceRectangle":{
            "left":597,
            "top":162,
            "width":248,
            "height":248
          },
          "confidence":0.999028444
        }
      ],
      "landmarks":[
        {
          "name":"Forbidden City",
          "confidence":0.9978346
        }
      ]
    }
  }
]
```

## <a name="list-the-domain-specific-models"></a>Enumeración de modelos específicos de dominio

Actualmente, Computer Vision admite los siguientes modelos específicos de dominio:

| Nombre | Descripción |
|------|-------------|
| celebrities | Reconocimiento de celebridades, compatible con imágenes clasificadas en la categoría `people_`. |
| landmarks | Reconocimiento de puntos de referencia, compatible con imágenes clasificadas en las categorías `outdoor_` o `building_`. |

Una llamada a la API [Models](https://westcentralus.dev.cognitive.microsoft.com/docs/services/computer-vision-v3-2/operations/56f91f2e778daf14a499f20e) devolverá esta información junto con las categorías a las que se puede aplicar cada modelo:

```json
{
  "models":[
    {
      "name":"celebrities",
      "categories":[
        "people_",
        "人_",
        "pessoas_",
        "gente_"
      ]
    },
    {
      "name":"landmarks",
      "categories":[
        "outdoor_",
        "户外_",
        "屋外_",
        "aoarlivre_",
        "alairelibre_",
        "building_",
        "建筑_",
        "建物_",
        "edifício_"
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Conozca los conceptos de [categorización de imágenes](concept-categorizing-images.md).
